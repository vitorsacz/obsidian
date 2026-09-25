---
tags: [projeto, site, yarth]
url: https://yarth.com.br
repo: /Users/MAC/VITOR/yarth
branch: feature/new-design-system
data: 2026-08-31
relacionado: "[[Yarth - Análise da Estrutura do Projeto]]"
---

# Yarth — yarth.com.br

> Auditoria refeita em 2026-08-31 a partir do **código-fonte local** (`/Users/MAC/VITOR/yarth`), não do site publicado — o repo é a fonte de verdade e o conteúdo real. A versão anterior desta nota (21/07) descrevia um site one-page com âncoras; o site evoluiu para uma **SPA multi-rota** (React Router) com home one-page + 4 páginas dedicadas. Esta nota substitui a anterior.

Site institucional da **Yarth**, empresa de soluções em ferro, alumínio e vidro (esquadrias de alumínio, vidraçaria, serralheria e mobiliário sob medida), sediada em Mairiporã (SP) com atendimento nacional.

- **Título da aba:** Yarth - Soluções personalizadas
- **Idioma declarado no HTML:** `en` `[não verificado se corrigido — o `<html lang="en">` está incorreto, todo o conteúdo é em pt-BR]`
- **Desenvolvido por:** Vitor Santos ([vitorsantos.dev.br](https://www.vitorsantos.dev.br))
- **Copyright:** © 2026 Yarth. Todos os direitos reservados

## Stack técnica

Vite 6 + React 19 + TypeScript + Tailwind CSS v4 + React Router v7 (rotas client-side) + Framer Motion (`motion/react`, animações de entrada/scroll) + Lucide Icons. Fontes via Google Fonts: **Inter** (corpo/`font-sans`) e **Montserrat** (títulos/`font-display`).

Conteúdo centralizado em `src/data/content.ts` (data-driven, sem textos hardcoded nos componentes).

## Contato / dados de negócio

| Canal | Valor |
|---|---|
| Email | contato@yarth.com.br |
| Telefone/WhatsApp | +55 (11) 96644-6215 |
| Endereço | Rua dos Cedros, 133, Mairiporã - SP |
| Instagram | [@grupoyarth](https://www.instagram.com/grupoyarth/) |
| Google (avaliações) | 4.9 de média, 130 avaliações |
| WhatsApp link (todos os CTAs) | `https://wa.me/5511966446215?text=Olá,%20vim%20pelo%20site%20de%20vocês.%20Tenho%20interesse%20em%20fazer%20um%20orçamento!` |

Todos os CTAs do site ("Solicitar Orçamento", "Saiba Mais", "Falar com um Consultor", "Falar com Especialista") apontam para o **mesmo link de WhatsApp** com a mesma mensagem pré-formatada — não há formulário de contato no site.

---

## Sitemap (rotas React Router)

| Rota | Página | Conteúdo |
|---|---|---|
| `/` | HomePage | Hero + About + Portfolio + Reviews + WhyUs (one-page com âncoras) |
| `/esquadrias` | ServicePage | Esquadrias de Alumínio |
| `/vidracaria` | ServicePage | Vidraçaria Completa |
| `/serralheria` | ServicePage | Serralheria Moderna |
| `/mobiliario` | FurniturePage | Mobiliário Personalizado |

Todas as rotas compartilham o mesmo `Layout`: Header fixo + `<Outlet>` + Footer + widget VLibras + modal Lightbox global.

---

## Componentes globais (presentes em todas as páginas)

### Header (fixo, topo)

- Logo Yarth (link para `/`), encolhe ao rolar a página (`scrollY > 50` → muda de transparente para fundo branco com sombra `glass-nav`).
- Nav desktop: **A Yarth** (`#about`) · **Serviços** (dropdown hover) · **Portfólio** (`#portfolio`) · **Avaliações** (`#reviews`) · **Diferenciais** (`#why-us`) · **Contato** (`#contact`).
- Dropdown "Serviços": Esquadrias de Alumínio, Vidraçaria Completa, Serralheria Moderna, Mobiliário → cada um leva à respectiva rota dedicada.
- Nos links de âncora, se a rota atual não for `/`, o href vira `/#âncora` (navega de volta pra home e rola).
- CTA fixo à direita: **"Falar com Especialista"** → WhatsApp (abre em nova aba).
- Mobile: menu hamburguer (ícone `Menu`/`X`), overlay com slide-down animado (Framer Motion); item "Serviços" vira accordion (expande/colapsa a lista de serviços); CTA WhatsApp com ícone no rodapé do menu.

### Footer (`id="contact"`)

- Coluna Email (`contato@yarth.com.br`, `mailto:`) + Endereço (Rua dos Cedros, 133, Mairiporã - SP).
- Telefone em destaque (`+55 (11) 96644-6215`, `tel:`), botões redondos Instagram e WhatsApp (ícone customizado `WhatsAppIcon`).
- Duas imagens lado a lado: foto da fachada da empresa (`fachada-empresa.webp`) + **iframe do Google Maps embed** (`google.com/maps?q=<endereço>&output=embed`, lazy-loaded, sem API key).
- Linha final: "© 2026 Yarth. Todos os direitos reservados" + "Desenvolvido por Vitor Santos" (link externo, nova aba).

### Lightbox (modal de galeria — global, disparado por qualquer card de portfólio)

- Overlay preto 95% opaco, fecha ao clicar fora ou no X.
- Navegação anterior/próxima (setas, `ChevronLeft`/`ChevronRight`) quando há mais de 1 imagem, com contador "X / N".
- Transições de entrada/saída com Framer Motion (fade + scale).

### VLibras (acessibilidade)

- Widget oficial do governo brasileiro (Libras — Língua Brasileira de Sinais), botão de acesso flutuante.
- Carrega script externo `https://vlibras.gov.br/app/vlibras-plugin.js` em runtime (iframe/embed de terceiro).

---

## Home (`/`) — seções em ordem

### 1. Hero (`id="home"`)

- Selo: "Soluções Personalizadas"
- **Headline:** "Soluções de Alto Padrão em **Ferro, Alumínio e Vidro.**"
- **Subheadline:** "Elegância e precisão técnica para projetos residenciais e corporativos de alto padrão em todo o Brasil."
- CTA: **"Solicitar Orçamento"** → WhatsApp
- Estatísticas: **20+** Anos de Experiência · **500+** Clientes Atendidos
- Painel visual à direita: imagem `g_6.webp` (alt "Modern Detail"), moldura decorativa com texto "YARTH" rotacionado, card sobreposto "Destaque — Fachadas em Structural Glazing"
- Animação: fade-in + slide-up no texto (0.8s), fade-in + scale no painel visual (1s)

### 2. A Yarth (`id="about"`)

- Bloco imagem + citação: imagem `g_4.webp` (alt "Quality Details") com card flutuante sobreposto: selo "Compromisso" + citação *"Transformamos ideias visionárias em obras de arte funcionais."*
- **Heading:** "A Yarth" / "Excelência técnica que define novos padrões."
- Texto institucional (2 parágrafos):
  1. "Com mais de uma década de experiência, a Yarth nasceu com um propósito claro: oferecer soluções completas e de alto padrão em esquadrias de alumínio, vidraçaria e serralheria."
  2. "Sediada em Mairiporã (SP), nossa operação abraça todo o Brasil, entregando excelência em obras residenciais, comerciais e corporativas, independentemente do tamanho do projeto."
- Estatística: **+1000** Projetos Executados
- **Painel full-bleed "Nossa Filosofia"** (imagem de fundo `g_6.webp` + overlay escuro 75%):
  - 2 parágrafos sobre união de tecnologia/design/experiência e acompanhamento técnico do 1º atendimento ao pós-venda.
  - **Nossa Essência** (card com blur):
    - **Missão:** "Transformar ideias em projetos reais, com máxima qualidade e confiança."
    - **Visão:** "Ser a grande referência nacional em soluções de alumínio, vidro e estruturas metálicas."
    - **Valores:** "Compromisso, inovação, respeito e excelência."
  - 3 cards de serviço sobrepostos ao painel (imagem + título + "Saiba Mais →"), cada um linkando para `/esquadrias`, `/vidracaria`, `/serralheria` — hover com zoom na imagem.

### 3. Portfólio (`id="portfolio"`)

- **Heading:** "Portfólio" / "Galeria de Projetos Realizados."
- Subtexto: "Uma seleção de nossas melhores entregas em residências e espaços corporativos."
- Grid de 12 categorias (2 col mobile / 4 col desktop), cada card = 1ª imagem da categoria + título revelado no hover (overlay gradiente escuro); clique abre o Lightbox com todas as imagens daquela categoria. Ver seção **"Galeria completa"** abaixo para a lista.
- Animação: fade + scale-in por card ao entrar no viewport (delay escalonado).

### 4. Avaliações (`id="reviews"`)

- **Heading:** "Avaliações" / "O que nossos clientes dizem."
- Bloco de resumo: ícone Google + nota **4.9** com 5 estrelas + "130 avaliações no Google".
- Carrossel **marquee automático infinito** (CSS `@keyframes marquee`, 45s linear, loop contínuo da lista duplicada), **pausa no hover**. 6 depoimentos reais (texto completo abaixo em "Avaliações — conteúdo completo").

### 5. Diferenciais (`id="why-us"`)

- Heading: "Por que escolher a Yarth?"
- 4 pilares em grid (ver tabela abaixo).
- Painel de conversão final: "Inicie seu projeto com quem entende." / "Atendimento especializado em Mairiporã e execução nacional." → CTA "Falar com um Consultor" (WhatsApp). Fade-in ao entrar no viewport.

---

## Páginas de Serviço — `/esquadrias`, `/vidracaria`, `/serralheria`

Template único (`ServicePageLayout`), reaproveitado pelas 3 rotas + pela página de Mobiliário. Estrutura, em ordem:

1. **Banner full-bleed** (imagem de capa do serviço + overlay escuro 70%): breadcrumb "Serviços / [Nome]", H1 com o título, tagline (mesma `description` curta usada nos cards da home), CTA "Solicitar Orçamento" (WhatsApp).
2. **Texto + destaques**: parágrafos explicativos (`longDescription`) à esquerda, grid de bullets (`highlights`) à direita, cada um com um traço decorativo.
3. **Galeria filtrada**: subconjunto de `PROJECT_GALLERY` cujas `areas` incluem a tag do serviço (`Esquadrias` / `Vidraçaria` / `Serralheria`) — mesmo componente de card + Lightbox do portfólio da home.
4. **CTA final**: mesmo bloco "Inicie seu projeto com quem entende." → "Falar com um Consultor".

### Conteúdo de cada serviço

**Esquadrias de Alumínio** (`slug: esquadrias`)
- Descrição curta: "Portas, janelas, fachadas e divisórias personalizadas com isolamento acústico e design minimalista."
- Parágrafos: "Desenvolvemos esquadrias de alumínio sob medida para projetos residenciais, comerciais e corporativos, unindo design minimalista, isolamento acústico e alta durabilidade." / "Da especificação técnica à instalação final, cada peça é pensada para se integrar perfeitamente à arquitetura do ambiente, com acabamento impecável e resistência às condições externas."
- Destaques: Portas e janelas de correr e abrir · Fachadas e divisórias personalizadas · Fechamento de sacadas · Vedação térmica e acústica

**Vidraçaria Completa** (`slug: vidracaria`)
- Descrição curta: "Boxes, guarda-corpos em vidro, espelhos e envidraçamento de sacadas com segurança e sofisticação."
- Parágrafos: "Trabalhamos com vidro temperado e laminado para criar soluções que unem segurança, transparência e sofisticação em cada ambiente." / "De boxes e espelhos a guarda-corpos e fachadas envidraçadas, cada projeto é executado com precisão técnica e acabamento de alto padrão."
- Destaques: Boxes de banheiro sob medida · Guarda-corpos e sacadas envidraçadas · Espelhos e escadas de vidro · Fachadas e coberturas em vidro

**Serralheria Moderna** (`slug: serralheria`)
- Descrição curta: "Portões automáticos, estruturas metálicas sob medida e corrimãos com acabamento impecável."
- Parágrafos: "Projetamos e executamos estruturas metálicas sob medida — de portões automáticos a corrimãos e coberturas — com acabamento impecável e resistência para uso residencial e comercial." / "Nossa equipe acompanha cada etapa da obra, da fabricação à instalação, garantindo segurança e durabilidade em cada estrutura entregue."
- Destaques: Portões automáticos e gradis · Guarda-corpos e corrimãos metálicos · Estruturas e coberturas sob medida · Acabamento e pintura eletrostática

---

## Página Mobiliário (`/mobiliario`)

Mesmo template `ServicePageLayout`, sem breadcrumb de "Serviços" separado na Header (acessível só pelo dropdown "Serviços" ou link direto).

- **Título:** "Mobiliário Personalizado"
- **Tagline:** "Peças exclusivas em vidro, ferro, alumínio e madeira, desenvolvidas sob medida para cada ambiente."
- **Introdução (2 parágrafos):** "Desenvolvemos peças exclusivas que unem a leveza do vidro à resistência do alumínio e do ferro — adegas, cristaleiras, estantes e detalhes que transformam ambientes comuns em espaços extraordinários." / "Cada peça é projetada sob medida, com acabamento de alta fixação e integração de iluminação LED, complementando os projetos de esquadrias, vidraçaria e serralheria da Yarth."
- **Destaques:** Design Sob Medida · Acabamentos de Alta Fixação · Integração com Iluminação LED
- **Galeria (8 itens, cada um vira 1 card com 1 imagem):** Design de Interiores · Mobiliário Exclusivo · Cristaleiras e Adegas · Estruturas Minimalistas · Conceito Yarth · Estanteria Premium · Detalhes Técnicos · Ambientes Integrados

---

## Galeria completa do Portfólio (12 categorias, `PROJECT_GALLERY`)

| Categoria | Nº de imagens | Áreas associadas | Aparece em |
|---|---|---|---|
| Portas | 6 | Esquadrias, Vidraçaria | Home, `/esquadrias`, `/vidracaria` |
| Guarda-Corpo | 6 | Esquadrias, Vidraçaria, Serralheria | Home, todos os 3 serviços |
| Fechamento de Sacada | 2 | Esquadrias, Vidraçaria | Home, `/esquadrias`, `/vidracaria` |
| Telhado de Vidro | 3 | Esquadrias, Vidraçaria | Home, `/esquadrias`, `/vidracaria` |
| Box de Vidro | 6 | Vidraçaria, Serralheria | Home, `/vidracaria`, `/serralheria` |
| Janela Integrada | 1 | Esquadrias | Home, `/esquadrias` |
| Escada de Vidro | 2 | Vidraçaria | Home, `/vidracaria` |
| Espelhos | 5 | Vidraçaria | Home, `/vidracaria` |
| Fachada de Vidro | 3 | Vidraçaria | Home, `/vidracaria` |
| Estruturas Metálicas | 3 | Serralheria | Home, `/serralheria` |
| Mobiliário | 9 | Mobiliário | Home (não aparece em nenhuma página de serviço — tag não bate com nenhum `areaTag`) |
| Instalação | 6 | Esquadrias, Vidraçaria, Serralheria | Home, todos os 3 serviços |

Total: **52 imagens de projeto** distintas em `src/assets/portfolio/` + `src/assets/m_1..8.svg` (linha do mobiliário), todas `.webp` exceto a linha de mobiliário (`.svg`).

---

## Avaliações — conteúdo completo (`GOOGLE_REVIEWS`)

| Nome | Iniciais | Nota | Data relativa | Texto |
|---|---|---|---|---|
| Marina Ferreira | MF | ★★★★★ | há 2 semanas | "Equipe extremamente profissional, o acabamento das esquadrias ficou impecável. Recomendo muito a Yarth!" |
| Roberto Almeida | RA | ★★★★★ | há 1 mês | "Contratei a Yarth para o box do banheiro e ficou show. Prazo cumprido à risca e time muito atencioso." |
| Camila Souza | CS | ★★★★★ | há 3 meses | "Serviço de serralheria excelente, o portão automático funciona perfeitamente até hoje. Nota 10." |
| Eduardo Lima | EL | ★★★★☆ | há 4 meses | "Bom atendimento e material de qualidade, só demorou um pouco além do combinado. No mais, sem reclamações." |
| Patrícia Nogueira | PN | ★★★★★ | há 5 meses | "A vidraçaria da minha sacada ficou linda, superou minhas expectativas. Equipe muito educada e limpa no serviço." |
| Fernando Costa | FC | ★★★★★ | há 6 meses | "Profissionalismo do início ao fim. Já fechei outros dois projetos com eles e a qualidade se mantém sempre alta." |

`[não verificado]` se as datas relativas ("há 2 semanas" etc.) são estáticas (hardcoded, não recalculadas) — no código são strings fixas em `content.ts`, não calculadas a partir de uma data real.

## Diferenciais — 4 pilares (`WHY_US`)

| Título | Descrição |
|---|---|
| Atendimento Personalizado | Soluções sob medida para a sua necessidade. |
| Qualidade Premium | Uso exclusivo de materiais de altíssima qualidade. |
| Expertise | Equipe técnica altamente especializada. |
| Compromisso | Entregas rigorosamente dentro do prazo. |

---

## Animações e interações (resumo)

- **Scroll do Header:** transparente/grande → fundo branco com sombra + logo menor, ao passar de 50px de scroll.
- **Reveal ao entrar no viewport** (Framer Motion `whileInView`, `once: true`): cards de Serviços, cards de Portfólio, painel final de Diferenciais — fade + leve translação/escala, com delay escalonado por índice.
- **Hover:** cards de serviço e de portfólio dão zoom na imagem (scale 1.1) via `transition-transform`; cards de Serviços (não usados na home, ver observação abaixo) tinham efeito grayscale→cor no hover.
- **Marquee de avaliações:** loop horizontal infinito automático, pausa ao passar o mouse.
- **Menu mobile:** slide-down com fade (Framer Motion `AnimatePresence`); accordion de "Serviços" expande/colapsa altura com fade.
- **Lightbox:** fade in/out do overlay + scale in/out da imagem ativa a cada troca.

## Formulários

Não há formulário de contato em nenhuma página. Todo o funil de conversão é via link `wa.me` (WhatsApp) com mensagem pré-preenchida idêntica em todos os CTAs.

## Iframes / embeds de terceiros

| Onde | O quê | Propósito |
|---|---|---|
| Footer (todas as páginas) | `iframe` Google Maps (`google.com/maps?q=...&output=embed`) | Mostrar localização em Mairiporã-SP |
| Layout global (todas as páginas) | Script `vlibras.gov.br/app/vlibras-plugin.js` + widget | Tradução em Libras (acessibilidade) |

---

## Observações técnicas / achados da auditoria

- **`src/components/Services.tsx` é código morto** — não é importado por nenhuma página (`HomePage` usa Hero/About/Portfolio/Reviews/WhyUs, sem `Services`). O conteúdo de "serviços" que aparece de fato na home vem dos 3 cards dentro de `About.tsx`. Ao recriar o site, decidir se esse componente (grid de 3 serviços com imagem grayscale→cor no hover) deve ser resgatado como seção própria ou descartado.
- **`<html lang="en">`** em `index.html`, mas todo o conteúdo é em português — inconsistência a corrigir no redesign.
- **Pastas de assets brutos não referenciadas**: `src/assets/Esquadrias em aluminio/`, `src/assets/Serralheria/`, `src/assets/Vidraçaria/`, `src/assets/Fotos equipe/`, `src/assets/moveis/` (SVGs) não são importadas em nenhum lugar do código — parecem ser originais/rascunhos não utilizados na versão atual do conteúdo.
- Repositório do projeto: `/Users/MAC/VITOR/yarth`, branch atual `feature/new-design-system` (branch onde este redesign está sendo desenvolvido).
- Hospedagem/domínio de produção: `[não verificado]` — a nota anterior indicava `yarth.vercel.app`, mas o `url` de referência mais recente é `yarth.com.br`; não confirmado neste levantamento por não ter sido feita navegação ao vivo.
