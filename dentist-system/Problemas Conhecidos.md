#projeto #dentist-system

Ver [[Arquitetura]] pro código, [[Roadmap]] pro que falta construir.

## Bugs reais já encontrados e corrigidos

### Prisma `Decimal` serializa como string, quebrava `.toFixed()` no front
Todo campo `Decimal` do Prisma (valores monetários: orçamento, procedimento,
atendimento, aluguel) chega da API como **string** no JSON (`"900"`, não
`900`) — `Decimal.toJSON()` do Prisma faz isso por padrão. O front chamava
`.toFixed(2)` nesses valores e quebrava a tela inteira (`TypeError` no React,
componente sumia). Corrigido com um interceptor global
(`common/interceptors/decimal.interceptor.ts`) que converte todo `Decimal`
pra `number` antes de qualquer resposta HTTP sair da API — resolve pra
sempre, não precisa lembrar de converter campo por campo em cada service.

### Campo de data opcional vazio quebrava validação sem avisar
`<input type="date">` vazio manda `""` pro form, não `undefined`. Os schemas
Zod tinham `birthDate`/`expiryDate` como `z.coerce.date().optional()` — mas
`.optional()` só aceita `undefined`, não `""`, e `new Date("")` é inválido.
Resultado: o formulário simplesmente não submetia (nenhum request saía, sem
erro visível) toda vez que o campo de data ficava em branco. Corrigido com um
preprocess compartilhado (`packages/shared-types/src/utils.ts`,
`optionalCoercedDate`) que trata `""`/`null` como "não preenchido" antes de
tentar converter pra data.

### Logout não encerrava a sessão de verdade em produção (achado pelo Vitor testando manualmente)
`res.clearCookie(REFRESH_COOKIE)` no `logout()` era chamado **sem** os atributos
`sameSite`/`secure` usados na criação do cookie (`setRefreshCookie`). Como o
cookie de refresh precisa de `SameSite=None; Secure` pra funcionar entre domínios
diferentes (Vercel↔Render), um navegador real **rejeita silenciosamente** um
Set-Cookie de limpeza que não declare esses mesmos atributos — o cookie original
continuava válido, e o "Sair" só limpava a tela, não a sessão de verdade.
Resultado: mesmo depois de clicar "Sair", visitar qualquer rota de novo (ex.:
`/robots.txt`, que cai no `index.html` pelo rewrite de SPA) restaurava o login
sozinho via o cookie que nunca foi realmente apagado.

**Por que não apareceu nos testes anteriores**: em dev local os dois domínios são
`localhost`, então `sameSite` é sempre `"lax"` nos dois lados — só quebra quando
front e back estão em domínios diferentes de verdade (produção). `curl` também
não reproduz isso sozinho, porque não aplica as mesmas regras de SameSite que um
navegador real aplica — só foi possível confirmar testando no navegador contra a
Vercel/Render de produção.

Corrigido reaproveitando as mesmas opções (`getRefreshCookieOptions()`) tanto na
criação quanto na limpeza do cookie, pra nunca mais divergir.

## Avisos do editor que NÃO são bugs (já investigados, pode ignorar)

### VS Code marca `url`/`directUrl` do `schema.prisma` como erro
A extensão Prisma do VS Code valida contra as regras do Prisma 7 (que
descontinuou essas propriedades em favor de `prisma.config.ts`), mas o
projeto usa Prisma **5.22** de propósito (mesmo padrão do hubassistent), onde
`url`/`directUrl` no datasource continuam sendo a forma correta. Confirmado
com `prisma validate` rodando limpo. Nenhuma ação necessária — decisão
tomada em 2026-07-28 de não migrar pra Prisma 7 agora.

### VS Code marca `baseUrl`/`moduleResolution: Node` do `tsconfig.json` como depreciado
Mesma causa raiz: o TypeScript embutido no VS Code é mais novo que o
TypeScript **5.9.3** que o projeto usa de verdade. Rodando `tsc` do próprio
projeto direto no `apps/api/tsconfig.json`, zero erros/avisos. Corrigido de
forma permanente com `.vscode/settings.json` (`typescript.tsdk:
"node_modules/typescript/lib"`) apontando o editor pra versão do workspace —
se o aviso reaparecer, confirmar no VS Code que "Use Workspace Version" foi
aceito (paleta de comandos → "TypeScript: Select TypeScript Version").

### Sidebar rolava junto com a página (front, 2026-09-22)
Depois da sidebar fixa global (ver [[Roadmap]]), rolar uma página com conteúdo
mais alto que a tela arrastava a sidebar inteira junto — deveria ficar travada,
só o conteúdo à direita deveria ter scroll próprio. Causa raiz: o container
raiz do `app-shell.tsx` usava `min-h-screen` (define só uma altura *mínima*,
não trava em 100vh) e o `<main>` não tinha `min-h-0`. Sem `min-h-0` um item
flex nunca encolhe abaixo do tamanho do próprio conteúdo — o `overflow-y-auto`
do `<main>` nunca entrava em ação de verdade, e era o `<html>`/`<body>` inteiro
que crescia e rolava, arrastando a sidebar. Corrigido: container raiz vira
`h-screen overflow-hidden` (trava exatamente na altura da viewport, documento
nunca rola) e `<main>` ganhou `min-h-0` (permite o flex item encolher e o
`overflow-y-auto` realmente funcionar). Confirmado injetando um elemento de
3000px dentro do `<main>` via JS e checando que só `main.scrollTop` responde,
`document.documentElement.scrollHeight`/`window.scrollY` continuam travados.

## Limitações conhecidas (não são bugs, são escopo)
- O sistema hoje trata todos os dentistas/recepcionistas como vendo os
  **mesmos** pacientes/agenda — não há isolamento por "consultório de cada
  dentista". Se um dia precisar disso, é mudança de modelo de dados, não só
  de tela.
- **Agenda, Home Dashboard e Procedimentos (redesenho, 2026-09-21/22) são
  telas mockadas** — layout, cores, sidebar de dentistas e fluxo de criação
  de agendamento estão prontos e testados visualmente, mas os dados vêm de
  hooks `useMock*` locais (`useMemo`/`localStorage`), não do backend real. Só
  Locations (`clinicsApi.list()`), pacientes e procedimentos do picker do
  modal de agendamento são reais — o resto (calendários, agendamentos,
  cadastro/cor de dentista) é maquete. Ver [[Roadmap]] pra escopo de quando
  isso vira backend de verdade, e [[Arquitetura]] pro padrão técnico usado.
  **Consequência concreta**: `GET /appointments` (backend real) continua
  `@Roles("DENTIST", "RECEPTIONIST")` — ADMIN só ganhou acesso de leitura a
  `/clinics`, `/patients`, `/procedures` (pro picker do modal mockado
  funcionar), **não** a `/appointments`. Se a Agenda virar real um dia, tem
  que lembrar de estender esse `@Roles` também.
