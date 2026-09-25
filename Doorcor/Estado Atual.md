#projeto #doorcor #estado-atual

Ver [[Visão Geral]] para a linha do tempo completa. Este é um snapshot do que
está pronto vs. pendente em **2026-09-02**.

## Pronto e verificado

- Todas as 7 seções implementadas: Header, Hero, Marquee, Sobre, Produtos,
  Diferenciais, Galeria, Contato, Footer — ver [[Arquitetura]].
- Design system completo aplicado — paleta, tipografia, botões, grid
  estrutural, overlays de foto — ver [[Design System]].
- 12 fotos reais da DoorCor integradas (Hero, Sobre, Galeria) — ver
  [[Arquitetura]] e [[Estrutura de Conteúdo]].
- Todos os CTAs do site redirecionam pro WhatsApp (+55 11 93215-2858) com
  mensagem contextual — nenhum formulário no site.
- Legibilidade de texto sobre fundo escuro ajustada (cores `mist`/`fog`
  clareadas, peso de fonte do corpo aumentado).
- `npx tsc -b --noEmit`, `npm run build` e `npm run lint` passam limpos.
- Verificado visualmente via Chrome automation em desktop (1512px) e mobile
  (390px), incluindo menu hambúrguer mobile funcional.
- Todos os 6 pedidos pontuais da rodada de revisão de 2026-09-02 confirmados
  em tela: links de WhatsApp com mensagem certa, legendas em Sobre, botão
  "Ver Galeria de Projetos" em Produtos, Diferenciais antes de Galeria,
  Contato sem formulário, Footer com crédito do dev.

## Pendente / não feito

- **Nenhum commit no git além do scaffold inicial** (`8b0489f`). Toda a
  rodada de revisão de 2026-09-02 (WhatsApp em tudo, legendas, reordenação,
  Contact reescrito, Footer) está no working tree mas não commitada — a
  sessão nunca recebeu pedido explícito do Vitor pra commitar.
- **Sem re-checagem mobile** da rodada de revisão mais recente — a checagem
  de responsividade mobile foi feita numa rodada anterior, antes das
  mudanças de 2026-09-02.
- **Dados de `brand` não confirmados oficialmente** — telefone, Instagram e
  nome vêm de scraping do Instagram público, não de informação oficial da
  empresa. O próprio `content.ts` tem um comentário avisando disso.
- **Menu de navegação (`navLinks`) desatualizado** em relação à ordem real
  das seções — lista "Projetos" antes de "Diferenciais", mas a página
  renderiza Diferenciais primeiro desde a reordenação de 2026-09-02. Ver
  [[Estrutura de Conteúdo]].
- **25 das 37 fotos fornecidas não são usadas** em lugar nenhum do site —
  só 12 estão referenciadas em `content.ts`/`Hero.tsx`.
- **Paginação da Galeria é decorativa** — "01/08" e as setas não filtram nem
  paginam nada de verdade, é só os 8 itens sempre visíveis no grid.
- **Sem repositório remoto** configurado (git local, `origin` não verificado
  nesta sessão).

Ver [[Próximos Passos]] pra decisões em aberto derivadas desses pontos.
