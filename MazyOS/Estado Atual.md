#projeto #mazyos #estado-atual

Ver [[Visão Geral]] para contexto. Snapshot de **2026-08-01**, clone local em
`~/VITOR/MazyOS`.

## O que confirma que `/instalar` ainda não rodou

- `_memoria/empresa.md`, `preferencias.md`, `estrategia.md` — todos os campos
  em branco (só os títulos de seção do template).
- `identidade/design-guide.md` — cores, tipografia, elementos-chave, logo:
  tudo em branco.
- `CLAUDE.md` raiz — é o template genérico do MazyOS (o próprio README diz
  que o `/instalar` "complementa o final dessa página com as regras
  específicas do seu negócio" — isso não aconteceu ainda).
- A pasta continua chamada `MazyOS` (o README instrui renomear pro nome do
  negócio depois do `/instalar`).
- `scripts/` está vazia — nenhuma integração (Instagram, Meta, OpenAI) foi
  configurada ainda.
- `dados/`, `marketing/`, `saidas/` só têm os `README.md` de instrução, sem
  nenhum output real gerado ainda.

## O que já existe e funciona sem `/instalar`

- As 15 skills em `.claude/skills/` estão completas e utilizáveis — elas só
  ficam menos calibradas (sem tom de voz, sem cores de marca) até a memória
  ser preenchida.
- `templates/` (perfis de CLAUDE.md, catálogo de skills, catálogo de
  ferramentas) está completo — é material de referência do próprio MazyOS,
  não precisa de instalação.
- Git local está limpo, branch `main`, remote já aponta pro repo original
  `mazzeoia/MazyOS.git` (isso provavelmente precisa mudar pro repo do
  negócio do Vitor depois do `/instalar`+renomear, ou pelo menos é algo a
  decidir).

## Consequência prática

Qualquer skill de conteúdo/marketing chamada agora (`/carrossel`,
`/publicar-tema`, `/seo`...) vai rodar sem contexto de negócio real — sem
saber quem é a empresa, o tom de voz, ou as cores da marca. Faz sentido
rodar `/instalar` antes de qualquer produção real. Ver [[Próximos Passos]].
