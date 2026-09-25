#projeto #dentist-system #roadmap #agenda

Ver [[Visão Geral]] pro contexto do produto, [[Roadmap]] pro estado geral do MVP,
[[Arquitetura]] pro desenho técnico atual, [[Roadmap]] (seção "Evolução SaaS") pra
sequência de multi-tenancy original (Fase 1) — este plano **antecipa** parte dela,
ver decisão abaixo.

Nota criada em 2026-09-17 (consolidação de uma conversa técnica sobre a agenda
estilo Google Calendar). Vivia como `roadmap_agenda_plano_tecnico.md` na raiz do
vault; movida pra dentro da pasta do projeto pra ficar junto do resto da
documentação.

**Atualização do mesmo dia**: a fundação de multi-tenancy (`Organization`,
isolamento por `organizationId` em todo model de negócio, separação de role de
banco documentada) foi **implementada e testada localmente** — ver
[[Arquitetura]] pro desenho final. `Location`/`Calendar` e a UI estilo Google
Calendar (FullCalendar) **ainda não** — é o que falta desta nota daqui pra
frente.

**⚠️ Segunda atualização, ainda no mesmo dia**: o modelo `Membership`
descrito no restante desta nota (tabelas abaixo) **foi removido** — substituído
por identidade isolada por tenant (`User.organizationId` direto, Super Admin,
capitania de clínica), seguindo [[feature - painel-admin-rbac-lgpd]]. Antes de
retomar a implementação da agenda, reconciliar o modelo de dados abaixo
(`Calendar`/`Location` referenciando `Membership`) com o desenho atual em
[[Arquitetura]] — provavelmente `Calendar.dentist_user_id` passa a referenciar
`User.id` direto, sem a tabela de junção.

## Objetivo e escopo

Construir a agenda do MVP com visão multi-calendário estilo Google Calendar: cada
consultório/local de atendimento tem uma cor própria (ex.: Bragança verde, Atibaia
azul, São Paulo vermelho), e o acesso a cada calendário é controlado por papel —
não por interface, por regra de autorização no backend.

Esse desenho precisa servir dois públicos com a mesma estrutura, sem produto
paralelo: uma clínica com vários dentistas e um dentista freelancer que atende em
vários locais.

## Decisão de escopo (2026-09-17)

Avaliamos três opções: (1) só melhorar a UI da agenda em cima do modelo atual
(`Clinic`/`Appointment`), sem multi-tenancy; (2) multi-tenancy completa agora,
conforme este plano; (3) meio-termo, adiantando só o `tenantId` já desenhado na
Fase 1 do [[Roadmap]] (seção "Evolução SaaS"), sem `Organization`/`Membership`/`Calendar`.

**Escolhida: opção 2 — multi-tenancy completa agora**, adiantando a Fase 1 do
plano SaaS antes do gatilho original ("só quando a clínica 2 for confirmada").
Decisão consciente do Vitor, não um esquecimento da regra antiga.

**Nuance a reconciliar depois** (não resolvida ainda, não bloqueia começar): o
modelo `Organization`/`Membership`/`Location`/`Calendar` abaixo é mais rico do que
o desenho original da Fase 1 (que previa só adicionar `tenantId` nos models
existentes + renomear `Clinic`→`ClinicUnit`, com `tenantId` imutável por sessão no
JWT). Aqui, `Membership` permite um usuário pertencer a mais de uma `Organization`
ao mesmo tempo (dentista que atende em 2 clínicas) — isso implica escolher/trocar
"organização ativa" na sessão, o que a Fase 1 original não previa. Avaliar se o
JWT carrega uma `organizationId` ativa (trocável via novo login/endpoint) ou se o
contexto é resolvido por request a partir de todas as memberships do usuário.

## Modelo de dados

| Entidade | Campos principais | Relacionamento |
| --- | --- | --- |
| Organization | id, name, type | Raiz do tenant — uma clínica ou o workspace de um freelancer |
| Membership | id, user_id, organization_id, role | Liga um usuário a uma organização com um papel (admin, receptionist, dentist) |
| User | id, name, email | Pessoa; pode ter mais de uma Membership (ex.: dentista que atende em 2 clínicas) |
| Location | id, organization_id, name, default_color | Consultório/local físico; a cor exibida na agenda vem daqui |
| Calendar | id, location_id, dentist_user_id, color_override | Uma agenda = combinação de local + dentista |
| Appointment | id, calendar_id, patient_id, start_time, end_time, status | Consulta marcada numa agenda específica |
| Patient | id, organization_id, name | Paciente, sempre dentro de uma organização |

**Por que essa estrutura serve os dois públicos:** uma clínica com 3 dentistas é 1
Organization + 1 Location + 3 Calendars (uma por dentista). Um freelancer que
atende em 3 cidades é 1 Organization (ele mesmo) + 3 Locations (uma por cidade,
cada uma com sua cor) + 3 Calendars — como ele é o único dentist_user_id em
todas, a agenda colorida por local já aparece sem regra especial.

