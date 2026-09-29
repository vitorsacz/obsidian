---
tags: [projeto, dentist-system, rbac, requisitos, produto]
status: proposta — requisitos levantados em 2026-09-25, decisões em aberto no fim da nota
relacionado: "[[Pontos de Atenção — Análise 2026-09-25]], [[feature - painel-admin-rbac-lgpd]], [[Arquitetura]], [[Roadmap]]"
---

# Responsabilidades por Perfil e Novos Requisitos

Separação do que é responsabilidade de cada perfil da plataforma, comparando
com o que o sistema faz **hoje** (conferido no código em 2026-09-25, `main`
em `f8146c5`), e a lista de requisitos novos para ajustar.

Perfis:
1. **Super Admin** — Vitor, dono da plataforma, fora de qualquer organização.
2. **Admin da clínica** — gestor de um tenant `CLINIC`.
3. **Dentista da clínica** — profissional da equipe de um tenant `CLINIC`.
4. **Dentista freelancer** — dono e único usuário de um tenant `FREELANCER`.
5. (**Recepcionista da clínica** — não estava na pergunta, mas existe; incluído
   no fim pra matriz ficar completa.)

---

## O problema de fundo: papel × tipo de tenant

Hoje o sistema só tem 3 papéis (`ADMIN`, `DENTIST`, `RECEPTIONIST`) + a flag
`isSuperAdmin`. **O freelancer não é um papel** — é um usuário `DENTIST`
dentro de uma organização `type: FREELANCER` (ver `seed-test-personas.ts`).

Consequência: dentista da clínica e dentista freelancer têm **exatamente as
mesmas permissões** em quase tudo. O tipo de tenant só muda regra em 2
pontos do backend:
- `POST /clinics` → 403 para `CLINIC` (`clinics.service.ts`).
- `ClinicFinancialTerms` → 403 para quem não é `FREELANCER`
  (`clinic-financial-terms.service.ts`).

E no front, só a Agenda olha o tipo (`resolveCalendarAxis()`). O menu, as
rotas e todos os outros `@Roles` ignoram o tipo.

Por isso o **dentista da clínica herdou poderes de dono** que fazem sentido
para o freelancer (editar catálogo e preços, editar consultórios, ver o
financeiro inteiro), e o **admin da clínica ficou sem poderes de gestor**
(não vê financeiro, não edita catálogo nem consultório, não mexe na agenda
real).

**Diretriz proposta**: tratar permissão como `papel + tipo de tenant`,
decidida num ponto central (backend e front), em vez de `type === "CLINIC"`
espalhado por services. Na prática o freelancer funciona como
"admin + dentista" da própria conta.

---

## 1. Super Admin (Vitor)

**Responsabilidade**: operar a plataforma — criar e manter os clientes
(tenants), dar suporte, acompanhar números agregados. **Nunca** mexe em dado
clínico ou financeiro de paciente.

**Hoje**:
- ✅ Dashboard agregado (`/platform`), lista e detalhe de organizações.
- ✅ Cria organização + admin fundador; transfere a capitania.
- ❌ Só cria tipo `CLINIC` — freelancer só nasce via script.
- ❌ Organização nasce sem consultório e sem procedimento, e a clínica não
  consegue criar consultório (ver [[Pontos de Atenção — Análise 2026-09-25]],
  pontos 1 e 2).
- ❌ Não suspende, reativa nem exclui organização (o campo `status` existe,
  mas não tem endpoint e o login não checa).
- ❌ Não vê os usuários de uma organização nem ajuda num acesso perdido
  (ex.: admin fundador esqueceu a senha — ninguém consegue redefinir,
  porque só o fundador edita outros admins).
- ❌ Sem dados cadastrais do cliente (CNPJ/CPF, contato, endereço).

## 2. Admin da clínica

**Responsabilidade**: gerir a clínica como negócio — equipe, configuração
(consultórios, catálogo e preços), agenda de todos e financeiro consolidado.
**Não** acessa dado clínico (anamnese, prontuário, odontograma).

**Hoje**:
- ✅ Gestão de equipe (`/admin/users`): cria, edita papel, desativa,
  redefine senha, com a regra de capitania.
