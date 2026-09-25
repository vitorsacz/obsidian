---
tags: [projeto, dentist-system, design-system, referencia]
status: publicado como Artifact (Claude) — fonte de verdade para tokens visuais
relacionado: "[[Roadmap]], [[Arquitetura]], [[feature - painel-admin-rbac-lgpd]]"
artifact-url: https://claude.ai/artifact/3bgisvfDjSWKofoYqgcVw4
---

# Plataforma Odontológica — Design System

> Este arquivo e o `design-system-plataforma-odontologica-tokens.json` ao lado são a versão exportada do Design System publicado como Artifact no Claude. Se você pedir ao Claude Code para ler esta pasta do Obsidian, ele tem tudo que precisa aqui — sem depender de acesso à URL do artifact.

Este sistema documenta o padrão visual extraído do **TailAdmin** (`https://react-demo.tailadmin.com/`, template React + Tailwind + ApexCharts, inspecionado ao vivo via Claude in Chrome em 2026-09-18) para uso como referência de estilo e estrutura na plataforma de gestão odontológica — **não o template em si**, mas a linguagem visual: bordas finas em vez de sombra pesada, paleta semântica clara, tipografia geométrica e componentes de dashboard densos e legíveis. Aplicado de verdade no redesenho visual de 2026-09-21/22 (Home Dashboard, Agenda, Sidebar, Procedimentos — ver [[Roadmap]]).

## Fundamentos

- **Apoiado em borda, não em sombra.** Cards e inputs usam `border-default` fino (`border border-gray-200`, sem sombra pesada); sombra aparece só no botão primário, e de forma quase imperceptível (`shadow-theme-xs`).
- **Paleta semântica consistente.** Todo estado (sucesso, erro, pendência, informação) tem três variantes prontas: fundo claro + texto colorido (uso do dia a dia), fundo sólido + texto branco (ênfase), e a mesma cor usada com ícone. Formato de badge sempre pill (`rounded-full`), `text-sm`/`text-xs`, padding `px-2.5 py-0.5`.
- **Densidade com respiro.** Tabelas e cards de estatística priorizam leitura rápida de números — números grandes, labels pequenos, badges de tendência para contexto imediato.

## Cores

Extraídas via computed style direto dos elementos renderizados do TailAdmin (não é chute).

### Brand (cor de ação — accent do produto)
| Token | Hex | Uso |
| --- | --- | --- |
| `brand-50` | `#ECF3FF` | fundo claro (badge primary, ícone ativo) |
| `brand-300` | *(sem valor exato extraído — ver Gaps abaixo)* | borda de foco |
| `brand-500` | `#465FFF` | cor principal — botões, links ativos, barras de gráfico |
| `brand-600` | *(sem valor exato extraído — ver Gaps abaixo)* | hover do botão primário |

### Cores semânticas (status) — sempre 3 tokens: `-solid`, `-text`, `-light`
| Semântica | Solid (bg sólido) | Texto (sobre bg claro) | Fundo claro |
| --- | --- | --- | --- |
| Success | `#12B76A` | `#039855` | `#ECFDF3` |
| Error | `#F04438` | `#D92D20` | `#FEF3F2` |
| Warning | `#F79009` | `#DC6803` | `#FFFAEB` |
| Info | `#0BA5EC` | `#0BA5EC` | `#F0F9FF` |

Mapeamento direto pros enums do domínio: verde (`success`) = aprovado/ativo/
concluído; laranja (`warning`) = pendente; vermelho (`error`) = cancelado/
erro/suspenso; azul (`info`) = informativo.

### Escala de cinza (neutros)
| Token | Hex | Uso |
| --- | --- | --- |
| `gray-50` | `#F9FAFB` | fundo da página (atrás dos cards) |
| `gray-100` | `#F2F4F7` | fundo de ícone/badge "light" |
| `gray-200` | `#E4E7EC` | bordas de card/input/tabela |
| `gray-400` | *(sem valor exato)* | placeholder |
| `gray-500` | `#667085` | texto secundário, header de tabela |
| `gray-700` | `#344054` | label de formulário |
| `gray-800` | `#1D2939` | texto principal, títulos |
| `gray-900` | `#101828` | texto mais escuro / sidebar em dark mode |
| white | `#FFFFFF` | fundo de card, sidebar, header |

**Tema escuro**: só teve confirmação explícita para `surface-page`,
`surface-card`, `border-default` e `text-primary` (viram `gray-900`,
`gray-900`, `gray-800`, `white/90` respectivamente). `text-secondary` e
`text-label` herdam o valor do tema claro até serem auditados — não usar às
cegas em produção sem checar contraste. Existe toggle de tema no header do
TailAdmin (ícone de lua) — não implementado no produto ainda.