## Modelo de permissões (RBAC)

| Papel (Membership.role) | Escopo de acesso |
| --- | --- |
| admin | Todas as Calendars da Organization à qual está vinculado — **em aberto, ver abaixo** |
| receptionist | Todas as Calendars da Organization à qual está vinculado — apenas essa clínica, confirmado: recepcionista não tem acesso a outras organizações |
| dentist | Apenas Calendars onde dentist_user_id = usuário logado, dentro das Organizations às quais está vinculado |

"Todas as agendas" significa sempre todas **dentro da mesma Organization**, nunca
todas do sistema.

**Em aberto (não resolvido em 2026-09-17):** hoje o `RolesGuard` do sistema
exclui `ADMIN` de propósito de todo dado clínico/financeiro (ver [[Arquitetura]]
— tabela de acesso por módulo). Dar ao admin visão de todas as agendas expõe
nome de paciente e horário, o que reverte essa política atual. Duas opções em
aberto:
- manter admin fora de dados clínicos (agenda incluída), só recepcionista e
  dentista veem Calendars — consistente com a regra atual;
- mudar a política pra dar ao admin visão de agenda (não de prontuário/anamnese),
  como este plano propõe.

## Enforcement técnico

**Decisão (2026-09-17): só Prisma Client Extension por enquanto — RLS mapeado,
não implementado.**

1. **Filtro na aplicação** (caminho escolhido para o MVP): toda query de
   Calendar/Appointment passa pela mesma extension do Prisma já desenhada na
   Fase 1 do [[Roadmap]] (seção "Evolução SaaS"), que injeta o filtro de tenant
   conforme o papel.

```sql
-- admin / receptionist
SELECT * FROM calendar WHERE organization_id = :org_id;

-- dentist
SELECT * FROM calendar
WHERE organization_id = :org_id AND dentist_user_id = :user_id;
```

2. **Separação de role no banco (nova, incluída neste plano)** — independente da
   decisão de RLS, é boa prática adotar agora: hoje `DATABASE_URL` (pooler
   transação) e `DIRECT_URL` (pooler sessão) usam a **mesma role** dona das
   tabelas. Criar uma role `app_user` sem ownership, usada só pela API em
   runtime; `DIRECT_URL`/migrations continuam com a role dona.
   ```sql
   CREATE ROLE app_user LOGIN PASSWORD '...';
   GRANT USAGE ON SCHEMA public TO app_user;
   GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
   ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;
   ```
   Reduz o raio de um vazamento de `DATABASE_URL` de produção (leitura/escrita de
   dado, não `DROP TABLE`/alterar schema/desativar policy). No Supabase, precisa
   registrar a role no pooler (Supavisor/PgBouncer) além do `CREATE ROLE` — passo
   a mais em relação a um Postgres direto.

3. **RLS no Postgres — mapeado para o futuro, não implementado agora.** Motivo:
   o pooler de transação do Supabase (porta 6543) pode trocar a conexão física
   entre statements de um mesmo request, tornando `SET LOCAL`/variável de sessão
   pouco confiável sem embrulhar toda request numa transação Prisma explícita.
   Isso **não é exclusivo do Supabase** — é característica de qualquer pooler em
   modo transação (PgBouncer transaction mode, RDS Proxy em modo transação,
   pooler da Neon). Se um dia o projeto migrar pra um Postgres com conexão
   direta (sem pooler transacional) — inclusive dentro do próprio Supabase, se o
   plano pago viabilizar IPv4 direto — o RLS passa a ser confiável, mas ainda
   assim exige: a role separada do item 2 (RLS é ignorado pelo dono da tabela
   por padrão, precisa de `ALTER TABLE ... FORCE ROW LEVEL SECURITY`), `CREATE
   POLICY` em SQL cru em cada migration (Prisma não gerencia RLS nativamente), e
   cada request embrulhada numa transação explícita. Revisitar só se/quando
   migrar de banco — não é prioridade com o Supabase atual.

Regra de ouro, independente de qual camada: a checagem de papel acontece sempre
no servidor. Nenhum endpoint aceita "quero ver a agenda do dentista X" sem
confirmar que quem pediu tem papel admin/receptionist na mesma organização
daquele dentista.

## UI estilo Google Calendar

- **Sidebar com checklist de calendários**: cada Calendar aparece com um
  quadradinho colorido (cor da Location) e um checkbox para ligar/desligar a
  visualização.
- **Dentista**: sidebar mostra só a(s) sua(s) própria(s) agenda(s) — se atende em
  2 locais, aparecem 2 linhas, cada uma com a cor do respectivo consultório, sem
  nenhuma agenda de colega.
