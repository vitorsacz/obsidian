---
tags: [assets, inventario, dra-patricia-vieira]
projeto: dra-patricia-vieira
atualizado: 2026-08-13
---

# Inventário de assets

> [!info]
> Todo arquivo de imagem/fonte usado no site, o que ele mostra e onde aparece. Caminhos relativos a `/Users/MAC/VITOR/dra-patricia-vieira`. Para o raciocínio de *por que* cada registro fotográfico existe, ver [[Design System — Clínica Patrícia Vieira#4. Direção fotográfica — os 3 registros]].

## Logos (`public/images/logos/`)

| Arquivo | Conteúdo | Usado em |
|---|---|---|
| `logo-patricia-preta.png` | Só o ícone (monograma "PV"), traço preto, fundo transparente | Disponível, não usado diretamente hoje (o site usa a versão "full") |
| `logo-patricia-branca.png` | Só o ícone, traço branco | Idem |
| `logo-patricia-full-preta.png` | Ícone + "Clínica Patrícia Vieira" por extenso, preto — **arquivo original**, conteúdo ocupa só ~35% da altura do canvas | Fonte pro recorte abaixo |
| `logo-patricia-full-branca.png` | Idem, branco | Fonte pro recorte abaixo |
| `logo-patricia-full-preta-trim.png` | **Versão em uso** — recorte sem a margem transparente (1157×337px) | `Header.tsx` (estado "vidro"), `Footer.tsx` |
| `logo-patricia-full-branca-trim.png` | **Versão em uso** — recorte (1159×338px) | `Header.tsx` (estado transparente, sobre o hero) |

## Favicon (`app/`, gerados a partir da logo-ícone preta)

| Arquivo | Tamanho | Uso |
|---|---|---|
| `icon.png` | 512×512 | Favicon moderno (convenção Next.js App Router) |
| `apple-icon.png` | 180×180, fundo sólido | Ícone de tela inicial iOS |
| `favicon.ico` | 16/32/48px, multi-resolução real | Favicon legado |

Todos com fundo `--color-bg-alt` (`#FAF8F5`) — nunca transparente (some em abas claras e escuras).

## Fotos do ambiente (`public/images/clinica/`)

| Arquivo | Mostra | Usado em |
|---|---|---|
| `recepcao-1.jpg` | Recepção — sofá curvo, corredor, obra de arte na parede | Grid da seção "A Clínica" (legenda "Recepção") |
| `recepcao-2.jpg` | Recepção — balcão de mármore, letreiro luminoso, logo na parede | Fundo do Hero (`object-position: center 30%`) |
| `consultorio-1.jpg` | Sala interna, poltronas, mesa | CTA final ("Vamos conversar"), com overlay escuro forte |
| `consultorio-2.jpg` | Sala de atendimento com vista da cidade | Grid da seção "A Clínica" (legenda "Sala de atendimento") |
| `consultorio-3.jpg` | Consultório — cadeira odontológica, bancada de mármore | Grid da seção "A Clínica" (legenda "Consultório") |

## Fotos de tecnologia (`public/images/tecnologia/`)

| Arquivo | Mostra | Legenda no site |
|---|---|---|
| `tecnologia-scanner.png` | Dra. Patricia segurando tablet com foto de sorriso, tela de scanner intraoral ao fundo | "Scanner intraoral" |
| `tecnologia-doutora.png` | Dra. Patricia ao lado do carrinho do scanner iTero e de um aparelho a laser | "Equipamentos" |
| `ceramica.png` | Close em facetas/coroas de cerâmica sobre um modelo dental | "Precisão" |

## Equipe e fundadora (`public/images/`)

| Arquivo | Mostra | Status |
|---|---|---|
| `dra-patricia.png` | Dra. Patricia segurando uma câmera fotográfica, escritório ao fundo | **Em uso** — card da fundadora, seção Equipe |
| `equipe-clinica.png` | 5 mulheres em pé, roupa preta, foto de grupo **horizontal** (corta 2 pessoas em proporção 4:5) | Substituída — não usada mais |
| `equipe-vertical.png` | Mesmo grupo, mesma roupa, enquadramento **vertical/corpo inteiro** | **Em uso** — card da equipe, proporção 3:4, mostra todo mundo |

## Fachada (`public/images/`)

| Arquivo | Mostra | Usado em |
|---|---|---|
| `fachada-predio.png` | Prédio comercial moderno, entrada com destaque vermelho, palmeiras | Seção Localização, ao lado do mapa |

## Resultados / antes-depois (`public/images/resultados/`)

| Arquivo | Status |
|---|---|
| `caso-1.png` | Em uso — formato pronto do fotógrafo (topo = antes, base = depois, marca d'água "PV") |
| `caso-2.png` | Em uso |
| `caso-3.png` | Em uso |
| `caso-4.png` | **Referenciado no código, arquivo ainda não entregue** — aparece sozinho assim que chegar (ver [[Pendências de conteúdo]]) |

## Fontes (`app/fonts/`)

| Arquivo | Família | Peso/estilo | Papel |
|---|---|---|---|
| `montserrat-700.woff` | Montserrat | 700 | Títulos (display) |
| `montserrat-500.woff` | Montserrat | 500 | Labels, navegação, botões |
| `playfair-display-italic.woff` | Playfair Display | 400 itálico | Acentos humanos dentro de títulos |
| `inter-variable.woff2` | Inter | 300–600 (variável) | Corpo de texto |

Todas self-hosted via `next/font/local` — ver motivo em [[Design System — Clínica Patrícia Vieira#3.2 Tipografia]].
