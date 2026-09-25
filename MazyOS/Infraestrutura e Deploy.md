#projeto #mazyos #infraestrutura

Ver [[Arquitetura]] para onde essas peças entram na estrutura de pastas.

## Não é um serviço rodando — é local + GitHub

MazyOS não tem servidor, banco de dados ou deploy próprio. "Deploy" aqui
significa: os arquivos que as skills geram (posts, relatórios, CSVs) ficam
no disco local e vão pro GitHub via `/salvar`. O único lugar onde MazyOS
*dispara* um deploy de verdade é indireto — via `/aprovar-post`, que dá
push num repo de **site/blog externo** (Netlify/Vercel deployam esse push
automaticamente).

## Scripts (`scripts/`) — nascem vazios, sob demanda

A pasta vem vazia por design. Quando uma skill precisa de um script que não
existe, o Claude detecta a ausência, pergunta se quer configurar agora, guia
o setup de chaves de API, cria o script e roda a skill — nada é pré-instalado.

| Script esperado | Skill que usa | O que faz |
|---|---|---|
| `gerar-imagem.js` | `/carrossel` (com foto IA) | Gera foto realista via OpenAI DALL-E 3 |
| `render.js` (por pasta de conteúdo) | `/carrossel` | Playwright → screenshot 1080×1350 de cada slide HTML |
| `postar-instagram.js` | `/aprovar-post` | Publica carrossel no Instagram via Meta Graph API |
| `postar-facebook.js` | `/aprovar-post` | Publica carrossel no Facebook via Meta Graph API |

## Variáveis de ambiente (`.env` na raiz, não versionado)

```bash
OPENAI_API_KEY=sk-...               # gerar-imagem.js
META_PAGE_ACCESS_TOKEN=...          # postar-instagram.js + postar-facebook.js
META_PAGE_ID=...
META_IG_USER_ID=...
SITE_URL=https://seudominio.com.br
```

Pré-requisitos de máquina: **Node.js 20+** e **Playwright**
(`npm install playwright && npx playwright install chromium`).

## Ferramentas/APIs catalogadas (`templates/ferramentas/catalogo.md`)

Referência de tudo que uma skill nova pode usar, nem tudo ativo hoje:

- **Playwright CLI** — HTML → PNG, local, sem conta (`npx playwright screenshot
  --viewport-size=1080,1350 ...`). Base do `/carrossel`.
- **Cloudflare Pages API** — publica HTML com link público (propostas,
  landing pages). Precisa `CLOUDFLARE_API_TOKEN` + `CLOUDFLARE_ACCOUNT_ID`.
- **Post for Me API** (postforme.dev) — alternativa de publicação em
  Instagram/TikTok, via `POSTFORME_API_KEY`.
- **Jina Reader** (`https://r.jina.ai/{URL}`) — URL → markdown limpo, melhor
  que WebFetch pra artigos longos.
- **yt-dlp** — transcrição de vídeo do YouTube (`brew install yt-dlp`).
- **Gemini / DALL-E** — geração de imagem por texto (`GEMINI_API_KEY` /
  `OPENAI_API_KEY`).

### MCPs (conectores) sugeridos, instaláveis via `claude mcp add`
Notion, Gmail, Google Calendar, Canva, Facebook Ads (Meta), Google Ads,
N8N, Supabase, Telegram. Nenhum confirmado como já instalado neste clone —
checar com `claude mcp list`.

## Skills externas prontas (`templates/skills/catalogo.md`)

Catálogo de skills não-nativas do MazyOS que podem ser instaladas conforme
necessidade: **Schwartz Copy** (`/schwartz-copy`, copy de resposta direta) e
**Ogilvy Copy** (`/ogilvy-copy`, copy institucional/branding) — descritas
como "já vêm como skill global" no catálogo, mas isso é sobre a instalação
*pretendida*, não confirma que já estão em `~/.claude/skills/` nesta máquina.
As demais do catálogo (Frontend Design, Canvas Design, PDF/DOCX/PPTX/XLSX,
Doc Co-Authoring, YT Transcript, Webapp Testing, Skill Creator) são nativas
do Claude Code ou dependem só de `yt-dlp`.
