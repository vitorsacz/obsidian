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

### Prisma 5.22: duas transações interativas na mesma linha → 500 (P2028) (2026-09-26, S3)
Achado no teste no Chrome do refresh rotativo: várias abas abrindo juntas
mandavam o mesmo refresh token, e uma delas recebia **500** (`Transaction API
error: Unable to start a transaction in the given time`, código `P2028`) em
vez de 401. Reproduzido com o **Prisma puro, fora do Nest**: quando duas
transações interativas (`$transaction(async (tx) => …)`) disputam a mesma
linha, a segunda falha depois de 2 s (o `maxWait` padrão) em vez de esperar o
lock, que dura milissegundos. Acontece também com `connection_limit` alto.

Corrigido sem transação interativa: a sessão é reivindicada com um único
`UPDATE … WHERE revokedAt IS NULL` (atômico no Postgres; quem perde recebe
`count = 0` e é tratado como reuso → 401), e só depois a próxima sessão e o
`replacedById` são gravados numa **transação em lote** (`$transaction([...])`).
Teste e2e: dois refresh simultâneos com o mesmo token dão exatamente um 201 e
um 401, em 5 rodadas. **Regra pro projeto:** em código com concorrência na
mesma linha, preferir UPDATE condicional + transação em lote.

Efeito colateral que o mesmo teste mostrou: o controller apagava o cookie de
refresh em **qualquer** erro, então o 500 deslogava a aba seguinte. Agora só
apaga em 401.

### 401 do login reativava uma sessão antiga (2026-09-26, visto no S1, corrigido no S3)
O `api-client` do front tentava `auth/refresh` em **qualquer** 401, inclusive
no do próprio `POST /auth/login`. Se ainda existisse um cookie de sessão antiga
no navegador, uma tentativa de login com senha errada renovava essa sessão
antiga em memória. Corrigido: `/auth/login` e `/auth/refresh` não disparam
refresh no 401.

### Modal deslocado 24 px pelo `space-y-*` do container (2026-09-26, PR #21)
O diálogo de confirmação de Admin, renderizado dentro da página, herdava o
`margin-top` do `space-y-6` (o Tailwind põe margem em todo filho, inclusive
um `fixed inset-0`), e o fundo escuro não cobria o topo da tela. Achado
medindo no navegador (`getBoundingClientRect().top === 24`, `marginTop:
24px`). Corrigido com **portal no `<body>`** (`createPortal`). Os modais da
Agenda usam o mesmo padrão `fixed inset-0` dentro da página — conferir se
aparecerem deslocados.

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
  restrito a DENTIST/RECEPTIONIST (capacidade `appointments.manage` na matriz
  `ACCESS`) — ADMIN só ganhou acesso de leitura a `/clinics`, `/patients`,
  `/procedures` (pro picker do modal mockado funcionar), **não** a
  `/appointments`. Se a Agenda virar real (Fase 4 do
  [[roadmap-papeis-permissoes-financeiro]]), lembrar de rever
  `appointments.manage` no `ACCESS`.
- **Rate limit em memória e por IP** (S2): com mais de uma instância da API
  cada uma conta separado (precisaria de Redis); e numa clínica em que todos
  saem pelo mesmo IP, 5 logins errados de uma pessoa travam o login de todos
  por 1 minuto.
- **Access token continua valendo até 15 min** depois de logout, `logout-all`
  ou desativação — só o refresh para de funcionar (S3). Redefinir a senha pelo
  admin **não** derruba as sessões (não era escopo do S3; vale considerar).
- **Tabela `RefreshSession` cresce** uma linha por refresh; as expiradas só são
  apagadas no próximo login do próprio usuário.
- **`GET /organization` do Super Admin responde 400** (não 403): a rota é
  `@AllowAuthenticated` e o 400 vem de dentro do controller. Mantido assim de
  propósito na matriz `ACCESS` (PR #18) pra não mudar comportamento.