- ✅ Vê "Minha Clínica" (só leitura) e a Agenda (mock).
- ✅ Leitura de pacientes, consultórios e procedimentos (só para a Agenda).
- ❌ Não edita o nome/dados da clínica.
- ❌ Não edita consultórios nem catálogo/preços (escrita é só `DENTIST`).
- ❌ Não vê nada de financeiro (`/attendances`, `/reports` são só
  `DENTIST`).
- ❌ Sem acesso a `/appointments` (agenda real).
- ❌ Não tem tela "Início": cai direto em Usuários.
- ⚠️ Não pode ser admin **e** dentista na mesma conta — e, como o e-mail é
  único dentro do tenant, também não pode ter duas contas com o mesmo
  e-mail na mesma clínica. Caso comum de clínica pequena (dono que atende)
  não é coberto. Ver decisão em aberto D1.

## 3. Dentista da clínica

**Responsabilidade**: atender — prontuário, anamnese, odontograma,
orçamento, lançar os **próprios** atendimentos, ver a **própria** agenda e a
**própria** produção/repasse.

**Hoje**:
- ✅ Todo o módulo clínico (anamnese, prontuário, odontograma, orçamento).
- ✅ Lança atendimento com baixa de estoque.
- ⚠️ **Vê o financeiro da clínica inteira** — `reports/financial` soma todos
  os `Attendance` da organização, sem filtrar por dentista
  (`reports.service.ts`). Com 2+ dentistas, um vê o faturamento do outro.
- ⚠️ O cálculo do relatório desconta **aluguel do consultório** do repasse
  do dentista — isso é regra de freelancer/dentista que aluga; na clínica,
  quem paga aluguel é a clínica.
- ⚠️ Edita catálogo de procedimentos e **preços** da clínica, e edita
  consultórios (inclusive tipo e diária) — deveria ser do admin.
- ❌ Agendamento não tem dentista (`Appointment` não tem `dentistId`) — não
  existe "minha agenda" de verdade no backend.
- ℹ️ Vê pacientes e prontuários de todos os dentistas da clínica (o paciente
  é do tenant, não do profissional — decisão do painel RBAC). Ver D3.

## 4. Dentista freelancer

**Responsabilidade**: tudo da própria conta — é dono e profissional ao mesmo
tempo. Gerencia os locais onde atende (consultórios próprios, alugados, com
comissão), catálogo e preços, agenda, atendimento clínico, estoque e o
próprio financeiro, incluindo o acordo com cada consultório.

**Hoje**:
- ✅ Cria/edita os próprios consultórios; Agenda por consultório (roster
  real).
- ✅ Todo o módulo clínico e financeiro (é `DENTIST`).
- ✅ `ClinicFinancialTerms` no backend (aluguel fixo / comissão / por
  serviço).
- ❌ `ClinicFinancialTerms` **não tem tela** e o **relatório financeiro
  ignora** esses termos — continua usando só `Clinic.type` +
  `dailyRentValue`. Comissão e aluguel semanal/mensal não entram no cálculo.
  Hoje existem dois conceitos paralelos de "custo do consultório".
- ❌ Só existe criado por script — nem Super Admin nem self-service.
- ⚠️ Menu mostra "Minha Clínica" (lista de membros de um tenant de 1
  pessoa) — texto e itens pensados para clínica.

## 5. Recepcionista da clínica (para completar)

**Responsabilidade**: agenda de todos, cadastro de paciente, orçamento,
retornos, estoque. Sem dado clínico e sem financeiro. **Hoje** bate com isso,
exceto pelo que já é problema geral: agenda é mock e retorno não tem tela.

---

## Matriz de responsabilidades proposta

Legenda: ✏️ gerencia (lê e escreve) · 👁 só leitura · 🙋 só o próprio ·
— sem acesso. Entre colchetes, **o que muda** em relação a hoje.

