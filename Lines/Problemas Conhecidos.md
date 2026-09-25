#projeto #lines

Bugs concretos e código quebrado encontrados na análise de 2026-07-23. Ver [[Visão Geral]] para contexto e [[Dependências e Riscos]] para os riscos de dependências/segurança.

## Navegação quebrada

- **`navigation.navigate('home')` para uma rota que não existe** — todos os formulários de alerta que têm botão "Enviar" (`Alertas/SOS`, `eletrica`, `lentidao`, `obras`, `movimento`, `acidente/descarrilamento`, `tecnico`, `climatico`) chamam `navigation.navigate('home')` (minúsculo). As rotas registradas são `"Main"` (stack) → `"Home"` (maiúsculo, dentro da tab navigator) — não existe rota `'home'`. Isso não retorna o usuário para a Home de fato; no melhor caso é um no-op silencioso.
- **Menu de Perfil aponta para "Em Construção" em vez das telas reais** — `src/pages/perfil/index.js` linhas ~372-479: 8 dos 9 itens do menu (`Informações Pessoais`, `Locais Salvos`, `Minhas Passagens`, `Últimos Alertas`, `Formas de Pagamento`, `Caminhos Compartilhados`, `Notificações`, `Configurações`) navegam para `"paginaConstrucao"` — mesmo quando as telas reais (`LocaisSalvos`, `Configuracoes`) já existem e estão registradas nas rotas.

## Dados perdidos / não persistidos

- **Formulários de Alertas não salvam nada** — comentário e foto ficam só em estado local do componente e são descartados ao "enviar" (sem escrita no Firestore, sem chamada de API). O texto no rodapé ("Todos os Alertas são públicos e podem sofrer atualizações") promete algo que essas telas específicas não entregam — existe uma segunda implementação de SOS, dentro de `Home/index.js`, que é a única que realmente escreve no Firestore + dispara email.
- **Configurações não persiste** — os 3 switches funcionais (Notificações, Economia, Lembrete de alerta) só setam estado local; nada é salvo em Firestore/AsyncStorage, então tudo volta ao padrão a cada reabertura do app.

## Erros latentes (`ReferenceError` potencial)

- `src/pages/configuracoes/index.js` — `handleSwitchChange` tem branches de `switch` que chamam `setTema`, `setGeral`, `setPrivacidade`, `setSobre` — **nenhuma dessas funções existe** (não há `useState` correspondente). Não quebra hoje porque nenhuma UI aciona esses branches especificamente, mas é uma bomba-relógio se alguém conectar esses itens de menu no futuro sem notar.

## Tratamento de erro fraco

- Praticamente todo `catch` do app (`perfil/index.js`, `Home/index.js`, `Cadastro.js`, `Login.js`, `Operacao/index.js`, `noticias/index.js`, `noticiasExame/index.js`) só faz `console.log`/`console.error` do erro e segue em frente — sem estado de erro visível para o usuário, sem retry, sem reporte. Em `Cadastro.js`/`Login.js` isso é agravado por mostrar sempre a mesma mensagem genérica ("Preencha todos os campos corretamente!") mesmo para erros que nada têm a ver com campos vazios (ex: senha fraca, email duplicado) — dificulta diagnóstico tanto para o usuário quanto em log.

## `console.log` esquecidos em produção

~28 ocorrências, incluindo alguns reveladores de debug ao vivo:
- `App.js:17,21` — `console.log("loggedin")` / `console.log("no log, no log")`
- `src/pages/compartilharLocalizacao/index.js:34` — comentário literal `// Adicione este log para verificar o URL`, nunca removido
- `src/pages/noticiasExame/index.js:40` — loga a URL de toda imagem raspada, a cada render

## Bug visual/cosmético

- `Login.js` e `Cadastro.js` mostram o texto fixo `"********@gmail.com"` nos modais de confirmação de reset/verificação, em vez de interpolar o email real digitado pelo usuário.
- `extrato/index.js` — a tela inteira do extrato está envolta em `{ opacity: 0.2 }`, deixando tudo visualmente "apagado"/desabilitado — parece um estado de placeholder esquecido.
- `noticiasExame/index.js` — `renderItem` ignora o valor real de `item.imagem` e sempre mostra a imagem placeholder local, mesmo quando existe uma imagem raspada de verdade.

## Código morto / órfão

- **`src/pages/menu/menu.js`** — nenhum `TouchableOpacity` tem `onPress`, e o arquivo não é importado em lugar nenhum do app (substituído pelo menu lateral inline em `stackRoutes.js`).
- **`src/pages/padrao/index.js`** — tela template órfã, sem nenhuma referência no app, com um import de imagem de caminho relativo quebrado (inofensivo só porque nunca carrega).
- **`src/pages/Home/index.js:69`** — `import construcao from "./../construcao"` nunca é usado no resto do arquivo de 16 mil linhas.
- **`stackRoutes.js`** — blocos comentados de itens de menu ("Operações", "Notícias", "Perfil") e constantes de URL não removidas.
- **`perfil/index.js:456-467`** — item de menu do Alexa inteiro comentado, mesmo a feature Alexa existindo e estando registrada nas rotas.
- Blocos JSX grandes comentados dentro de `Home/index.js` (modal antigo de SOS, modal antigo de detalhes de estação, por volta da linha 15550+).

## Cópia/colagem sem ajuste

- `Alertas/acidente/colisao`, `climatico`, `tecnico` — todos copiados de `descarrilamento/index.js` sem renomear a classe interna (`class Descarrilamento extends Component` em todos os quatro arquivos) — funciona (o nome da classe não afeta o export), mas confunde debugging/stack traces.

## Arquitetura sub-ótima (não é "bug", mas facilita bugs futuros)

- `src/pages/Home/index.js` com 16.409 linhas — todas as coordenadas de estações/linhas de metrô/trem de SP hardcoded inline no mesmo arquivo que a lógica de UI/Firestore. Deveria estar em módulos de dados separados (JSON).
- Mistura de componentes de classe (`Alertas/*`, `menu/menu.js`) e componentes funcionais com hooks (resto do app) — inconsistência de padrão dentro do mesmo projeto.
- 4 nomes de import diferentes (`MetroSp`, `MetroCPTM`, `MetroNoticiando`, `DiarioDoTransporte`) apontando todos para o mesmo arquivo `webViewTwitter/index.js` — funciona, mas confunde leitura do código.

## Notas relacionadas
- [[Visão Geral]]
- [[Funcionalidades]]
- [[Dependências e Riscos]]
- [[Próximos Passos]]
