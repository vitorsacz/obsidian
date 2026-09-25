#projeto #mazyos #arquitetura

Ver [[Visão Geral]] para o contexto do produto.

## Estrutura de pastas

```
MazyOS/
├── CLAUDE.md                 regras de operação (lido sempre, override de comportamento)
├── README.md                 pitch + comandos de instalação
├── .claude/skills/            as 15 skills (comandos /nome)
├── _memoria/                  cérebro — empresa.md, preferencias.md, estrategia.md
├── identidade/                rosto — design-guide.md (cores, tipografia, regras visuais)
├── marketing/                 histórico vivo — conteúdo, SEO, campanhas (versionado)
├── saidas/                    outputs pontuais — análises, emails, relatórios avulsos
├── dados/                     drop zone — CSV/PDF/planilha que o usuário solta pra skill ler
├── scripts/                   Node/Python que skills chamam (nasce vazio, populado sob demanda)
└── templates/                 material de referência do próprio MazyOS
    ├── skills/catalogo.md      skills externas prontas pra instalar (Schwartz Copy, PDF, etc.)
    ├── ferramentas/catalogo.md APIs/CLIs/MCPs disponíveis (Playwright, Gemini, Notion, Gmail…)
    ├── perfis/                 4 moldes de CLAUDE.md (agência, empresa, freelancer, solopreneur)
    └── identidade/exemplos/    design-guides de exemplo por perfil
```

## Como as camadas se conectam

`CLAUDE.md` (raiz) é a única fonte de regras de comportamento — ele instrui o
Claude a:
1. Ler `_memoria/empresa.md`, `preferencias.md`, `estrategia.md` no início de
   toda conversa (contexto do negócio).
2. Consultar `identidade/design-guide.md` antes de qualquer tarefa visual.
3. Verificar se existe skill relevante em `.claude/skills/` antes de executar
   qualquer tarefa à mão.
4. Perguntar (nunca decidir sozinho) antes de: salvar uma correção do usuário
   em memória, virar uma tarefa repetível em skill nova, ou atualizar memória
   depois de uma mudança relevante. Isso é o loop de "aprender com correções"
   descrito no próprio `CLAUDE.md`.

Ou seja: **o `CLAUDE.md` raiz é meta — ele não fala do negócio, fala de como
o sistema deve se comportar.** Quem fala do negócio são os arquivos em
`_memoria/` e `identidade/`, que hoje estão todos vazios (ver [[Estado Atual]]).

## Skills — como funcionam

Cada skill é uma pasta `.claude/skills/<nome>/SKILL.md` com frontmatter YAML
(`name`, `description` — que define os gatilhos de linguagem natural que
disparam a skill) + instruções em markdown de workflow. O Claude Code
descobre as skills automaticamente pelo `description`; não precisa chamar
`/nome` literalmente, frases como "cria um carrossel sobre X" já disparam
`/carrossel`.

Skills declaram **Dependências** explícitas no topo (ex: "Contexto do
negócio: `_memoria/empresa.md`") — é assim que elas ficam calibradas ao tom
e ao negócio sem re-perguntar toda vez.

Ver [[Funcionalidades]] para a lista completa e [[Infraestrutura e Deploy]]
para o que cada skill precisa de fora (chaves de API, scripts, MCPs).

## Perfis de negócio (`templates/perfis/`)

O `/instalar` provavelmente escolhe um desses 4 moldes de `CLAUDE.md` conforme
o tipo de negócio do usuário, e adapta a estrutura de pastas em cima da base
comum:
- **Agência** — soma `clientes/`, `briefings/`, `propostas/`
- **Empresa estruturada** — soma `comercial/`, `financeiro/`, `rh/`, `operacoes/`
- **Freelancer** — soma `clientes/`, `propostas/`, `tarefas.md`
- **Solopreneur** — soma `produtos/`, `audiencia/`

Isso não foi confirmado lendo o código do `/instalar` em detalhe — é inferido
da existência dos 4 templates em `templates/perfis/`.

## Versionamento

Tudo em `marketing/`, `saidas/`, `_memoria/`, `identidade/` versiona no Git via
a skill `/salvar` (commit + push, configura o remote na primeira vez). O
`.gitignore` exclui `.claude/projects|sessions|shell-snapshots|telemetry|
backups` (estado local de sessão do Claude Code), `node_modules/`, e o
conteúdo de `dados/` (drop zone — só o `README.md` dela é versionado).
