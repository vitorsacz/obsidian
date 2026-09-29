#projeto #dentist-system

## O que é

Sistema de gestão para uma dentista que atende em mais de um consultório (próprio
e/ou alugado). Fase de teste (MVP), uso inicial por uma única dentista, cobrindo
pacientes/prontuário, orçamento, agenda e financeiro com repasse por consultório.

Especificação original: `prompt-sistema-odontologico` (documento que o Vitor
escreveu antes de começarmos a construir) — **não encontrado no vault** na
reorganização de 2026-09-22, link ficava apontando pra um arquivo que não
existe na raiz nem em nenhuma pasta; se reaparecer, restaurar o link.

Repositório: `~/VITOR/dentist-system` — remote já configurado em
[GitHub — vitorsacz/dentist-system](https://github.com/vitorsacz/dentist-system).
Primeiro commit feito localmente (`c6b1efb`), **ainda sem push**.

## Por que a arquitetura se parece com o hubassistent

O Vitor já tinha resolvido esse tipo de projeto antes (monorepo React+Vite /
NestJS+Prisma, deploy Vercel+Render+Supabase) no [[hubassistent/Visão Geral|hubassistent]].
Em vez de redescobrir decisões, replicamos a base validada — mesma estrutura de
pastas, mesmo padrão de auth JWT, mesma forma de lidar com o pooler do Supabase —
e ajustamos só o que o domínio odontológico exige de diferente.

**A diferença estrutural mais importante**: no hubassistent cada usuário é dono
isolado dos próprios dados. Aqui, dentro de uma mesma organização, dentista e
recepcionista **compartilham os mesmos pacientes/agenda/orçamentos** — a
separação não é por dono do dado, é por **papel** (`role`), que restringe quais
campos/endpoints cada perfil acessa. Isso levou à criação de um `RolesGuard`
que não existia no hubassistent. Desde 2026-09-17 existe também isolamento
entre organizações diferentes (multi-tenancy — ver [[Arquitetura]]), mas isso
é ortogonal ao ponto acima: dentro da mesma organização, o compartilhamento
por papel continua exatamente como sempre foi.

## Notas relacionadas

**Roadmap** (o que fazer / prioridade):
- [[roadmap-papeis-permissoes-financeiro]] — **roadmap ativo** (desde
  2026-09-25): papéis, permissões e financeiro, fases 0 a 7, com o progresso
  de cada item e os PRs
- [[Roadmap]] — histórico do que já foi construído até 2026-09-22 + evolução
  SaaS e visão de produto

**Conhecimento** (como as coisas são hoje, referência técnica):
- [[Funcionalidades e Endpoints]] — **norte funcional**: todo endpoint da API com
  exemplo real de request/response, regra de negócio de cada um, pegadinhas a lembrar
- [[Arquitetura]] — stack, schema de dados, módulos do backend, estrutura do frontend
- [[Infraestrutura e Deploy]] — banco local de dev, credenciais de teste, plano de deploy
- [[Problemas Conhecidos]] — bugs já encontrados/corrigidos e avisos de editor que não são bugs

**Análise/planejamento** (embasam decisão futura, ainda não é compromisso de escopo):
- [[Roadmap]] (seção "Evolução SaaS") — proposta de evolução pra multi-tenant (clínica 2),
  painel de plataforma, geração de documentos e billing — análise crítica de onde
  aplicar e em que ordem (Fase 1 já implementada, com desenho diferente do original)
- [[Roadmap]] (seção "Visão de produto mais ampla") —
  visão de produto comparada a concorrentes
- [[feature - painel-admin-rbac-lgpd|Painel Administrativo — RBAC, Multi-tenant e LGPD]] —
  decisões 🔴 P0 (identidade isolada por tenant) já implementadas em
  2026-09-17 (ver linha do tempo abaixo); itens 🟡 P1 (cadastro progressivo,
  LGPD, auditoria, Freelancer) seguem como backlog, ver [[Roadmap]]

## Linha do tempo de decisões
- **2026-07-28** — plano de arquitetura definido (monorepo pnpm+Turborepo,
  NestJS+Prisma no backend, React+Vite no frontend, Postgres via Supabase em
  produção). Backend completo implementado: 14 módulos cobrindo todos os
  módulos do MVP descrito no prompt original, incluindo odontograma e retorno
  (que o prompt já previa como "podem entrar na segunda iteração" — entraram
  direto porque o custo incremental era baixo). Frontend completo com todas as
  telas. Testado ponta a ponta localmente (Postgres via Homebrew, não Supabase
  ainda) via curl e navegador real.
- **2026-07-28 (mesmo dia)** — Vitor pediu um papel de **admin** separado, com
  painel próprio para criar/gerenciar dentistas e recepcionistas — até então
  quem criava usuários era a própria dentista. Isso levou a uma revisão de
  segurança: descobri que vários controllers (pacientes, orçamentos, agenda,
  financeiro, estoque, consultórios, procedimentos) não tinham `@Roles(...)`
  nenhum, ou seja, qualquer papel autenticado passava. Adicionei restrição
  explícita em todos, deixando o admin de fora de tudo que é clínico/financeiro
  por padrão (ver [[Arquitetura]] para a tabela completa de quem acessa o quê).
- **2026-08-01** — primeiro commit do repositório (`c6b1efb`, ainda local).
  Odontograma ganhou mapa visual da arcada (32 dentes clicáveis, cor por
  status calculado a partir do registro mais recente do dente), substituindo
  a lista/tabela original — ver [[Roadmap]].
- **2026-08-03** — deploy real no ar (Vercel+Render+Supabase), reset de senha
  pelo admin, botão "Marcar como realizado" no odontograma, correção do bug de
  logout não limpando o cookie em produção — ver [[Infraestrutura e Deploy]] e
  [[Problemas Conhecidos]].
- **2026-08-06** — orçamento e odontograma passam a se conversar: formulário
  de novo orçamento sugere os dentes com status Planejado como chips
  clicáveis, tentando casar o procedimento (texto livre) com o catálogo por
  substring — ver [[Funcionalidades e Endpoints]].
- **2026-09-17** — planejamento técnico da agenda multi-consultório (visão
  estilo Google Calendar, cor por consultório) antecipa a Fase 1 de
  multi-tenancy do [[Roadmap]] (seção "Evolução SaaS"), antes do gatilho original
  de "clínica 2 confirmada" — decisão consciente do Vitor. Também chegou ao
  vault um documento de visão de produto mais amplo, comparando com
  concorrentes (Clínica Experts, Clinicorp, Codental) — ver
  [[Roadmap]], seção "Visão de produto mais ampla" — ainda não reconciliado
  com o MVP já em produção.
- **2026-09-17 (mesmo dia)** — fundação de multi-tenancy **implementada e
  testada localmente** (não só planejada): `Organization`+`Membership`,
  `organizationId` nos 14 models de negócio, isolamento via Prisma Client
  Extension + `AsyncLocalStorage`, migração em 3 passos com backfill,
  separação de role de banco documentada (script pronto, não aplicado em
  produção ainda), suíte e2e de isolamento cross-tenant rodando no CI. Duas
  pegadinhas reais descobertas no processo: TypeScript exige `organizationId`
  explícito em todo `create()` mesmo a extension injetando em runtime (a
  coluna é `NOT NULL`, TS não vê a injeção dinâmica); `PrismaPromise` é
  preguiçosa, então o callback do `AsyncLocalStorage.run()` precisa ser
  `async` pra não escapar do contexto antes da query disparar de verdade. Ver
  [[Arquitetura]] pro desenho completo. **Ainda não migrado em produção** —
  falta rodar os scripts de backfill + migration final contra o Supabase real.
  A camada de agenda (`Location`/`Calendar`, FullCalendar) continua como
  próxima etapa — ver
  [[Implementação da Agenda Multi-Consultório — Plano Técnico]].
- **2026-09-17 (mesmo dia, terceira rodada)** — o Vitor trouxe uma análise
  crítica feita numa sessão separada (Claude na web) sobre o painel
  administrativo, registrada em [[feature - painel-admin-rbac-lgpd]], e pediu
  pra seguir esse modelo à risca — o que significou **substituir** a
  fundação de `Membership` implementada horas antes por um modelo de
  identidade **isolada por tenant** (motivo: privacidade — uma clínica nunca
  pode saber que um dentista atende em outro lugar, mesmo sendo a mesma
  pessoa). `Membership` foi criada e removida no mesmo dia. Implementado:
  `User.organizationId` direto (nullable só pra Super Admin), e-mail único só
  dentro do tenant, `nickname` global opcional, Super Admin fora de qualquer
  organização (módulo `platform` — cria clínicas + admin fundador numa
  transação), capitania da clínica (só o fundador edita outro admin da mesma
  clínica), login em 2 passos (`auth/lookup` → `auth/login`, escolhe
  organização só se o e-mail se repetir entre tenants). Fora desta rodada,
  documentado mas não implementado: cadastro progressivo, consentimento LGPD,
  auditoria, tenant tipo Freelancer, impersonation. Testado com 18 testes e2e
  (5 da extension + 13 de isolamento, incluindo os 4 casos novos: login com
  e-mail repetido, login por nickname, capitania, `SuperAdminGuard`) e smoke
  test manual completo (login dos 3 perfis + Super Admin criando uma clínica
  nova pelo painel). Ver [[Arquitetura]] e [[Funcionalidades e Endpoints]]
  pro desenho final — essas notas já refletem só o modelo atual, sem histórico
  do `Membership` (isso fica registrado aqui e no [[Roadmap]]).
- **2026-09-18** — `/platform` ganhou um dashboard de verdade pro Super Admin
  (stat cards, gráficos ApexCharts, drill-down por clínica), seguindo
  [[design-system-plataforma-odontologica]]. Mecanismo técnico novo: token DI
  `PRISMA_UNSCOPED_SERVICE` pra agregação cross-tenant (ver [[Arquitetura]]).
  No mesmo dia, produção (Supabase) foi **resetada do zero**
  (`prisma migrate reset --force`, sem dado produtivo real em jogo,
  confirmado antes) e recriada já com o schema de multi-tenancy — login
  confirmado via navegador real pros 3 perfis na URL de produção. Ver
  [[Infraestrutura e Deploy]] pro gotcha de bash `source`+`$` que corrompeu a
  primeira tentativa de seed.
- **2026-09-21** — sequência de trabalho de redesenho visual completo,
  seguindo a filosofia "camada visual/UX primeiro com dado mockado, backend
  depois" (ver [[Arquitetura]]):
  - GitHub Repository Ruleset criado pra bloquear push direto em `main` —
    branch protection clássica **não bloqueava** de verdade (testado e
    descartado antes de trocar de abordagem).
  - CI corrigido de ponta a ponta (nunca tinha rodado completo) — cadeia de
    5 falhas em sequência, cada uma só visível depois de corrigir a anterior
    (ordem setup-node/pnpm, cache quebrado, Jest exigindo ts-node, ESLint sem
    globals de Node, Turborepo modo estrito descartando env vars). Ver
    [[Roadmap]] pro detalhe de cada uma.
  - Home Dashboard redesenhada (header, stat cards, gráficos) + migração dos
    tokens do Tailwind config pros valores oficiais do design system.
  - Agenda reconstruída do zero em cima do FullCalendar — grade estilo
    Google Calendar (dia/semana/mês), sidebar de consultórios coloridos,
    criação por arrastar-e-soltar, painel lateral de detalhe (não modal).
  - Menu horizontal substituído por sidebar fixa global — grupos por seção,
    colapsável, rodapé com avatar/nome/papel/sair, aplicada em todas as
    páginas via `AppShell`.
- **2026-09-22** — Catálogo de Procedimentos redesenhado (filtros, categoria,
  paginação, modal criar/editar). Agenda ganhou visão multi-dentista pro
  admin/recepcionista (cor por dentista, sidebar de checklist, cadastro
  mínimo de dentista mockado, agendamentos sobrepostos lado a lado) — no
  processo, descoberto e corrigido um RBAC real incompleto (`ADMIN` não
  conseguia nem abrir a Agenda: `/clinics`, `/patients`, `/procedures`
  respondiam 403 pra esse papel). Corrigido também um bug real de layout: a
  sidebar rolava junto com a página em vez de ficar fixa (`min-h-screen` sem
  `min-h-0` no `<main>`, ver [[Problemas Conhecidos]]). Vault reorganizado no
  mesmo dia — arquivos duplicados/soltos (design system, roadmap de produto,
  plano SaaS distilado, prompt de contexto, conversa duplicada do Clinicorp)
  consolidados em menos notas, e todo o trabalho acima documentado aqui pela
  primeira vez.
- **2026-09-22 (mais tarde no mesmo dia)** — 4 PRs sequenciais (#9→#12)
  implementados e mergeados em `main`, evoluindo o modelo de tenant
  Freelancer que ainda era só simulado: consultório de clínica passou a ser
  fixo (só Freelancer cria novos, PR #9); `TenantType.FREELANCER` ganhou
  existência real no schema + script de personas de teste (PR #10);
  `ClinicFinancialTerms` (modelo novo, relação financeira do freelancer com
  cada consultório — alugado fixo/comissão/por serviço) e seu endpoint,
  seguindo o plano técnico já aprovado (PR #11); e a sidebar da Agenda
  trocou o roster mockado por `Clinic`/`User` reais nos dois eixos, com
  `colorToken` real no schema (PR #12) — os agendamentos/calendários em si
  continuam mockados, só quem aparece na sidebar virou real. Ver [[Roadmap]]
  seção "Feito" pro detalhe técnico de cada PR.
- **2026-09-25** — análise do projeto a partir do vault gerou
  [[Pontos de Atenção — Análise 2026-09-25]] (clínica nova sem consultório nem
  procedimento, agenda e catálogo mockados, recall sem tela…) e
  [[Responsabilidades por Perfil e Novos Requisitos]] (Super Admin, admin da
  clínica, dentista da clínica, freelancer). O Vitor consolidou tudo em
  [[roadmap-papeis-permissoes-financeiro]], com decisões fechadas (D1 a D6) e
  fases 0 a 7. Ambiente local montado no Windows (Postgres em container
  Docker) — ver [[Infraestrutura e Deploy]].
- **2026-09-26** — execução do roadmap novo, um prompt por item, cada um com
  branch, testes e2e, teste no Chrome, PR e merge com CI verde. **Fase 0 de
  segurança inteira na `main`**: S5 cobertura do isolamento (#13), S4 RBAC
  negando por padrão (#14), S1 login sem `auth/lookup` (#15), S2 rate limit +
  helmet (#16), S3 refresh token opaco, rotativo e revogável (#17). Depois,
  matriz de permissões centralizada em `ACCESS` (#18) e **R1 múltiplos papéis
  por usuário** (#19, `role` → `roles[]`, migration expandir → contrair; a
  parte que remove `role`, #20, foi mergeada depois que o deploy do #19
  ficou Live); confirmação ao dar/tirar Admin (#21). Suíte e2e foi de 4 para
  10 arquivos (82 testes). Bugs reais achados no teste no navegador
  (P2028 do Prisma com transações concorrentes, modal deslocado) estão em
  [[Problemas Conhecidos]]. Decisões tomadas no caminho (capitania mantida,
  refresh 30/min, Web Locks entre abas…) em
  [[roadmap-papeis-permissoes-financeiro#Progresso]]. **Ao fazer o deploy,
  todos os usuários precisam logar de novo** (S3). Próximo: R3 → P3.
