---
tags: [projeto, linesapp, infraestrutura, seguranca]
criado: 2026-07-30
---

# LinesApp — Infraestrutura

← [[LinesApp - Visão Geral]]

## ⚠️ Segredos / variáveis de ambiente

- `.env` (raiz) tem 2 chaves: `GOOGLE_MAPS_API_KEY`, `EXPO_PUBLIC_FIREBASE_API_KEY` — **corretamente** no `.gitignore` e confirmado fora do git.
- `app.config.js` lê essas chaves via `process.env.*` para injetar no build (iOS e Android) — forma certa de fazer.
- `src/Componentes/Firebase/Firebase.js` lê `EXPO_PUBLIC_FIREBASE_API_KEY`; os outros campos do config do Firebase (authDomain, projectId, storageBucket, messagingSenderId, appId) estão hardcoded no arquivo — **isso é normal**, config web do Firebase não é segredo.

### 🔴 Problema real encontrado

**`android/app/src/main/AndroidManifest.xml:18`** — a chave do Google Maps está **hardcoded em texto puro e commitada no git**, em vez de vir só do `.env` via plugin de config no `prebuild`. Como a pasta `android/` é rastreada pelo git, essa chave está exposta no histórico do repositório `github.com/loctimegroup/LinesV1`.

→ Ação: restringir a chave atual no Google Cloud Console (por pacote Android + SHA-1) e/ou gerar uma nova, e confirmar que o próximo `expo prebuild` não deixa o valor literal exposto fora do `.gitignore` de forma desnecessária. Ver [[Plano de Melhorias]].

## Firebase

- Único ponto de inicialização: `src/Componentes/Firebase/Firebase.js`.
- SDK **JS puro** (não há `google-services.json` nativo no repo) — mistura API **compat** (`firebase/compat/app`, `firebase/compat/auth`, `firebase/compat/firestore`) com API **modular** (`initializeAuth`, `getReactNativePersistence`).
- Produtos usados: **Auth** e **Firestore**. Sem Storage/Functions/Analytics.
- Persistência de auth via `@react-native-async-storage/async-storage`.

## Build / Deploy (EAS)

`eas.json` define 3 perfis:
- `development` — dev client, distribuição interna, APK
- `preview` — distribuição interna
- `production` — **APK** (não AAB — atípico para publicação na Play Store, que hoje exige AAB) + `submit.production` vazio (sem config de submissão)

- **Sem CI/CD**: nenhum `.github/workflows` ou config de pipeline no repo.
- `package.json` só tem os scripts padrão do Expo (`start`, `android`, `ios`, `web`) — nenhum script de lint/test/build customizado.

## Projeto nativo Android

- `android/build.gradle`, `android/app/build.gradle`, `android/gradle.properties` são boilerplate padrão gerado pelo `expo prebuild` (hermes ligado, gif/webp habilitados) — sem edição manual anômala, **exceto** a chave do Maps hardcoded (seção de segredos acima).
- `versionCode 1` / `versionName "1.0.0"` — em sincronia com `package.json`/`app.config.js`.
- **Assinatura**: o build de `release` reusa `signingConfigs.debug` — ou seja, **release usa a keystore de debug**. Não é seguro para publicar; precisa de keystore de produção própria antes de ir pra Play Store.

## Higiene de dependências

- `react-native-paper` (^5.12.3) está instalado e **não é usado em nenhum lugar do código** — candidato a remoção.
- `react-native-vector-icons` é usado diretamente em `src/pages/perfil/index.js`, duplicando `@expo/vector-icons` (que já encapsula `react-native-vector-icons`) — redundante.
- `"fix": "^0.0.3"` no `package.json` — nome genérico, exige Node `0.2.5` (engine incompatível/obsoleta), não corresponde a nada identificável em uso — parece dependência adicionada por acidente.
- Libs de uso único e específico, mas não necessariamente "mortas": `react-native-communications`, `react-native-email`, `react-native-app-intro-slider`, `react-native-modalize` — vale revisar se ainda são necessárias.
- Sem conflito real de libs de parsing HTML: apenas `cheerio` está no `package.json` (usado em `noticias`/`noticiasExame`).

## Testes

- **Nenhuma infraestrutura de teste**: sem config de Jest, sem pasta `__tests__`, sem `*.test.js`/`*.spec.js` em nenhum lugar do projeto (fora de `node_modules`).

## Lint / formatação

- **Sem ESLint nem Prettier** configurados na raiz do projeto.

## Documentação

- Sem `README.md` (ou qualquer `README*`) na raiz — zero onboarding para um novo colaborador entender como rodar o projeto.

## Recomendações de infraestrutura

1. 🔴 Restringir/rotacionar a chave do Google Maps exposta no `AndroidManifest.xml`.
2. 🔴 Configurar keystore de release própria (não reusar a de debug) e mudar o build de produção para AAB.
3. 🟠 Adicionar `README.md` com passos de setup, variáveis de ambiente necessárias e como rodar (`expo start`, `expo run:android`).
4. 🟠 Remover `react-native-paper` e a dependência `fix`; consolidar em `@expo/vector-icons` só.
5. 🟡 Adicionar ESLint + Prettier com config mínima.
6. 🟡 Adicionar ao menos testes básicos (Jest) para `src/services/` e `src/utils/`, que são as partes mais testáveis/isoladas hoje.
7. 🟢 Configurar CI simples (GitHub Actions) rodando lint + build check em PRs.

Ver detalhes de arquitetura em [[Arquitetura]] e nomenclatura em [[Nomenclatura - Auditoria]]. Checklist consolidado em [[Plano de Melhorias]].
