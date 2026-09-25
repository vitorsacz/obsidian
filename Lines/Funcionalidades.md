#projeto #lines

Status de cada área funcional do app, por tela. Ver [[Visão Geral]] para contexto.

## ✅ Funciona de verdade

| Feature | Onde | Detalhe |
|---|---|---|
| **Autenticação** | `src/pages/Login/Login.js`, `src/pages/Cadastro/Cadastro.js` | Firebase Auth real (email/senha), envio de verificação de email, reset de senha. Acesso ao app (`Main`) é bloqueado até `emailVerified === true`. |
| **Perfil** | `src/pages/perfil/index.js` | Dados do usuário em tempo real via Firestore (`onSnapshot`), upload/troca de foto via Firebase Storage (com limpeza da imagem antiga), edição de nome, logout. É a tela mais completa do app — mas 7 dos 9 itens do menu apontam para a tela de placeholder (ver abaixo). |
| **Mapa + pings comunitários** | `src/pages/Home/index.js` | `MapView` com todas as estações/linhas de Metrô/CPTM/ViaMobilidade, toggle claro/escuro, câmera 3D. Feature real: qualquer usuário pode reportar um incidente que vira um marcador Firestore (`pings`) em tempo real para todos via `onSnapshot`, com deduplicação por distância (raio de 15m). |
| **Status das linhas** | `src/pages/Operacao/index.js` | Consome a API não-oficial `diretodostrens.com.br/api/status` de verdade, com pull-to-refresh. Sem estado de erro visível ao usuário se a API cair (só `console.error`). |
| **Compartilhar localização** | `src/pages/compartilharLocalizacao/index.js` | Pega GPS via `expo-location` e abre o WhatsApp com um link de localização — implementação client-side legítima, funciona. |

## ⚠️ Mock / placeholder (parece funcionar, mas não é real)

| Feature | Onde | Detalhe |
|---|---|---|
| **Passagens/Pagamento** | `passagens`, `pagamento`, `aprovado`, `suaPassagem`, `extrato`, `valida` | 100% simulado. Preço fixo (`R$ 5,00`), "cartão cadastrado" hardcoded, nenhum gateway de pagamento real, "aprovado" sempre acontece. `suaPassagem`/`extrato`/`valida` mostram a mesma imagem estática `qrcode.jpg` com datas fixas (algumas literalmente `00/00/2222`) e IDs fake sequenciais. `extrato` inteiro está renderizado com `opacity: 0.2`. |
| **Alexa** | `src/pages/alexa/` | Simulação visual de vínculo de conta — "Autorizar" sempre "funciona" e leva para uma tela de sucesso. Não existe integração real de OAuth/Login with Amazon neste repositório (a skill real, se existir, está em outro projeto). |
| **SOS (fluxo real)** | Botão SOS dentro de `Home/index.js` | Não é integração com serviço de emergência — é um `mailto:` (via `react-native-email`) para `loctime.group@gmail.com`, mais um marcador Firestore. Funciona, mas o nome "S.O.S." sugere mais do que realmente entrega. |

## ❌ Não funciona / stub / código morto

| Feature | Onde | Detalhe |
|---|---|---|
| **Formulários de Alertas** | `Alertas/SOS`, `Alertas/eletrica`, `Alertas/lentidao`, `Alertas/obras`, `Alertas/acidente/*` (descarrilamento, colisão, climático, técnico) | Nenhum salva o comentário/foto em lugar nenhum (sem Firestore, sem API). O botão "Enviar" só navega para a rota `'home'` (minúsculo), **que não existe** no navigator — não confirma nada e não retorna de fato para a Home. Existe uma segunda implementação de SOS (dentro de `Home.js`) que é a única que realmente funciona — as duas são inconsistentes entre si. Ver [[Problemas Conhecidos]]. |
| **`colisao`, `climatico`, `tecnico`** | `Alertas/acidente/*` | Copiados de `descarrilamento/index.js` sem renomear a classe interna (`class Descarrilamento` em todos) — funciona por acidente (o nome do export não importa), mas é copy-paste sujo. |
| **Configurações** | `src/pages/configuracoes/index.js` | Só 3 switches locais (sem persistência — perdem estado ao reabrir o app). O resto dos itens (Geral, Som, Tema, Privacidade, Sobre, Sair) não tem `onPress` nenhum. Há branches de `switch` (`setTema`, `setGeral`, `setPrivacidade`, `setSobre`) que **chamam funções que não existem** — dispararia `ReferenceError` se algum dia fossem acionadas. |
| **Menu de Perfil quebrado** | `src/pages/perfil/index.js` | 8 dos 9 itens do menu ("Informações Pessoais", "Locais Salvos", "Minhas Passagens", "Últimos Alertas", "Formas de Pagamento", "Caminhos Compartilhados", "Notificações", "Configurações") navegam para a tela genérica "Em Construção" (`paginaConstrucao`) — **mesmo quando a tela real já existe e está registrada** (ex: `locaisSalvos`, `configuracoes`). Ou seja, há telas prontas que ficaram inacessíveis pelo menu principal. |
| **Locais Salvos** | `src/pages/locaisSalvos/index.js` | Tela vazia, só um botão "Voltar". Nenhuma funcionalidade de salvar locais implementada. |
| **`menu/menu.js`** | — | Código morto — nenhum `TouchableOpacity` tem `onPress`, e o arquivo não é importado em lugar nenhum do app. Foi substituído pelo menu lateral inline em `stackRoutes.js`. |
| **`padrao/index.js`** | — | Tela órfã (template genérico "Acidente" sem corpo), não importada em nenhum lugar. Tem até um import de imagem com caminho relativo quebrado — inofensivo só porque nunca é carregada. |
| **`noticiasExame/index.js`** | — | Segundo scraper de notícias (Exame), aparentemente não referenciado nas rotas ativas (a tab "Notícias" usa o scraper do MetroCPTM) — parece código morto/alternativo abandonado. Também tem um bug: sempre mostra a imagem placeholder local em vez da imagem real raspada. |
| **Twitter/X embeds** | `src/pages/webview/twitter/*` | URLs hardcoded para `twitter.com` (pré-rebrand para X) de 4 contas — X costuma bloquear visualização embedada/deslogada em WebView, então essas telas provavelmente não carregam bem na prática. |
| **"Mapa Offline"** | `src/pages/webview/mapaoffline` | Nome enganoso: é uma `WebView` carregando uma imagem JPEG remota de um site de notícias (`vejasp.abril.com.br`) — não é offline, e depende desse site continuar hospedando a imagem. |
| **Jogos** | `src/pages/jogos/index.js` | `WebView` de um jogo do Poki.com com parâmetros de rastreamento de uma campanha do Google Ads colados na URL (`gclid=...`) — resto de uma sessão de navegação copiada, não uma integração limpa. |

## Notas relacionadas
- [[Visão Geral]]
- [[Problemas Conhecidos]]
- [[Próximos Passos]]
