---
tags: [projeto, linesapp, nomenclatura, refactor]
criado: 2026-07-30
---

# LinesApp — Auditoria de Nomenclatura

← [[LinesApp - Visão Geral]]

**Escopo revisado:** 210 arquivos (`src/img` 111, `src/pages` 83, `assets/` 4, restante em `src/Componentes`, `src/data`, `src/Routes`, `src/services`, `src/utils`).

**Resultado:** nenhuma convenção de casing é aplicada de forma consistente em nenhum lugar da árvore. Em `src/pages` (31 pastas de rota de topo): 8 PascalCase, 16 lowercase-sem-separador, 7 camelCase — **3 convenções competindo na mesma pasta, nenhuma dominante** (~52% no máximo). Estimativa geral: **menos de 20%** dos arquivos/pastas seguem uma convenção consistente com os "vizinhos"; **mais de 80%** divergem.

Só duas ilhas de consistência real: `src/services/` (`directionsService.js`, `placesService.js`) e a família de assets `pin-*`.

## 1. Casing misturado na mesma pasta

`src/pages/` (nível raiz) mistura:
- **PascalCase**: `Alertas`, `Cadastro`, `Home`, `Login`, `NavegacaoAoVivo`, `Operacao`, `RotaBusca`, `RotaOpcoes`
- **lowercase-sem-separador**: `alexa`, `aprovado`, `configuracoes`, `construcao`, `extrato`, `inicio`, `introducao`, `jogos`, `loading`, `noticias`, `pagamento`, `passagens`, `perfil`, `sair`, `valida`, `webview`
- **camelCase**: `compartilharLocalizacao`, `locaisSalvos`, `noticiasExame`, `politicaPrivacidade`, `sobreNos`, `suaPassagem`, `telaAjuda`
- **ALLCAPS**: `src/pages/Alertas/SOS/`

**Arquivo de estilo**: `style.js` (21 arquivos, ex. `Home/style.js`, `Login/style.js`, `Cadastro/style.js`) vs `styles.js` (17 arquivos, ex. `Alertas/styles.js`, `alexa/styles.js`, `construcao/styles.js`) — alternando sem critério.

`src/Routes/`: `index.js` (lowercase) ao lado de `stackRoutes.js`, `tabRoutes.js` (camelCase).

Assets — mesma família, casing diferente:
- `src/img/seta*`: `setaDireita.png`, `setaVoltar.png` (camelCase) vs `seta-clara.png`, `seta-escura.png`, `seta-esq-branca.png`, `seta-esq-escura.png` (kebab-case)
- `src/img/logo*`: `logo-alexa.png`, `logo-exame.png`, `logo-loctime.png` (kebab) vs `logoLines.png`, `logoLinesAzul.png` (camelCase)
- `Metro.png` (PascalCase) vs `metro-escuro.png` (kebab) — mesmo assunto, casing diferente

## 2. Padrão de arquivo de entrada inconsistente

- `src/pages/Cadastro/Cadastro.js` e `src/pages/Login/Login.js` divergem do `index.js` usado pelas outras 29 pastas de página. **Nota (revisão abaixo):** dos dois lados, é `Cadastro`/`Login` que está certo — a convenção proposta na seção final inverteu a recomendação original deste documento, de "todo mundo vira `index.js`" para "todo mundo vira `NomeDaTela.js`, como `Cadastro`/`Login` já fazem".
- `src/pages/Alertas/alerta.js` — arquivo solto de nome genérico, ao lado do `index.js` da pasta, sem seguir o padrão `index.js`/`styles.js`.
- `src/pages/webview/twitter/webViewTwitter/index.js` — caminho aninhado redundante, repete o conceito "webview" três vezes.

## 3. Português/Inglês misturado

- Pastas de arquitetura em inglês (`Routes`, `services`, `utils`, `data`) vs pastas de domínio quase todas em português (`acidente`, `configuracoes`, `noticias`, `passagens`, `perfil`, `sair`, `valida`...), com uns poucos nomes em inglês (`Home`, `Login`) misturados sem critério.
- `src/Componentes/` (português) é a pasta-mãe de componentes, mas seu conteúdo é em inglês: `Firebase.js`, `BackHeader.js`.
- `src/img/iconeUser.png` (inglês "User") vs `src/img/iconeUsuario.png` e `src/img/iconeUsuarioLogin.png` (português "Usuario") — mesmo conceito, idioma inconsistente, provavelmente quase-duplicatas.

## 4. Assets duplicados / quase-duplicados

- `src/img/icons8-a-carregar-90 (1).png` e `src/img/icons8-ok-96 (1).png` — sufixo `(1)` clássico de download duplicado, nunca renomeado.
- `src/img/passagens.png` vs `src/img/Passagens5.png` — sufixo numérico + casing diferente.
- `src/img/LocaisSalvos.png` vs `src/img/locaisSalvosMenu.png` — mesmo conceito, casing/sufixo diferente.
- `src/img/alexa-lines.png`, `logo-alexa.png`, `icon-alexa.png`, `speakers-alexa.png` — mesmo conceito "alexa" espalhado com prefixo/sufixo sem padrão.
- Possível sobreposição de conceito de tela: `src/pages/Home/`, `src/pages/inicio/`, `src/pages/introducao/`, `src/pages/loading/` (todas plausivelmente telas de "abertura/início" sob nomes diferentes — vale confirmar se todas são realmente distintas).
- `src/img/chuvaT.png`, `crimeT.png`, `descarrilamentoT.png`, `engrenagemT.png`, `quedaT.png` — sufixo "T" sem significado documentado nem contraparte sem o sufixo.

