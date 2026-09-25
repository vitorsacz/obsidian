---
tags: [design-system, template, odontologia, next-js, tailwind]
projeto: dra-patricia-vieira
stack: "Next.js 16 (App Router) + TypeScript + Tailwind CSS v4"
criado: 2026-08-13
status: base reutilizável
---

# Design System — Clínica Patrícia Vieira

> [!tip] Navegação do vault
> Esta é uma de seis notas — ver o mapa completo em [[Índice]]. Relacionadas: [[Histórico de Decisões]] · [[Estrutura Técnica e Código]] · [[Inventário de Assets]] · [[Conteúdo do Site]] · [[Pendências de conteúdo]]

> [!abstract] Sobre este documento
> Este arquivo documenta **tudo** o que foi decidido e construído para o site institucional da Clínica Patrícia Vieira (Atibaia/SP): identidade visual, estrutura, comportamento, tom de voz e as razões humanas/psicológicas por trás de cada escolha — não só o "o quê", mas o "porquê".
>
> **Ele foi escrito para ser reaproveitado.** A ideia é que, para o próximo cliente parecido (clínica odontológica, esteticista, profissional de saúde premium), a gente reabra este arquivo, troque os pontos marcados como `🔁 TROCAR` (cores, nome, endereço, fotos, depoimentos) e mantenha tudo o que está marcado como `🧱 ESTRUTURA` (padrões de layout, comportamento, raciocínio de design) — porque é isso que faz o site *parecer premium e pensado*, não a cor específica do dourado.

---

## 0. Resumo executivo

