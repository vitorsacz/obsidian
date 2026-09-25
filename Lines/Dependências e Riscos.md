#projeto #lines

Auditoria de dependências e riscos de segurança/compatibilidade. Ver [[Visão Geral]] para contexto. Analisado em 2026-07-23.

## ✅ Resolvido em 2026-07-23

### 1. `package.json` incoerente — resolvido, versão final é **SDK 54** (não 57)

Estava com `expo` em `^57.0.8` mas o resto das dependências ainda na geração SDK 52. Primeira tentativa: completar upgrade para SDK 57 via `npx expo install --fix` (raciocínio: "Expo Go só roda o SDK mais recente"). **Não funcionou na prática** — a aprovação do Expo Go compatível com SDK 57 estava atrasada na App Store (mesmo problema que já tinha acontecido com SDK 55 em maio/2026). O usuário testou com projetos Expo em branco e confirmou que o Expo Go instalado no iPhone dele roda **SDK 54**. Projeto reajustado para `"expo": "~54.0.0"` + `npx expo install --fix`, resultando em `react-native` → 0.81.5, `react` → 19.1.0, `reanimated` → ~4.1.1, `gesture-handler` ~2.28, `screens` ~4.16, `safe-area-context` ~5.6, `react-native-maps` 1.20.1, `react-native-webview` 13.15.0, `babel-preset-expo` ~54.0.10, todos os `expo-*`.

No processo (nas duas tentativas), `react-native-fast-image` (já confirmado sem uso, ver seção "sem uso" abaixo) travava o `npm install` por exigir React 17/18 — foi removido de vez. `@expo/vector-icons` (usado em `stackRoutes.js`) e `babel-preset-expo` precisaram ser adicionados como dependências diretas (em SDK 52 vinham embutidos indiretamente). Validado com `npx expo export --platform ios` — bundle sem erros. Detalhe completo, incluindo o porquê da mudança de rota, em [[Ambiente de Desenvolvimento (Mac)]].

⚠️ Isso significa que a versão do Expo Go suportada por vez nas lojas pode continuar mudando (Apple tem atrasado aprovações repetidamente) — se o app parar de abrir no Expo Go de novo no futuro, o primeiro passo é testar qual SDK um projeto Expo em branco consegue abrir, antes de mexer no LinesApp.

### 2. Chaves do Google Maps e do Firebase — tiradas do código-fonte

O usuário já tinha rotacionado as duas chaves (valores novos, diferentes dos antigos commitados) e criado um `.env` — faltava só a mecânica funcionar de verdade. Concluído: `app.json` (JSON estático, não lê `.env`) virou `app.config.js`, que lê `process.env.GOOGLE_MAPS_API_KEY` para Android **e** iOS (antes só existia para Android); `Firebase.js` lê `process.env.EXPO_PUBLIC_FIREBASE_API_KEY`. `.env` foi adicionado ao `.gitignore` (não estava — as chaves novas estavam a um `git add .` de serem commitadas de novo). Detalhe completo em [[Ambiente de Desenvolvimento (Mac)]].

⚠️ As chaves **antigas** (`AIzaSyB_7_8xkzquPPWR93cihI7AwRkIYWrgBag` para Maps, e a antiga do Firebase) continuam permanentemente no histórico do git de commits anteriores — isso não tem como ser desfeito sem reescrever histórico. Como já foram rotacionadas, o risco prático é baixo, mas vale confirmar no Google Cloud Console que a chave antiga do Maps foi de fato desativada/restringida, não só substituída no app.

`Firebase.js` ainda mistura a API legada `firebase/compat/*` com a API modular (`initializeAuth`, `getReactNativePersistence`) — funciona, mas seria bom unificar numa migração futura (ver [[Próximos Passos]]).

## 🟡 Desatualizado, vale planejar upgrade

| Pacote | Atual | Observação |
|---|---|---|
| `firebase` | `^10.10.0` (instalado `10.14.1`) | SDK já está na linha v12 (12.16.0 em jul/2026) — duas major versions atrás. Migração v10→v12 tem breaking changes. |
| `react-native-maps` | `1.18.0` | Atual é `1.29.0`. Pacote ainda ativamente mantido (não é urgente, só desatualizado). |
| `react-native-vector-icons` | `^10.0.2` | Único uso é em `perfil/index.js` (`Icon` de `Ionicons`). Não há linking nativo configurado no Android (falta `fonts.gradle`) — risco de ícones não aparecerem (tofu boxes) em build de release. Resto do app já usa `@expo/vector-icons`, que inclui `Ionicons` — mais simples trocar esse import único e remover a dependência inteira. |
| `react-native-app-intro-slider`, `react-native-communications`, `react-native-email` | — | Usados de verdade, mas pouco mantidos. Baixa urgência; candidatos a substituir por `Linking`/soluções nativas no futuro. |

## 🟢 Sem uso nenhum no código — seguro remover

`react-native-fast-image` **já foi removido** (2026-07-23) — travava a instalação por exigir React 17/18, incompatível com React 19 do SDK 57 (ver acima).

Ainda no `package.json`, confirmado via grep em `src/` e `App.js` — zero imports:

`fix`, `uuid`, `react-native-elements`, `react-native-mail`, `htmlparser2`, `node-html-parser`, `react-native-html-parser`, `react-native-htmlview`, `domhandler`

Destaque: `"fix": "^0.0.3"` no `dependencies` quase certamente é lixo/typo acidental (nome de pacote sem relação nenhuma com o projeto) — o `npm install` inclusive avisa `EBADENGINE` porque ele e sua dependência `pipe@0.0.1` pedem Node `0.2.5` (versão de ~2011), confirmando que é um pacote abandonado/irrelevante. De 4 bibliotecas de parsing de HTML instaladas (`cheerio`, `htmlparser2`, `node-html-parser`, `react-native-html-parser` + `react-native-htmlview`), **só `cheerio` é usado de fato** (nos dois scrapers de notícias).

## 🔧 Tooling ausente

Nenhum ESLint, Prettier, Jest ou TypeScript configurado (`devDependencies` só tem `@babel/core`). Não há `.eslintrc*`, `tsconfig.json` ou `jest.config.*` no repo. Para um projeto com ~35 telas isso é uma lacuna real de manutenibilidade.

## Sinal de manutenção

Git log mostra desenvolvimento ativo até abril/2025 (`f157b63`), depois silêncio total até esta análise (jul/2026) — mais de um ano parado. O upgrade de SDK que tinha ficado pela metade foi completado em 2026-07-23 (ver acima), mas as mudanças ainda não foram commitadas.

## Notas relacionadas
- [[Visão Geral]]
- [[Problemas Conhecidos]]
- [[Próximos Passos]]