| Área | Super Admin | Admin clínica | Dentista clínica | Recepção | Freelancer |
|---|---|---|---|---|---|
| Organizações (criar, suspender, tipo) | ✏️ [+tipo, +suspender] | — | — | — | — |
| Dados cadastrais da clínica | ✏️ | ✏️ [novo] | 👁 | 👁 | ✏️ (própria conta) |
| Equipe / usuários | 👁 + suporte [novo] | ✏️ | — | — | — (conta única) |
| Consultórios | ✏️ na criação [novo] | ✏️ [hoje é do dentista] | 👁 [perde escrita] | 👁 | ✏️ |
| Termos financeiros do consultório | — | — | — | — | ✏️ [+tela] |
| Catálogo e preços | — (só catálogo padrão inicial) | ✏️ [hoje é do dentista] | 👁 [perde escrita] | 👁 | ✏️ |
| Pacientes (cadastro) | — | 👁 | ✏️ | ✏️ | ✏️ |
| Anamnese, prontuário, odontograma | — | — | ✏️ | — | ✏️ |
| Orçamento | — | 👁 [novo] | ✏️ | ✏️ | ✏️ |
| Agenda | — | ✏️ todos [novo] | 🙋 a própria [+dentistId] | ✏️ todos | ✏️ |
| Atendimento (lançamento) | — | — | 🙋 os próprios | — | ✏️ |
| Relatório financeiro | só agregado da plataforma | 👁 clínica toda, por dentista [novo] | 🙋 a própria produção [hoje vê tudo] | — | ✏️ com termos por consultório |
| Estoque | — | 👁 [novo] | ✏️ | ✏️ | ✏️ |
| Retornos | — | 👁 | ✏️ | ✏️ | ✏️ |

---

## Novos requisitos

Prioridade: **P0** = bloqueia uso real por um cliente novo · **P1** = corrige
uma responsabilidade no lugar errado · **P2** = evolução.

### Transversais
- **TR-01 (P1)** — Política de permissão central por `papel + tipo de
  tenant` no backend (um helper/guard que responde "pode fazer X?"), usada
  também para montar menu e rotas no front. Substitui os
  `type === "CLINIC"` espalhados.
- **TR-02 (P1)** — `Appointment.dentistId` (obrigatório em tenant `CLINIC`)
  e dentista responsável explícito no `Attendance` (hoje só `createdByUserId`).
  Base para "minha agenda" e "minha produção".
- **TR-03 (P1)** — Testes e2e por perfil: uma suíte por persona (as 5),
  provando a matriz acima, no mesmo estilo de `tenant-isolation.e2e-spec.ts`.

### Super Admin
- **SA-01 (P0)** — Criar organização escolhendo o **tipo** (`CLINIC` /
  `FREELANCER`); para freelancer, o usuário criado é o próprio dentista.
- **SA-02 (P0)** — Cadastrar os consultórios da clínica na criação da
  organização e depois, pelo detalhe da organização.
- **SA-03 (P0)** — Toda organização nova nasce com o catálogo padrão de
  procedimentos (hoje só no `seed.ts`).
- **SA-04 (P1)** — Suspender / reativar / excluir (soft delete) organização,
  com bloqueio no login **e** no `jwt.strategy` (sessão aberta cai).
- **SA-05 (P1)** — Ver os usuários de uma organização (nome, papel, ativo —
  sem dado clínico) e redefinir a senha do admin fundador como suporte.
- **SA-06 (P2)** — Dados cadastrais do cliente: CNPJ/CPF, responsável,
  contato, endereço.
- **SA-07 (P2)** — Log de auditoria das ações do Super Admin (já previsto
  no [[feature - painel-admin-rbac-lgpd]]).

### Admin da clínica
- **AD-01 (P1)** — Assumir a escrita de **consultórios** (nome, cor, tipo,
  diária) e do **catálogo/preços** no tenant `CLINIC`; dentista da clínica
  passa a só ler.
- **AD-02 (P1)** — Relatório financeiro **da clínica**, com quebra por
  dentista e por consultório (faturamento, repasse de cada dentista, custo
  de material).
- **AD-03 (P1)** — Agenda real de todos os dentistas (`/appointments` com
  `ADMIN`), junto com a Agenda sair do mock.
- **AD-04 (P1)** — Editar os dados da clínica (nome, e futuramente
  logo/contato) em "Minha Clínica".
