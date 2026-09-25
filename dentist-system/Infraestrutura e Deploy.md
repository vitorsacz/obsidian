#projeto #dentist-system #infra

Ver [[Visão Geral]] para contexto, [[Arquitetura]] para o código por trás disso.

## Atualização (2026-09-18): produção resetada e migrada pra multi-tenancy

Produção (Supabase) rodou `prisma migrate reset --force` — apagou o schema
`public` inteiro e recriou do zero com as 7 migrations atuais (incluindo
multi-tenancy + identidade isolada por tenant, [[Arquitetura]]) + seed novo.
Decisão possível porque, na data, produção só tinha dado de teste — nenhuma
conta real de paciente/dentista em uso de verdade (confirmado com o Vitor;
qualquer referência anterior nesta nota a uma "conta real da dentista"
criada em 2026-08-03 está desatualizada, não existe mais).

**Gotcha real descoberto no processo**: `source .env.local` no bash faz
**expansão de variável** em cima de `$` dentro de valores com aspas duplas —
`SEED_SUPER_ADMIN_PASSWORD="V$acz26092407"` virava só `"V"` ao ser
sourceado (bash tentava expandir `$acz26092407` como variável), e
`"Admin$26092407"` virava `"Admin6092407"` (`$2` interpretado como parâmetro
posicional). O primeiro reset rodou com essas senhas corrompidas — os
usuários foram criados com hash de senha errado, só percebido ao testar
login pelo navegador (que usa a senha real, digitada) enquanto os testes via
`curl` "passavam" porque usavam a mesma variável corrompida pra login E pro
seed, criando falsa confiança. **Fix**: usar aspas simples (`'...'`) no
`.env.local` pra valores com `$` — aspas simples não sofrem expansão do bash
no `source`, e o Prisma/dotenv trata aspas simples e duplas igual. Refeito o
reset com aspas simples, confirmado login funcionando via `curl` e navegador
real (Super Admin, Admin, Dentista, todos 3).

Credenciais de produção atuais (`vitorsuper@dentist.com`,
`admin@dentist.com`, `dentista@dentist.com`) ficam em `apps/api/.env.local`
— **não** são mais `admin@example.com`/`dentista@example.com` como a seção
"Credenciais de teste" abaixo ainda documenta (desatualizada, mantida por
enquanto só como histórico do formato/fluxo, ver seção nova de credenciais
via `.env.local`).

## Estado atual (2026-08-03): deploy real no ar, validado ponta a ponta

| Componente | Onde roda | URL |
|---|---|---|
| Frontend (`apps/web`) | Vercel | `https://dentist-system-web-ruddy.vercel.app` |
| API (`apps/api`) | Render | `https://dentist-system-api.onrender.com` |
| Postgres | Supabase (projeto `dentist-system`, região `sa-east-1`/São Paulo, ref `fjkzcqcimtlrddhvmkgr`) | — |

Testado de verdade no navegador: login (`dentista@example.com`) funciona, sessão
sobrevive a reload (cookie de refresh `SameSite=None; Secure` cross-domain
Vercel↔Render OK), CORS liberado só pra origem da Vercel, health check estável.

**Render free tier "dorme" após inatividade** — primeira requisição depois de um
tempo parado pode levar até ~50s pra responder (cold start). Normal, não é bug.

### Bug de deploy encontrado e corrigido: SPA 404 na Vercel
Acessar uma rota direto (`/login`, refresh numa página interna) dava 404 da
própria Vercel — ela procurava um arquivo real nesse path em vez de deixar o
React Router tratar no client. Corrigido com rewrite catch-all em
`apps/web/vercel.json`:
```json
"rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
```

### Decisões tomadas na criação do Supabase
- Região **South America (São Paulo)** — latência baixa pro uso real da dentista.
- **Data API desligada** (Enable Data API / Automatically expose new tables /
  Enable automatic RLS todos desmarcados) — o projeto não usa PostgREST/
  supabase-js, só Postgres puro via Prisma; deixar ligado seria expor dado de
  saúde por um caminho HTTP que ninguém do sistema controla. RLS do Supabase não
  se aplica ao nosso caminho de acesso de qualquer forma — ver
  [[Roadmap]] (seção "Evolução SaaS") sobre por que o isolamento futuro (multi-tenant)
  será via Prisma Client Extension, não RLS.
