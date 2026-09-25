#projeto #hubassistent #arquitetura

Ver [[Visão Geral]] para o contexto do produto.

## Monorepo

pnpm workspaces + Turborepo.

```
hubassistent/
├── apps/
│   ├── api/     # NestJS + Prisma
│   └── web/     # React + Vite
└── packages/
    ├── shared-types/    # schemas Zod compartilhados (contrato de API)
    ├── eslint-config/
    └── tsconfig/
```

`shared-types` é buildado em dual-format (CJS + ESM via tsup) porque o `apps/api` (NestJS/CommonJS) e o `apps/web` (Vite/ESM) precisam consumi-lo de formas diferentes. Barrel exports usam `export { x } from "./y"` nomeado explicitamente, nunca `export *` — um `export *` compilado pra CommonJS quebra a análise estática do Rollup no build do Vite (bug real que já apareceu e foi corrigido).

## Backend (NestJS)

- Auth própria (não Supabase Auth) — JWT access + refresh, refresh token em cookie httpOnly. Mantém um único modelo de usuário, sem acoplar a um provedor de identidade externo.
- **Todo domínio é isolado por `userId`** — é o padrão estrutural que faz "virar multiusuário" ser uma mudança pequena no futuro, não uma reescrita.
- Módulos existentes: `AuthModule`, `UsersModule` (implícito via auth), `AccountsModule`, `CardsModule`, `CategoriesModule`, `TransactionsModule`, `InvoicesModule`.
- `InvoicesModule` tem, além do CRUD padrão, upload de arquivo (`FileInterceptor` do `@nestjs/platform-express` com `memoryStorage()` — arquivo fica só em memória, nunca toca o disco efêmero do Render) e um serviço separado de parsing (`InvoicePdfParserService`, sem acesso a banco). `multer` precisou virar dependência direta do `apps/api` (não só transitiva via `@nestjs/platform-express`) porque em workspace pnpm imports transitivos não resolvem.
- `InvoicePdfParserService` extrai o texto do PDF com `pdf-parse` **v2** — API mudou bastante da v1: agora é `new PDFParse({ data: buffer })` + `await parser.getText()` + `await parser.destroy()`, não mais a função direta `pdfParse(buffer)` — e manda esse texto pro **Gemini** (`@google/genai`, `new GoogleGenAI({apiKey}).models.generateContent({...})`) junto de um prompt que pede JSON estruturado (banco, mês/ano, vencimento, transações). O prompt foi portado do projeto Python `~/VITOR/invoice-reader`. Cuidado ao mexer no prompt: a instrução de "identifique o mês de referência com base no vencimento" precisou ser refinada pra priorizar texto explícito no PDF (ex: "Fatura de julho") — sem isso o Gemini inferia o mês errado a partir só do vencimento, ignorando o texto explícito do documento.
- Validação via Zod (`ZodValidationPipe`), **sempre vinculado a um parâmetro específico** (`@Body(new ZodValidationPipe(schema))`), nunca via `@UsePipes` em nível de método — `@UsePipes` no método aplica o pipe a *todos* os parâmetros (incluindo `@Param`, `@CurrentUser()`), o que quebra endpoints com múltiplos parâmetros heterogêneos. Bug real já encontrado e corrigido nesse projeto.
- Prisma: schema em `apps/api/prisma/schema.prisma`. O hook de postinstall do `@prisma/client` não acha esse schema sozinho num monorepo — por isso `apps/api/package.json` tem `"postinstall": "prisma generate"` explícito. Sem isso o client gerado fica vazio (sem tipos dos models) e builda localmente (se alguém já rodou generate manual antes) mas quebra num ambiente limpo tipo Render.

## Frontend (React)

- Vite (não Next.js — a API já é separada, SSR não seria usado).
- TanStack Query para dados de servidor, React Hook Form + Zod para formulários.
- Cuidado com `reset()` do React Hook Form: passar `undefined` pra um campo é interpretado como "não mexe no valor atual", não como "limpa". Pra limpar de verdade, usar `""` (ou o equivalente vazio do tipo). Bug real: formulário de transação não limpava depois de criar/cancelar editar até isso ser corrigido.

Ver [[Design System]] para tokens visuais e [[Infraestrutura e Deploy]] para como tudo isso roda em produção.