## 5. Pastas / pluralização inconsistente

- `src/img/` (singular, 111 arquivos) e `assets/` da raiz (4 arquivos, ícones do Expo) são dois locais de asset desconectados — não existe um `assets/images/` único.
- `src/pages/webview/twitter/`, `.../mapaoffline/`, `.../sampaPe/` — `sampaPe` em camelCase dentro do mesmo pai que usa lowercase-sem-separador nas outras duas.

## 6. Extensão de arquivo inconsistente

- `src/img/icones.psd` — arquivo fonte do Photoshop commitado junto dos PNGs de produção.
- `src/img/Mapa-Metropolitano.pdf` ao lado de `Mapa-Metropolitano.png` — mesmo nome-base, dois formatos.
- `src/img/Loading.gif` — único `.gif` numa pasta majoritariamente `.png`/`.jpg`.
- `src/img/qrcode.jpg` — único `.jpg` minúsculo numa pasta majoritariamente `.png`.
- Nenhum `.jsx` em `src/` (tudo é `.js`, inclusive componentes React) — consistente internamente, mas fora da convenção comum de RN/Expo de usar `.jsx` para componentes.

## 7. Espaços / caracteres especiais

- `src/img/icons8-a-carregar-90 (1).png` — espaço + parênteses.
- `src/img/icons8-ok-96 (1).png` — espaço + parênteses.
- `src/img/icons8-cartão-de-crédito-96.png` — caracteres acentuados no nome do arquivo.

---

## Convenção proposta (para adotar daqui pra frente)

Não há convenção legada forte o bastante para "vencer" — então a proposta é greenfield, escolhendo o que já é mais usado como base:

1. **Pastas de tela** (`src/pages/**`, futuramente `src/screens/**`): `PascalCase`, sempre em **um único idioma** (recomendo português, já que é a maioria e é o idioma do domínio/negócio do app) — ex. `Login`, `Cadastro`, `Perfil`, `TelaAjuda` (não `telaAjuda`).
2. **Arquivo de entrada da tela**: ~~sempre `index.js`~~ **revisado → sempre `<NomeDaTela>.js`, descritivo e igual ao nome da pasta** (ex. `Login/Login.js`, `Perfil/Perfil.js`, `TelaAjuda/TelaAjuda.js`). `index.js` genérico repetido em ~30 pastas prejudica navegação por abas do editor, busca fuzzy (Ctrl+P), leitura de stack trace de erro e diffs de PR — tudo aparece com o mesmo nome, sem contexto de qual tela é. `Cadastro/Cadastro.js` e `Login/Login.js` já seguem esse padrão certo hoje; são os outros 29 que devem se adequar a eles, não o contrário.
3. **Arquivo de estilo**: ~~sempre `styles.js`~~ **revisado → sempre `<NomeDaTela>.styles.js`** (ex. `Login/Login.styles.js`), pelo mesmo motivo do item acima — nome descritivo e co-localizado em vez de `styles.js` genérico repetido em toda pasta.
4. **Pastas de arquitetura** (`Routes`, `services`, `utils`, `data`, `components`): inglês, `camelCase` para arquivos dentro delas (já é o padrão em `services/`).
5. **Renomear `Componentes/` → `components/`** por consistência com as demais pastas de arquitetura.
6. **Assets** (`src/img/` → sugestão `src/assets/images/`): `kebab-case` sempre, sem acento, sem espaço, sem sufixo de download tipo `(1)`; variantes de cor/tema com sufixo padronizado (`-light`/`-dark` em vez de `-clara`/`-escura` misturado com `Voltar`/`Direita`).
7. Nunca commitar arquivos fonte de design (`.psd`) nem duplicatas de formato (`.pdf` + `.png` do mesmo asset) na pasta de produção — mover pra uma pasta `design/` fora do bundle do app, se precisar manter.

⚠️ Aplicar essa convenção em massa exige renomear ~200 arquivos **e** atualizar todos os imports correspondentes — está listado como item separado em [[Plano de Melhorias]], não deve ser feito manualmente arquivo por arquivo.

**Custo de migração dos itens 2 e 3 é baixo, na prática:** os imports de tela estão centralizados em só **2 arquivos** (`src/Routes/stackRoutes.js` e `src/Routes/tabRoutes.js` — nenhum outro lugar do `src/` importa de `pages/`). Trocar de resolução implícita (`./Login` → `Login/index.js`) para explícita (`./Login/Login`) só toca esses dois arquivos de rota, mais a linha `import styles from './styles'` dentro de cada tela (mecânico, entra no mesmo script de rename).

Contexto de arquitetura em [[Arquitetura]], infraestrutura em [[Infraestrutura]].
