#projeto #dentist-system #saas

Ver [[Roadmap]] (seção "Evolução SaaS") pra versão destilada (decisão tomada, sequência
acordada). Este arquivo é a conversa/análise completa que originou aquela nota —
movido do repo (`PLANO_IMPLEMENTACAO.md`) pro vault em 2026-08-03, já que
documentação de processo/planejamento fica só aqui, não no GitHub.

---

# Plano de Implementação — dentist-system → SaaS Multi-tenant

> Este documento reúne: (A) a conversa original de arquitetura com o Gemini, (B) a análise crítica feita em cima dela, (C) a validação do projeto real `dentist-system` com as divergências encontradas, e (D) o plano de implementação em 4 fases. Serve como ponto de continuidade para seguir a implementação via Claude Code a partir de onde paramos.

---

# PARTE A — Conversa original com o Gemini (arquitetura teórica da Clinicorp)

**Usuário:** voce consegue me dizer, como funciona o software da Clinicorp?

**Gemini:** O Clinicorp é um dos softwares de gestão (ERP e CRM) em nuvem mais utilizados no Brasil, desenvolvido especificamente para atender às necessidades de clínicas odontológicas e de estética. Opera no modelo SaaS e centraliza, digitaliza e automatiza toda a operação de um consultório ou rede de clínicas.

Estrutura por módulos integrados — uma ação alimenta o ecossistema inteiro (ex: orçamento aprovado atualiza financeiro, agenda e estoque):

| Módulo | Principais Funcionalidades | Impacto na Operação |
|---|---|---|
| Recepção e Agenda | Agendamento online, fila de espera, confirmações via WhatsApp | Reduz faltas e ociosidade |
| Gestão Clínica | Prontuário eletrônico, odontograma, planos de tratamento, anamnese via WhatsApp | Elimina papelada, assinatura eletrônica com validade jurídica |
| Gestão Financeira | Contas a pagar/receber, NFs, boletos, SPC/Serasa, maquininha própria (Clinipay) | Automatiza conciliação e repasses |
| Comercial (CRM) | Funil de vendas, orçamentos em aberto, réguas de cobrança | Aumenta taxa de fechamento |
| Inteligência e Gestão | Dashboards, IA integrada, metas, estoque, comissões | Visão baseada em dados |

Acesso: 100% web, App do Doutor e App do Paciente, controle de permissões hierárquico.

---

**Usuário:** baseado no que a clinicorp faz, voce como um arquiteto de sistemas senior, como voce pensaria e estruturaria a criação de uma aplicação dessa?

**Gemini:**

**1. Design Estratégico e Bounded Contexts (DDD):** Contexto Clínico (core domain), Agenda/Recepção, Financeiro, CRM/Comercial, Identidade e Acesso (auth, RBAC, multi-tenancy).

**2. Estilo Arquitetural:** Microsserviços por contexto + Arquitetura Hexagonal (Ports and Adapters)/Clean Architecture dentro de cada serviço — domínio isolado no centro, banco/UI/integrações externas como adaptadores.

**3. Infraestrutura:** Docker + Kubernetes para auto-scaling e resiliência.

**4. Multi-tenancy:** banco isolado por tenant (ou schemas separados) para dados clínicos sensíveis; pool com `tenant_id` para serviços menos críticos.

**5. SRE e Observabilidade:** logs centralizados, métricas, tracing distribuído.

---

**Usuário:** agora tendo em vista ser um programador que irá montar isso com vibe coding, quais etapas voce sugere que sejam seguidas e as tecnologias que voce sugeriria. Antes de fazer qualquer tipo de deploy produtivo em uma clinica, podemos rodar toda a aplicação free, sem custo.

**Gemini:**

**Stack 100% gratuito no MVP:**
- IDE/IA: Cursor (Claude 3.5 Sonnet ou GPT-4o)
- Framework: Next.js (App Router) + TypeScript, monolito modular, hospedagem grátis na Vercel
- UI: Tailwind CSS + shadcn/ui
- BaaS: Supabase (Postgres + Auth + Storage, plano grátis generoso)
- ORM: Drizzle ORM ou Prisma

