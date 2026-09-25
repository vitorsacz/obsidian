---
tags: [projeto, yarth, code-review, opiniao]
data: 2026-07-26
relacionado: "[[Yarth - Site]]"
---

# Yarth — Opinião sobre a estrutura do projeto

Análise do repositório local `/Users/MAC/VITOR/yarth` (branch `feat/adjust-to-deploy`), feita a pedido do Vitor após rodar o lint (`tsc --noEmit`, sem erros).

## Veredito geral

Estrutura **funcional, mas melhorável**. Para um one-pager de marketing ela cumpre o papel, porém tem sinais de scaffold não limpo e falta de modularização.

## Pontos positivos

- **Conteúdo data-driven**: `NAV_LINKS`, `SERVICES`, `FEATURES`, `GALLERY`, `FURNITURE_GALLERY` são arrays declarados no topo de `App.tsx` e renderizados via `.map()` — não é JSX repetido manualmente.
- **Stack moderna e adequada**: Vite + React 19 + TypeScript + Tailwind CSS v4 + Framer Motion (`motion/react`) + Lucide icons.
- `tsc --noEmit` passa sem erros — tipagem consistente.
- Assets (imagens `.webp`, logos e ícones `.svg`) organizados em `src/assets/`.

## Pontos a melhorar

1. **Monolito em `App.tsx` (627 linhas)**
   Navbar, Hero, About, Services, Furniture, Portfolio, WhyUs, Footer e o Lightbox modal estão todos no mesmo componente/arquivo.
   → Recomendação: quebrar em `src/components/` (um arquivo por seção: `Navbar.tsx`, `Hero.tsx`, `About.tsx`, `Services.tsx`, `Furniture.tsx`, `Portfolio.tsx`, `WhyUs.tsx`, `Footer.tsx`, `Lightbox.tsx`).
   Isso é inclusive o que o próprio `README.md` já **descreve** na seção "Estrutura do Projeto" — mas a pasta `components/` não existe de fato. É deriva de documentação (doc diz uma coisa, código é outra).

2. **Dependências mortas (leftover de scaffold AI Studio)**
   `@google/genai`, `express`, `dotenv`, `tsx`, `@types/express` estão no `package.json` mas nenhuma é referenciada em `src/`. `metadata.json` ainda declara `"majorCapabilities": ["MAJOR_CAPABILITY_SERVER_SIDE_GEMINI_API"]`, sem uso real no código.
   → Infla o `npm install` (213 pacotes) e confunde quem entra no projeto pensando que há integração com IA/servidor.

3. **`.env.example` fora de contexto**
   Pede `GEMINI_API_KEY` e `APP_URL`, herdados do template do AI Studio — não se aplicam a este site estático.

4. **`package.json` → `"name": "react-example"`**
   Deveria ser `"yarth"` (cosmético, mas incoerente com o projeto real).

5. **Lint é só type-check**
   O script `lint` roda `tsc --noEmit`, que verifica tipos mas não é ESLint de verdade — não há checagem de regras de hooks (`react-hooks/exhaustive-deps`), acessibilidade (`jsx-a11y`), variáveis não usadas, etc.

## Prioridade sugerida (se formos mexer)

1. Remover dependências mortas + limpar `metadata.json`/`.env.example` (baixo risco, ganho imediato).
2. Quebrar `App.tsx` em componentes por seção (melhora manutenibilidade, sem mudar comportamento).
3. Corrigir `package.json` name.
4. Adicionar ESLint real, se o projeto for crescer.

Nada disso é bloqueante — o site funciona e builda normalmente. São melhorias de manutenibilidade, não bugs.

## Status — 2026-07-26

Todos os pontos foram aplicados na branch `feat/adjust-to-deploy`:

- **Commit `398bf08`**: removidas as dependências mortas (`@google/genai`, `express`, `dotenv`, `tsx`, `@types/express`), o `define` do Gemini em `vite.config.ts`, a capability `SERVER_SIDE_GEMINI_API` do `metadata.json` e o `.env.example` obsoleto. `package.json` renomeado para `"yarth"`. Adicionado ESLint real (`eslint.config.js` com `typescript-eslint` + `react-hooks` + `react-refresh`), o que revelou e permitiu limpar imports/constantes mortos em `App.tsx` (`Phone`, `Mail`, `MapPin`, `ShieldCheck`, `Clock`, `Gem`, `logoBranco`, `FEATURES`).
- **Commit `1d5af0a`**: `App.tsx` (627 linhas) quebrado em `src/components/` (`Header`, `Hero`, `About`, `Services`, `Furniture`, `Portfolio`, `WhyUs`, `Footer`, `Lightbox`, `WhatsAppIcon`) + `src/data/content.ts` com todo o conteúdo estático (nav, serviços, galerias). `App.tsx` agora só compõe as seções e guarda o estado do Lightbox (28 linhas). Validado com `npm run lint`, `npm run build` e teste manual no navegador (navegação, lightbox, galeria de mobiliário).

Estrutura do projeto agora corresponde ao que o `README.md` já descrevia.