- Integração GitHub conectada, mas **não usamos o fluxo de schema-as-code do
  Supabase** — schema é 100% gerenciado pelo Prisma (`prisma migrate deploy`,
  já embutido no `buildCommand` do `render.yaml`).

### Separação dev/prod
`apps/api/.env` local aponta pro Postgres do Homebrew (`dentist_system`) —
nunca aponta direto pro Supabase no dia a dia, pra nenhum teste/reset local
arriscar tocar em dado real da dentista depois que ela começar a usar de
verdade. Credenciais do Supabase (prod) ficam em `apps/api/.env.local`
(gitignored, só referência local) e nas env vars do Render.

### Conta real da dentista (2026-08-03) — histórico, não existe mais
Criada via painel admin (`POST /users`, não pelo seed) direto em produção:
`yasmimcruz_almeida@hotmail.com`, papel `DENTIST`, testada com login real
(retornou access token). **Apagada no reset de 2026-09-18** (ver seção acima)
— confirmado com o Vitor que não tinha mais valor real na data do reset.
Decisão de produto que segue de pé independente disso: não construir tela de
cadastro público — o sistema continua sem registro público de propósito
(dado de saúde), quem cria conta é o Admin/Super Admin pelo painel. Ver
discussão em [[Roadmap]] (seção "Evolução SaaS") sobre por que abrir cadastro
público exigiria rate-limit/confirmação de e-mail antes de considerar.

## Infra como código (repo)
- **`render.yaml`** (raiz) — Blueprint do serviço da API. `buildCommand` roda
  `prisma migrate deploy` a cada deploy. Health check em `/health`. Secrets
  (`DATABASE_URL`, `DIRECT_URL`, `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`,
  `CORS_ORIGIN`, `SEED_*`) preenchidos manualmente no dashboard do Render,
  nunca ficam no git.
- **`apps/web/vercel.json`** — build/install/output pro monorepo pnpm+Turborepo
  + rewrite de SPA (ver bug acima). Root Directory = `apps/web` no projeto Vercel.
- **`.github/workflows/ci.yml`** — lint + typecheck + build em todo push/PR pra
  `main`, antes de qualquer deploy.

## Credenciais de teste

**Local/dev** (`apps/api/.env`): `admin@example.com`/`dentista@example.com`,
senha em `SEED_ADMIN_PASSWORD`/`SEED_DENTIST_PASSWORD` do `.env`.

**Produção** (desde o reset de 2026-09-18): `vitorsuper@dentist.com` (Super
Admin), `admin@dentist.com` (Admin), `dentista@dentist.com` (Dentist) — todas
as senhas ficam em `apps/api/.env.local` (gitignored), nunca aqui no vault.

Rodar seed de novo: `pnpm db:seed` local, ou `prisma migrate reset --force`
com as env vars de produção pra recriar do zero (ver seção do gotcha do `$`
acima antes de rodar via `source`).

## Variáveis de ambiente — referência
`apps/api/.env`:
- `DATABASE_URL` — transaction pooler (porta 6543, `?pgbouncer=true`).
  `DIRECT_URL` — session pooler (porta 5432). Conexão direta do Supabase é
  IPv6-only e falha do Render/máquina local, por isso os poolers.
- `JWT_ACCESS_SECRET`/`JWT_REFRESH_SECRET` — gerados com
  `crypto.randomBytes(32).toString("hex")`; os de produção são diferentes dos
  de dev (nunca reusar).
- `CORS_ORIGIN` — dev: `http://localhost:5173`; prod: URL da Vercel.
- `SEED_ADMIN_*`/`SEED_DENTIST_*` — só usados pelo `seed.ts`; se email/senha
  não definidos, o seed pula aquele usuário sem quebrar.

`apps/web/.env` / env var na Vercel:
- `VITE_API_URL` — fica embutido no build, precisa redeploy da Vercel ao trocar.

## Próximos passos (ver [[Roadmap]])
- Criar de novo a conta real da dentista pelo painel (Admin cria em
  `/admin/users`, ou Super Admin cria a organização dela em `/platform` se
  for virar uma clínica separada) antes de repassar o acesso pra ela.
- Depois da validação dela: ver [[Roadmap]] (seção "Evolução SaaS") pra clínica 2.
