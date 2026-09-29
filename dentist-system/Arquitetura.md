#projeto #dentist-system #arquitetura

Ver [[Visão Geral]] para o contexto do produto.

## Monorepo

pnpm workspaces + Turborepo (mesmo padrão do [[hubassistent/Arquitetura|hubassistent]]).

```
dentist-system/
├── apps/
│   ├── api/     # NestJS + Prisma
│   └── web/     # React + Vite
└── packages/
    ├── shared-types/    # schemas Zod compartilhados (contrato de API)
    ├── eslint-config/
    └── tsconfig/
```

## Backend (NestJS)

Prisma 5.22 (fixado deliberadamente, não é a versão mais nova — ver
[[Problemas Conhecidos]] sobre os avisos do editor por causa disso).

### Modelo de dados

Entidades principais: `User` (identidade **isolada por tenant** — ver Multi-tenancy
abaixo), `Clinic` (consultório, `OWN`/`RENTED` + `dailyRentValue`), `Patient`,
`Anamnesis` (1:1 com paciente), `ClinicalRecord` (evolução, N por paciente),
`ToothRecord` (odontograma, histórico por dente em notação FDI),
`ProcedureCatalog`, `Budget`/`BudgetItem`, `Appointment`, `Attendance`
(registro financeiro do atendimento), `Material`/`MaterialBatch`/`MaterialUsage`
(estoque com baixa por lote), `Recall` (controle de retorno).

