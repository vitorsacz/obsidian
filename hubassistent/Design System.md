#projeto #hubassistent #design

Ver [[Visão Geral]] para contexto. Ver [[Arquitetura]] para onde isso vive no código (`apps/web`).

## Como chegamos aqui

O visual inicial (Tailwind padrão: fundo `slate-950`, botão `indigo-600`) foi identificado como genérico/"cara de IA" antes de continuar com features novas. Processo:
1. 3 direções visuais mostradas via mockup (Artifact): **Ledger** (editorial/precisão), **Sol** (quente/acolhedor), **Signal** (ousado/marinho-latão).
2. Vitor escolheu **Ledger**, mas pediu foco em azul/branco/preto em vez do verde-petróleo original.
3. 3 variações de azul mostradas: Cobalt, Ink & Azure, **Navy Duotone**.
4. Escolhida: **Navy Duotone** — aplicada em todo o app em 2026-07-21.

## Conceito "Ledger" (linguagem estrutural — manter em telas novas)

Precisão de um livro-razão bem escriturado: hairlines em vez de sombra/cards muito arredondados, números alinhados à direita, rótulos em caixa alta com tracking, uso de cor restrito (gastos normais ficam na tinta neutra, não em vermelho — vermelho/`bad` é reservado pra alertas reais, verde/`good` é só pra entradas).

## Paleta Navy Duotone

| Token | Claro | Escuro |
|---|---|---|
| `--color-bg` (fundo do app) | `#F3F5F8` | `#0A0F1C` |
| `--color-surface` (cards) | `#FFFFFF` | `#101728` |
| `--color-ink` (texto principal) | `#101E3D` (marinho) | `#E8ECF5` |
| `--color-ink-muted` (texto secundário) | `#5C6B85` | `#8790A8` |
| `--color-accent` (cobalto) | `#3563E9` | `#6E8FFF` |
| `--color-accent-soft` | `#E3E9FB` | `#17203A` |
| `--color-line` (hairlines) | `#DEE4EF` | `#1C2438` |
| `--color-good` | `#227A52` | `#4FBE85` |
| `--color-bad` | `#B23B3B` | `#E08A73` |

A tinta (`ink`) do texto É o azul-marinho, não preto — essa é a assinatura do "Duotone": duas tonalidades de azul (marinho profundo pra texto, cobalto pra ação/destaque) fazendo o trabalho que preto+um-acento faria em outro sistema.

## Tipografia

- **Instrument Serif** (itálico) — números em destaque (saldo, valores de transação) e títulos de seção. Hospedada localmente via `@fontsource/instrument-serif` (não CDN do Google — evita CSP/dependência externa).
- Sans do sistema — corpo de texto, labels, UI geral.
- `font-variant-numeric: tabular-nums` (classe utilitária `.tabular`) em qualquer coluna de números.

## Tema claro/escuro

Implementado de verdade (não só `prefers-color-scheme`): `ThemeProvider` em `apps/web/src/lib/theme-context.tsx`, toggle em `apps/web/src/components/theme-toggle.tsx`, persistido em `localStorage` (chave `hub-theme`), padrão inicial = preferência do sistema. Tailwind configurado com `darkMode: 'class'`.

## Onde aplicar em telas novas

Reusar os tokens Tailwind já existentes — `bg-app`, `bg-surface`, `text-ink`, `text-muted`, `border-line`, `bg-accent` / `text-accent`, `bg-accent-soft`, `text-good` / `text-bad`, `font-display italic` (Instrument Serif). Não inventar cor nova sem necessidade — checar `apps/web/tailwind.config.js` e `index.css` primeiro.
