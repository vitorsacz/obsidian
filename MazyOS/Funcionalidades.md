#projeto #mazyos #funcionalidades

Ver [[Visão Geral]] para contexto e [[Arquitetura]] para como as skills se
conectam à memória/identidade. As 15 skills vivem em `.claude/skills/<nome>/SKILL.md`.

## Núcleo — jeito de operar o dia a dia

- **`/abrir`** — carrega a memória (empresa, preferências, estratégia,
  identidade) e devolve um resumo de uma frase pra começar a sessão.
- **`/salvar`** — commit + push no GitHub. Configura o remote na primeira vez.
  Pensada pra quem nunca usou git.
- **`/atualizar`** — varre o workspace e propõe atualizações pros arquivos de
  contexto (`_memoria/*`, `CLAUDE.md`, `design-guide.md`) que ficaram
  desatualizados em relação ao estado real.
- **`/novo-projeto`** — entrevista curta (cliente, objetivo, entregas) e cria
  pasta isolada com `CLAUDE.md` próprio que herda contexto da raiz.
- **`/mapear-rotinas`** — entrevista sobre o que o usuário repete toda semana
  e cria skills personalizadas novas em `.claude/skills/` pras aprovadas.

## Conteúdo e SEO — vitrine pública

- **`/carrossel`** — HTML estilizado → PNG 1080×1350 via Playwright, com
  identidade da marca. Suporta texto puro, foto gerada por IA (DALL-E via
  OpenAI), ou post único. Formato mudou de 1080×1080 pra 1080×1350 (4:5,
  padrão Instagram atual) num commit recente do repo.
- **`/publicar-tema`** — orquestradora: pega um tema → artigo de blog
  completo + chama `/carrossel` + gera 3 legendas (Instagram, Facebook,
  LinkedIn), tudo amarrado (carrossel aponta pro blog).
- **`/seo`** — fluxo de 8 passos: pesquisa de demanda → concorrência →
  Google Meu Negócio → on-page → estratégia de conteúdo → Google Ads →
  checklist de monitoramento → GEO (otimização pra aparecer em respostas de
  ChatGPT/Gemini/Perplexity). Preenche `marketing/seo/01..08-*.md`.
- **`/responder-avaliacoes`** — respostas curtas e humanas pra reviews do
  Google Meu Negócio (nome do cliente, agradecimento variado, frase
  concreta sobre o serviço — deliberadamente **não** soa como resposta
  automática de empresa grande).
- **`/aprovar-post`** — fecha o loop: pega um post já criado por
  `/publicar-tema`, muda status de draft→published no blog, copia os PNGs
  pro `public/` do site, faz commit+push (dispara deploy Netlify/Vercel),
  espera o deploy, e publica o carrossel no Instagram + Facebook via Meta
  Graph API.

## Anúncios pagos — onde o dinheiro entra

- **`/anuncio-google`** — monta campanha Search inteira (grupos de
  anúncios, RSAs, extensões, negativas) direto em CSV pronto pra importar
  no Google Ads Editor. Lê o briefing de `_memoria/empresa.md` e da
  pesquisa SEO se existir — pula a montagem manual grupo-por-grupo na UI.
- **`/relatorio-ads`** — lê exports de CSV do Google Ads + Meta Ads (ou
  prints) e devolve relatório semanal: KPIs, top criativos, alertas
  (queima de orçamento, CTR baixo, conversão caindo), recomendações.

## Produção — ferramentas do dia a dia

- **`/analisar-dados`** — lê CSV/XLSX/PDF/TXT/JSON solto em `dados/` e
  gera resumo executivo com insights e recomendações.
- **`/email-profissional`** — rascunha email a partir de contexto livre,
  calibrando tom ao destinatário e objetivo.

## Padrão comum entre as skills

Toda skill que produz algo declara de onde lê contexto ("Dependências:
`_memoria/empresa.md`, `_memoria/preferencias.md`") e pra onde escreve —
`marketing/conteudo/`, `marketing/seo/`, `marketing/campanhas/`,
`saidas/analises/`, etc. — então o usuário nunca precisa decidir onde
salvar; a skill já sabe (ver `marketing/README.md` e `saidas/README.md`
pra estrutura completa de destino).
