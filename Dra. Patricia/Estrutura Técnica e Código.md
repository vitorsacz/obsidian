---
tags: [tecnico, codigo, dra-patricia-vieira]
projeto: dra-patricia-vieira
atualizado: 2026-08-13
---

# Estrutura técnica e código

> [!info]
> Onde cada coisa vive no repositório `/Users/MAC/VITOR/dra-patricia-vieira`. Para o *porquê* das decisões, ver [[Design System — Clínica Patrícia Vieira]]; para a linha do tempo, ver [[Histórico de Decisões]].

## Stack

- **Framework:** Next.js 16 (App Router), Server Components por padrão
- **Linguagem:** TypeScript
- **Estilo:** Tailwind CSS v4 (`@theme inline` em `app/globals.css`)
- **Fontes:** self-hosted via `next/font/local` (não `next/font/google` — ver bug em [[Design System — Clínica Patrícia Vieira#10. Notas técnicas e armadilhas conhecidas]])
- **Imagens:** `next/image`
- **Processamento de imagem (build-time only):** `sharp` — usado num script único pra gerar o favicon e recortar a logo, não faz parte do runtime do site
- **Sem banco de dados, sem backend próprio, sem formulário** — 100% estático/SSR

## Como rodar localmente

```bash
cd /Users/MAC/VITOR/dra-patricia-vieira
npm install
npm run dev
```

Comandos de verificação usados durante todo o projeto:

```bash
npx tsc --noEmit   # type-check
npx eslint .       # lint
```

## Estrutura de pastas

```
dra-patricia-vieira/
  app/
    layout.tsx           # RootLayout — fontes, <Header/>, <Footer/>, <WhatsAppButton/>, JSON-LD
    page.tsx              # Home — página única com todas as seções âncora
    globals.css            # design tokens, classes de overlay (.hero-scrim, .photo-scrim...), marquee
    icon.png                # favicon moderno (gerado)
    apple-icon.png           # ícone iOS (gerado)
    favicon.ico               # favicon legado, multi-resolução (gerado)
    sitemap.ts                 # rotas: "/", "/faq"
    robots.ts
    fonts/                      # .woff/.woff2 self-hosted
    faq/
      page.tsx                   # única página de conteúdo além da Home
  components/
    Header.tsx              # nav fixo, transparente→vidro, logo dinâmica
    Footer.tsx               # colunas de navegação/contato + linha legal centralizada
    WhatsAppButton.tsx        # botão flutuante fixo, presente em 100% das páginas
    PhotoPlaceholder.tsx       # wrapper de foto com fallback gracioso + overlay opcional
    SpecialtyList.tsx           # lista numerada de especialidades com hover
    ui.tsx                       # PillLink, Eyebrow, SectionHeading (componentes pequenos reusáveis)
  lib/
    content.ts               # TODA a informação/copy do site (ver [[Conteúdo do Site]])
    assets.ts                 # assetExists() — fallback gracioso de fotos (ver abaixo)
  public/
    images/
      clinica/                  # fotos do ambiente
      tecnologia/                # fotos de equipamentos
      resultados/                  # casos antes/depois
      logos/                        # logo oficial (4 variantes + 2 recortadas)
      dra-patricia.png, equipe-vertical.png, fachada-predio.png
```

## Páginas / rotas

| Rota | Conteúdo |
|---|---|
| `/` | Home — Hero, `#clinica`, `#tecnologia`, `#especialidades`, `#equipe`, `#resultados`, `#avaliacoes`, CTA final, `#localizacao` |
| `/faq` | Perguntas frequentes (hoje só 1 pergunta com resposta real — ver [[Pendências de conteúdo]]) |
| ~~`/clinica` `/equipe` `/especialidades` `/resultados`~~ | **Removidas** — viraram âncoras da Home (ver [[Histórico de Decisões#Fase 4]]) |
| ~~`/contato`~~ | **Removida** — formulário eliminado, todo CTA vai pro WhatsApp (ver [[Histórico de Decisões#Fase 6]]) |

## Componentes — o que cada um faz

- **`Header`** (`"use client"`) — fixo no topo, calcula `transparent = isHome && scrollY <= 60` pra decidir entre fundo transparente (sobre o hero) e `.glass` (blur, ao rolar ou em qualquer página que não seja Home). Nav em grid 3 colunas (`grid-cols-[auto_1fr_auto]`) pra ficar centralizado. Logo troca de variante (branca/preta) junto com o estado.
- **`Footer`** — 3 colunas (identidade/logo, navegação, contato) + linha legal centralizada no rodapé do rodapé.
- **`WhatsAppButton`** — flutuante, `position: fixed`, `glass-dark`, sempre visível.
- **`PhotoPlaceholder`** — o componente mais reaproveitado do projeto. Recebe `variant` (gradiente de fallback: `ambiente`/`macro`/`prova`/`studio`), `src` opcional (se ausente, mostra o gradiente), `caption` opcional, `overlay`/`overlayStrength` opcional (escurece a foto pra legibilidade de texto por cima).
- **`SpecialtyList`** — lista numerada, hover via `group`/`group-hover:` puro CSS.
- **`ui.tsx`** — `PillLink` (3 variantes: `ghost`, `filled`, `ghost-light`), `Eyebrow`, `SectionHeading` (padrão repetido em toda seção: número + título + nota).

## Padrão `assetExists` — fallback gracioso de fotos

```ts
// lib/assets.ts
import fs from "node:fs";
import path from "node:path";

export function assetExists(publicPath: string): boolean {
  try {
    return fs.existsSync(path.join(process.cwd(), "public", publicPath));
  } catch {
    return false;
  }
}
```

Usado em todo Server Component pra checar, em tempo de request, se uma foto real já foi entregue — se não, `PhotoPlaceholder` mostra um gradiente elegante em vez de ícone quebrado. **Nunca importar isso num componente `"use client"`** (`node:fs` não roda no bundle do cliente).

Onde é usado hoje: fotos de tecnologia, foto da fundadora, foto da equipe, foto da fachada, e filtro do grid de Resultados (`resultCases.filter(item => assetExists(item.src))` — permite adicionar `caso-4.png` só soltando o arquivo, sem tocar em código).

## Scripts pontuais (não fazem parte do site, rodados uma vez)

Dois scripts Node temporários foram criados, rodados, e deletados durante o projeto (não existem mais no repo — documentados aqui caso precise recriar algo parecido pra outro cliente):

1. **Geração de favicon** — trim da logo-ícone preta + composição num quadrado com fundo `--color-bg-alt` em 3 tamanhos (512, 180, 16/32/48) + empacotamento manual de `.ico` real (header ICONDIR + PNG embutido, sem depender de lib externa de `.ico`).
2. **Trim das logos full** — `sharp().trim()` pra remover a margem transparente das logos `full-preta`/`full-branca` antes de usá-las no header/rodapé.

Ambos usaram `sharp` (instalado como dependência do projeto — também é a lib recomendada pelo Next.js pra otimização de imagem em produção self-hosted).
