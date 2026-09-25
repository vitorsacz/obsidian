#projeto #doorcor

## O que é

Site institucional em React para a **DoorCor**, fabricante brasileira de
portas premium em ACM (Alumínio Composto) para arquitetura residencial e
corporativa de alto padrão — portas externas/internas, tecnologia embarcada
(fechaduras eletrônicas, biometria, automação) e fachadas em ACM.

A marca real existe no Instagram [@doorcor.acm](https://www.instagram.com/doorcor.acm/).
O projeto nasceu de uma análise desse perfil (identidade visual, produtos,
pontos fortes) e virou um site completo construído em cima de um design
system definido junto com o Vitor.

Repositório local: `~/VITOR/doorcor`. Sem repositório remoto configurado até
2026-09-02 — nenhum commit foi feito além do scaffold inicial (ver
[[Estado Atual]]).

## Como pensar sobre ele

Três camadas, nessa ordem de dependência:
1. **Conteúdo** (`src/lib/content.ts`) — fonte única de verdade pra todo
   texto, número de telefone, links e listas (serviços, diferenciais, fotos).
   Nenhum componente tem copy hardcoded.
2. **Design system** (`src/index.css` via `@theme` do Tailwind v4) — paleta,
   tipografia, componentes reutilizáveis (`.btn-ghost`, `.btn-solid`,
   `.eyebrow`, `.index-number`). Ver [[Design System]].
3. **Componentes de seção** (`src/components/`) — um componente por seção da
   página, montados em ordem fixa em `App.tsx`. Ver [[Arquitetura]].

O contato é **exclusivamente via WhatsApp** — não existe formulário no site.
Todo CTA (botão) do site abre `wa.me` com uma mensagem pré-preenchida
diferente conforme a intenção (orçamento vs. ver produtos). Essa foi uma
decisão explícita do Vitor, não uma limitação técnica.

## Notas relacionadas
- [[Design System]] — paleta de cores, tipografia, componentes visuais, regras de estilo
- [[Arquitetura]] — estrutura de pastas, stack técnica, como as seções se conectam
- [[Estrutura de Conteúdo]] — o que está em `content.ts`, textos e dados reais do site
- [[Estado Atual]] — o que está pronto, verificado e pendente
- [[Próximos Passos]] — decisões em aberto pro Vitor

## Linha do tempo de decisões
- **2026-08-13** — análise do Instagram @doorcor.acm feita via Chrome
  automation (identidade visual, produtos, pontos fortes). Base pro design
  system e pro copy inicial do site.
- **2026-08-13** — design system detalhado definido em conjunto com o Vitor:
  paleta escura sóbria/elegante com acentos terrosos/metálicos, tipografia
  geométrica oversized com toques serifados/script em itálico, grid
  estrutural visível, cards modulares, botões pill (ghost/solid), header fixo
  transparente, hero full-bleed com overlay escuro, paginação numerada estilo
  "01/03" com setas finas longas.
- **2026-08-13** — site scaffolded com Vite + React 19 + TypeScript +
  Tailwind v4, todas as 7 seções implementadas e verificadas em desktop
  (1512px) e mobile (390px) via browser automation.
- **2026-08-20 (aprox.)** — pedido de imagens reais de projetos na galeria.
  Tentativa inicial de extrair imagens do Instagram via browser automation
  esbarrou em bloqueios (CDN com tokens assinados na URL, proteção do Chrome
  contra múltiplos downloads automáticos). Abandonada quando o Vitor avisou
  que já tinha adicionado mais de 30 fotos reais direto em `public/img/`.
- **2026-08-20 (aprox.)** — pivô pra usar as 37 fotos reais fornecidas pelo
  Vitor. Criado o componente `ProjectPhoto` (substitui um antigo
  `PlaceholderPhoto`), 12 fotos selecionadas manualmente (evitando fotos com
  marca de terceiros visível ou pessoas identificáveis), redimensionadas via
  `sips` e distribuídas entre Hero, Sobre e Galeria.
- **2026-09-02** — rodada de revisão com 6 pedidos pontuais do Vitor: todos
  os CTAs redirecionando pro WhatsApp (+55 11 93215-2858) com mensagens
  específicas, legendas nas fotos da seção Sobre, botão de atalho pra galeria
  na seção Produtos, seção "Por que a DoorCor" reordenada pra antes da
  Galeria, Contato reescrito sem formulário (só WhatsApp), rodapé com direitos
  reservados + crédito "Desenvolvido por Vitor Santos | vitorsantos.dev.br".
  Acompanhado de ajuste geral de legibilidade (cores de texto secundário mais
  claras, peso de fonte do corpo mais pesado). Tudo implementado, `tsc`/`build`/
  `lint` limpos, verificado visualmente via browser automation.
