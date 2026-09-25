---
tags: [projeto, lines, ambiente, mac]
data: 2026-07-23
---

#projeto #lines

Avaliação de se/como rodar o LinesApp na máquina do usuário, e correção dos bloqueios de código que impediam o app de buildar. Ver [[Visão Geral]] para contexto do projeto. Máquina: MacBook Apple **M4**, macOS **26.5.1**, 311 GB livres — hardware não é limitação.

## O que já está instalado

| Ferramenta | Status |
|---|---|
| Node.js | ✅ v25.9.0 — mas é uma versão **Current/não-LTS** (releases ímpares do Node não viram LTS). Tooling do React Native/Expo é testado contra LTS (18/20/22/24); recomendo instalar Node 22 LTS via `nvm` para evitar erros obscuros de bundling. Ainda assim, o build de teste funcionou com a v25. |
| npm | ✅ 11.12.1 |
| Homebrew | ✅ 6.0.12 — facilita instalar o que falta |
| Java | ✅ 21 (LTS) — só é usado se algum dia precisar buildar Android |
| Xcode | ⚠️ Só as **Command Line Tools** — não o Xcode completo. Sem `xcodebuild`, sem simulador iOS disponível ainda (não é necessário para o caminho recomendado, ver abaixo). |
| CocoaPods | ❌ não instalado (só necessário se instalar o Xcode completo depois) |
| Watchman | ❌ não instalado (recomendado pelo Metro/RN, opcional) |
| Android Studio / SDK | ❌ nada instalado, `ANDROID_HOME` vazio, sem `adb` |
| Dispositivo físico Android | ❌ usuário não tem |
| iPhone físico | ✅ tem — caminho de teste recomendado |

## ✅ Bloqueios de código — resolvidos em 2026-07-23

Todos os itens abaixo foram corrigidos diretamente no repositório (`/Users/MAC/VITOR/LinesV1`) e validados com um bundle real (`npx expo export --platform ios`, sucesso).

### 1. SDK do Expo — **SDK 54** (não 57, correção de rota)

Primeiro completei o upgrade para SDK 57 (raciocínio: "Expo Go só roda o SDK mais recente"). Na prática isso não funcionou — a aprovação do Expo Go compatível com SDK 57 na App Store está atrasada (mesmo problema recorrente que já tinha acontecido com o SDK 55 em maio/2026). O usuário testou na própria máquina criando projetos do zero: **SDK 57 não abriu no Expo Go dele, SDK 54 abriu normalmente** — então o Expo Go instalado no iPhone dele está na geração 54. Reajustei o projeto para SDK 54 (`npx expo install --fix` com `"expo": "~54.0.0"`), que realinhou:

- `react-native` → **0.81.5**
- `react` → **19.1.0**
- `react-native-reanimated` → **~4.1.1**
- `react-native-gesture-handler` ~2.28, `react-native-screens` ~4.16, `react-native-safe-area-context` ~5.6, `react-native-maps` 1.20.1, `react-native-webview` 13.15.0, `babel-preset-expo` ~54.0.10, todos os `expo-*` → versões SDK 54.

Durante o processo (tanto na tentativa 57 quanto na 54), três problemas adicionais apareceram e foram resolvidos:
- **`react-native-fast-image` removido** — já estava confirmado como sem nenhum uso no código ([[Dependências e Riscos]]), e travava a resolução de dependências por exigir React 17/18 (incompatível com React 19). Removê-lo desbloqueou o `npm install`.
- **`@expo/vector-icons` adicionado como dependência direta** — em SDK 52 vinha embutido via `expo`; em SDKs mais novos precisa ser instalado explicitamente (`npx expo install @expo/vector-icons`); sem isso o bundle falhava ao resolver os ícones usados no menu (`stackRoutes.js`).
- **`babel-preset-expo` adicionado como devDependency direta** — antes só existia aninhado dentro de `node_modules/expo`, e o Babel não conseguia resolvê-lo na raiz.
- **`expo-status-bar` removido de `plugins` no `app.config.js`** — tentei registrá-lo como config plugin (sugestão do `expo install --fix` durante a tentativa com SDK 57), mas no SDK 54 ele não exporta um config plugin válido e isso gera erro; removido, não é necessário.

