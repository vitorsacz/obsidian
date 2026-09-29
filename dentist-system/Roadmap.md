#projeto #dentist-system #roadmap

Ver [[Visão Geral]] para contexto completo. Prioridade original do MVP estava
em `prompt-sistema-odontologico` (arquivo não encontrado no vault, ver nota
em [[Visão Geral]]) — todas as fases sugeridas lá já foram implementadas,
incluindo as que o próprio prompt dizia que podiam ficar pra "segunda
iteração" (odontograma e retorno).

> **Desde 2026-09-25 o roadmap ativo é [[roadmap-papeis-permissoes-financeiro]]**
> (papéis, permissões e financeiro — fases 0 a 7, com progresso por item e
> PRs). Esta nota segue como histórico do que foi construído até lá, mais a
> evolução SaaS e a visão de produto. A Fase 0 (segurança) e o R1 (múltiplos
> papéis) de lá já estão na `main` — resumo em "Feito" abaixo.

Esta nota é o **roadmap de verdade** — o que já foi construído e o que está
priorizado pra construir a seguir, incluindo a visão de produto mais ampla
(comparativo com concorrentes) e a evolução SaaS de infraestrutura, ambas
consolidadas aqui em vez de viverem em notas soltas. Notas de **conhecimento**
(arquitetura, endpoints, infra, bugs já resolvidos — material de referência,
não itens de tarefa) ficam separadas na seção final "Notas de conhecimento"
abaixo.

## Feito

- [x] **Base** — monorepo, Prisma schema completo, auth (login + refresh),
      CORS/env validado com Zod
- [x] **Pacientes** — cadastro + anamnese + queixa principal + prontuário/evolução
- [x] **Orçamento** — tabela de procedimentos + geração de orçamento com valor
      editável por item, vínculo opcional a dente
- [x] **Agenda (versão original)** — agendamento por consultório, visão por dia,
      mudança de status — **substituída pelo redesenho de 2026-09-21, ver abaixo**
- [x] **Financeiro** — registro de atendimento (consultório, % repasse, custo
      de material) + relatório consolidado com regra de aluguel por dia
- [x] **Estoque** — materiais, lotes (FEFO), alerta de estoque baixo e
      validade próxima, baixa vinculada a atendimento
- [x] **Odontograma visual** — mapa gráfico da arcada (32 dentes, notação FDI,
      quadrantes separados por uma linha central), cor por status (sem
      registro/planejado/realizado calculado pelo registro mais recente do
      dente), clique no dente abre histórico + formulário de novo registro já
      escopado pra aquele dente (ver [[Arquitetura]]). Inclui botão "Marcar
      como realizado" pra transicionar um registro Planejado existente sem
      duplicar entrada no histórico (`PATCH tooth-records/:id/status`)
- [x] **Orçamento ↔ odontograma** — formulário de novo orçamento sugere os
      dentes com status Planejado como chips clicáveis; adicionar uma sugestão
      preenche dente/observação e tenta casar o texto livre do odontograma
      com um procedimento do catálogo por substring pra puxar valor padrão
      (ver [[Funcionalidades e Endpoints]])
- [x] **Controle de retorno (recall)** — lembrete por paciente com status
- [x] **Papel ADMIN + painel de usuários** — criar/editar papel/ativar-desativar
      dentistas e recepcionistas; auditoria de permissões em todos os módulos
      (ver [[Arquitetura]] pra tabela completa de quem acessa o quê)
- [x] **Testado ponta a ponta localmente** — via curl (cálculo de repasse,
      RBAC de cada papel) e navegador real (login, cadastro, orçamento,
      agenda, financeiro, estoque, painel admin)
- [x] **Deploy real** — Supabase + Render + Vercel no ar, testado ponta a ponta
      no navegador (login, sessão persistindo via cookie cross-domain) — ver
      [[Infraestrutura e Deploy]]