**Plano de ataque:**
1. Setup: Next.js + shadcn + Supabase Auth, tabelas `clinics`/`users`, RLS (Row Level Security) no Postgres do Supabase para isolar dados entre clínicas.
2. `/modules/scheduling`: CRUD de pacientes + calendário (react-big-calendar/FullCalendar).
3. `/modules/clinical`: prontuário, DDD/Hexagonal leve, regras de negócio isoladas das rotas Next.js.
4. `/modules/finance`: orçamentos → contas a receber, export CSV.
5. Sentry (grátis) para erros; estrutura modular permite extrair para microsserviços/Docker/K8s no futuro se tracionar.

---

**Usuário:** [15 perguntas críticas sobre Cursor vs VS Code/Claude Code, Supabase e vendor lock-in, banco de dados e ORM, deploy full-stack vs separado, Docker/K8s, infraestrutura futura, painel administrativo, multi-tenancy, RBAC, agenda, documentos/backup/LGPD, módulo financeiro, observabilidade/auditoria, revisão crítica geral como CTO, e decisões mais difíceis de reverter — íntegra disponível no arquivo `conversa_arquitetura_clinicorp.md` original]

**Gemini (resumo das respostas, atuando como CTO):**

1. **Cursor vs VS Code:** manter Cursor no MVP — indexação de codebase multi-arquivo ainda superior para vibe coding; VS Code + Claude Code Pro também atende.
2. **Supabase/vendor lock-in:** manter Supabase; mitigar lock-in isolando regras de negócio do SDK via interfaces (Arquitetura Hexagonal) — `IAuthRepository`/`IStorageService`.
3. **Banco/ORM:** Postgres (Supabase) + Drizzle (mais leve que Prisma em serverless/edge, sem engine oculta).
4. **Deploy:** Monolito Modular em Next.js, não separar front/back no dia 1 — separar adiciona latência, CORS, dois pipelines de CI/CD para um dev solo.
5. **Docker/K8s:** abandonar K8s no MVP; deploy serverless na Vercel.
6. **Infra futura:** Vercel/Supabase → Render/Fly.io → AWS/GCP, só migrar quando a conta de engenharia doer mais que a financeira.
7. **Painel admin:** aplicação separada — ciclo de vida e criticidade diferentes do sistema da clínica.
8. **Multi-tenancy:** subdomínios (não white-label no MVP); `tenant_id` em todo registro + RLS.
9. **RBAC:** RBAC estrito (ABAC é over-engineering agora); tabelas `roles`/`permissions`/`role_permissions`/`user_roles`.
10. **Agenda:** react-big-calendar/FullCalendar; recorrência via RRULE (RFC 5545), lock otimista, tudo em UTC no banco.
11. **Documentos/LGPD:** backup é responsabilidade da plataforma; Signed URLs (15 min); soft delete + anonimização em vez de hard delete.
12. **Financeiro:** MVP válido mas gera atrito se não fizer cobrança automática; sugeriu importação de OFX para conciliação.
13. **Observabilidade/auditoria:** Sentry para erros; tabela `audit_logs` append-only (trigger Postgres ou interceptador no ORM) para ações sensíveis (quem acessou prontuário, alterou orçamento, etc.).
14. **Veredito do CTO:** cortar microsserviços/K8s (síndrome de arquiteto); manter DDD/Bounded Contexts lógicos, Clean Architecture, Supabase+Next.js+Vercel; risco = construir a "agenda perfeita" e esquecer que a clínica só paga se CRM/Financeiro resolverem a dor dela.
15. **One-way doors:** (1) estrutura de multi-tenancy — o mais crítico, vazamento entre tenants mata a empresa por LGPD; (2) identidade/Auth — trocar depois de milhares de sessões ativas é caótico; (3) timezone/moeda — salvar UTC absoluto e dinheiro como inteiro em centavos desde o início.

---

# PARTE B — Análise crítica feita sobre a proposta do Gemini

Pontos fortes confirmados: transição de "arquitetura ideal" para "pragmática para dev solo" correta; monolito modular, adiar K8s, RLS+tenant_id, RBAC simples, UTC, dinheiro em centavos — tudo consistente com boas práticas.

Gaps e ressalvas identificados:

1. **Drizzle + RLS — furo técnico real.** Se a conexão do Drizzle for direta na connection string do Postgres (motivo de usar Drizzle em vez do client do Supabase), ela normalmente usa uma role de serviço que **bypassa RLS por padrão**. Sem configurar `SET LOCAL` com os claims do JWT por request, o isolamento entre clínicas vira ilusório num sistema multi-tenant de saúde. (No projeto real, esse ponto nem se aplica da forma como o Gemini desenhou — ver Parte C, item 6.)
2. **Cobrança das assinaturas SaaS não foi endereçada.** O pedido original incluía "gerenciar assinaturas" no painel admin, mas a resposta só cobriu o financeiro da clínica cobrando o paciente, nunca a plataforma cobrando a clínica.
3. **Jobs assíncronos/background processing ausentes.** Lembretes WhatsApp, relatórios, importação OFX, backups não deveriam rodar em function serverless da Vercel (timeout curto). Faltou recomendar algo como Inngest/Trigger.dev ou cron via Supabase Edge Functions.
4. **Cursor vs Claude Code** — opinião apresentada com mais força do que o caso permite; hoje o Claude Code também indexa codebase inteiro com contexto multi-arquivo comparável.
5. Pontos bem resolvidos sem ressalva: RRULE, lock otimista, Signed URLs + soft delete/anonimização (LGPD), `audit_logs` append-only, e a lista de "one-way doors" (multi-tenancy, auth, timezone/moeda).

---

# PARTE C — Validação do projeto real `dentist-system` e divergências

O projeto já existe e está funcional — não foi construído do zero seguindo a receita do Gemini. Principais divergências entre o que foi discutido e o que está implementado:

| Tema | Gemini recomendou | Projeto real usa | Situação |
|---|---|---|---|
| Arquitetura da app | Monolito modular em Next.js (front+back juntos) | Monorepo pnpm/Turborepo com `apps/api` (NestJS) e `apps/web` (Vite+React) **separados** — exatamente o que o Gemini desaconselhou para dev solo | Já construído e funcionando; decisão é evoluir nesse formato, não migrar |
| Backend/DB | Supabase (Auth+DB+Storage+RLS prontos) | Postgres direto + Prisma + JWT próprio (bcrypt, access+refresh token via cookie httpOnly). `.env` usa padrão pooler (porta 6543) + direct (5432), típico de Supabase, mas só como Postgres puro — Auth/Storage do Supabase não estão em uso | RLS do Supabase não se aplica; isolamento multi-tenant precisa ser via aplicação (Prisma Client Extension) — ver Parte D, Fase 1.3 |
| ORM | Drizzle (preferência do Gemini) | Prisma | Mantido Prisma — já maduro no projeto, migrations e seed já escritos em cima dele |
| Multi-tenancy | `tenant_id` em toda tabela + RLS desde o dia 1 (**"one-way door" #1**) | **Inexistente.** `Clinic` representa unidade física de uma única operação, não um tenant/cliente. Dados compartilhados entre todos os usuários | Confirmado com o usuário: intenção é SaaS multi-tenant → retrofit é a Fase 1 do plano, prioridade máxima |
| RBAC | Tabelas `roles`/`permissions`/`role_permissions`/`user_roles` (suporta cargos customizados por tenant) | Enum fixo `Role` (`ADMIN`/`DENTIST`/`RECEPTIONIST`) hardcoded, guards globais (`JwtAuthGuard`+`RolesGuard`) | Mais simples que o desenhado; não suporta cargos personalizados ainda (o usuário pediu isso nas perguntas originais) — avaliar se entra em fase futura |
| Dinheiro | Inteiro em centavos | `Decimal` do Postgres via Prisma (`@db.Decimal`) | Não é o mesmo approach, mas é tecnicamente seguro (Decimal é exato, diferente de float) — não precisa mudar |
| Deploy | Vercel (tudo) | Vercel (front) + Render (back), já configurado (comentário sobre `SameSite=None` cross-domain no código) | Consistente com a separação front/back já adotada |
| Painel admin separado | App separado | Ainda não existe — só existe `admin-users-page.tsx` dentro do `apps/web` (gerencia usuários do próprio tenant, não plataforma) | Vira Fase 2 do plano — novo app `apps/admin` |
| Geração de documentos | Não fazia parte do escopo original | Módulo `reports` existe mas só gera dados para tela (`financial-report.tsx`), não exporta PDF/Word/Excel | Pedido novo do usuário (via skills do Cowork) → Fase 3 do plano |
| Billing das assinaturas (cobrar as clínicas) | Não endereçado (gap apontado na Parte B) | Não existe | Fase 4 do plano |

Módulos já implementados no backend (cobrem quase tudo da Clinicorp, exceto CRM/funil de vendas e emissão de NF/cobrança automatizada): auth, users, patients, anamnesis, clinical-records, clinics, procedures, budgets, appointments, attendances, reports, materials, odontogram, recalls.

---

# PARTE D — Plano de implementação (4 fases)

## FASE 1 — Multi-tenancy (bloqueante, fazer primeiro)

### 1.1 Schema Prisma

Criar novo model `Tenant` (a clínica-cliente que paga o SaaS):

```prisma
enum TenantPlan {
  TRIAL
  BASIC
  PRO
}

model Tenant {
  id        String     @id @default(cuid())
  name      String
  slug      String     @unique // usado no subdomínio: {slug}.suaempresa.com.br
  plan      TenantPlan @default(TRIAL)
  active    Boolean    @default(true)
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt

  users           User[]
  clinicUnits     ClinicUnit[]
  patients        Patient[]
  // ... demais relações inversas conforme for adicionando tenantId abaixo
}
```

Renomear o model `Clinic` atual para `ClinicUnit` (representa unidade física, não tenant — o nome `Clinic` fica reservado para o conceito de tenant agora). Atualizar todas as referências (o campo `clinicId` pode manter o nome, só o model muda).

**Adicionar `tenantId String` + relação `tenant Tenant @relation(fields: [tenantId], references: [id])` + `@@index([tenantId])` em TODOS os models, sem exceção**: `User`, `ClinicUnit`, `Patient`, `Anamnesis`, `ClinicalRecord`, `ToothRecord`, `ProcedureCatalog`, `Budget`, `BudgetItem`, `Appointment`, `Attendance`, `Material`, `MaterialBatch`, `MaterialUsage`, `Recall`.

Por quê em todos, mesmo os que já têm um pai tenant-scoped (ex: `BudgetItem` já pendura em `Budget`)? Porque a estratégia de isolamento da Fase 1.3 (Prisma Client Extension) precisa que **todo model tenant-scoped tenha o campo diretamente** para aplicar o filtro de forma uniforme, sem casos especiais por join. É mais dado gravado, mas é a diferença entre uma regra simples e auditável e uma cheia de exceções — exceções são exatamente onde vazamento de dado acontece.

Gerar migration (`pnpm db:migrate`). Como já existem dados de teste, o script de migração precisa de um passo de backfill: criar um `Tenant` "default" e popular `tenantId` em todas as linhas existentes antes de tornar a coluna `NOT NULL` (fazer em duas migrations: 1) coluna nullable + backfill via SQL, 2) `ALTER COLUMN ... SET NOT NULL`).

### 1.2 Contexto de tenant por request

Criar `apps/api/src/tenant/tenant-context.ts` com um wrapper simples sobre `AsyncLocalStorage` (não precisa de dependência nova como `nestjs-cls` — o projeto é pequeno o suficiente para não precisar):

```ts
import { AsyncLocalStorage } from "node:async_hooks";

interface TenantStore {
  tenantId: string;
}

const storage = new AsyncLocalStorage<TenantStore>();

export const TenantContext = {
  run<T>(tenantId: string, fn: () => T): T {
    return storage.run({ tenantId }, fn);
  },
  get tenantId(): string {
    const store = storage.getStore();
    if (!store) {
      throw new Error("TenantContext acessado fora de uma request autenticada");
    }
    return store.tenantId;
  },
};
```

Criar `TenantInterceptor` (`APP_INTERCEPTOR`, registrado **depois** do `JwtAuthGuard`/`RolesGuard` no `app.module.ts`) que lê `request.user.tenantId` (populado pelo JWT — ver 1.4) e envolve `next.handle()` com `TenantContext.run(...)`. Rotas públicas (`@Public()`, ex: login, register-tenant) não têm tenant ainda nesse ponto — o interceptor deve pular se não houver `request.user`.

