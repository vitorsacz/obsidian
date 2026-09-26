---
tags: [projeto, linesapp, execucao]
criado: 2026-07-30
---

# LinesApp — Execução Rodada 1 (2026-07-30)

← [[LinesApp - Visão Geral]] · relacionado: [[Plano de Melhorias]], [[Nomenclatura - Auditoria]]

Registro do que foi efetivamente executado no código a partir do [[Plano de Melhorias]], em modo autônomo, numa sessão só. **Nada foi commitado** — as mudanças ficaram na working tree do branch `feature/infra-clean-code` (topo em `3464bd3`), para o usuário revisar o diff antes de decidir commitar.

## Descoberta importante no início

O branch ativo tinha mudado de `fix/version-and-compability` (onde a migração pro Expo SDK 54 tinha sido feita numa sessão anterior, mas nunca commitada) para `feature/infra-clean-code`. As mudanças da migração SDK 54 ficaram preservadas em `git stash` (`stash@{0}: WIP on fix/version-and-compability`), intocado. O branch atual estava com o `package.json` já em `expo ~54.0.0`, mas faltando peer deps (`expo-font`, `react-native-worklets`, `expo-system-ui`) e com a pasta `android/` nativa ainda gerada para SDK 52 (chave do Maps antiga hardcoded). Reapliquei essa correção diretamente neste branch (sem tocar no stash antigo) antes de seguir com o resto.

## O que foi feito

**Fundação (SDK 54):**
- Instaladas as peer deps faltantes (`expo-font`, `react-native-worklets`, `expo-system-ui`) e regenerada a pasta `android/` via `expo prebuild --clean` (agora `compileSdk`/`targetSdk` 36, gradle wrapper 8.14.3).

**🟠 Código morto e dependências:**
- Removidos `src/pages/noticiasExame/`, `src/pages/sair/`, `src/pages/locaisSalvos/` (confirmado, via grep, que não eram referenciados em nenhuma rota).
- Removida a dependência `react-native-paper` (zero usos no código).
- Consolidados os ícones: `src/pages/perfil/perfil.js` agora usa `Ionicons` de `@expo/vector-icons` em vez de importar `react-native-vector-icons` direto; a dependência `react-native-vector-icons` foi removida do `package.json`.
- Corrigido de brinde um bug real encontrado pelo lint: `src/pages/telaAjuda/style.js` tinha a chave `titulo` duplicada no objeto de estilos (a segunda sobrescrevia a primeira silenciosamente) — removida a duplicata.

**🔴 Infra:**
- `eas.json`: perfil `production` mudado de `buildType: "apk"` para `"app-bundle"` (AAB), alinhado ao que a Play Store exige hoje.

**📄 Documentação e tooling:**
- Criado `README.md` (stack, setup, variáveis de ambiente, comandos, build EAS) e `.env.example`.
- Criado `eslint.config.js` (flat config, `eslint-config-expo` + `eslint-config-prettier`) e `.prettierrc.json`. Scripts `npm run lint` e `npm run format` adicionados ao `package.json`.
- Criado `.github/workflows/lint.yml` — roda `npm ci` + `npm run lint` em push para `main` e em PRs.
- **Baseline do lint, não corrigido nesta rodada** (é trabalho 🟢 separado, ~poucas horas de triagem manual): 43 erros + 164 avisos, majoritariamente `no-unused-vars` (imports/variáveis não usados) espalhados pelas telas antigas.

**🟡 Nomenclatura (itens 2 e 3 da convenção em [[Nomenclatura - Auditoria]]):**
- Renomeados os arquivos de entrada e estilo de **39 telas** (incluindo as aninhadas em `Alertas/**` e `webview/**`) de `index.js`/`style(s).js` para `<NomeDaTela>.js`/`<NomeDaTela>.styles.js`.
- Atualizados os imports em `src/Routes/stackRoutes.js` e `src/Routes/tabRoutes.js` (os 2 únicos arquivos que importam de `pages/`, confirmado antes de começar) e a linha de import de estilo dentro de cada tela.
- Renomeada a pasta `src/Componentes/` → `src/components/` (item 5 da convenção), com os 6 imports que apontavam pra ela corrigidos (`App.js`, `stackRoutes.js`, e as 4 telas que chamam Firebase direto: `Home`, `Login`, `Cadastro`, `perfil`).
- **Imprevistos corrigidos na mão** (o script de rename cobriu o padrão comum `./style`/`./styles`, mas 7 arquivos usavam variações que ele não previa):
  - `Login/Login.js` importava estilo via `../Login/style` (caminho redundante) em vez de `./style`.
  - `aprovado`, `pagamento`, `suaPassagem`, `valida` também usavam a variação `./../<própria-pasta>/style`.
  - **Achado curioso**: `extrato/extrato.js` importava o estilo de `../valida/style` — ou seja, a tela "Extrato" sempre usou os estilos da tela "Valida" (provavelmente `extrato` foi criada copiando `valida` como base e o import nunca foi trocado). Mantive esse comportamento exatamente como estava (agora aponta para `../valida/valida.styles`) para não mudar a aparência da tela sem essa ser a tarefa — mas é uma pista de que a tela `extrato` pode não ter estilo próprio de verdade. Vale investigar depois.

## O que ficou de fora (de propósito)

Itens do [[Plano de Melhorias]] que exigem acesso/decisão do usuário ou são refactors grandes demais para fazer "às cegas" num projeto sem testes automatizados:

- 🔴 **Rotacionar/restringir a chave do Google Maps exposta** — exige acesso ao Google Cloud Console do usuário, não dá pra fazer via código.
- 🔴 **Keystore de release própria** — decisão de alto risco (perder a keystore trava updates futuros na Play Store para sempre); melhor caminho é `eas credentials` com o usuário logado na conta Expo dele, não algo pra gerar sozinho.
- 🟠 **Resolver a duplicação das 4 rotas `webViewTwitter`** — não dá pra saber sem o usuário se cada uma deveria mostrar conteúdo diferente ou se é pra consolidar numa rota só.
- 🟡 Quebrar o monólito `Home/Home.js` (2497 linhas), criar camada de dados pro Firebase, `AuthContext`, migrar classes pra function components, mover o scraping de notícias pra server-side — refactors de comportamento real, precisam de QA manual tela por tela (sem suíte de testes hoje pra pegar regressão), então ficam para uma próxima rodada revisada com o usuário.
- 🟢 Corrigir os 43 erros/164 avisos de lint, testes automatizados, TypeScript incremental — trabalho contínuo, tooling já está pronto pra isso.
- Renomeação de assets (`src/img/**`, item 6 da convenção) — deliberadamente fora de escopo: são ~111 arquivos referenciados via `require()` espalhados pelo código, e um path errado não dá erro de build (só imagem faltando em runtime), então só é seguro fazer com QA visual tela por tela.

## Verificação feita

- `npx expo-doctor`: 17/18 (só o aviso esperado de "app config fields not synced" em projeto com prebuild).
- Bundle Android via Metro (`expo start` + fetch do bundle) compilou com sucesso (200 OK, ~8,8MB) depois de corrigir os 7 imports de estilo com padrão fora do comum.
- `npx eslint src App.js`: 43 erros / 164 avisos — mesmo baseline de antes das mudanças (nenhum erro novo introduzido pelas renomeações).

## Estado final

Working tree do branch `feature/infra-clean-code` tem ~159 entradas modificadas (78 renomeações, 5 remoções, o resto são as mudanças de conteúdo/config), nada commitado. Próximo passo é o usuário revisar (`git diff`/`git status`) e decidir o commit.