**Se no futuro a Apple aprovar o Expo Go de uma versão mais nova e o app parar de abrir de novo**: o sintoma é o Expo Go recusando o projeto por "SDK incompatível" — nesse caso, criar um projeto Expo em branco (`npx create-expo-app`) é a forma mais rápida de descobrir qual SDK o Expo Go instalado realmente suporta, antes de mexer no LinesApp.

### 2. Import quebrado em `Firebase.js` — corrigido

*(Correção do meu próprio erro: eu tinha dito antes que o `.env` estava com valores vazios — não estava, o usuário já tinha preenchido com chaves reais e novas/rotacionadas; meu grep anterior só escondia os valores no output, não confirmava que estavam vazios.)*

O projeto não precisa de `react-native-dotenv` — o Expo (desde o SDK 49) já carrega `.env` automaticamente e expõe variáveis prefixadas com `EXPO_PUBLIC_` direto no bundle JS, sem pacote nem plugin extra. Ajustes feitos:
- `.env`: `FIREBASE_API_KEY` renomeada para `EXPO_PUBLIC_FIREBASE_API_KEY` (precisa do prefixo pra ficar acessível no código do cliente). `GOOGLE_MAPS_API_KEY` mantida sem prefixo (só é usada em `app.config.js`, do lado do build, nunca vai pro bundle do cliente).
- `src/Componentes/Firebase/Firebase.js`: removida a linha de import quebrada (`import FIREBASE_API_KEY from "@env"`); a config agora lê `process.env.EXPO_PUBLIC_FIREBASE_API_KEY` diretamente.

### 3. `app.json` → `app.config.js` — migrado

`app.json` é JSON estático e nunca conseguiria ler `.env`. Convertido para `app.config.js` (módulo JS, lido em Node no momento do build), que já resolve `process.env.GOOGLE_MAPS_API_KEY` de verdade — confirmado com `npx expo config` (a chave aparece corretamente injetada tanto em `android.config.googleMaps.apiKey` quanto em `ios.config.googleMapsApiKey`).

### 4. Chave do Google Maps para iOS — adicionada

`Home/index.js` força `provider={MapView.PROVIDER_GOOGLE}`, mas só havia chave configurada para Android. Adicionei `ios.config.googleMapsApiKey` em `app.config.js`, reaproveitando a mesma variável `GOOGLE_MAPS_API_KEY` do `.env` (a mesma chave do Cloud Console cobre os dois, desde que não esteja restrita só a um app ID Android — vale checar as restrições da chave no Google Cloud Console se o mapa não carregar no iOS).

### 5. `.env` não estava no `.gitignore`

Achado durante essa correção, fora do que eu tinha listado antes: `.gitignore` só ignorava `.env*.local`, não `.env` puro — as chaves novas (já rotacionadas pelo usuário) estavam a um `git add .` de serem commitadas de novo. Adicionada a linha `.env` ao `.gitignore`. Confirmado com `git status --ignored` que o arquivo agora é ignorado.

## Caminho recomendado: Expo Go no iPhone (16 Pro Max, iOS mais recente)

Não precisa de Xcode nem Android Studio:

1. Instalar o app **Expo Go** (App Store) no iPhone.
2. No Mac, dentro de `/Users/MAC/VITOR/LinesV1`: `npx expo start` (usar `--tunnel` se o iPhone não estiver na mesma rede Wi-Fi do Mac — o Mac foi visto conectado a uma rede via hotspot/172.20.10.x em vez de Wi-Fi tradicional numa das checagens, o que pode exigir `--tunnel`).
3. Escanear o QR code com o Expo Go.

Isso cobre a lógica JS, Firebase, navegação, mapa etc. **Não** valida a configuração nativa específica do Android (isso só dá pra testar com Android Studio + emulador, ou um aparelho Android físico — nenhum dos dois o usuário tem hoje).

## Se quiser rodar num simulador iOS no Mac (opcional)

Instalar o **Xcode completo** pela App Store, depois:
```
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -license accept
brew install cocoapods
```
Com isso, `npx expo start` + tecla `i` abre no simulador. Não é necessário para o caminho recomendado.

## Android — sem alternativa sem Android Studio ou aparelho físico

Continua sem solução local — precisaria de Android Studio (emulador) ou um aparelho Android real. Fora do escopo desta correção.

## Notas relacionadas
- [[Visão Geral]]
- [[Dependências e Riscos]]
- [[Próximos Passos]]