### 1.3 Prisma Client Extension — isolamento automático

Ponto mais sensível tecnicamente. Objetivo: nenhum service pode esquecer de filtrar por tenant — o filtro é injetado automaticamente na camada do Prisma Client.

Criar `apps/api/src/prisma/tenant-scoped.extension.ts`. Regras por tipo de operação (Prisma Client Extensions, `$allModels.$allOperations`):

- `findMany`, `count`, `aggregate`, `groupBy`, `updateMany`, `deleteMany`: aceitam `where` arbitrário → injetar `args.where = { ...args.where, tenantId: TenantContext.tenantId }`.
- `create`, `createMany`: injetar `tenantId` em `args.data` (ou em cada item de `data` se for array).
- `findUnique`, `findUniqueOrThrow`, `findFirst`, `findFirstOrThrow`: **atenção** — `findUnique` só aceita campos únicos em `where`, não dá pra simplesmente adicionar `tenantId` (Prisma rejeita). Solução: deixar a query rodar normalmente e **checar o resultado depois**: se `result.tenantId !== TenantContext.tenantId`, retornar `null` (ou lançar, nas variantes `OrThrow`) como se o registro não existisse. Isso fecha o vazamento sem quebrar a API do Prisma.
- `update`, `delete`, `upsert` (operações também feitas por chave única): antes de executar, fazer um `findUnique` (já protegido pela regra acima) para confirmar que o registro pertence ao tenant atual; se não pertencer ou não existir, lançar `NotFoundException`. Só então deixar a operação original prosseguir.

Registrar a extensão no `PrismaService` (`apps/api/src/prisma/prisma.service.ts`), aplicando `this.$extends(tenantScopedExtension)` — cuidado: `$extends` retorna um client novo, então o padrão comum é a classe expor um client estendido em vez de estender `PrismaClient` diretamente, ou usar o padrão de composição recomendado pela doc do Prisma para extensions com NestJS (procurar "Prisma Client Extensions NestJS custom provider" na doc oficial ao implementar).

**Limitação a documentar no código**: queries via `$queryRaw`/`$executeRaw` **não** passam pela extension — se algum relatório usar SQL raw (o módulo `reports` é candidato, por causa de agregações financeiras), o `tenantId` precisa ser adicionado manualmente na cláusula `WHERE` desse SQL. Auditar `reports.service.ts` especificamente por isso.

### 1.4 JWT e login

- `auth.service.ts`: `JwtPayload` passa a incluir `tenantId: string`. `issueTokens` recebe e assina o `tenantId` do usuário.
- `jwt.strategy.ts`: `validate()` retorna também `tenantId` (buscar do `User` no banco, não confiar cegamente no payload para dados sensíveis — mas `tenantId` pode vir do payload já que é imutável por sessão).
- `current-user.decorator.ts`: `AuthenticatedUser` ganha `tenantId: string`.
- Novo endpoint `POST /auth/register-tenant`: cria um `Tenant` novo (com `slug` derivado do nome, checando unicidade) + primeiro `User` com role `ADMIN` vinculado a esse tenant, em uma transação. Esse é o fluxo de "clínica nova se cadastra no SaaS".

### 1.5 Seed e testes de isolamento

Atualizar `prisma/seed.ts` para criar **dois tenants de teste** (ex: "Clínica Sorriso" e "Clínica Vida"), cada um com seu admin/dentista/recepcionista e alguns pacientes. Depois, escrever um teste (ou roteiro manual) que loga como usuário do tenant A e tenta acessar por ID um paciente/orçamento/agendamento do tenant B — todos os endpoints devem responder 404, nunca vazar o dado. Esse é o critério de aceite da Fase 1: sem isso validado, não seguir para produção com clientes reais.

---

## FASE 2 — Painel administrativo da plataforma

Decisão de design: **super-admins da plataforma não são `User` de nenhum tenant.** Criar um model separado `PlatformAdmin` (id, email, passwordHash, name, createdAt) fora do escopo tenant-scoped, com seu próprio módulo de auth (endpoints diferentes, ex: `/platform-auth/login`). Isso evita ter que tornar `tenantId` opcional em `User` — o que quebraria a garantia "todo model tenant-scoped tem tenantId NOT NULL" da Fase 1 e reabriria a possibilidade de esquecer uma checagem.