## Tipografia

Fonte única: **Outfit** (Google Font, geométrica, se aproxima de Inter/Poppins) — `font-family: Outfit, sans-serif` no `<body>`. Cinco estilos cobrem o essencial:
- **Título de página** (`page-title`) — `20px`/`font-weight: 600`, cor `gray-800`.
- **Label de formulário** (`form-label`) — `14px`/`font-weight: 500`, cor `gray-700`, margem inferior `6px`.
- **Corpo/tabela** (`body`) — `14px`, normal.
- **Número grande de estatística** (`stat-number`) — classe `text-title-sm` + `font-bold`, ~24-28px, `font-weight: 700`.
- **Badge/status** (`badge`) — `12-14px`, `font-weight: 500`.

## Espaçamento, borda e sombra

- **Cards**: `radius-card` (16px, `rounded-2xl`), `border-default` (`border border-gray-200`, sem `box-shadow` pesado), padding `card-padding-mobile` (20px, `p-5`) / `card-padding-desktop` (24px, `p-6`).
- **Botões e inputs**: `radius-control` (8px, `rounded-lg`).
- **Badges**: `radius-pill` (`rounded-full`).
- **Botão primário**: `bg-brand-500`, texto branco, `shadow-theme-xs` (sombra bem sutil — valor CSS exato não extraído, ver Gaps), padding varia por tamanho (sm: `12px 16px`, md: `14px 20px`).
- **Input**: `border border-gray-200`, `bg-transparent`, foco vira `border-brand-300` + `ring-3 ring-brand-500/10` (anel de foco suave, não borda grossa).
- **Tabela**: sem bordas verticais, só `border-bottom` fina (`gray-100/200`) separando linhas; cabeçalho sem fundo, texto `gray-500 font-medium`.

## Padrões de componente

Mapeamento direto para o domínio da plataforma (pacientes, agenda, orçamento, usuários, clínica):

1. **Stat card** — ícone dentro de caixa `radius-control bg-gray-100`, label pequeno, número grande (`stat-number`) em negrito, badge de tendência (pill verde `↑ 11%` ou vermelho `↓ 9%`, `success-light`/`error-light`) alinhado à direita. Uso: totais do dashboard (clínicas ativas, dentistas, armazenamento, atendimentos do mês).
2. **Card de gráfico** — título + ícone de menu "⋮" no canto superior direito, corpo do gráfico abaixo.
3. **Gauge/radial** ("Monthly Target") — arco de progresso com % grande no centro, texto de apoio abaixo, barra de 3 colunas no rodapé (Target/Revenue/Today) separada por borda superior. Uso: % de ocupação da agenda, % de meta de faturamento.
4. **Tabela de listagem** — avatar + nome em negrito + subtítulo (papel/cargo) na mesma célula; status como badge; coluna numérica alinhada à direita. Uso direto: lista de tenants, pacientes, usuários.
5. **Badge de status** — mapeia diretamente para os enums do domínio (`BudgetStatus`, `AppointmentStatus`, `RecallStatus`, `active/inactive` de usuário).
6. **Formulário** — label acima do campo (`form-label`), borda fina, ícone dentro do campo quando aplicável (email, telefone), agrupado em cards por seção.
7. **Header fixo** — menu hambúrguer (colapsa sidebar), busca global (`⌘K`), toggle de tema, notificações (sino com bolinha), avatar/nome com chevron (dropdown). *(No produto real, o header dedicado da Home foi descontinuado quando a sidebar global entrou — ver [[Roadmap]]; busca e notificações eram só decorativas/mock e saíram nessa troca.)*
8. **Sidebar** — fixa à esquerda (~290px no TailAdmin, 240px no produto), logo no topo, itens com ícone outline + label, seções agrupadas por título pequeno maiúsculo (`MENU`, `OTHERS`, `SUPPORT` no TailAdmin; `Menu`/`Clínica`/`Plataforma` no produto), item ativo com texto/fundo `brand-500`/`brand-50`, colapsa pra só ícones com tooltip.
9. **Breadcrumb** — "Home > Nome da Página", alinhado à direita do título no topo do conteúdo. No produto: componente único `PageHeader` (`breadcrumb`+`title`+`action?`), raiz sempre "Início".

## Gráficos

Biblioteca de referência: **ApexCharts** (`react-apexcharts`, confirmado via DOM `.apexcharts-canvas`/`.apexcharts-svg`). Tipos no template TailAdmin e o que faz sentido pro nosso domínio:

| Tipo | Onde aparece no TailAdmin | Uso real na plataforma |
| --- | --- | --- |
| Bar | Vendas mensais (barras `brand-500`, cantos levemente arredondados, sem borda) | Atendimentos ou receita por mês (financeiro), novos tenants por mês |
| Área/linha | Métricas ao longo do tempo (2 séries sobrepostas, gradiente translúcido) | Baixa prioridade pro nosso caso |
| Radial bar (gauge) | Meta mensal/progresso (arco único, % grande no centro) | % de meta de faturamento do mês, % de ocupação da agenda |
| Donut/pie | Distribuição por categoria (paleta multi-cor, legenda com bolinha) | Distribuição de orçamentos por status, repasse por consultório, organizações por status |
| Radar | Comparação multi-eixo | Não inspecionado em detalhe, baixa prioridade |

## Estrutura de navegação do TailAdmin — o que reaproveitar, o que descartar

Site map completo do template inspecionado, listado pra deixar claro o que é
"produto genérico de SaaS/e-commerce de demonstração" (descartar) vs. o que
mapeia de verdade pro domínio odontológico (aproveitar):

- **Descartar**: dashboards por vertical (Ecommerce, Analytics, Marketing,
  CRM, Stocks, SaaS, Logistics — só mostram variedade do template), AI tools
  (geradores de texto/imagem/código/vídeo), Comércio (Produtos, Faturas,
  Transações), Authentication/Error pages/Layouts variantes (baixa
  prioridade).
- **Aproveitar direto**: Calendar e User Profile (mapeiam pra Agenda e um
  futuro "meu perfil" — hoje só existe "Minha Clínica"); Tables (Basic/Data —
  é exatamente o padrão que `admin-users-page.tsx`/`my-clinic-page.tsx`/
  `patients-page.tsx` já seguem); Charts Bar/Donut/Radial (ver tabela acima);
  UI Elements (Alerts, Badge, Buttons, Cards, Modals, Pagination, Dropdowns —
  praticamente o checklist do design system mínimo do produto).
- **Aproveitar só o padrão visual, não a página pronta**: Forms (já se usa
  React Hook Form + Zod, só falta alinhar o estilo); Task List/Kanban
  (inspira uma visão futura de orçamentos por status, não construído).

## Como isso virou trabalho real (2026-09-21/22)

Ordem em que os tokens/componentes entraram no produto, do maior pro menor
valor: (1) paleta de cores + tipografia como tokens reais no Tailwind config
do `apps/web` (re-hexado, tokens legados mantidos com nomes antigos apontando
pros valores oficiais, pra não quebrar todo o app de uma vez), (2) componente
`Badge` reutilizável pros status já existentes, (3) padrão de tabela aplicado
em `/admin/users`, `/my-clinic`, `/patients`, (4) Home Dashboard com stat
cards, (5) sidebar fixa global substituindo o menu horizontal, (6) Agenda
estilo Google Calendar (FullCalendar, cor por consultório/dentista via os
mesmos 5 tokens semânticos ciclados por índice — `paletteColorForIndex` em
`apps/web/src/features/agenda/location-colors.ts`, arbitrário/sem significado
de status, diferente do `Badge` semântico), (7) catálogo de Procedimentos.
Ver [[Roadmap]] pro estado de cada um e [[Arquitetura]] pro padrão técnico
"mock primeiro, backend depois" usado em todas essas telas.

## Gaps conhecidos (a capturar depois, se algum dia importar)

- `brand-300` (borda de foco) e `brand-600` (hover do botão primário)
  apareceram na extração sem valor hexadecimal exato — não foram incluídos
  como token pra evitar aproximação. Capturar via inspeção direta se algum
  dia a implementação chegar nesse nível de detalhe.
- `text-secondary`/`text-label` do tema escuro herdam o valor do tema claro,
  não auditados — não usar às cegas se/quando o produto ganhar dark mode de
  verdade.
- A sombra do botão primário (`shadow-theme-xs`) não teve valor CSS exato
  extraído — só a descrição "quase imperceptível".

## Fonte

Extraído por inspeção ao vivo de `https://react-demo.tailadmin.com/` em
2026-09-18, via Claude in Chrome — usado como referência de estilo e
estrutura, não como template a ser copiado literalmente. Publicado como
Artifact no Claude (`design-system-plataforma-odontologica.md` +
`design-system-plataforma-odontologica-tokens.json` são a versão exportada
pra uso direto pelo Claude Code, sem depender de acesso à URL do artifact).