- **Admin/recepcionista**: sidebar lista todos os dentistas da organização,
  agrupados por local ou por dentista (como o Google Calendar faz com "Minhas
  agendas" vs. "Outras agendas") — sujeito à decisão em aberto sobre acesso do
  admin, acima.
- **Biblioteca recomendada**: FullCalendar (open-source) — suporta múltiplos
  calendários sobrepostos, cor por evento/recurso e visões dia/semana/mês,
  evitando construir do zero a grade de horários, o arrastar-e-soltar e a
  detecção de conflito.

## Endpoints da API

```
GET  /calendars                        → lista calendários visíveis (filtrado por papel no backend)
GET  /calendars/:id/appointments?from=&to=
POST /appointments
PUT  /appointments/:id
GET  /locations                        → popula cor e nome no seletor
```

O filtro por papel acontece dentro do handler de `GET /calendars`, nunca no
cliente — a lista que a API devolve já é a lista permitida.

## Passos de implementação

- [x] Modelar Organization, Membership no banco (2026-09-17) — `Location`/
      `Calendar` **ainda não modelados**, ficam pra quando esta nota entrar em
      implementação de verdade
- [x] Construir a camada de autorização (Prisma Client Extension) que resolve,
      a partir do usuário logado (`organizationId` no JWT), o isolamento por
      organização — feito de forma genérica pra todo model de negócio, não só
      Calendar (que ainda não existe)
- [x] Teste automatizado de isolamento entre 2 organizations (critério de
      aceite herdado da Fase 1 do [[Roadmap]] (seção "Evolução SaaS") — acesso
      cruzado por ID responde 404 em todo endpoint hoje existente). Rodando
      no CI
- [ ] Criar a role `app_user` separada da role dona das tabelas — **script
      pronto** (`apps/api/prisma/scripts/create-app-role.sql`), ainda não
      aplicado em produção
- [ ] Aplicar a migração de multi-tenancy em produção (Supabase) — feita e
      testada só localmente até agora
- [ ] Modelar Location, Calendar + seed de dados de teste (1 clínica
      multi-dentista + 1 freelancer multi-local)
- [ ] Implementar `GET /calendars` já filtrado, antes de tocar em UI
- [ ] Integrar FullCalendar no front, renderizando um recurso/cor por Calendar
      retornado
- [ ] Construir a sidebar com toggle de visibilidade (estado só no front, sem
      ida ao servidor)
- [ ] Testar os três cenários do zero: dentista solo, clínica multi-dentista,
      freelancer multi-local

## Casos de borda já endereçados pelo modelo

| Caso | Como o modelo resolve |
| --- | --- |
| Freelancer atende em vários locais | Várias Locations sob a mesma Organization (o próprio profissional); cada uma com sua cor |
| Dentista atende em mais de uma clínica | Uma segunda Membership numa segunda Organization — os dados de uma clínica nunca vazam para a outra |
| Recepcionista vinculada a mais de uma clínica (futuro) | Basta uma Membership adicional; a UI usaria um seletor de organização ativa para não misturar agendas de clínicas diferentes |
| Recepcionista vinculada a apenas uma clínica (caso atual confirmado) | Já funciona sem nenhuma regra extra — só existe uma Membership dela, então o filtro por organization_id já a restringe a essa clínica |

## Decisões validadas e pontos em aberto

**Validado:**
- Multi-tenancy completa (Organization/Membership/Location/Calendar) entra
  agora, adiantando a Fase 1 do plano SaaS (2026-09-17)
- Admin e recepcionista veem todas as agendas — sempre restrito à Organization à
  qual estão vinculados, nunca ao sistema todo
- Recepcionista tem acesso apenas à clínica à qual está vinculada, hoje sem
  cenário multi-clínica
- Dentista vê apenas sua própria agenda
- Separação de role de banco (app_user vs. role dona) entra como parte deste
  plano, independente de RLS
- RLS fica mapeado (ver Enforcement técnico) mas **não** entra nesta
  implementação — revisitar só se sair do pooler transacional do Supabase

**Em aberto para as próximas etapas técnicas:**
- Acesso do admin a agendas (ver dados clínicos) vs. manter exclusão atual —
  não decidido
- "Organização ativa" quando um usuário tem mais de uma Membership —
  **parcialmente resolvido na implementação de 2026-09-17**: o login já
  resolve a primeira membership automaticamente (`orderBy: createdAt asc`) e o
  JWT carrega `organizationId`/`membershipId` fixos pra sessão inteira
  (refresh fica ancorado na mesma membership, não escolhe de novo). O que
  ainda falta é um seletor de verdade pra quando existir um usuário real com
  mais de uma membership — não construído de propósito, sem caso de uso real
  ainda.
- Fluxo de agendamento em si — criação de consulta, detecção de conflito,
  confirmação via WhatsApp
- Regra de cores quando o número de locais ultrapassar a paleta fixa
- Reconciliar este modelo com a visão de produto mais ampla — [[Roadmap]],
  seção "Visão de produto mais ampla" (trilha MVP já prevê "múltiplas agendas
  por dentista, alertas de conflito, compromisso recorrente")