- **AD-05 (P2)** — Tela "Início" do admin com indicadores da clínica (hoje
  cai direto em Usuários).
- **AD-06 (P2)** — CRO + UF no cadastro de dentista (já no [[Roadmap]]).
- **AD-07 (P2)** — Leitura de orçamentos, estoque e retornos (sem dado
  clínico) para acompanhar a operação.

### Dentista da clínica
- **DC-01 (P1)** — Relatório financeiro filtrado **só pelos próprios
  atendimentos**, sem aluguel de consultório (quem paga é a clínica).
  Mostra produção bruta e repasse.
- **DC-02 (P1)** — Perde escrita de consultórios e catálogo (par do AD-01);
  menu esconde "Consultórios" e deixa "Procedimentos" só leitura.
- **DC-03 (P1)** — Agenda mostra só os próprios agendamentos (depende de
  TR-02).
- **DC-04 (P1)** — Lançar atendimento sempre em nome de quem está logado
  (sem escolher outro dentista).

### Dentista freelancer
- **FL-01 (P1)** — Tela de termos financeiros por consultório (aluguel
  fixo diário/semanal/mensal, comissão, por serviço) — o endpoint já
  existe.
- **FL-02 (P1)** — Relatório financeiro do freelancer calculado a partir de
  `ClinicFinancialTerms` (comissão sobre o bruto, aluguel no período,
  valor por serviço), unificando com `Clinic.type`/`dailyRentValue` para
  não haver dois conceitos de custo de consultório.
- **FL-02b (P1)** — Decidir o destino de `Clinic.type` (`OWN`/`RENTED`) +
  `dailyRentValue`: ou vira só o caso "aluguel diário" dos termos
  financeiros, ou fica só para clínica. Hoje os dois coexistem.
- **FL-03 (P1)** — Menu e textos próprios do freelancer: sem "Minha
  Clínica"/equipe; "Meus consultórios", "Meu financeiro".
- **FL-04 (P2)** — Cadastro self-service (cadastro progressivo Fase 1/2 +
  LGPD, já descritos no [[feature - painel-admin-rbac-lgpd]]). Até lá, SA-01
  cobre a criação manual.

---

## Decisões em aberto (precisam de resposta do Vitor)

- **D1 — Dono da clínica que também atende.** Admin e dentista na mesma
  conta? Opções: (a) permitir mais de um papel por usuário (`roles[]`);
  (b) exigir duas contas com e-mails diferentes; (c) papel novo
  "Admin-Dentista". Recomendação: (a) — é o caso mais comum em clínica
  pequena e mexe pouco no resto.
- **D2 — Quem paga o quê na clínica.** O repasse do dentista da clínica é
  sempre percentual do bruto? Existe dentista CLT/fixo? Isso define o
  relatório do AD-02/DC-01.
- **D3 — Prontuário entre dentistas da mesma clínica.** Hoje todos veem
  todos os pacientes e prontuários. Manter (paciente é da clínica) ou
  restringir ao dentista responsável?
- **D4 — Freelancer com secretária.** O painel RBAC decidiu "sempre 1
  usuário". Continua valendo? (Se não, o freelancer precisa de um
  recepcionista e da tela de usuários.)
- **D5 — Admin da clínica vê dado financeiro por paciente?** Ou só
  agregados por dentista/consultório?
- **D6 — Consultório de clínica: quem cadastra.** Só o Super Admin (fixo,
  como decidido no PR #9) ou o admin da clínica também (AD-01)?
  Recomendação: admin da clínica — evita depender do Vitor para cada
  mudança de sala.

## Ordem sugerida

1. **P0 do Super Admin** (SA-01, SA-02, SA-03) — destrava cliente novo.
2. **TR-01 + AD-01/DC-02** — tira do dentista da clínica os poderes de dono.
3. **DC-01 + AD-02** — separa o financeiro (hoje o maior vazamento entre
   dentistas).
4. **TR-02 + AD-03/DC-03** — junto com a Agenda real.
5. **FL-01/FL-02** — financeiro do freelancer com os termos por
   consultório.
6. Resto (P2) conforme a demanda.
