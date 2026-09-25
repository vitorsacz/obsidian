#projeto #doorcor #design-system

Ver [[Visão Geral]] para o contexto do produto. Implementado em
`src/index.css` via `@theme` do Tailwind v4 (tokens viram classes utilitárias
automaticamente, ex: `--color-sand` → `bg-sand`, `text-sand`, `border-sand`).

## Especificação original (briefing do Vitor)

Paleta escura, sóbria e elegante, com acentos terrosos/metálicos. Tipografia
sans geométrica oversized, com toques ocasionais serifados/script em itálico
pra contraste. Grid estrutural visível (linhas finas de divisão). Layout em
cards modulares. Botões pill — ghost (contorno) e sólido. Header fixo,
minimalista, transparente até o scroll. Hero full-bleed com fotografia
dramática e overlay escuro. Paginação numerada estilo "01/03" com setas finas
longas.

## Paleta de cores

| Token | Hex | Uso |
|---|---|---|
| `ink` | `#000000` | Fundo principal (seções escuras, header, footer top) |
| `graphite` | `#121212` | Fundo alternado (Produtos, Diferenciais) |
| `charcoal` | `#1a1a1a` | Fundo de placeholder de imagem antes de carregar |
| `paper` | `#ffffff` | Fundo do footer (única seção clara) |
| `bone` | `#f5f5f5` | Texto principal sobre fundo escuro |
| `mist` | `#c6c6c6` | Texto secundário (era `#a0a0a0`, clareado 2026-09-02 por legibilidade) |
| `fog` | `#a8a8a8` | Texto terciário/labels (era `#888888`, clareado 2026-09-02) |
| `sand` | `#d8c9ab` | Acento principal — números, itálicos, hover states, botão sólido |
| `ocre` | `#b08d4f` | Acento secundário — hover do botão sólido, links no footer claro |
| `copper` | `#a97452` | Acento terciário — reservado, pouco usado nos componentes atuais |

**Nota de legibilidade (2026-09-02):** `mist` e `fog` foram clareados e o
peso base do corpo do texto subiu de 300 (light) pra 400 (regular) depois de
feedback do Vitor de que o texto sobre fundo escuro estava pouco evidente.
Todas as ocorrências de `font-light` nos parágrafos dos componentes também
foram removidas.

## Tipografia

Três famílias, cada uma com um papel fixo:
- **`--font-display`**: `"Archivo", "Helvetica Neue", Arial, sans-serif` —
  títulos, números de índice, labels em uppercase tracking largo, botões.
  É a fonte "oversized geométrica" do briefing (ex: H1 do Hero em
  `text-[16vw]` no mobile).
- **`--font-body`**: `"Inter", "Helvetica Neue", Arial, sans-serif` —
  parágrafos de corpo.
- **`--font-accent`**: `"Cormorant Garamond", "Times New Roman", serif` —
  usada só em itálico (`italic font-accent font-normal`), sempre em `sand`,
  pra dar o toque serifado/script de contraste dentro de headings (ex: "alto
  padrão" no H1, "peça de arquitetura" na seção Sobre, "obra de alto luxo"
  em Diferenciais).

## Componentes visuais reutilizáveis (`@layer components`)

- **`.container-edge`** — wrapper de largura máxima (1440px) com padding
  responsivo; usado em toda seção pra alinhar o conteúdo às bordas do grid.
- **`.eyebrow`** — label pequeno uppercase com tracking largo (`0.35em`), cor
  `mist`; usado como subtítulo de seção logo abaixo do número de índice.
- **`.index-number`** — força `font-display` + `font-variant-numeric:
  tabular-nums`, garante que "01", "02" etc. fiquem alinhados/monoespaçados.
  É o elemento central do padrão de paginação numerada do briefing.
- **`.btn-ghost`** — pill outline (`border-white/25`, `rounded-full`), texto
  uppercase tracking largo, hover muda borda/texto pra `sand`.
- **`.btn-solid`** — pill preenchido em `sand`, texto `ink`, hover escurece
  pra `ocre`.
- **`.hairline`** — `border-white/10`, usado implicitamente via a regra
  global `* { border-color: rgb(255 255 255 / 0.1) }` — é assim que as
  linhas de grid estrutural (divisórias entre cards, entre seções) ficam
  sempre finas e sutis sem precisar declarar cor em cada `border`.

## Grid estrutural

O efeito de "linhas de grid visíveis" do briefing é feito de duas formas:
1. **Bordas finas** (`border-white/10`) em `border-t` no topo de cada bloco
   de seção, e em grids de cards (`border-t border-l` + cada célula com
   `border-r border-b`) — cria uma grade contínua sem gap.
2. **Grids "de 1px"**: containers com `gap-px bg-white/10` e células opacas
   por cima — técnica usada em `About` (fotos) e `Gallery` (cards de
   projeto) pra criar divisórias de exatamente 1px entre itens de tamanhos
   variáveis, mais confiável que `border` em grids responsivos.

## Fotografia / overlays

Componente único `ProjectPhoto` (ver [[Arquitetura]]) controla todo overlay
de imagem via a prop `overlay`:
- `"hero"` — camada `bg-ink/45` constante + gradiente
  `from-ink via-ink/50 to-ink/20` de baixo pra cima. Garante contraste do
  texto do Hero mesmo com fotos muito claras, sem escurecer demais o topo
  (onde fica o header).
- `"card"` — gradiente mais sutil `from-ink/90 via-ink/10 to-transparent`
  que escurece só a base do card (onde fica texto), com hover suavizando
  pra `from-ink/70`.
- `"none"` — sem overlay, imagem crua.

Todo `<img>` usa `contrast-105 brightness-95` como tratamento de cor padrão
e `loading="eager"` (não `"lazy"` — ver [[Arquitetura]] sobre por quê).

## Botões / CTA — regra de conteúdo

Design system à parte, há uma regra de **produto** que atravessa todos os
botões: todo CTA do site é um link `wa.me` (ver `buildWhatsAppUrl` em
[[Estrutura de Conteúdo]]), nunca um `<button>` de formulário. Isso não está
no CSS, mas é tão estrutural quanto — qualquer novo botão adicionado ao site
deve seguir esse padrão por padrão, a menos que o Vitor peça o contrário.
