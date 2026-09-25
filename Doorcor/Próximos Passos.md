#projeto #doorcor #próximos-passos

Ver [[Estado Atual]] pra origem de cada item. Nada aqui é pedido confirmado
do Vitor — são pontos em aberto identificados durante o desenvolvimento que
valem uma decisão dele antes de seguir.

## Decisões a validar com o Vitor

- **Commitar a rodada de revisão de 2026-09-02?** Mudanças de WhatsApp/
  legendas/reordenação/Contact/Footer estão no working tree, sem commit.
- **Confirmar dados oficiais da marca** — telefone, endereço, CNPJ, nome
  legal — antes de considerar o site pronto pra publicar. Hoje tudo em
  `brand` (`content.ts`) vem de scraping do Instagram público.
- **Corrigir a ordem do menu de navegação** (`navLinks` em `content.ts`) pra
  bater com a ordem real das seções (Diferenciais antes de Projetos).
- **Aproveitar o resto das fotos** — 25 das 37 fornecidas não estão em uso.
  Se o Vitor quiser uma galeria maior ou trocar alguma das 12 atuais, é só
  apontar quais.
- **Paginação real na Galeria?** Hoje é só decorativa (8 itens, sempre todos
  visíveis). Se a galeria crescer bastante, paginação/filtro funcional por
  categoria (Interno/Residencial/Tecnologia) pode valer a pena.
- **Deploy** — nenhuma decisão de hosting foi tomada ainda (Vercel, Netlify,
  etc.) nem domínio configurado.
- **Repositório remoto** — projeto só existe localmente em `~/VITOR/doorcor`,
  sem GitHub/remote configurado nesta sessão.