- [x] **Multi-tenancy (fundação)** — implementado e testado localmente em
      2026-09-17, antecipando a Fase 1 da evolução SaaS (ver seção "Evolução
      SaaS" abaixo). **Revisado no mesmo dia** — a versão inicial usava `User`
      global + `Membership` many-to-many; foi substituída pelo modelo de
      identidade isolada por tenant (ver item logo abaixo) antes mesmo de
      chegar em produção. O que sobreviveu da fundação original: `Organization`
      como raiz do tenant, `organizationId` nos 14 models de negócio,
      isolamento via Prisma Client Extension + `AsyncLocalStorage` (RLS
      avaliado e adiado, não implementado), separação de role de banco
      (`app_user`, script pronto, ainda não aplicado em produção), suíte e2e
      cobrindo todo endpoint com 2 organizations rodando no CI. Ver
      [[Arquitetura]] pro desenho completo e as pegadinhas reais descobertas
      na implementação.
- [x] **Identidade isolada por tenant + Super Admin + capitania** (2026-09-17,
      mesmo dia) — segue as decisões 🔴 P0 de
      [[feature - painel-admin-rbac-lgpd|Painel Administrativo — RBAC, Multi-tenant e LGPD]]
      à risca: `Membership` removida (criada e removida no mesmo dia);
      `User.organizationId` direto (nullable só pra Super Admin), e-mail único
      só dentro do tenant, `nickname` opcional único global. Super Admin
      (`isSuperAdmin`, fora de qualquer organização) cria clínicas novas pelo
      módulo `platform`. Capitania da clínica
      (`Organization.foundingAdminUserId`) — só o fundador edita outro ADMIN
      da mesma clínica, transferência só via Super Admin. Login em 2 passos
      (`POST auth/lookup` → `POST auth/login`). Fora desta rodada, documentado
      mas não implementado: cadastro progressivo Fase 1/2, consentimento LGPD,
      `AUDIT_LOG`, tenant tipo Freelancer (implementado depois, PR #10,
      2026-09-22, ver mais abaixo), impersonation (ver 🟡 P1 na seção
      "Em planejamento" abaixo). Ver [[Arquitetura]] e
      [[Funcionalidades e Endpoints]].
- [x] **"Minha Clínica"** (2026-09-17) — `GET /organization` (qualquer papel
      autenticado) + página no front mostrando nome da clínica e lista de
      quem faz parte dela.
- [x] **Dashboard "Visão Geral da Plataforma" (Super Admin)** (2026-09-18) —
      `/platform` virou um dashboard de verdade (stat cards, badges,
      ApexCharts): organizações por status, usuários por papel, dentistas
      (ativos vs. total), atendimentos no mês, gráfico de novos tenants (6
      meses), donut por status, ranking top 10 clínicas, drill-down cadastral
      por clínica. Mecanismo técnico novo: token DI `PRISMA_UNSCOPED_SERVICE`
      (client Prisma sem a extension de tenant) — ver [[Arquitetura]].
- [x] **Multi-tenancy aplicada em produção** (2026-09-18) — PR #1 mergeado;
      produção (Supabase) resetada e recriada com as 7 migrations + seed novo.
      Login confirmado via navegador real (Super Admin, Admin, Dentista) na
      URL de produção. Ver [[Infraestrutura e Deploy]].

- [x] **CI corrigido de ponta a ponta** (PR #3, 2026-09-21) — pipeline do
      GitHub Actions nunca tinha rodado completo até então; cada camada de
      falha só aparecia depois de corrigir a anterior: ordem
      `setup-node`/`pnpm` (Node 20 não rodava pnpm 11) → Node 22.13 + reordenado
      → `cache: pnpm` quebrava (`pnpm` ainda não instalado no PATH nesse ponto)
      → trocado por `actions/cache` manual na pasta do pnpm store →
      `jest-e2e.config.ts` exigia `ts-node` → convertido pra `.js` puro →
      ESLint reclamava de `module`/`require` indefinidos nesse `.js` →
      `globals.node` no `eslint.config.mjs` da API → testes falhando com auth
      errada no Postgres → Turborepo em **modo estrito de env** descartava
      `TEST_DATABASE_URL`/`JWT_*_SECRET` por não estarem declaradas no
      `turbo.json` → declaradas na task `test`. CI verde desde então em todas
      as PRs seguintes (#3 a #8).
- [x] **`main` só aceita merge via PR** (2026-09-21) — GitHub Repository
      Ruleset (`require-pr-for-main`: `pull_request` + `non_fast_forward` +
      `deletion`, `bypass_actors: []`) — não é a branch protection clássica
      (essa foi testada e **não bloqueava** push direto mesmo com
      `enforce_admins`). Verificado com um push direto real sendo rejeitado
      (`GH013`).

- [x] **Redesenho visual completo (2026-09-21/22)** — leu-se
      [[design-system-plataforma-odontologica]] e aplicou-se os tokens
      oficiais (cor, tipografia Outfit, espaçamento, raio) no lugar dos
      valores antigos do Tailwind config. Filosofia adotada em todas as telas
      abaixo: **construir a camada visual/UX primeiro com dado mockado,
      plugar o backend depois** — ver [[Arquitetura]], seção "Padrão mock
      primeiro, backend depois", pro mecanismo técnico exato.
  - **Home Dashboard** (PR #2) — header, busca, notificações, avatar,
        stat cards, gráficos — tudo mockado.
  - **Sidebar de navegação fixa** (PR #5) — substituiu o menu horizontal;
        sidebar à esquerda, colapsável, grupos por seção, rodapé com
        avatar/nome/papel/sair, transição de página fluida (CSS puro, sem lib
        nova). Bug de scroll (sidebar rolando junto com a página) corrigido
        depois, ver [[Problemas Conhecidos]].
  - **Agenda estilo Google Calendar** (PR #4) — FullCalendar, grade
        semanal/dia/mês, sidebar de consultórios coloridos, criação por
        arrastar-e-soltar (modal), painel lateral de detalhe (não modal) com
        Confirmar/Remarcar/Cancelar. 100% mockada, exceto Locations
        (`clinicsApi.list()` real) e os pickers de paciente/procedimento.
  - **Catálogo de Procedimentos** (PR #6) — filtros (nome/categoria/status),
        badge de categoria colorida, paginação, modal de criar/editar,
        soft-delete only. Schema mockado já pensado pra um futuro
        `PROCEDURE_LOG` (registro do que foi de fato realizado, `id` estável
        pra FK futura).
  - **Agenda — múltiplos dentistas com cor própria** (PR #7, 2026-09-22) —
        ajuste em cima da Agenda existente (não reconstrução): admin/
        recepcionista agora agrupam e coloram por **dentista**, não só por
        consultório. Sidebar nova (`dentist-sidebar.tsx`) com checklist,
        cadastro mínimo de dentista (nome, CRO+UF, cor via seletor de 5
        swatches — mockado em `localStorage`, `dentist-store.ts`, ver
        [[Arquitetura]]), agendamentos sobrepostos no mesmo horário
        renderizam lado a lado (`slotEventOverlap={false}`, sem plugin de
        resource view). Dentista logado continua vendo a agenda exatamente
        como antes — RBAC real na camada de dados (`useAgendaMockData`
        devolve `dentists: []` pra esse papel, sidebar de dentistas nem
        recebe dado). Descoberto e corrigido no caminho: rota `/agenda` e
        `GET /clinics`/`/patients`/`/procedures` não deixavam ADMIN nem abrir
        a tela (403) — RBAC real estendido, ver [[Arquitetura]] e
        [[Funcionalidades e Endpoints]].
  - **Sidebar fixa de verdade** (PR #8, 2026-09-22) — bugfix de layout puro,
        ver [[Problemas Conhecidos]].

- [x] **Consultório de clínica fixo — criação só pra tenant Freelancer**
      (PR #9, 2026-09-22) — `POST /clinics` bloqueia (403) quando
      `organization.type === "CLINIC"`, independente de quem pede — regra
      de negócio no serviço, não só escondida na UI. `GET /organization`
      passa a expor `type` pro front decidir quando mostrar "Novo
      consultório". Ver [[Arquitetura]] e [[Funcionalidades e Endpoints]].
- [x] **`TenantType.FREELANCER` de verdade + personas de teste** (PR #10,
      2026-09-22) — migration aditiva (`ALTER TYPE ... ADD VALUE`), sem
      impacto em dado existente. Script idempotente
      `apps/api/prisma/scripts/seed-test-personas.ts` cria 4 contas com
      senha padrão `senha123456`: uma dentista freelancer (tenant próprio, 2
      consultórios já seedados) e uma clínica com admin fundador + dentista
      de equipe + recepcionista — ver [[Infraestrutura e Deploy]].
- [x] **Relação financeira do freelancer por consultório —
      `ClinicFinancialTerms`** (PR #11, 2026-09-22) — plano aprovado sem
      mudança de desenho, ver
      [[Relação Financeira do Freelancer por Consultório — Plano Técnico]].
      Model novo 1:1 com `Clinic`, opcional, `relationshipType`
      (`RENTED_FIXED`/`COMMISSION`/`PER_SERVICE`, discriminated union no
      Zod) + campos específicos por tipo + `ownerLabel` livre. Endpoint novo
      `GET`/`PUT clinics/:clinicId/financial-terms`, `DENTIST`-only (mesmo
      escopo de `attendances`/`reports/financial`) — só tenant Freelancer
      pode ter/editar, 403 pra tenant Clínica mesmo no próprio consultório.
      Trocar de tipo zera de verdade os campos do tipo anterior. Fora do v1,
      por decisão consciente: `ClinicProcedureRate` (rate por procedimento),
      `YEARLY` em `RentPeriodicity`, `ownerLabel` como entidade.
- [x] **Sidebar da Agenda conectada a consultórios e dentistas reais**
      (PR #12, 2026-09-22) — plano aprovado sem mudança de desenho, ver
      [[Conectar Agenda a Dados Reais de Consultório e Dentistas — Plano
      Técnico]]. Migration aditiva: enum `PaletteColorToken` +
      `Clinic.colorToken` + `User.colorToken` (nullable, fallback por índice
      quando ausente, mesma lógica que antes vivia no front). Endpoint novo
      `GET organization/dentists` (`ADMIN`/`RECEPTIONIST` — `DENTIST` recebe
      403 de verdade, não filtro de UI; só dentista ativo). Front:
      `useAgendaMockData` → `useAgendaData`, com `resolveCalendarAxis()`
      como único ponto de decisão do eixo (`organization.type` + papel):
      `"location"` (freelancer, `Clinic` reais), `"dentist"` (clínica
      admin/recepcionista, `User` reais `role: DENTIST`), `"none"` (clínica
      dentista, sem sidebar, só a própria agenda). `dentist-store.ts`
      (mock/`localStorage`) removido. `calendar-sidebar.tsx` ganhou "+ Novo
      consultório" (freelancer, `POST /clinics` real) e seletor de cor
      (`PATCH /clinics/:id`); `dentist-sidebar.tsx` perdeu o cadastro de
      dentista (fica em Usuários) e usa `PATCH /users/:id` pra cor.
      **Agendamentos/calendários em si continuam mockados** — só o roster
      (quem aparece na sidebar) virou real, ver "Em planejamento" abaixo.
- [x] **4 PRs (#9–#12) mergeados em `main`** (2026-09-22) — sequência
      completa, todos com CI verde, sem conflito.

- [x] **Fase 0 de segurança + matriz de acesso + múltiplos papéis**
      (2026-09-26, PRs #13–#21) — itens do
      [[roadmap-papeis-permissoes-financeiro]]: cobertura do isolamento
      (#13), RBAC negando por padrão (#14), login sem `auth/lookup` (#15),
      rate limit + helmet (#16), refresh token revogável com rotação (#17),
      matriz `ACCESS` centralizada (#18), `User.roles` (#19) e confirmação ao
      dar/tirar Admin (#21), e remoção da coluna antiga `User.role` (#20,
      mergeado depois do deploy do #19). Detalhes em [[Arquitetura]] e
      [[Funcionalidades e Endpoints]]; avisos de deploy em
      [[Infraestrutura e Deploy]].

## Evolução SaaS (infraestrutura/multi-tenant, fases)

Consolidação do que antes vivia espalhado — decisão de sequência e status
atual de cada fase, cruzando com o que realmente foi implementado:

| Fase | O quê | Status |
| --- | --- | --- |
| **Fase 1 — Multi-tenancy** | `Organization`, isolamento por `organizationId`, Prisma Client Extension | ✅ Implementada em 2026-09-17 (ver "Feito" acima), com desenho final diferente do plano original (identidade isolada por tenant em vez de `Membership`) |
| **Fase 2 — Painel administrativo da plataforma** | App `apps/admin` separado pra Super Admin | Adiada — hoje o Super Admin já tem `/platform` dentro do `apps/web` (dashboard implementado em 2026-09-18), suficiente enquanto o volume de tenants cabe em provisionamento manual |
| **Fase 3 — Geração de documentos** (PDF/Word/Excel) | `pdfkit`/`docx`/`exceljs` — orçamento assinável, anamnese/prontuário exportável, relatório financeiro em Excel | Não iniciada — não depende de nenhuma fase anterior, pode entrar a qualquer momento que vire prioridade |
| **Fase 4 — Billing das assinaturas** | Stripe Billing, cobrar as clínicas (não o paciente) | Adiada — com poucos clientes nomeados, cobrança é manual (Pix/transferência); só vale a pena com volume que não caiba mais em contato direto |

**Contexto da decisão de sequência (2026-08-02)**: 1 dentista validaria o MVP
em uso real primeiro; só depois disso entraria uma segunda clínica — não era
"SaaS especulativo", eram 2 clientes nomeados com ordem definida. O Vitor
antecipou a Fase 1 em 2026-09-17 por causa da agenda multi-consultório, antes
do gatilho original ("só quando a clínica 2 for confirmada") — decisão
consciente, não esquecimento da regra antiga.

**Desenho original (Fase 1) vs. o que foi implementado**: o plano original
(schema Prisma sketch, `TenantContext` via `AsyncLocalStorage`, Prisma Client
Extension, JWT com `tenantId`) propunha `tenantId` solto em cada model e
renomear `Clinic`→`ClinicUnit`. O que foi implementado de fato usa
`Organization` como raiz do tenant (não só um campo solto) e manteve `Clinic`
como está — renomear fica pra quando a camada `Location`/`Calendar` da agenda
entrar. O essencial se confirmou certo: isolamento via Prisma Client
Extension (não RLS), `AsyncLocalStorage` pro contexto, migração com backfill
em vez de `NOT NULL` direto. Ver [[Arquitetura]] pro desenho técnico final.

**Riscos avaliados, status pós-implementação**: dinheiro continua `Decimal`
(não migrado pra inteiro em centavos) — seguro, `Decimal` do Prisma não é
`float`. Zero `$queryRaw` em todo `apps/api` (auditado) — a extension cobre
100% das queries, nenhum SQL cru escapando do isolamento. Migração com
backfill feita em 3 passos reais contra dev local; produção foi resetada do
zero em vez de migrada incrementalmente (ver [[Infraestrutura e Deploy]]).

Documento fonte completo (conversa de arquitetura original com o Gemini +
análise crítica + os sketches técnicos completos das 4 fases):
[[Plano de Implementação SaaS - Conversa Completa]] (seção "PARTE D" tem o
schema/pseudocódigo originais).

## Visão de produto mais ampla (comparativo com concorrentes)

Trazida em 2026-09-17, ainda **não reconciliada com o MVP já em produção** —
tratar como análise de mercado, não como compromisso de escopo até um item
daqui virar entrada explícita em "Feito"/"Em planejamento" acima. Baseada em
comparativo com Clínica Experts, Clinicorp e Codental.

**Três trilhas**: MVP (sequencial, dia 1) → Add-ons e Incrementos (paralelos
entre si, começam quando o MVP estabilizar em produção — Add-ons atende
freelancer/início de carreira e pode vender a qualquer momento; Incrementos
evolui o produto todo, fase após fase, guiado por adoção real, não calendário
fixo).

- **MVP** (critério: presente no plano de entrada de 2+ concorrentes) —
  agenda online c/ link público, prontuário digital completo, financeiro
  básico, confirmação via WhatsApp, usuários ilimitados, app mobile básico,
  suporte chat/WhatsApp. **Fora por decisão consciente**: CRM, estoque,
  comissões, NF-e, campanhas automáticas, assinatura digital ilimitada, IA.
- **Add-ons** (pós-MVP, paralelo, cobrança por profissional ativo não por
  clínica) — CRM leve de captação, link de agendamento público + mini-site,
  integração Instagram, site institucional, assinatura digital ilimitada.
  Marco antes de expandir: validar CRM leve + mini-site com 5–10 freelancers
  reais.
- **Incremento 1 — Crescimento** (gatilho: base validada + pedidos
  recorrentes de estoque/comissões) — comissões automáticas, estoque,
  campanhas automáticas, tags de paciente, contrato digital, boleto com baixa
  automática.
- **Incremento 2 — Escala** (gatilho: primeiras clínicas multi-unidade) — CRM
  completo, NF-e/NFS-e nativa, multi-unidade (central de acesso/ranking),
  dashboard analítico avançado, meios de pagamento integrados, câmera
  intraoral.
- **Incremento 3 — IA e diferenciação** (gatilho: volume de dado/conversa
  suficiente pra IA agregar valor real, não vitrine vazia) — assistente IA
  via WhatsApp, transcrição de evolução clínica por IA, pesquisa de
  satisfação automática, teleconsulta, chatbot de triagem/pré-anamnese.

## Em planejamento

- **Agenda multi-consultório (backend real)** — ver
  [[Implementação da Agenda Multi-Consultório — Plano Técnico]]. O roster
  da sidebar (consultórios/dentistas + cor) já é real (PR #12, ver "Feito"
  acima) — o que falta agora é só o `Location`/`Calendar` de verdade no
  schema e `GET /calendars` filtrado por papel; **os agendamentos em si
  continuam 100% mockados** (`useAgendaMockData` pro conteúdo do
  calendário). RBAC de admin/recepcionista vendo todas as agendas e
  dentista só a própria já está real pro roster (`GET organization/dentists`),
  falta pro conteúdo do calendário em si. Pontos ainda em aberto no plano
  técnico: fluxo de agendamento com detecção de conflito, confirmação via
  WhatsApp, regra de cor além da paleta fixa de 5.
- **Painel administrativo — RBAC, multi-tenant e LGPD** — decisões 🔴 P0 de
  [[feature - painel-admin-rbac-lgpd]] **já implementadas** (ver "Feito"
  acima), incluindo `TenantType.FREELANCER` de verdade (PR #10, 2026-09-22).
  O que resta desse documento (🟡 P1, por decisão consciente): cadastro
  progressivo Fase 1/2 com prazo de 7 dias, consentimento LGPD em dois
  momentos, `AUDIT_LOG`, retenção de prontuário (10 anos mínimo, CFO
  91/2009), impersonation.
- **Painel do Super Admin — próximas fases** — o dashboard de 2026-09-18 (ver
  "Feito") cobre só o que o schema atual sustenta. Fora por depender de
  dado/modelo que ainda não existe: plano/tier por tenant e billing/MRR/ARR
  (depende da Fase 4 da Evolução SaaS, adiada), add-ons e sua monetização,
  dados cadastrais tipo CNPJ/endereço/geolocalização, `AUDIT_LOG`, tracking
  de login/uso (health score, churn), armazenamento por clínica, alertas
  proativos.
- **Cadastro de dentistas — versão real** — nome/email/senha/papel/cor já são
  reais via `/admin/users` (`POST`/`PATCH /users`, `colorToken` incluso
  desde PR #12). Falta só migrar CRO+UF (ainda não existe nem mockado nem
  real) pra campos reais quando esse dado for exigido de fato.
- **KPIs em cima de `ClinicFinancialTerms`** — o model/endpoint já estão
  implementados (ver "Feito" acima); os KPIs propostos no plano técnico
  (ex.: custo total de aluguel/comissão por consultório) ainda não têm tela
  nem cálculo — ficam pra quando o relatório financeiro for revisado. Ver
  [[Relação Financeira do Freelancer por Consultório — Plano Técnico]].

## Não feito ainda / próximos passos naturais
- [x] Conta real da dentista criada pelo painel admin — depois apagada no
      reset de produção de 2026-09-18 (ver [[Infraestrutura e Deploy]]),
      recriar antes de repassar acesso de novo.
- [x] **Reset de senha pelo admin** — `PATCH /users/:id/password`.
- [ ] Ver seção "Evolução SaaS" acima pro que vem depois da fundação de
      multi-tenancy (já implementada) — Fase 3 (documentos) é a próxima com
      valor imediato e sem dependência.
- [ ] Camada de agenda multi-consultório **backend real** (`Location`/
      `Calendar`, `GET /calendars` filtrado) — a UI já está pronta, ver "Em
      planejamento" acima.
- [ ] Migrar Home Dashboard, Procedimentos e a parte ainda mockada da Agenda
      (calendários/agendamentos em si — o roster de consultórios/dentistas
      já é real desde PR #12) de `useMock*` pra endpoints reais.
- [ ] Integração com convênios, nota fiscal automática, lembrete por
      WhatsApp/SMS — seguem fora do escopo, sem mudança.

## Notas de conhecimento (referência técnica, não são itens de roadmap)

Material de consulta sobre o que já existe e como funciona — não representa
trabalho pendente, é o "como as coisas são hoje":

- [[Arquitetura]] — stack, schema de dados, módulos do backend, estrutura do
  frontend, desenho de multi-tenancy, padrão mock-primeiro do redesenho visual
- [[Funcionalidades e Endpoints]] — todo endpoint da API com exemplo real de
  request/response e regra de negócio
- [[Infraestrutura e Deploy]] — banco local de dev, credenciais de teste,
  deploy em produção
- [[Problemas Conhecidos]] — bugs já corrigidos e avisos de editor que não
  são bugs de verdade
- [[Visão Geral]] — o que é o produto + linha do tempo de decisões
- [[design-system-plataforma-odontologica]] — tokens visuais oficiais (cor,
  tipografia, espaçamento, raio) + referência de estrutura extraída do
  TailAdmin — ler antes de qualquer trabalho de estilo
- [[contexto-geral-plataforma-odontologica]] — nota de re-briefing rápido
  (decisões de produto fixas + prompt pronto pra colar no início de uma
  sessão nova), complementar a esta e a [[Visão Geral]]

Documentos de **análise/discussão** que embasam decisões futuras, mas não são
nem "feito" nem um item de roadmap já priorizado até aparecerem explicitamente
nas seções acima:

- [[Plano de Implementação SaaS - Conversa Completa]] — conversa completa de
  arquitetura que originou a seção "Evolução SaaS" acima (Fases 1-4)
- [[feature - painel-admin-rbac-lgpd|Painel Administrativo — RBAC, Multi-tenant e LGPD]] —
  decisões 🔴 P0 já implementadas (ver "Feito"); tenant tipo Freelancer (P1)
  também já implementado (PR #10, 2026-09-22); itens 🟡 P1 restantes
  (cadastro progressivo, LGPD, auditoria, impersonation) seguem como
  análise, não roadmap priorizado ainda
