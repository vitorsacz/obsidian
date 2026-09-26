---
tags: [projeto, linesapp, lint, execucao]
criado: 2026-07-30
---

# LinesApp — Execução Rodada 2: correção do lint (2026-07-30)

← [[LinesApp - Visão Geral]] · relacionado: [[Lint - Auditoria]], [[Execução - Rodada 1 (2026-07-30)]]

Correção de **todos** os achados mapeados em [[Lint - Auditoria]], no mesmo dia do mapeamento. `npm run lint` foi de **207 problemas (43 erros / 164 avisos)** para **0**. Trabalho feito sobre o branch `feature/infra-clean-code`, em cima do que já tinha sido commitado em `04d79ed` (ver [[Execução - Rodada 1 (2026-07-30)]]).

## O que foi corrigido, por categoria

**🔴 Bugs reais (36 `no-dupe-keys` + 5 `no-undef` + 1 `no-unused-expressions`):**
- 36 chaves de estilo duplicadas removidas em 14 arquivos `*.styles.js` (principalmente `Alertas/**`), mantendo sempre o valor que já estava vigente em runtime (o da segunda ocorrência, que é o que o JS aplica quando há chave repetida).
- `Home.js`: `Alert.alert(...)` era chamado sem importar `Alert` de `react-native` — adicionado ao import.
- `configuracoes.js`: 4 `case` do switch (`Tema`, `Geral`, `Privacidade`, `Sobre`) chamavam setters (`setTema`, etc.) que nunca existiram — eram código morto inalcançável (os itens de menu correspondentes não têm `onPress` que dispare esses `case`s), removidos.
- `perfil.js:483`: operador vírgula por engano (`firebase.auth().signOut(), navigation.navigate(...)`) em vez de dois statements — separado em duas linhas, comportamento idêntico.

**`react-hooks/exhaustive-deps` (4×):** dependências faltando adicionadas (`navigation`, `db`, `firestore` × 2) — todas são valores estáveis entre renders (`useNavigation()`, `getFirestore()` retornam a mesma referência), então não introduzem loop extra de re-render.

**`import/no-duplicates` (25×):** imports duplicados consolidados. Destaque: as 4 rotas `MetroSp`/`MetroCPTM`/`MetroNoticiando`/`DiarioDoTransporte` em `stackRoutes.js` importavam o mesmo `webViewTwitter` 4 vezes sob nomes diferentes — consolidado em um único `import WebViewTwitter`, reusado nas 4 `<Stack.Screen>`. **A duplicação das 4 rotas em si continua a mesma** (não mudei quantas rotas existem nem seus nomes) — só o import redundante foi limpo.

**`no-unused-vars` (123×):** imports, variáveis e states não usados removidos em 31 arquivos. Quando só o setter de um `useState` era usado (ex. `const [selectedStation, setSelectedStation]` onde só `setSelectedStation` era chamado), troquei por elision (`const [, setSelectedStation]`) para não quebrar as chamadas existentes. Em dois `catch (err)` onde o erro nunca era lido, troquei para `catch {}` (optional catch binding).

**Regras menores:**
- `eqeqeq` (6×): `==`/`!=` → `===`/`!==`.
- `import/no-named-as-default` (4×): 4 arquivos de estilo (`Alertas`, `Cadastro`, `Login`, `politicaPrivacidade`) exportavam `styles` tanto como `export const` quanto `export default` — removida a exportação nomeada redundante (nada no código fazia `import { styles }`, só `import styles`).
- `react/no-unescaped-entities` (2×): aspas em JSX trocadas por `&quot;`.
- `import/no-named-as-default-member` (1×): `noticias.js` trocou `import cheerio from 'cheerio'; cheerio.load(...)` por `import { load } from 'cheerio'; load(...)`.

## Erro cometido e corrigido no processo

Ao remover imports não usados em `suaPassagem.js`, removi `TouchableOpacity` por engano (ele estava sendo usado em 3 lugares no JSX, só não tinha sido flagrado pelo ESLint na mesma leitura rápida). Percebido e corrigido antes de seguir adiante — reforça por que a verificação final via bundler é necessária, não só re-rodar o lint.

## Verificação

- `npx eslint src App.js` → **0 erros, 0 avisos**.
- Bundle Android via Metro (`expo start` + fetch do bundle) compilou com sucesso (200 OK, ~8,8MB) depois de todas as mudanças.
- Nenhum `no-undef` novo apareceu em nenhum momento do processo, o que indica que nenhuma remoção de import quebrou uma referência real (fora do caso do `TouchableOpacity` acima, pego antes do lint final).

## Estado final

51 arquivos alterados (61 inserções, 213 remoções — a maior parte é código morto/duplicado saindo). **Ainda não commitado.** Branch `feature/infra-clean-code`, em cima do commit `04d79ed`.