| | |
|---|---|
| **Cliente** | Clínica Patrícia Vieira — Dra. Patricia Vieira, Odontologia Estética |
| **Cidade** | Atibaia, SP |
| **Origem da marca** | Extraída de dois perfis reais do Instagram: [@drapatriciavieira](https://www.instagram.com/drapatriciavieira) (pessoal, 15,3 mil seguidores) e [@clinicapatriciavieira](https://www.instagram.com/clinicapatriciavieira) (institucional) |
| **Posicionamento** | "Realçando sorrisos com naturalidade e delicadeza" / "A clínica mais linda de Atibaia" |
| **Direção visual** | Editorial, arquitetural, *light mode* de alto padrão — luxo silencioso, não gritado |
| **Stack** | Next.js 16 (App Router) · TypeScript · Tailwind CSS v4 · fontes self-hosted via `next/font/local` |
| **Repositório** | `/Users/MAC/VITOR/dra-patricia-vieira` |
| **Conversão** | 100% WhatsApp — sem formulário de contato, uma única mensagem padrão em todos os botões |

---

## 1. Contexto e origem da marca

O ponto de partida **não foi um briefing em branco** — foi uma auditoria de identidade visual em dois perfis reais do Instagram que já tinham tração:

- **@drapatriciavieira** — a marca pessoal da médica. Autoridade + humanização: ela mistura credencial técnica (CROSP, "Odontologia Estética") com rosto e vida pessoal. Isso é o que gera confiança sem parecer distante.
- **@clinicapatriciavieira** — o perfil institucional, que revelou que a clínica é **multiprofissional** (Dra. Patricia + outras dentistas), não um consultório solo. Isso mudou o enquadramento do site: de "site da Patricia" para "site da clínica, com a Patricia como fundadora e rosto principal".

**Achados que viraram regra de design:**
- Prova social forte via antes/depois em close, sem moldura, com marca d'água discreta "PV"
- Paleta bege / branco / dourado / preto já estabelecida nos destaques e no logo
- Tipografia serifada elegante misturada com sans-serif limpa
- Funil de conversão já validado: WhatsApp, sem fricção

> [!tip] 🧱 ESTRUTURA — princípio geral
> Sempre que possível, **audite o Instagram/redes existentes do cliente antes de desenhar algo do zero**. A identidade "correta" quase sempre já existe em fragmentos (paleta que a cliente já usa, frases que ela já repete, fotos que already convertem) — o trabalho de design é *sistematizar*, não inventar por cima.

---

## 2. Posicionamento de marca e tom de voz

| Elemento | Texto | Papel |
|---|---|---|
| Frase-âncora pessoal | **"Realçando sorrisos com naturalidade e delicadeza"** | Headline principal do hero — promessa emocional |
| Frase-âncora institucional | **"A clínica mais linda de Atibaia"** | Acento itálico logo abaixo — orgulho local, prova de ambiente |
| Mensagem de WhatsApp única | **"Olá, tudo bem? Queria fazer um agendamento na clínica."** | Usada em 100% dos CTAs do site |
| CROSP | 138.008 | Aparece **apenas** junto ao nome de uma profissional específica (nunca solto no hero/rodapé) |

**Tom de voz:** português informal-profissional, frases curtas, sem jargão técnico excessivo. Nunca inventa números (avaliações, contagens, credenciais) — quando a informação real não está confirmada, o texto diz explicitamente "a confirmar com a clínica" em vez de estimar ou fingir.

> [!warning] 🧱 ESTRUTURA — regra inegociável
> **Nunca fabricar prova social.** Depoimentos, notas, contagem de avaliações, credenciais — tudo isso só entra no site se vier de uma fonte real fornecida pelo cliente (Google Maps, print, texto colado). Quando falta um dado real (ex: horário completo de funcionamento, foto de um membro da equipe), o site usa um placeholder visualmente elegante com uma nota discreta "a confirmar com a clínica" — nunca inventa o número.

---

## 3. Identidade visual

### 3.1 Paleta de cores

> [!info] 🔁 TROCAR por cliente — mas manter a **estrutura de 6 tokens** e a lógica "high-key + 1 acento"

| Token CSS | Hex | Uso | O que comunica psicologicamente |
|---|---|---|---|
| `--color-bg` | `#FFFFFF` | Fundo predominante | Higiene, clareza clínica — essencial numa categoria (odontologia) onde o medo/ansiedade do paciente precisa ser neutralizado visualmente antes mesmo de ler uma palavra |
| `--color-bg-alt` | `#FAF8F5` | Off-white quente para diferenciar seções (ex: painel "Nossa história") | Aquece o branco puro sem perder a sensação de limpeza — evita que o site pareça um hospital frio |
| `--color-ink` | `#1A1A1A` | Texto principal | Grafite, não preto 100% — preto puro cansa a vista em telas claras e soa "impresso"; o grafite é mais suave, mais editorial |
| `--color-ink-muted` | `#86807A` | Texto secundário, legendas | Cinza com viés quente (não é um cinza neutro) — mantém a paleta coesa mesmo em texto de apoio |
| `--color-accent` | `#D9A62A` (dourado vivo) | Botões primários, ícones, numeração editorial | Luxo acessível — dourado comunica "premium" sem ser frio como prata/preto puro. **Foi intencionalmente tornado mais vivo/saturado** que um dourado envelhecido clássico, a pedido do cliente, para ter mais presença em telas grandes |
| `--color-accent-strong` | `#A6790A` | Estados de hover, texto sobre fundo claro que precisa de mais contraste | Versão mais escura do acento para acessibilidade (o dourado vivo puro não tem contraste suficiente para texto pequeno) |
| `--color-accent-ink` | `#3A2408` | Texto sobre botões preenchidos em dourado | Marrom escuro, não preto puro — mantém a paleta quente até no detalhe do texto-sobre-botão |
| `--color-line` | `#E5E2DC` | Linhas estruturais finas (1px) que dividem o grid | O "esqueleto" arquitetural do layout — organiza sem pesar |

**Tokens de vidro/glassmorphism** (usados para sobrepor texto em fotos):

```css
--glass-bg: rgba(255, 255, 255, 0.55);
--glass-border: rgba(255, 255, 255, 0.45);
--glass-scrim: rgba(20, 17, 12, 0.42);
--glass-blur: 14px;
```

> [!question] O que o "vidro" (glassmorphism) comunica?
> Transparência com blur é a forma visual de dizer "sofisticação e profundidade" sem esconder o que está por trás. Foi um pedido explícito do cliente ("quero explorar transparência para passar sofisticação") — a ideia é que o card de texto sobre a foto do hero nunca "tampe" a foto real da clínica, ele *flutua* sobre ela. Isso comunica: *a clínica não tem nada a esconder, o texto é só uma camada de apoio*.

### 3.2 Tipografia

> [!info] 🔁 TROCAR as fontes específicas se o cliente pedir — mas manter a **estrutura de 3 papéis** (display geométrica + acento serifado humano + corpo leve)

| Papel | Fonte | Peso | Uso | Por que essa escolha |
|---|---|---|---|---|
| **Display** | Montserrat | 700 (títulos), 500 (labels/nav) | Títulos em caixa alta, botões, navegação | Sans-serif geométrica e imponente — transmite estrutura, precisão, modernidade. Título em caixa alta = autoridade editorial (como uma capa de revista) |
| **Acento** | Playfair Display (itálico) | 400 | Palavras-chave dentro de títulos ("naturalidade", "delicadeza"), tagline, nome dos membros da equipe | Serifada clássica em itálico — quebra deliberadamente a rigidez da sans-serif para trazer **calor humano**. É o contraponto emocional dentro de um sistema mecânico/geométrico. Sempre usada com moderação (uma palavra ou frase curta, nunca um parágrafo) |
| **Corpo** | Inter | 300–600 (variável) | Parágrafos, descrições | Peso leve (300), entrelinha generosa (1.65) — pede leitura calma, "sem pressa". Isso é proposital: o visitante de um site de odontologia estética muitas vezes está ansioso; tipografia apertada/pesada aumenta a tensão, tipografia leve e espaçada acalma |

**Implementação técnica:** as 3 fontes são **self-hosted** via `next/font/local` (arquivos `.woff`/`.woff2` em `app/fonts/`), não via `next/font/google`. Isso foi uma decisão técnica forçada por um bug do Turbopack no Next.js 16 com `next/font/google` (ver [[#10. Notas técnicas e armadilhas conhecidas]]), mas acabou sendo *melhor* de qualquer forma: zero dependência de rede externa no load, mesmo resultado visual.

> [!question] Por que misturar uma serifada com uma geométrica?
> É o mesmo princípio usado no Instagram original da cliente: título em caixa alta e reto + uma palavra em itálico manuscrito-like no meio. A dualidade tipográfica *é* a personalidade da marca — "somos precisos E somos humanos", não um ou outro. Nunca usar as duas fontes com o mesmo peso de destaque; a serifada é sempre a exceção pontual, nunca o corpo do texto.

### 3.2b Logo oficial

> [!info] 🔁 TROCAR — mas manter a estrutura de **4 variantes**

A clínica forneceu a marca oficial (monograma "PV" line-art, geométrico, mesmo espírito da tipografia display) em 4 arquivos, todos PNG com fundo transparente, em `public/images/logos/`:

| Arquivo | Conteúdo | Uso |
|---|---|---|
| `logo-patricia-preta.png` | Só o ícone (monograma), traço preto | Sobre fundo claro — header em estado "vidro", rodapé |
| `logo-patricia-branca.png` | Só o ícone, traço branco | Sobre fundo escuro/foto — header transparente sobre o hero |
| `logo-patricia-full-preta.png` | Ícone + nome por extenso, preto | Reservado para usos institucionais maiores (papelaria, apresentações) — não usado no site em si pra não duplicar o nome que já aparece em texto ao lado da marca |
| `logo-patricia-full-branca.png` | Ícone + nome por extenso, branco | Idem, variante clara |

**Regra de aplicação:** todo lugar que mostra a marca precisa saber, em tempo real, se está sobre um fundo claro ou escuro/foto, e trocar entre a variante preta e branca — nunca usar uma cor fixa. No Header isso reaproveita a mesma variável `transparent` que já controla a cor do texto de navegação.

**Favicon:** gerado a partir do ícone preto (`logo-patricia-preta.png`), recortado (trim da margem transparente) e centralizado num quadrado com fundo `--color-bg-alt` (`#FAF8F5`) — nunca usar o ícone puro com fundo transparente como favicon, porque um traço fino preto sobre transparente some tanto em abas de navegador no modo claro quanto no escuro. Gerado via script Node + `sharp` (não há CLI pronta pra isso): `app/icon.png` (512×512), `app/apple-icon.png` (180×180, fundo obrigatoriamente sólido — iOS não aceita transparência), e um `app/favicon.ico` real (multi-resolução 16/32/48, PNG embutido no contêiner ICO).

### 3.3 Grid, layout e numeração editorial

- **Linhas estruturais finas** (`--color-line`, 1px) dividem seções e itens de lista — dá uma sensação "analítica", como um blueprint de arquitetura, reforçando a mensagem de precisão técnica.
- **Numeração editorial** (`01`, `02`, `03`...) em fonte monoespaçada com `tabular-nums`, acompanhada de uma borda esquerda dourada — usada em dois lugares:
  1. Lista de especialidades (`01 — Lentes de Contato Dental`, `02 — Implantes`...)
  2. Eyebrows de cada seção da home (`01 — A clínica`, `02 — Tecnologia`...)

  > [!important] Regra sobre numeração
  > **Só numerar o que realmente é uma sequência ou lista finita real** (especialidades, seções da página). Nunca usar número como decoração pura — se não há uma ordem/contagem real por trás, não numera.

- **Espaço negativo generoso** — o site "respira"; nenhuma seção é espremida. Isso é tratado como elemento de luxo (marcas premium raramente têm layouts densos).
- **Assimetria controlada** — quando há duas colunas de conteúdos diferentes (ex: foto + texto), evita-se a divisão 50/50 óbvia quando as proporções naturais das fotos pedem outra coisa (ver seção Equipe abaixo — isso gerou uma correção real no projeto).

### 3.4 Componentes de UI

| Componente | Especificação | Papel psicológico |
|---|---|---|
| **Botão pill (ghost)** | `border-radius: 999px`, borda fina, sem preenchimento | Convite leve, não agressivo — usado para ações secundárias ("Ver especialidades", "Perguntas frequentes") |
| **Botão pill (filled)** | Fundo `--color-accent`, texto `--color-accent-ink` | Ação primária — sempre é o WhatsApp. É o único botão "cheio" de cor da paleta, then ele se destaca por contraste, não por gritar |
| **Botão pill (ghost-light)** | Como o ghost, mas em branco — usado sobre fotos escuras | Mesma linguagem de "convite leve", adaptada para fundo escuro sem perder legibilidade |
| **Card de vidro (`.glass`)** | Fundo semi-transparente + blur 14px | Sobrepõe texto a fotos sem esconder a foto — ver seção 3.1 |
| **Scrim de foto (`.photo-scrim` / `.photo-scrim-strong`)** | Gradiente preto linear (não usar utilitário de opacidade do Tailwind — ver bug em [[#10. Notas técnicas e armadilhas conhecidas]]) | Escurece a base da foto o suficiente pra legibilidade de texto branco, sem transformar a foto real num filtro escuro genérico. A versão "strong" existe especificamente pro CTA final, onde o texto precisa ser lido rápido, sem esforço |

---

## 4. Direção fotográfica — os 3 registros

Esse é um dos pilares mais importantes do sistema: **toda foto do site pertence a um de três registros deliberados**, nunca escolhida por acaso.

| Registro | Onde é usado | O que mostra | Por que existe |
|---|---|---|---|
| **Ambiente** (`variant="ambiente"`) | Hero, seção "A Clínica", fachada, equipe | Luz natural, mármore, vidro, plantas — *high-key* | Reduz a ansiedade pré-consulta. Antes de qualquer texto, a pessoa vê que o lugar é bonito, limpo, calmo — não um consultório genérico |
| **Macro / precisão** (`variant="macro"`) | Seção "Tecnologia" | Close em equipamentos, telas de scanner, material cerâmico | Comunica competência técnica sem precisar de texto explicando — "eles usam tecnologia de verdade" |
| **Prova social** (`variant="prova"`) | Seção "Resultados" | Antes/depois em cor, sem dessaturar, sem moldura, tocando as linhas do grid | É o maior gatilho de conversão que já existia no Instagram da cliente — **nunca estilizar demais essa categoria**. Ela precisa parecer autêntica e imediata, não um antes/depois "produzido" |

> [!question] Por que não dessaturar as fotos de "prova social" para combinar com o resto do site?
> Porque o realismo *é* o argumento de venda ali. Um antes/depois em preto-e-branco ou com filtro artístico perderia credibilidade — o cérebro do visitante reconhece "isso parece Photoshop" mais rápido do que "isso é dourado premium". Prova social sempre vence estilo.

**Padrão de fallback:** quando uma foto real ainda não foi entregue pelo cliente, o site nunca mostra um ícone quebrado — ele mostra um gradiente elegante na paleta da marca (componente `PhotoPlaceholder`) com uma legenda discreta indicando o que vai entrar ali. Ver [[#9. Stack técnica#Padrão assetExists — fallback gracioso de fotos]].

---

## 5. Estrutura do site, seção por seção

> [!info] 🧱 ESTRUTURA — a ordem e a lógica de cada seção deve ser mantida; o conteúdo específico é `🔁 TROCAR`

A home é **uma página única e longa, com âncoras** (não múltiplas páginas separadas) — decisão tomada explicitamente para consolidar conteúdo que antes estava espalhado em `/clinica`, `/equipe`, `/especialidades`, `/resultados`. Só ficaram como páginas separadas `/faq` e (antes de ser removido) `/contato`.

### Header
- **Fixo** (`position: fixed`), nunca `sticky` — flutua por cima do hero.
- **Estado transparente** no topo da home (nav em branco sobre a foto) → **estado "vidro"** (`glass`, blur, fundo semi-opaco) assim que o usuário rola a página **ou** está em qualquer outra página que não seja a home.
- **Nav centralizado** (logo à esquerda, itens de navegação centralizados, botão de agendamento à direita) — layout em grid 3 colunas (`grid-cols-[auto_1fr_auto]`), não flexbox `justify-between`.
- Item ativo (página atual) ganha borda/texto na cor de acento.

> [!question] Por que transparente → vidro, e nunca uma cor sólida?
> Cor sólida no topo do hero "cortaria" a foto imediatamente, antes do usuário processar a primeira impressão. Transparente deixa a foto ser a primeira coisa vista, 100%. O "vidro" ao rolar existe pra manter a navegação legível sobre qualquer conteúdo subsequente, sem nunca virar uma barra opaca pesada — reforça a identidade "sofisticação com transparência" do resto do site.

### Hero (banner)
- **Full-bleed**: a foto cobre a tela inteira, incluindo a área atrás do header (técnica: `main` recebe `padding-top` igual à altura do header; a seção do hero cancela esse padding com margem negativa e compensa a altura, então ela "vaza" para trás do header).
- Overlay escuro (`.hero-scrim`) — gradiente sutil, mais forte no topo/base, mais claro no meio, só o suficiente pra legibilidade do texto branco.
- **Conteúdo centralizado** (não mais em card lateral): eyebrow pequeno → título grande em caixa alta → tagline em itálico → **um único botão** ("Entrar em contato conosco" / "Agendar pelo WhatsApp").
- Sem CRO nem outros selos no hero — informação de credencial fica reservada para a seção Equipe.

> [!question] Por que só um botão, centralizado, no hero?
> Múltiplos CTAs no primeiro contato geram paralisia de decisão. Um visitante ansioso (categoria odontológica estética) toma a decisão mais rápido quando só existe um caminho óbvio. O centro (em vez de canto) sinaliza "isso é o ponto principal da página", não "isso é um detalhe a mais".

### 01 — A Clínica
- Grid de 3 fotos reais do ambiente (recepção, consultório, sala de atendimento), cada uma com legenda em "vidro escuro" no canto.
- Bloco **"Nossa história"**: painel com fundo `--color-bg-alt`, título em Playfair itálico, texto corrido. **Não contém mais informação de endereço/localização** — isso foi separado numa seção própria (item 07) porque misturar "por que existimos" com "onde nos achar" dilui as duas mensagens.

### 02 — Tecnologia
- 3 fotos no registro *macro*: scanner intraoral em uso, equipamentos, detalhe de material (cerâmica). Existe para responder à pergunta silenciosa "esse lugar é realmente bem equipado?" sem precisar de uma lista de specs.

### 03 — Especialidades
- Lista numerada (`01`–`05`), cada linha com número + nome + descrição curta.
- **Interação de hover**: ao passar o cursor sobre uma linha, o fundo ganha um leve `--color-bg-alt`, o número e o nome mudam para `--color-accent-strong`, com transição suave (200ms). Implementado com `group`/`group-hover` do Tailwind, **sem JavaScript** — é puro CSS `:hover`.

> [!question] Por que dar destaque de hover a uma lista de texto simples?
> Convida a exploração. Uma lista estática de 5 itens é lida em bloco e esquecida; uma lista que "responde" ao mouse faz o visitante parar em cada item individualmente por mais tempo — aumenta o tempo de leitura de cada especialidade sem precisar de mais texto.

### 04 — Equipe
- **Card da fundadora** (Dra. Patricia): foto vertical (proporção ~3:4), nome, papel ("Fundadora & CEO"), especialidade, **CROSP visível** — é o único lugar do site onde a credencial aparece, porque é o único lugar onde ela está ligada a uma pessoa específica.
- **Card da equipe**: uma única foto de grupo, **sem nomes, sem CRO, sem cargos individuais** — só a legenda "Nossa equipe" e uma frase genérica de acolhimento.
- As duas fotos usam a **mesma proporção** (3:4, vertical) lado a lado em colunas iguais — isso não era a versão original (ver correção de layout abaixo).

> [!question] Por que a Dra. Patricia é nomeada e a equipe não?
> Ela é a fundadora — a marca pessoal dela é o que trouxe autoridade e seguidores no Instagram (ver seção 1). O resto da equipe existe para comunicar "isso não é um consultório solo, tem estrutura" sem expor dados pessoais de funcionárias que não pediram para ser a fachada pública da marca. É uma escolha tanto de storytelling (fundadora em destaque) quanto de respeito (privacidade da equipe).

> [!bug] Correção de layout real que aconteceu neste projeto
> Na primeira versão, a foto da Dra. Patricia era vertical (retrato) e a foto da equipe era **horizontal** (foto de grupo tirada na largura) — ambas forçadas na mesma proporção 4:5, o que cortava 2 das 5 pessoas da foto de equipe. O cliente pediu uma segunda foto de equipe, dessa vez **vertical/corpo inteiro** (mesmo grupo, mesma roupa, enquadramento vertical), especificamente para poder usar a mesma proporção 3:4 dos dois lados e mostrar todo mundo. **Lição:** sempre perguntar/verificar a orientação nativa de uma foto de grupo antes de decidir a proporção do card — não force um crop que corta pessoas.

### 05 — Resultados
- Grid de fotos "antes/depois" reais (formato já vem pronto do fotógrafo da clínica: topo = antes, base = depois, marca d'água "PV" no meio), legendadas "Caso 01", "Caso 02"...
- Título mudou de "Antes & Depois" para **"Resultados dos pacientes"** — a pedido do cliente, para soar menos "antes/depois de anúncio" e mais "resultado real documentado".
- **Nunca preencher com placeholders genéricos aqui.** Se só existem 3 casos reais, mostra 3. O grid é dinamicamente filtrado (`resultCases.filter(item => assetExists(item.src))`) — quando a clínica manda um caso novo, ele aparece sozinho, sem precisar mexer em código.

### 06 — Avaliações
- **Carrossel/marquee infinito**, deslizando devagar para a esquerda (46s por ciclo), pausa no hover, respeita `prefers-reduced-motion`.
- Cards com 5 estrelas douradas (decorativas — nunca inventar uma nota média por card, só a nota agregada real do Google no texto acima: *"4,5 ★ · 17 avaliações no Google"*).
- **Só depoimentos reais e positivos** — nunca inclui a resposta pública da clínica ao comentário (isso é conversa entre clínica e paciente, não é material de marketing), nunca inclui avaliações negativas, nunca reescreve a fala do paciente (no máximo corta uma frase do meio, preservando as demais literalmente, erros de digitação inclusos).
- Botão único, **centralizado**, "Ver avaliações no Google" → link de busca do Google Maps.

> [!question] Por que marquee automático em vez de grid estático?
> Um grid estático de 8 depoimentos ocupa muito espaço vertical ou fica pequeno demais pra ler. O marquee deixa o volume de prova social "parecer maior" (sensação de fluxo contínuo de gente falando bem) sem exigir scroll nem espaço de tela — e o pause-on-hover garante que dá pra parar e ler com calma quando quiser.

### CTA final ("Vamos conversar")
- Foto do ambiente com **scrim escuro reforçado** (`.photo-scrim-strong`) — mais escuro que o padrão do resto do site, porque aqui o texto precisa ser lido instantaneamente, é a última chance de conversão antes do rodapé.
- Botão WhatsApp + link para FAQ.

### 07 — Localização
- **Grid de 2 colunas**: iframe do Google Maps (embed sem API key, via `google.com/maps?q=...&output=embed`) ao lado de foto da fachada do prédio + card com endereço, horário e telefone.
- Endereço e telefone **conferidos contra a listagem real do Google Meu Negócio** do cliente antes de publicar — nunca assumir/formatar um endereço sem checar contra a fonte.

### Rodapé
- Colunas: identidade (logo + tagline) · navegação · contato (WhatsApp + Instagram).
- Linha final **centralizada**: `© {ano} {nome}. Todos os direitos reservados.` + `Desenvolvido por Vitor Santos · vitorsantos.dev.br`.

---

## 6. Estratégia de conversão (CTAs)

> [!important] 🧱 ESTRUTURA — regra que define este projeto
> **Zero formulário de contato. Todo botão de ação no site inteiro abre o WhatsApp**, com a mesma mensagem padrão, definida uma única vez:
> ```
> Olá, tudo bem? Queria fazer um agendamento na clínica.
> ```
> Implementado como um valor-padrão de função em `lib/content.ts`:
> ```ts
> export function whatsappUrl(message: string = site.defaultWhatsappMessage) {
>   return `https://wa.me/${site.whatsappNumber}?text=${encodeURIComponent(message)}`;
> }
> ```
> Todo botão do site chama `whatsappUrl()` sem argumento. Isso existe porque o cliente já validou esse funil no Instagram — reinventar com formulário próprio (que existia numa versão anterior do site, com Resend + React Hook Form) só adicionava fricção e um canal a mais para checar. **Removido de propósito.**

Existe também um botão flutuante fixo de WhatsApp (`WhatsAppButton.tsx`, canto inferior direito, `glass-dark`) presente em 100% das páginas, sempre visível durante o scroll.

---

## 7. Conteúdo e ética de prova social

- Nunca fabricar depoimentos, contagens, ratings ou credenciais.
- Quando uma informação real está incompleta (ex: "4,517 comentários" veio ambíguo de um copy/paste do Google), **parar e verificar a fonte real** (nesse caso, o próprio widget do Google Maps confirmou "4,5 ★ · 17 avaliações") antes de publicar um número.
- Ao curar depoimentos de uma lista real (ex: colada do Google Meu Negócio), excluir ativamente avaliações negativas e respostas do proprietário — só entram avaliações de pacientes, positivas, verbatim.

> [!important] Mudança de política — textos "a confirmar" não ficam mais visíveis no site
> Na primeira versão do projeto, quando faltava um dado real, o site mostrava explicitamente uma nota tipo *"a confirmar com a clínica"* na tela. **O cliente pediu pra tirar isso** — um site com placeholders visíveis passa impressão de inacabado. A partir de 2026-08-13, a regra passou a ser:
> - Se existe conteúdo real (mesmo que parcial), mostra só o conteúdo real, sem ressalva na tela.
> - Se não existe conteúdo real nenhum (ex: resposta de FAQ), **remove o item da tela** em vez de deixar um espaço vazio ou uma nota de pendência.
> - Todo gap de conteúdo — o que foi removido e o que ficou parcial — é rastreado à parte em [[Pendências de conteúdo]], nunca no site em produção.

---

## 8. Transições e microinterações

| Elemento | Comportamento | Duração/easing |
|---|---|---|
| Header (transparente → vidro) | `transition-colors`, cross-fade de fundo | 300ms |
| Pills de navegação (hover/estado ativo) | Borda e cor de texto | 200ms, `transition-colors` |
| Linha de especialidade (hover) | Fundo + cor do número/nome | 200ms |
| Botões (hover) | Fundo passa de `--color-accent` para `--color-accent-strong` | 200ms |
| Botão flutuante do WhatsApp (hover) | `scale(1.05)` | transição padrão do navegador |
| Marquee de avaliações | Translação horizontal contínua, `linear`, loop infinito via duplicação da lista | 46s por ciclo, pausa total no `:hover` |
| Scroll geral | `scroll-behavior: smooth` no `<html>`, com `data-scroll-behavior="smooth"` (exigido pelo Next 16 para não conflitar com transições de rota) | — |
| Motion reduzida | `prefers-reduced-motion: reduce` zera todas as durações de animação/transição para ~0 | — |

---

## 9. Stack técnica

- **Framework:** Next.js 16 (App Router), TypeScript, Server Components por padrão.
- **Estilo:** Tailwind CSS v4 (`@theme inline` em `globals.css` para mapear tokens de cor/fonte para utilitários).
- **Fontes:** self-hosted via `next/font/local` (ver 3.2).
- **Imagens:** `next/image`, sempre com `sizes` explícito; hero usa `priority`.
- **Sem banco de dados, sem backend próprio** — site 100% estático/SSR, sem formulário, sem CMS.
- **Deploy alvo:** Vercel (não confirmado o domínio final).

### Padrão `assetExists` — fallback gracioso de fotos

Criado em `lib/assets.ts`:

```ts
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

Usado dentro de Server Components (nunca em componentes `"use client"`, porque `node:fs` não roda no bundle do cliente) para decidir, em tempo de request, se uma foto real já foi entregue:

```tsx
<PhotoPlaceholder
  src={assetExists(founder.photo) ? founder.photo : undefined}
  caption={assetExists(founder.photo) ? undefined : "Dra. Patricia"}
/>
```

> [!tip] 🧱 ESTRUTURA — por que isso importa pra reutilização
> Esse padrão é o que permite ao cliente **ir soltando fotos reais aos poucos** na pasta `public/images/...` sem precisar pedir uma alteração de código a cada entrega. O caminho do arquivo já fica "reservado" no `lib/content.ts` desde o início do projeto; quando o arquivo aparece, o site se atualiza sozinho no próximo build/request. Reaproveitar esse padrão em todo projeto novo.

---

## 10. Notas técnicas e armadilhas conhecidas

> [!bug] Bug real — gradientes de opacidade do Tailwind v4 (oklab)
> Utilitários do tipo `bg-gradient-to-b from-black/60 via-black/30 to-black/65` renderizaram **muito mais opacos do que deveriam** neste ambiente — o gradiente ficava quase 100% sólido em vez de deixar a foto por trás aparecer, mesmo com os valores de opacidade corretos no CSS computado. Causa provável: interpolação de cor em `oklab` (padrão do Tailwind v4 pra gradientes) entre dois tons pretos com alfas diferentes.
>
> **Correção aplicada:** parar de usar utilitários Tailwind de gradiente-com-opacidade para overlays de foto. Em vez disso, classes CSS dedicadas em `globals.css` com `rgba()` puro:
> ```css
> .hero-scrim { background-image: linear-gradient(to bottom, rgba(0,0,0,0.55) 0%, rgba(0,0,0,0.25) 45%, rgba(0,0,0,0.6) 100%); }
> .photo-scrim { background-image: linear-gradient(to top, rgba(0,0,0,0.7) 0%, rgba(0,0,0,0.25) 55%, rgba(0,0,0,0) 100%); }
> .photo-scrim-strong { background-image: linear-gradient(to top, rgba(0,0,0,0.88) 0%, rgba(0,0,0,0.6) 55%, rgba(0,0,0,0.25) 100%); }
> ```
> **Regra pra próximos projetos:** qualquer overlay escuro sobre foto usa uma dessas classes dedicadas (ou uma nova, no mesmo padrão), nunca `bg-gradient-to-* from-black/NN`.

> [!bug] Bug real — Next.js 16 + Turbopack + `next/font/google`
> `next/font/google` falhava em dev com `Module not found: Can't resolve '@vercel/turbopack-next/internal/font/google/font'` (bug específico dessa combinação de versões, no momento da build deste projeto). **Correção:** baixar os `.woff`/`.woff2` das fontes manualmente e usar `next/font/local` apontando pra `app/fonts/`. Resultado idêntico visualmente, e como bônus, zero dependência de rede externa.

> [!bug] Header `fixed` cobrindo conteúdo ao rolar
> Trocar o header de `sticky` para `fixed` (necessário pro efeito transparente-sobre-o-hero) tira ele do fluxo do documento — sem compensação, ele passa a **cobrir os primeiros ~80px de qualquer seção** que role até o topo da viewport, não só a primeira.
> **Correção:** `<main>` recebe `padding-top` igual à altura do header (`pt-20` = 5rem = 80px); a seção do hero cancela esse padding com margem negativa (`-mt-20`) e soma a mesma quantia à própria altura mínima (`min-h-[calc(100vh+5rem)]`) pra continuar cobrindo a tela inteira. Toda âncora de seção (`#clinica`, `#equipe`...) também recebe `scroll-mt-24` pra não ficar colada embaixo do header ao navegar direto por link.

> [!bug] `next/image` com `width: auto` esticando dentro de um `flex-col`
> Uma imagem de proporção fixa (`width`/`height` do `next/image` + classe `w-auto`) renderizou **distorcida/esticada horizontalmente** quando o elemento pai era um container `flex flex-col` (ex: a logo no rodapé, dentro de `<div className="flex flex-col gap-3">`). A mesma imagem, com o mesmo `className`, renderizava perfeita dentro de um container `flex` em linha (o header).
> **Causa:** é o comportamento padrão do Flexbox para elementos substituídos (`<img>`, `<video>`) — dentro de um flex container em coluna, o eixo cruzado é a *largura*, e `align-items: stretch` (padrão) estica a imagem pra preencher a largura disponível, ignorando `width: auto` e a proporção intrínseca da imagem.
> **Correção:** adicionar `self-start` (`align-self: flex-start`) na própria imagem sempre que ela estiver dentro de um `flex-col` e precisar manter sua proporção natural.
> **Regra pra próximos projetos:** qualquer `<Image>` de proporção fixa dentro de um container `flex-col` (não só `flex-row`) precisa de `self-start` (ou `self-center`) por padrão — não assumir que `w-auto`/`h-auto` sozinho basta.

> [!tip] Ferramenta de preview / browser instável neste ambiente
> Durante o desenvolvimento, o painel de preview do browser apresentou renderizações "fantasma" (screenshot em branco ou com conteúdo antigo) várias vezes, mesmo com o DOM/CSS corretos (confirmado via inspeção JS direta). **Sempre que a screenshot parecer errada mas o código estiver correto, abrir uma aba nova antes de assumir que é um bug real** — economiza tempo de debugging de um problema que não existe no código.

---

## 11. Como reaproveitar este sistema para um novo cliente

Checklist prático:

**🔁 Trocar (específico do cliente):**
- [ ] Nome, tagline, cidade, CROSP/registro profissional
- [ ] Número de WhatsApp e mensagem padrão
- [ ] Endereço, horário de funcionamento, telefone
- [ ] Paleta de cores (manter a estrutura de 6 tokens + regra "high-key + 1 acento", mas o hex específico do acento pode mudar)
- [ ] Fontes específicas (se o cliente já tiver uma identidade tipográfica própria)
- [ ] Todas as fotos (respeitando os 3 registros da seção 4)
- [ ] Especialidades/serviços oferecidos
- [ ] Equipe (decidir de novo: quem é "fundador visível com credencial" vs. "equipe coletiva sem nomes")
- [ ] Depoimentos reais (nunca reaproveitar os depoimentos de um cliente anterior)
- [ ] Copy de todas as seções

**🧱 Manter (estrutura validada):**
- [ ] Header fixo, transparente→vidro, nav centralizado
- [ ] Hero full-bleed com scrim + conteúdo centralizado + 1 CTA só
- [ ] Os 3 registros fotográficos deliberados (ambiente / macro-precisão / prova social)
- [ ] Numeração editorial só onde há sequência real
- [ ] Hover de destaque em listas de especialidades/serviços
- [ ] Padrão `assetExists` pra fallback gracioso de fotos
- [ ] Estratégia de CTA único e consistente (decidir WhatsApp vs. formulário conforme o canal que o cliente já usa — mas manter consistente em 100% do site, nunca misturar)
- [ ] Marquee de avaliações reais, nunca fabricadas
- [ ] Ética de conteúdo: nada de números/credenciais inventados
- [ ] Rodapé centralizado com crédito do desenvolvedor
- [ ] Usar as classes `.hero-scrim`/`.photo-scrim` (não gradientes Tailwind com opacidade) pra overlays

**Perguntas a fazer no início de todo projeto novo (antes de desenhar qualquer coisa):**
1. O cliente já tem Instagram/redes com identidade visual validada? Auditar antes de propor algo novo.
2. Qual é o canal de conversão que o cliente já usa e confia (WhatsApp, formulário, telefone)?
3. Quais fotos reais existem hoje, e quais faltam? Nunca travar o projeto esperando 100% das fotos — usar o padrão de fallback e liberar aos poucos.
4. Existe um "fundador visível" na marca, ou é uma equipe coletiva sem hierarquia de destaque?
5. Onde estão as avaliações/depoimentos reais (Google, Instagram, prints do cliente)? Pedir a fonte bruta, nunca resumir de memória.

---

## Notas finais

Este documento acompanha o repositório em `/Users/MAC/VITOR/dra-patricia-vieira`. Qualquer decisão de design nova tomada naquele projeto que **generalize bem** para outros clientes deve ser retro-adicionada aqui, na seção correspondente — este arquivo é vivo, não uma foto única do dia em que foi escrito.
