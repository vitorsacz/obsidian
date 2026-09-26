---
tags: [projeto, linesapp, arquitetura]
criado: 2026-07-30
---

# LinesApp — Arquitetura

← [[LinesApp - Visão Geral]]

## Estrutura de pastas (`src/`)

```
src/
  Componentes/   # PT — Firebase.js, Header
  data/
  img/           # 111 arquivos de imagem, solto (não é src/assets/)
  pages/         # estilo Next.js — deveria ser screens/ em RN
  Routes/        # navegação
  services/      # camada de API real (a única bem feita)
  utils/
```

Problemas de organização:
- **Mistura de idioma na própria arquitetura**: `Componentes` (PT) ao lado de `Routes`, `services`, `utils`, `data` (EN).
- Usa convenção `pages/` (Next.js) em vez do padrão de RN (`screens/`).
- Quase tudo mora dentro de `src/pages/**` (60+ pastas de tela); só existem dois componentes compartilhados de verdade: `BackHeader.js` e `Firebase.js` — pouquíssimo reaproveitamento de UI.
- `src/img/` (111 arquivos) e o `assets/` da raiz (ícone/splash do Expo, 4 arquivos) são dois locais de assets desconectados.

## Navegação

- `src/Routes/index.js` — envolve o `NavigationContainer`.
- `src/Routes/stackRoutes.js` — **~470 linhas**, `createNativeStackNavigator`, define ~45 telas, e ainda embute um menu lateral (`Modal` com slide-out) e seus estilos dentro do mesmo arquivo.
- `src/Routes/tabRoutes.js` — `createBottomTabNavigator`, 4 abas: Home / Operações / Notícias / Perfil.
- `initialRouteName` decidido por `loggedIn`/`loading`, passados via **prop drilling**: `App.js` → `Routes` → `StackRoutes` (sem Context de auth).

**Bug/código morto encontrado:** em `stackRoutes.js`, as rotas `MetroSp`, `MetroCPTM`, `MetroNoticiando` e `DiarioDoTransporte` apontam todas para o **mesmo componente** `src/pages/webview/twitter/webViewTwitter/index.js` — parece copy-paste nunca parametrizado (4 nomes de rota, 1 tela real).

## Gerenciamento de estado

- **Não existe** Context API, Redux, Zustand ou similar em nenhum lugar do `src/` (zero `createContext`/`useContext`/`Provider`).
- Auth (`loggedIn`) é `useState` local em `App.js`, alimentado por `onAuthStateChanged` do Firebase, repassado via props até `StackRoutes`.
- Todo o resto do estado (favoritos, formulários, dados buscados) é `useState`/`useEffect` local por tela.
- `src/pages/Home/index.js` sozinho tem **13 `useState`** e **2497 linhas** — mistura mapa, markers, leitura do Firebase e UI num único arquivo monolítico. Tem `//console.log(...)` de debug esquecidos comentados (ex.: linhas 112, 141, 243, 246).

## Camada de dados / API

- Firebase inicializado uma vez em `src/Componentes/Firebase/Firebase.js`, mas **chamado direto dentro das telas** (`perfil`, `Login`, `Home`, `Cadastro`) em vez de passar por uma camada de acesso a dados dedicada.
- `src/services/directionsService.js` e `src/services/placesService.js` são a **exceção boa**: wrappers limpos do Google Directions/Places com axios, tratamento de erro e JSDoc — modelo a seguir para o resto do app.
- **Scraping de HTML client-side com `cheerio`** dentro de componentes de tela: `src/pages/noticias/index.js` e `src/pages/noticiasExame/index.js` fazem `axios.get` em `metrocptm.com.br`/`exame.com` e usam `cheerio.load()` + seletores CSS para montar feed de notícias — funciona em RN só porque o `cheerio` é parser HTML puro-JS (sem DOM), mas é frágil (quebra se o site de origem mudar o HTML) e incomum como padrão de arquitetura.
- `noticiasExame` é quase idêntico a `noticias` (mesmo padrão de scraping, fonte diferente) e **não está referenciado em nenhuma rota** — código morto.

## Padrões de componente

- 100% JavaScript, sem TypeScript, sem `prop-types`.
- Majoritariamente function components + hooks, mas ainda existem **componentes de classe legados**: `src/pages/Alertas/alerta.js` e todos os arquivos em `src/pages/Alertas/acidente/**` e `src/pages/Alertas/{eletrica,lentidao,movimento,obras,SOS}/index.js` usam `class X extends Component`.

## Código morto / duplicado (achados)

- `src/pages/noticiasExame/index.js` — duplicata não roteada de `noticias`.
- `src/pages/sair/index.js` e `src/pages/locaisSalvos/index.js` — não referenciados em `stackRoutes`/`tabRoutes`; a rota "LocaisSalvos" de fato renderiza a tela `Construcao` (placeholder "em construção"), não o `locaisSalvos` real.
- 4 rotas de `webViewTwitter` apontando pro mesmo componente (ver seção Navegação).
- `Home/index.js`: monólito de 2497 linhas, forte candidato a ser quebrado em subcomponentes/hooks.

## Recomendações de arquitetura

1. Extrair uma camada de dados (`src/data/` ou `src/services/firebase/*`) para Auth/Firestore, seguindo o padrão já usado em `directionsService`/`placesService`.
2. Introduzir um `AuthContext` para eliminar o prop drilling de `loggedIn`/`loading`.
3. Quebrar `Home/index.js` em subcomponentes (mapa, markers, painel de alertas) + hooks customizados.
4. Remover código morto: `noticiasExame`, `sair`, `locaisSalvos`, e decidir o destino real das rotas duplicadas de `webViewTwitter`.
5. Migrar classes legadas de `Alertas/**` para function components, por consistência com o resto do app.
6. Padronizar em uma única língua para nomes de pasta/arquivo (ver [[Nomenclatura - Auditoria]]).

Detalhes de infraestrutura (build, segredos, dependências) em [[Infraestrutura]]. Checklist de ação em [[Plano de Melhorias]].
