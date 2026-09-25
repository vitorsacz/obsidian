#projeto #doorcor #arquitetura

Ver [[Visão Geral]] para o contexto do produto e [[Design System]] para os
tokens visuais usados nos componentes abaixo.

## Stack técnica

- **React 19.2** + **TypeScript** (`tsc -b` no build, modo strict padrão do
  template Vite)
- **Vite 8** como bundler/dev server
- **Tailwind CSS v4** via plugin `@tailwindcss/vite` — sem `tailwind.config.js`
  tradicional; tokens de design ficam em `@theme` dentro de `src/index.css`
  (ver [[Design System]])
- **oxlint** como linter (`npm run lint`)
- Sem dependências além de `react`/`react-dom` em produção — nenhuma lib de
  UI, roteamento ou formulário. Site é uma página única (SPA de uma seção só,
  sem rotas).

Scripts (`package.json`): `dev`, `build` (`tsc -b && vite build`), `lint`,
`preview`.

## Estrutura de pastas

```
doorcor/
├── src/
│   ├── App.tsx              monta as 7 seções na ordem fixa (ver abaixo)
│   ├── index.css             tokens @theme + @layer base/components (Design System)
│   ├── lib/
│   │   └── content.ts        fonte única de verdade pra todo texto/dado (Estrutura de Conteúdo)
│   └── components/
│       ├── Header.tsx        nav fixa, transparente→sólida no scroll, menu mobile
│       ├── Hero.tsx           full-bleed, H1 oversized, CTAs
│       ├── Marquee.tsx        ticker infinito de keywords (CSS @keyframes, sem JS de scroll)
│       ├── About.tsx          "Sobre" — grid de 3 fotos com legenda
│       ├── Services.tsx       "Produtos & Serviços" — grid de 4 cards
│       ├── Differentials.tsx  "Por que a DoorCor" — grid de 4 cards
│       ├── Gallery.tsx        "Projetos" — grid de 8 fotos reais + paginação "01/08"
│       ├── Contact.tsx        WhatsApp-only, sem formulário
│       ├── Footer.tsx         direitos reservados + crédito do dev
│       └── ProjectPhoto.tsx   componente único de imagem (overlay + fill)
├── public/
│   └── img/                   37 fotos reais fornecidas pelo Vitor (doorcor-01.jpg … doorcor-37.jpg)
└── README.md
```

## Ordem das seções (`App.tsx`)

```
Header (fixo, fora do fluxo)
Hero            → 01
Marquee         (sem número — é decorativo/transição)
About           → 01  "Sobre"
Services        → 02  "Produtos & Serviços"
Differentials   → 03  "Por que a DoorCor"
Gallery         → 04  "Projetos"
Contact         → 05  "Contato"
Footer
```

**Atenção**: os números de índice de `About`/`Hero` colidem (ambos "01") —
é intencional, o Hero tem seu próprio contador visual separado do fluxo de
seções numeradas que começa em About. Se `Differentials` e `Gallery` forem
reordenados de novo no futuro, os números `index-number` hardcoded em cada
componente (`"03"`, `"04"`) precisam ser trocados manualmente — não são
gerados a partir da posição no array, são literais em cada arquivo.

## `ProjectPhoto` — componente central de imagem

Único componente de imagem do site (substituiu um antigo `PlaceholderPhoto`
quando fotos reais entraram no projeto). Duas props resolvem os dois
problemas que apareceram durante o desenvolvimento:

- **`fill: boolean`** — alterna entre duas strings de classe completas e
  mutuamente exclusivas (`"absolute inset-0 h-full w-full"` vs.
  `"relative h-full w-full"`) em vez de deixar `className` externo tentar
  sobrescrever uma classe `relative` hardcoded via concatenação. Existe
  porque Tailwind não garante ordem de precedência entre `relative` e
  `absolute` quando ambos aparecem em strings de classe diferentes — o hero
  renderizava como item de flex pequeno em vez de background full-bleed até
  essa prop ser introduzida.
- **`loading="eager"`** (não `"lazy"`) — em contexto de browser automation
  (Chrome via extensão), `loading="lazy"` não disparava o carregamento da
  imagem de forma confiável (`img.complete`/`naturalWidth` ficavam presos).
  Trade-off aceito: todas as imagens carregam imediatamente, sem lazy-load
  real. Se performance de carregamento inicial virar problema (site tem
  ~12 imagens ativas hoje), vale revisitar com lazy-load nativo + fallback,
  mas não há sinal de que isso seja necessário agora.

## Fotos reais (`public/img/`)

37 arquivos renomeados de `doorcor-01.jpg` a `doorcor-37.jpg` (originais
exportados do WhatsApp, com espaços/parênteses no nome, renomeados via script
bash). **Só 12 estão referenciadas no código** hoje (via `content.ts` —
ver [[Estrutura de Conteúdo]]): `01, 02, 07, 08, 09, 13, 14, 16, 17, 18, 20,
30`. Essas 12 foram redimensionadas/comprimidas com `sips -Z 1920` antes de
entrar no repositório; as outras 25 estão na pasta mas não são usadas em
lugar nenhum — candidatas naturais pra ampliar a Galeria no futuro (ver
[[Próximos Passos]]).

Critério de seleção aplicado: evitadas fotos com marca/identidade visual de
terceiros aparecendo no enquadramento (ex: sinalização de uma clínica
odontológica refletida num vidro) e fotos com pessoas identificáveis — por
critério próprio de privacidade/direitos de imagem, não porque o Vitor pediu
especificamente.

## Ancoragem de scroll

Toda seção com `id` (`#sobre`, `#produtos`, `#diferenciais`, `#projetos`,
`#contato`) tem `scroll-mt-24` pra compensar a altura do header fixo — sem
isso, o header cobriria o topo da seção ao clicar num link de navegação.
