#projeto #mazyos

## O que é

Template/boilerplate de repositório Git — não um app, não um SaaS — pra rodar
a operação de marketing/conteúdo de um negócio dentro do Claude Code. A tese
do próprio projeto: "IA não é uma ferramenta que sua empresa usa, é o sistema
operacional em que ela roda". Na prática é `CLAUDE.md` + memória em arquivos
`.md` + 15 skills do Claude Code, tudo versionado no GitHub.

O nome "MazyOS" é o nome do template — a ideia é que depois do `/instalar`
a pasta seja renomeada pro nome do negócio real do usuário ("ela é o teu
negócio agora", diz o README). Feito pela agência/estúdio **mazzeoia**
([mazzeoia.com.br](https://mazzeoia.com.br)).

Repositório: [GitHub — mazzeoia/MazyOS](https://github.com/mazzeoia/MazyOS.git)
(clonado localmente em `~/VITOR/MazyOS`, ainda **não renomeado** — ver
[[Estado Atual]]).

## Como pensar sobre ele

Três camadas:
1. **Memória** (`_memoria/`) — quem é a empresa, como ela fala, o que importa
   agora. Lida pelo Claude no início de toda conversa (regra do `CLAUDE.md`
   raiz).
2. **Identidade** (`identidade/design-guide.md`) — cores, tipografia, regras
   visuais. Toda peça gerada (carrossel, post) respeita isso.
3. **Skills** (`.claude/skills/`) — 15 comandos prontos que produzem o
   trabalho de fato (conteúdo, SEO, ads, dados, email) e sabem onde salvar
   o resultado (`marketing/`, `saidas/`, `scripts/`).

Ver [[Funcionalidades]] pra lista completa dos comandos e [[Arquitetura]]
pra como as pastas se conectam.

## Notas relacionadas
- [[Arquitetura]] — estrutura de pastas, como memória/identidade/skills se conectam
- [[Funcionalidades]] — as 15 skills, agrupadas por área
- [[Infraestrutura e Deploy]] — dependências externas (APIs, MCPs, scripts)
- [[Estado Atual]] — o que já está preenchido vs. o que falta neste clone específico
- [[Próximos Passos]] — o que o README pede pra fazer a seguir

## Linha do tempo de decisões
- **2026-08-01** — repositório clonado em `~/VITOR/MazyOS`, análise inicial
  feita a pedido do Vitor. `/instalar` ainda **não foi rodado**: `_memoria/`,
  `identidade/design-guide.md` e o `CLAUDE.md` estão todos no estado
  template (campos em branco), a pasta continua com o nome `MazyOS`.