Novo app no monorepo: `apps/admin` (Vite + React, reaproveitando `packages/shared-types`, `packages/eslint-config`, `packages/tsconfig` — o padrão já existe no monorepo, é só replicar a estrutura de `apps/web`). Funcionalidades: listar/criar/desativar tenants, alterar plano, ver métricas básicas de uso por tenant (nº de pacientes, agendamentos no mês). Endpoints correspondentes em `apps/api` sob um módulo `platform-admin`, protegido por uma guard separada (não usa `JwtAuthGuard`/tenant JWT comum).

---

## FASE 3 — Geração real de documentos (PDF / Word / Excel)

Novo módulo `apps/api/src/modules/documents`. Bibliotecas sugeridas (Node, sem dependência de serviço externo):
- PDF: `pdfkit` (leve, controle total de layout) ou `@react-pdf/renderer` (se preferir compor com JSX, mais natural vindo de React).
- Word: `docx` (npm package, gera `.docx` nativo).
- Excel: `exceljs`.

Endpoints a implementar (todos autenticados, tenant-scoped automaticamente pela extension ao buscar os dados):
- `GET /documents/budgets/:id/pdf` — orçamento formatado, com itens, valores e espaço para assinatura do paciente.
- `GET /documents/patients/:id/anamnesis-pdf` — anamnese + dados do paciente, para impressão/assinatura.
- `GET /documents/patients/:id/clinical-record-pdf` — prontuário/evolução clínica exportável.
- `GET /documents/reports/financial-report.xlsx` — versão em Excel do que hoje só existe como `financial-report.tsx` na tela (o módulo `reports` já calcula os dados; aqui só formata para exportação).

Os arquivos podem ser gerados on-the-fly (stream de resposta, sem persistir em storage) para o MVP — não precisa de bucket/S3 ainda, a menos que o usuário queira histórico de documentos gerados.

---

## FASE 4 — Billing das assinaturas (cobrar as clínicas)

Isso é diferente do módulo financeiro que a clínica usa para cobrar o *paciente* — aqui é a plataforma cobrando o *tenant*. Sugestão: Stripe Billing.
- Adicionar `stripeCustomerId`, `stripeSubscriptionId`, `subscriptionStatus` ao model `Tenant`.
- Endpoint para criar Checkout Session de assinatura na hora do `register-tenant` ou depois, via painel admin do tenant.
- Webhook `POST /billing/webhook` tratando eventos `customer.subscription.updated/deleted` para atualizar `subscriptionStatus` e, se inadimplente, bloquear acesso (guard adicional checando `tenant.active`/`subscriptionStatus` antes de liberar qualquer rota tenant-scoped).

---

## Riscos e decisões a não esquecer (recap da análise arquitetural)

- **Dinheiro**: o schema atual usa `Decimal` do Postgres (via Prisma) para valores — correto e seguro (não é `float`), mas mantenha essa disciplina em todo código novo (documentos, billing).
- **Datas/timezone**: confirmar que tudo é salvo em UTC no banco (padrão do Postgres `timestamp`) e a conversão de fuso acontece só no frontend.
- **RLS como camada extra**: a extension do Prisma cobre a aplicação, mas não é RLS de banco. Depois que a Fase 1 estiver validada e estável, considerar adicionar políticas RLS no Postgres como defesa em profundidade (não bloqueia lançamento, mas reduz o risco de um bug futuro na extension).
- **`reports.service.ts`**: auditar por SQL raw que não passa pela extension (ver Fase 1.3).
- **Jobs assíncronos**: lembretes (WhatsApp), geração de relatório pesado e conciliação OFX não devem rodar dentro do ciclo de request normal se o backend for para serverless no futuro — hoje o backend roda em Render (processo long-running), então isso não é urgente, mas vale documentar antes de migrar de plataforma.
- **Ordem de execução**: não pular para Fase 2/3/4 antes da Fase 1 estar validada com o teste de isolamento — é a fundação que todo o resto depende.