**`ClinicFinancialTerms`** (2026-09-22, PR #11) — 1:1 opcional com `Clinic`,
relação financeira do dentista Freelancer com um consultório específico
(conceito paralelo a `Clinic.type`/`dailyRentValue`, que é pra clínica
pagando aluguel internamente). `relationshipType`
(`LocationRelationshipType`: `RENTED_FIXED`/`COMMISSION`/`PER_SERVICE`) +
campos exclusivos por tipo (`rentValue`+`rentPeriodicity`,
`commissionPercentage`, ou `defaultServiceRate`) + `ownerLabel` livre. Regra
de negócio no serviço, não no schema: só tenant `FREELANCER` pode ter/editar
(403 pra tenant `CLINIC`, ver `ClinicFinancialTermsService.upsert()`); trocar
`relationshipType` zera de verdade os campos do tipo anterior
(`toPersistedFields()`). Ver [[Relação Financeira do Freelancer por
Consultório — Plano Técnico]] e [[Funcionalidades e Endpoints]].

**`PaletteColorToken`** (2026-09-22, PR #12) — enum `BRAND`/`SUCCESS`/
`WARNING`/`ERROR`/`INFO`, campo `colorToken` nullable em `Clinic` e `User`
(identidade visual na sidebar da Agenda — nunca cor de status de
agendamento, propositalmente separado). Se omitido na criação, o serviço
cicla a paleta por índice (`ClinicsService.create()`/`UsersService.create()`
— antes essa lógica vivia no front). Front usa os mesmos 5 valores em
minúsculo (`@/lib/palette-colors`); `fromApiColorToken()`/`toApiColorToken()`
em `location-colors.ts` fazem a conversão de caixa.

**`User.roles`** (2026-09-26, R1, PR #19): `Role[]` no lugar do antigo
`role` único — um usuário pode ter mais de um papel (ex.: dono da clínica
que também atende = `[ADMIN, DENTIST]`). Vazio só pro Super Admin. Migração
em dois PRs (**expandir → contrair**): o #19 criou `roles` e copiou
`[role]` mantendo `role` sem uso (a API antiga segue funcionando durante o
deploy no Render); o **#20** (mergeado depois que o deploy do #19 ficou Live)
repreenche quem a API antiga criou no intervalo e faz `DROP COLUMN "role"`.
Desde então `User` só tem `roles`.
Filtros usam `roles: { has: "DENTIST" }`.

**`RefreshSession`** (2026-09-26, S3, PR #17): uma linha por refresh token.
Guarda só o **hash SHA-256** do token (o token é opaco, 32 bytes aleatórios,
e só existe no cookie), `familyId` (= um login num navegador), `expiresAt`,
`revokedAt`, `replacedById`, `userAgent`, `ip`. **Sem `organizationId`** de
propósito: a sessão é do usuário e é consultada no refresh, antes de existir
contexto de tenant (fica fora da extension, e o teste de cobertura do S5
continua passando sem exceção). Ver "Autenticação e papéis" abaixo.

### Multi-tenancy (2026-09-17, **revisado no mesmo dia** — ver aviso abaixo)

Todo model de negócio acima carrega `organizationId` (14 models, adicionado
por completo — inclusive filhos tecnicamente alcançáveis via relação, como
`BudgetItem`/`MaterialBatch`/`MaterialUsage`, porque vários deles são
consultados diretamente pelos services, não só via join). `ProcedureCatalog`
também é por organização — cada clínica tem seu próprio catálogo/preço.

**⚠️ Mudança de desenho no mesmo dia**: a primeira versão da fundação de
multi-tenancy usava `User` global + `Membership` many-to-many (um usuário podia
pertencer a várias organizations). O Vitor rodou uma sessão de análise crítica
separada (documentada em [[feature - painel-admin-rbac-lgpd]]) que decidiu o
oposto por motivo de **privacidade**, não só técnico: identidade deve ser
**isolada por tenant** — cada organização tem seus próprios usuários,
completamente independentes, mesmo que seja a mesma pessoa física em outra
clínica (uma clínica nunca pode saber que um dentista atende em outro lugar).
`Membership` foi **removida** do schema no mesmo dia em que foi criada. O que
seguiu válido da primeira versão: `Organization` como raiz do tenant, todo o
desenho de isolamento por `organizationId` nos 14 models de negócio, a Prisma
Client Extension, o `TenantContextInterceptor`. O que mudou:

- `User` agora tem `organizationId` **direto** (não via tabela de junção),
  nullable — `null` só pra Super Admin (ver abaixo). O papel voltou a viver em
  `User` — desde 2026-09-26 como lista `roles` (R1; antes `role` único).
- E-mail deixou de ser único globalmente — só dentro do tenant
  (`@@unique([organizationId, email])`). A mesma pessoa em 2 clínicas = 2
  contas completamente separadas, sem nenhuma referência cruzada no banco.
- `nickname` (opcional, único globalmente) permite logar sem escolher
  organização mesmo se o e-mail se repetir entre tenants.
- **Super Admin** (`User.isSuperAdmin`, `organizationId: null`, `roles: []`)
  é a identidade da plataforma (o Vitor), fora de qualquer organização — cria
  clínicas novas pelo módulo `platform` (ver abaixo). Nunca é a mesma conta
  que administra uma clínica.
- **Capitania da clínica**: `Organization.foundingAdminUserId` — só o admin
  fundador de uma organização pode editar/desativar outro `ADMIN` da mesma
  organização (`users.service.ts`, checado em `update()`). Transferência de
  capitania é exclusiva do Super Admin (`PATCH platform/organizations/:id/founding-admin`),
  nunca self-service.
- **Login multi-tenant sem vazar organizações** (2026-09-26, S1, PR #15 —
  substituiu o login em 2 passos com `POST auth/lookup`, que era público e
  devolvia os nomes das organizações de qualquer e-mail): `POST auth/login
  { identifier, password, organizationId? }` valida a senha **antes** de
  revelar qualquer organização. Nenhuma conta confere → `401` genérico (e-mail
  inexistente roda `bcrypt.compare` contra um hash fixo, pro tempo não
  denunciar); uma → tokens; mais de uma sem `organizationId` → `200` sem
  tokens com `{ requiresOrganizationSelection: true, accounts }`, só as
  organizações em que a senha conferiu. O front mostra a escolha e reenvia com
  `organizationId` (senha validada de novo).
- `Organization` ganhou `type` (`TenantType`) e `status`
  (`ACTIVE`/`SUSPENDED`/`DELETED`, soft delete). `TenantType.FREELANCER`
  ganhou existência real em 2026-09-22 (PR #10, migration aditiva `ALTER
  TYPE ... ADD VALUE`) — até então só `CLINIC` existia no banco. `type`
  agora dirige regra de negócio real, não só rótulo: consultório de tenant
  `CLINIC` é fixo (só `FREELANCER` cria novos, `POST /clinics` 403 pra
  `CLINIC`, PR #9) e `ClinicFinancialTerms` só existe pra `FREELANCER`
  (PR #11, ver "Modelo de dados" acima).

Fora desta rodada, documentado mas não implementado: cadastro progressivo
(Fase 1/Fase 2 com prazo de 7 dias), consentimento LGPD em dois momentos,
`AUDIT_LOG`, impersonation — ver [[feature - painel-admin-rbac-lgpd]] pros
detalhes de cada um.

**Isolamento via Prisma Client Extension**, não RLS do Postgres (decisão
documentada em [[Roadmap]] (seção "Evolução SaaS") e na nota da agenda) —
`apps/api/src/prisma/tenant.extension.ts` + `tenant-context.ts`
(`AsyncLocalStorage`). Um `TenantContextInterceptor` global popula o contexto
a partir de `request.user.organizationId` logo após o `JwtAuthGuard`. Regras
da extension por tipo de operação:
- `findMany`/`findFirst`/`updateMany`/`deleteMany`: injeta `organizationId` no
  `where`.
- `findUnique`: deixa rodar (leitura, sem risco) e faz post-check — se o
  `organizationId` do resultado não bater, retorna `null`.
- `create`/`createMany`: injeta `organizationId` no `data`.
- `update`/`delete`/`upsert`: injeta `organizationId` no `where` via
  **extended whereUnique filtering** (GA desde o Prisma 4.5, já disponível na
  5.22 fixada) — vira um único `UPDATE/DELETE ... WHERE id = $1 AND
  organizationId = $2` atômico, sem pre-check separado. Prisma responde
  `P2025` quando zero linhas batem; a extension converte isso em
  `NotFoundException`.

`User`/`Organization` ficam **fora** da extension (são o próprio mecanismo de
resolução de tenant — login precisa buscar `User` por e-mail sem saber a
organização ainda) — `users.service.ts`/`organization.service.ts`/
`platform.service.ts` filtram `organizationId` manualmente em toda query, sem
rede de segurança automática.

**Fuga deliberada do isolamento pro dashboard do Super Admin** (2026-09-18):
`getTenantContext()` lança erro quando não há contexto de tenant no
`AsyncLocalStorage` — e o Super Admin (`organizationId: null`) nunca tem esse
contexto populado (`TenantContextInterceptor` pula de propósito). Isso
significa que qualquer agregação cross-tenant em model tenant-scoped (ex.:
contar `Attendance` de todas as organizations pro `platform/stats/overview`)
quebraria se passasse pelo client normal (`PRISMA_SERVICE`). Solução: um
segundo token DI, `PRISMA_UNSCOPED_SERVICE` (`apps/api/src/prisma/prisma.service.ts`
+ `prisma.module.ts`), apontando pro `PrismaClient` cru, sem a extension —
usado **só** em `platform-stats.service.ts`. É o único ponto do código que
atravessa tenants de propósito; qualquer outro consumidor desse token fora do
módulo `platform` deve ser tratado como bug de revisão.

**O que a extension não resolve sozinha**: referência cross-tenant por FK no
`create` (ex.: criar um `Attendance` com `patientId` de outra organização) —
cada service que cria um registro com FK pra outro model tenant-scoped
reaproveita o `findOne()` do service correspondente antes de criar
(`appointments`, `recalls`, `budgets`, `attendances`). **Nested writes**
(`budget.create({ data: { items: { create: [...] } } })`) também não disparam
o hook da extension pro model aninhado — único call site no projeto com esse
formato, `organizationId` injetado manualmente em cada item.

**Pegadinha de tipagem descoberta na implementação**: como `organizationId` é
`NOT NULL` no schema, o TypeScript exige o campo em toda chamada de `create()`
mesmo a extension injetando em runtime — ele não "vê" a injeção dinâmica. Cada
`create()` nos services passa `organizationId` explicitamente (via
`getTenantContext()`), redundante com a extension mas necessário pro
`tsc` compilar; a extension continua sendo a rede de segurança real pros
`findMany`/`findUnique`/`update`/`delete`, onde esse problema de tipo não
existe (campos de `where` são sempre opcionais no Prisma).

**Pegadinha de `AsyncLocalStorage` + Prisma**: `prisma.model.create(...)`
retorna uma `PrismaPromise` preguiçosa — só dispara a query de verdade quando
algo dá `await`/`.then()` nela. Um callback como `() => prisma.x.create(...)`
passado pro `AsyncLocalStorage.run()` "sai" do contexto antes da query
realmente rodar, porque o `.then()` acontece depois, fora do escopo síncrono.
Resolvido garantindo `async () => prisma.x.create(...)` (função async,
mesmo sem `await` explícito) em todo ponto que usa o contexto diretamente —
nos services do NestJS isso já é natural, porque todo método usa
`await this.prisma...`.

Migração da fundação original (nunca `NOT NULL` num passo só numa tabela com
dados): `Organization`+`Membership` (aditivo) → `organizationId` nullable nos
14 models + backfill → `NOT NULL` + `DROP COLUMN "role"` de `User`. Depois, a
revisão pro modelo isolado por tenant (mesmo dia): migration aditiva com os
campos novos de `User`/`Organization` (todos nullable, `Membership` mantida
por enquanto) → script `flatten-membership-to-user.ts` (lê a única
`Membership` de cada `User`, escreve `organizationId`/`role` direto nele;
define `foundingAdminUserId` de cada organização como o primeiro `ADMIN`) →
migration final dropando a tabela `Membership`. Scripts em
`apps/api/prisma/scripts/`.

**Separação de role de banco**: `apps/api/prisma/scripts/create-app-role.sql`
cria uma role `app_user` sem ownership pra uso em runtime (`DATABASE_URL`),
independente da decisão de RLS — reduz o raio de um vazamento de credencial de
produção. Ainda não aplicado em produção (script pronto, é um passo manual).

**Teste de isolamento**: primeiro teste real do projeto (`apps/api`'s `test`
script era `echo "no tests yet"`). Jest + `@nestjs/testing` + `supertest`,
banco Postgres separado (`dentist_system_test` local / `TEST_DATABASE_URL` em
CI). `test/tenant-extension.e2e-spec.ts` prova a extension isoladamente
(incluindo propagação pro `tx` de `$transaction`, o risco mais alto do plano);
`test/tenant-isolation.e2e-spec.ts` cobre todo endpoint HTTP tenant-scoped com
2 organizations reais, sempre 404 em acesso cruzado. `test/platform-stats.e2e-spec.ts`
(2026-09-18) prova que `PRISMA_UNSCOPED_SERVICE` atravessa tenants sem lançar
"Tenant context indisponível" e que os agregados batem através de múltiplas
organizations. Rodando no CI (`.github/workflows/ci.yml`, serviço Postgres
dedicado).

**Suítes e2e em 2026-09-26** (10 suítes, 82 testes na `main`):
`tenant-extension`, `tenant-isolation`, `platform-stats`,
`tenant-models-coverage` (S5 — todo model com `organizationId` está em
`TENANT_SCOPED_MODELS`, via `Prisma.dmmf`), `route-policy-coverage` (S4 —
toda rota declara política; lista travada de rotas públicas), `auth-login`
(S1), `rate-limit` (S2), `auth-sessions` (S3), `user-roles` (R1) e
`user-roles-migration` (R1 — roda a **cadeia real** de migrations num schema
Postgres descartável e confere o preenchimento de `roles`). Todas sobem a app
com `configureApp()` (`src/app.setup.ts`), os mesmos middlewares da produção.
Rate limit desligado na suíte (`RATE_LIMIT_DISABLED` em `test/env-setup.ts`),
exceto em `rate-limit.e2e-spec.ts`.

**Regra de repasse**: `repasse = grossValue * (repassePercentage / 100)`,
incide sobre o valor bruto, nunca sobre bruto menos material.

**Regra de aluguel**: não é uma linha por atendimento — o `ReportsService`
calcula por `(clinicId, date)` distinto: se o consultório é `RENTED` e teve
pelo menos um atendimento naquele dia, soma `dailyRentValue` uma vez (não por
atendimento, mesmo que hajam vários no mesmo dia/consultório).

**Estoque**: baixa via `MaterialUsage` decrementa `MaterialBatch.quantity`
seguindo FEFO (usa primeiro o lote que vence antes) — ver
`AttendancesService.deductStock()`.

### Autenticação e papéis (`Role`: `ADMIN` / `DENTIST` / `RECEPTIONIST`)

JWT access (15min, em memória no front) + refresh token **opaco e revogável**
(30d, cookie httpOnly `refresh_token`). **Sem endpoint público de registro** —
o primeiro usuário/organização nascem do `prisma/seed.ts`; a partir daí só o
admin cria novos usuários, pelo painel.

Payload do JWT de acesso: `{ sub, email, organizationId, roles, isSuperAdmin }`
— `organizationId` null e `roles` vazio só pro Super Admin. Login: ver seção
Multi-tenancy acima (S1). `jwt.strategy.ts` relê o `User` do banco a cada
request, então papéis, ativo e organização valem sempre o estado atual (um
access token antigo com `role` no payload continua funcionando).

**Sessões de refresh revogáveis com rotação** (2026-09-26, S3, PR #17 —
`modules/sessions/refresh-sessions.service.ts`). Antes o refresh era um JWT de
30 dias sem estado: o logout só apagava o cookie e um token vazado valia até
expirar. Agora:
- cada `POST /auth/refresh` revoga a sessão atual e cria a próxima na mesma
  família (`replacedById`); **reusar um token já trocado revoga a família
  inteira** (sinal de roubo) e responde 401;
- a reivindicação é um `UPDATE … WHERE revokedAt IS NULL` fora de transação
  interativa — ver [[Problemas Conhecidos]] (P2028 do Prisma com duas
  transações interativas disputando a mesma linha);
- `POST /auth/logout` revoga a família do navegador atual; `POST
  /auth/logout-all` revoga todas as sessões do usuário; desativar um usuário
  também revoga;
- refresh recusado (401) apaga o cookie; erro de outro tipo não;
- `JWT_REFRESH_SECRET` deixou de existir.

**Rate limit e headers** (2026-09-26, S2, PR #16): `@nestjs/throttler` como
**primeiro** guard global, por IP do cliente — login 5/min, refresh 30/min
(folga porque o front faz refresh a cada carregamento de página e a clínica
sai por um IP só), demais rotas 100/min, `/health` sem limite. Valores em
`common/rate-limit/rate-limit.config.ts`. `trust proxy` com número de saltos
(`TRUST_PROXY_HOPS`, padrão 1 = Render). `helmet` com CSP `default-src 'none'`
(API só JSON). Tudo montado em `src/app.setup.ts` (`configureApp()`), usado
pelo `main.ts` e pelos testes.

`SuperAdminGuard` (`common/guards/super-admin.guard.ts`) checa `isSuperAdmin`
— usado via `@UseGuards` só no `PlatformController`, não é guard global (ao
contrário do `RolesGuard`/`JwtAuthGuard`).

Cookie de refresh usa `sameSite: "none", secure: true` em produção (porque
Vercel e Render são domínios diferentes — um cookie `Lax` não seria enviado
num fetch cross-site) e `sameSite: "lax", secure: false` em dev local sobre
http. Essa é uma correção deliberada em relação ao padrão do hubassistent, que
usa só `Lax` — lá provavelmente o refresh cross-origin nunca chegou a ser
testado de verdade em produção.

`RolesGuard` (`common/guards/roles.guard.ts` + decorator `@Roles(...)`) — não
existia no hubassistent. Registrado como `APP_GUARD` global, depois do
`ThrottlerGuard` e do `JwtAuthGuard`. Desde 2026-09-26:
- **nega por padrão** (S4, PR #14): rota sem `@Public`, `@Roles` ou
  `@AllowAuthenticated` (qualquer logado) responde 403; a política do handler
  vence a da classe. O teste `route-policy-coverage` percorre todos os
  handlers e falha se algum não tiver política — e trava a lista de rotas
  públicas (`POST auth/login`, `POST auth/refresh`, `GET health`);
- **matriz central `ACCESS`** (PR #18, `packages/shared-types/src/access.ts`):
  capacidade → papéis (`patients.read`, `clinical.write`, `users.manage`…).
  A API usa `@Roles(...ACCESS["capacidade"])` e o front `can(user,
  "capacidade")` — **mudar quem acessa algo = mudar só o `ACCESS`**. Regras
  por tipo de organização (termos financeiros e criação de consultório só pra
  `FREELANCER`, eixo da Agenda) ficam fora da matriz, nos services;
- **vários papéis** (R1, PR #19): libera se **qualquer** papel do usuário
  estiver na lista.

Tabela de quem acessa cada módulo — reflete o `ACCESS` em 2026-09-26 (Super
Admin não aparece — não bate em nenhum `@Roles(...)`, só acessa `/platform`
via `SuperAdminGuard`):

| Módulo | ADMIN | DENTIST | RECEPTIONIST |
|---|---|---|---|
| `/users` (painel admin) | ✅ | ❌ | ❌ |
| `/organization` (Minha Clínica) | ✅ | ✅ | ✅ |
| `/patients` — leitura (2026-09-22) | ✅ | ✅ | ✅ |
| `/patients` — escrita | ❌ | ✅ | ✅ |
| `/patients/:id/anamnesis` | ❌ | ✅ | ❌ |
| `/patients/:id/clinical-records` | ❌ | ✅ | ❌ |
| `/patients/:id/tooth-records` (odontograma) | ❌ | ✅ | ❌ |
| `/budgets` | ❌ | ✅ | ✅ |
| `/appointments` (agenda) — ⚠️ ver nota abaixo | ❌ | ✅ | ✅ |
| `/materials` (estoque) | ❌ | ✅ | ✅ |
| `/recalls` | ❌ | ✅ | ✅ |
| `/clinics` — leitura (2026-09-22) | ✅ | ✅ | ✅ |
| `/clinics` — escrita | ❌ | ✅ | ❌ |
| `/procedures` — leitura (2026-09-22) | ✅ | ✅ | ✅ |
| `/procedures` — escrita | ❌ | ✅ | ❌ |
| `/attendances`, `/reports/financial` | ❌ | ✅ | ❌ |
| `/platform/*` | só Super Admin (nenhum papel de tenant acessa) |

**Leitura liberada pro ADMIN em `/clinics`, `/patients`, `/procedures`
(2026-09-22)**: até então o `ADMIN` não acessava nenhum dado clínico — mas a
Agenda redesenhada (ver [[Roadmap]] e a nota de mock-first abaixo) tem um
modo de visão multi-dentista pro admin/recepcionista, e o modal de novo
agendamento precisa buscar paciente/procedimento reais mesmo estando numa
tela mockada. Ajustado só o suficiente: `@Roles` do controller ganhou
`ADMIN` nos três (escrita continua `DENTIST`/`RECEPTIONIST`, sem mudança).
**`/appointments` continua `DENTIST`/`RECEPTIONIST` apenas** — a Agenda do
ADMIN não bate nesse endpoint hoje porque é 100% mockada; se um dia os
agendamentos virarem reais, esse `@Roles` também precisa ser revisado.

**Gestão de usuários com a lista de papéis** (`users.service.ts`, R1), nesta
ordem: (1) a organização **mantém pelo menos um ADMIN ativo**; (2) o admin não
se desativa nem remove o próprio ADMIN; (3) **capitania como antes** — só o
fundador edita/desativa outro admin (a tarefa do R1 falava em "admin não edita
admin"; o Vitor decidiu manter a capitania); (4) todo dentista tem cor
(atribuída por índice na criação e ao ganhar DENTIST). Estatísticas da
plataforma: usuário com 2 papéis conta nos dois; `usersByRole.total` conta
pessoas.

Um `DecimalInterceptor` global (`common/interceptors/decimal.interceptor.ts`)
converte todo `Prisma.Decimal` pra `number` antes de serializar — ver
[[Problemas Conhecidos]] pro bug real que isso corrigiu.

### Módulos

`auth`, `sessions` (sessões de refresh, usado por `auth` e `users`), `users`
(painel do Tenant Admin, só ADMIN), `organization`
(`GET /organization`, qualquer papel autenticado — nome da clínica + lista de
membros, ver [[Funcionalidades e Endpoints]]), `platform` (só Super Admin —
criar/listar organizações, transferir capitania), `patients`, `anamnesis`,
`clinical-records`, `odontogram`, `clinics`, `procedures`, `budgets`,
`appointments`, `attendances`, `reports`, `materials`, `recalls`, `health`.

## Frontend (React + Vite)

Vite (não Next — API já é separada). TanStack Query + React Hook Form + Zod,
Tailwind sem kit de componentes (tudo hand-rolled, suficiente pro volume de
telas do MVP). Redesenho visual completo em 2026-09-21/22 (ver [[Roadmap]]
pra timeline) migrou os tokens do Tailwind pro design system oficial — ver
[[design-system-plataforma-odontologica]].

```
apps/web/src/
├── components/{app-shell.tsx, sidebar.tsx, ui/*}
├── features/{auth,admin,agenda,clinics,dashboard,financeiro,materials,my-clinic,patients,platform,procedures}/
├── lib/{access.ts, api-client.ts, auth-context.tsx, palette-colors.ts, initials.ts, use-click-outside.ts}
└── routes/{protected-route.tsx, home-route.tsx}
```
(Orçamento, anamnese, prontuário e odontograma são abas em
`features/patients/tabs/`.)

**Acesso no front** (PR #18): toda decisão "este usuário pode X?" passa por
`can(user, capacidade)` (`lib/access.ts`, lê `user.roles`), com a mesma matriz
`ACCESS` da API — nenhuma comparação direta com papel. `ProtectedRoute` aceita
`capability?` e/ou `superAdminOnly?`. `HomeRoute`: Super Admin → `/platform`;
quem não tem `dashboard.view` mas tem `users.manage` (ADMIN puro) →
`/admin/users`; senão `DashboardPage` (um `[ADMIN, DENTIST]` cai no
dashboard). Rótulo do papel na sidebar junta a lista ("Admin · Dentista").

**Tela de usuários** (R1 + PR #21): checkboxes de papéis (criação e tabela);
na própria linha ADMIN fica travado; o último papel não desmarca; dar ou tirar
ADMIN de outra pessoa pede confirmação num diálogo da app (portal no
`<body>`) antes do PATCH — PR #21, na `main` desde 2026-09-26.

**Navegação (2026-09-21)**: o antigo menu horizontal (`layout.tsx`) foi
substituído por `app-shell.tsx` + `sidebar.tsx` — sidebar fixa à esquerda
(240px, colapsável, tooltip no modo colapsado, estado em `localStorage`),
grupos de nav (Menu/Clínica/Plataforma), filtrados por `capability`
(`can()`) ou `superAdminOnly`. `app-shell.tsx` trava em `h-screen overflow-hidden`
e só o `<main>` (`min-h-0 overflow-y-auto`) rola — ver bug real corrigido em
[[Problemas Conhecidos]] (a primeira versão usava `min-h-screen` sem
`min-h-0`, e a página inteira rolava junto com a sidebar). Super Admin só vê
"Plataforma" no nav, nada de tenant.

`api-client.ts`: access token em memória, refresh automático via
`credentials: "include"` quando a API responde 401 (exceto no próprio
`/auth/login` e `/auth/refresh`). Como o refresh token é rotacionado e reuso
derruba a sessão, **só existe um refresh em andamento por vez**
(`refreshAccessToken()`): na mesma aba, chamadas paralelas esperam a mesma
promessa; entre abas, a **Web Locks API** serializa — a segunda aba só manda
o refresh depois da primeira, já com o cookie novo. O `AuthProvider` usa o
mesmo caminho ao abrir o app (o StrictMode chamava o refresh duas vezes).

### Padrão "mock primeiro, backend depois" (2026-09-21/22)

Home Dashboard, Agenda (redesenho estilo Google Calendar) e Procedimentos
(catálogo) foram reconstruídos visualmente **antes** de qualquer mudança de
schema/backend — decisão consciente pra validar UI/UX rápido sem acoplar a
migrações de banco. Padrão replicado nas três:

- Hook `useMockXxx()` = `useQuery` com uma `Promise`/`setTimeout` resolvendo
  dado semeado (exercita skeleton de loading de propósito), sem chamar a API.
- A página semeia `useState` local a partir do resultado da query (`useEffect`
  uma vez carregado); toda criação/edição/soft-delete depois disso mexe só
  nesse estado local — nenhuma mutação real vai pro backend.
- Onde já existe endpoint real e estável, ele é reaproveitado de verdade
  (não mockado): Clinics (`clinicsApi.list()`, vira "Consultórios"/Locations
  na Agenda), Patients e Procedures (pickers de paciente/procedimento no
  modal de novo agendamento).
- **Atualização 2026-09-22 (PR #12)**: o roster da sidebar da Agenda (quem
  aparece pra colorir/filtrar — consultórios ou dentistas) **deixou de ser
  mockado**. `useAgendaMockData` foi substituído por `useAgendaData`, com
  `resolveCalendarAxis()` decidindo o eixo real a partir de
  `organization.type` + capacidade: `"location"` (freelancer → `Clinic`
  reais), `"dentist"` (clínica, quem tem `organization.dentists` —
  admin/recepcionista → `User` reais com `roles has DENTIST` via `GET
  organization/dentists`, `403` de verdade pra `DENTIST`, não mais filtro de
  UI), `"none"` (clínica, sem essa capacidade — dentista, sem sidebar). `dentist-store.ts`
  (mock/`localStorage`) foi removido. **O que continua mockado**: os
  agendamentos/eventos do calendário em si (conteúdo, não o roster).
- Cor arbitrária (sem significado de status, diferente do `Badge` semântico)
  usa sempre os mesmos 5 tokens oficiais (`brand/success/warning/error/info`)
  ciclados por índice — `apps/web/src/features/agenda/location-colors.ts`
  (`locationColorForIndex`). Desde PR #12 é persistida de verdade
  (`Clinic.colorToken`/`User.colorToken`, ver "Modelo de dados" acima) via
  `PATCH /clinics/:id`/`PATCH /users/:id` — não mais `dentist-store.ts`/
  `localStorage`.
- Agenda usa FullCalendar (`@fullcalendar/{core,react,daygrid,timegrid,interaction}`,
  todos pinados na mesma versão `6.1.21` — v7 só tem `core`/`react` estáveis,
  os plugins ainda são RC). `slotEventOverlap={false}` faz agendamentos
  sobrepostos (dentistas diferentes, mesmo horário/consultório) renderizarem
  lado a lado em colunas em vez de empilhados — não precisou de plugin de
  resource view.

Consequência prática: Home Dashboard e Procedimentos continuam 100% mockados.
Na Agenda, Locations/Patients/Procedures e, desde 2026-09-22 (PR #12), o
roster da sidebar (consultórios ou dentistas, com cor) já são dado real —
só os agendamentos/eventos do calendário em si continuam mockados. Ver
[[Roadmap]] pro que falta pra virar backend de verdade (`Location`/
`Calendar` da agenda multi-consultório continuam só planejados, ver
[[Implementação da Agenda Multi-Consultório — Plano Técnico]]).

## Convenções replicadas do hubassistent (não redescobertas, copiadas de propósito)
- `ZodValidationPipe` sempre em parâmetro específico (`@Body(new ZodValidationPipe(schema))`),
  nunca `@UsePipes` de método — evita o bug já visto lá (pipe aplicado a todos
  os parâmetros do handler).
- `"postinstall": "prisma generate"` no `package.json` da API — sem isso o
  client gerado fica vazio num ambiente limpo tipo Render.
- Schema em `apps/api/prisma/schema.prisma`, `directUrl` separado de
  `url` pensando no pooler do Supabase (ver [[Infraestrutura e Deploy]]).
