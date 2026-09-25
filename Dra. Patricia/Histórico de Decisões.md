---
tags: [historico, changelog, dra-patricia-vieira]
projeto: dra-patricia-vieira
atualizado: 2026-08-13
---

# Histórico de decisões — do zero ao site em produção

> [!info]
> Linha do tempo de tudo que foi pensado, testado e construído neste projeto, em ordem cronológica. Ver também [[Índice]] para navegar pelas outras notas do vault.

---

## Fase 1 — Pesquisa de marca (Instagram)

- Análise do perfil pessoal **[@drapatriciavieira](https://www.instagram.com/drapatriciavieira)** (15,3 mil seguidores, verificado): identidade visual (bege/branco/dourado/preto, tipografia serifada + sans, monograma "PV"), posicionamento ("Realçando sorrisos com naturalidade e delicadeza"), prova social forte via antes/depois, funil já validado pro WhatsApp.
- Análise do perfil institucional **[@clinicapatriciavieira](https://www.instagram.com/clinicapatriciavieira)** (522 seguidores): revelou que a clínica é **multiprofissional** (não só a Dra. Patricia) — achado que definiu o enquadramento do site inteiro: site da *clínica*, com a Patricia como fundadora/rosto principal, não um site pessoal dela.
- Bio da clínica mapeada: "A clínica mais linda de Atibaia✨" + especialidades (Lentes, Implantes, Ortodontia, Clínica geral, Próteses).

## Fase 2 — Plano e primeira direção visual

- Planejamento do site: Next.js, sem formulário no início (isso mudou depois — ver Fase 5), estrutura de páginas separadas (`/clinica`, `/equipe`, `/especialidades`, `/resultados`, `/contato`) — **essa estrutura de páginas separadas foi abandonada na Fase 4**.
- Usuário forneceu um **design system de referência** (adaptação de uma estética "alto padrão/arquitetural" para light mode) — adaptado às cores já identificadas no Instagram da clínica.
- Criada uma **referência visual em Artifact** (paleta, tipografia, componentes, direção fotográfica) pra validar a direção antes de codar.
- Pedido de ajuste: **dourado mais vivo** (de um tom envelhecido/muted para `#D9A62A`, mais saturado) + explorar **transparência/glassmorphism** pra sobreposição de texto em imagem → adicionados tokens de vidro (`--glass-bg`, `--glass-blur` etc.) e uma seção de demonstração no Artifact.

## Fase 3 — Scaffold do projeto

- `create-next-app` em `/Users/MAC/VITOR/dra-patricia-vieira` — Next.js 16 (App Router), TypeScript, Tailwind CSS v4.
- Design tokens implementados em `app/globals.css` via `@theme inline`.
- **Bug encontrado:** `next/font/google` falhava no dev com Turbopack (Next 16). **Correção:** fontes baixadas manualmente e servidas via `next/font/local` (`app/fonts/*.woff`/`.woff2`).
- Componentes base criados: `Header`, `Footer`, `WhatsAppButton`, `PhotoPlaceholder` (com fallback gracioso pra fotos que ainda não existem), `SpecialtyList`, `ui.tsx` (`PillLink`, `SectionHeading`, `Eyebrow`).
- Primeira versão da Home (Hero + A Clínica + Especialidades + Equipe + Resultados) e páginas separadas `/contato` (com formulário — React Hook Form + Zod + Resend) e `/faq`.

## Fase 4 — Home como página única, com âncoras

- Decisão: consolidar `/clinica`, `/equipe`, `/especialidades`, `/resultados` como **seções âncora dentro da própria Home** (`/#clinica`, `/#equipe`...) em vez de páginas separadas.
- Mantidas como páginas de verdade só `/faq` e (na época) `/contato`.
- Nav e rodapé atualizados pra apontar pras âncoras.

## Fase 5 — Fotos reais chegando

- Fotos reais do ambiente (`consultorio-1/2/3.jpg`, `recepcao-1/2.jpg`) entregues e conectadas na seção "A Clínica" e no hero.
- 3 casos reais de antes/depois (`caso-1/2/3.png`) conectados na seção Resultados.
- Dados reais do Google Meu Negócio fornecidos (endereço, telefone, avaliações) → criada seção de **Avaliações** (grid, versão inicial) e seção de **Localização** (endereço, telefone, iframe do mapa).
- **Confirmação cruzada:** o widget do Google Maps embutido revelou a nota real da clínica — **4,5 ★ · 17 avaliações** — resolvendo uma ambiguidade de um texto colado ("4,517 comentários") que parecia um erro de formatação.

## Fase 6 — Lote grande de 12 mudanças (o maior salto do projeto)

Pedido único cobrindo 12 pontos:

1. **Hero full-bleed** — foto cobrindo a tela inteira, incluindo atrás do header; header vira `fixed`, transparente no topo da Home, "vidro" (blur) ao rolar; nav mais tarde centralizado.
2. **CRO removido** do hero e do rodapé — só aparece junto ao nome de uma profissional específica.
3. **Seção Tecnologia** criada (equipamentos/precisão).
4. **Hover de destaque** na lista de especialidades (puro CSS, `group`/`group-hover`).
5. **Equipe reestruturada**: card grande da fundadora (foto + nome + CRO) + foto única da equipe sem nomes individuais.
6. **"Antes & Depois" → "Resultados dos pacientes"** (mudança de copy).
7. **Avaliações viraram marquee** — carrossel infinito, devagar, pausa no hover.
8. **Formulário de contato removido** — `/contato`, `ContactForm.tsx`, API route e libs de e-mail deletados; **todo botão do site** passou a abrir WhatsApp com uma única mensagem padrão.
9. **Foto da fachada** adicionada à seção Localização.
10. **Rodapé** ganhou "Todos os direitos reservados" + crédito do desenvolvedor.
11. **CTA final** ("Vamos conversar") com overlay escuro.
12. Nav centralizado no header (ajustado nesta fase e refinado depois).

**Dois bugs técnicos reais encontrados e corrigidos nesta fase** (documentados em detalhe em [[Design System — Clínica Patrícia Vieira#10. Notas técnicas e armadilhas conhecidas]]):
- Gradientes `bg-gradient-to-b from-black/60...` do Tailwind v4 renderizando quase opacos (interpolação `oklab`) → trocados por classes CSS dedicadas com `rgba()` puro (`.hero-scrim`, `.photo-scrim`).
- Header `fixed` cobrindo o topo de todas as seções ao rolar → corrigido com `padding-top` no `<main>` compensado por margem negativa só no hero.

## Fase 7 — Mais fotos reais, mais avaliações reais

- Fotos de tecnologia (`tecnologia-scanner.png`, `tecnologia-doutora.png`, `ceramica.png`) e da fachada (`fachada-predio.png`) entregues e conectadas.
- Fotos da Dra. Patricia (`dra-patricia.png`) e da equipe (`equipe-clinica.png`, depois trocada por `equipe-vertical.png`) entregues.
- **Padrão `assetExists`** criado (`lib/assets.ts`) — verifica em tempo de request se um arquivo já existe em `public/`, permitindo que fotos apareçam sozinhas assim que forem entregues, sem precisar de nova alteração de código.
- Lista completa de avaliações reais do Google fornecida pelo cliente (incluindo uma avaliação negativa e respostas da clínica aos comentários) → curadoria: **só avaliações positivas de pacientes entraram**, negativa e respostas do proprietário excluídas ativamente. Foram de 3 para 8 depoimentos no marquee.

## Fase 8 — Ajustes finos de layout

- **Correção de proporção Equipe:** a primeira foto de equipe era horizontal (grupo de 5 pessoas na largura) forçada numa proporção 4:5, cortando 2 pessoas. Pedido um recorte vertical/corpo inteiro do mesmo grupo (`equipe-vertical.png`) → as duas fotos (fundadora + equipe) passaram a usar a mesma proporção 3:4, lado a lado, mostrando todo mundo.
- Card da Dra. Patricia ajustado pra ficar menor e mais vertical, conforme pedido.

## Fase 9 — Vault do Obsidian criado

- Vault próprio **"Dra. Patricia"** criado inicialmente em `/Users/MAC/VITOR/Dra. Patricia`, com o documento [[Design System — Clínica Patrícia Vieira]] cobrindo identidade visual, estrutura, raciocínio humano por trás de cada escolha, CTAs, transições e notas técnicas — pensado desde o início pra ser reaproveitável em clientes futuros.

## Fase 13 — Consolidado no vault "vitor"

- A pedido do cliente, todas as notas foram movidas do vault standalone pra dentro do vault principal **"vitor"** (`/Users/MAC/VITOR/vitor`), na subpasta `Dra. Patricia/` — mesmo padrão usado pros outros projetos do vault (`dentist-system/`, `MazyOS/`, `Lines/`, `hubassistent/`). O vault standalone foi removido.

## Fase 10 — Logo oficial e favicon

- 4 arquivos de logo oficial fornecidos (`logo-patricia-preta/branca.png` = ícone só; `logo-patricia-full-preta/branca.png` = ícone + nome), todos PNG com transparência real.
- Favicon gerado a partir do ícone preto via script Node + `sharp`: `app/icon.png`, `app/apple-icon.png`, `app/favicon.ico` (multi-resolução real, não só um placeholder).
- Logo do texto "PV" (que era só tipografia dentro de um círculo) trocada pela logo-ícone real no Header e Footer, alternando preta/branca conforme o fundo.
- Pedido de trocar pra usar a **logo full** (ícone + nome) em vez de só o ícone → **bug encontrado:** o arquivo original tinha o conteúdo ocupando só ~35% da altura do canvas (muita margem transparente), então a logo renderizava minúscula. **Correção:** versões recortadas geradas (`logo-patricia-full-preta-trim.png` / `-branca-trim.png`) com as dimensões reais mapeadas em `lib/content.ts`.
- **Segundo bug encontrado:** a logo full no rodapé renderizava **esticada/distorcida** — causa: comportamento padrão do Flexbox, que estica elementos substituídos (`<img>`) no eixo cruzado dentro de um container `flex-col`. **Correção:** `self-start` na imagem.
- Texto "Clínica Patrícia Vieira" removido de baixo da logo no rodapé (redundante, já que a logo full contém o nome).

## Fase 11 — Limpeza de conteúdo placeholder

- Decisão do cliente: **nenhum texto visível de "a confirmar com a clínica"** no site em produção — passa impressão de inacabado.
- Todos os textos desse tipo removidos:
  - "Nossa história" → manteve o texto real, tirou só a ressalva.
  - Horário de funcionamento → ficou só "Fecha às 18h." (o que é real e confirmado).
  - Notas de foto pendente (equipe, fachada) → já eram blocos condicionais mortos (fotos já existiam), código limpo.
  - **FAQ:** 4 das 5 perguntas não tinham resposta real (só placeholder) → removidas da página, ficou só "Como agendar uma primeira avaliação?".
- Todo esse conteúdo pendente passou a ser rastreado só internamente, em [[Pendências de conteúdo]] — nunca mais como texto visível no site.

## Fase 12 — Este vault, expandido

- A pedido do cliente, o vault foi expandido além do documento de design system: [[Histórico de Decisões]] (esta nota), [[Estrutura Técnica e Código]], [[Inventário de Assets]] e [[Conteúdo do Site]] — cobrindo, além do *porquê* das decisões de design, o *o quê* e o *onde* de tudo que foi construído.
