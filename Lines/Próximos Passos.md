#projeto #lines

Lista priorizada de ações, a partir da análise de 2026-07-23. Ver [[Dependências e Riscos]] e [[Problemas Conhecidos]] para o detalhe de cada item.

## ✅ 1. SDK do Expo — feito (2026-07-23)

Completado o upgrade para SDK 57 (não revertido — Expo Go só suporta o SDK mais recente). Detalhe em [[Dependências e Riscos]] e [[Ambiente de Desenvolvimento (Mac)]]. **Falta commitar** as mudanças (`package.json`, `package-lock.json`, `app.config.js`, `Firebase.js`, `.gitignore` — ver `git status` no repositório).

## ✅ 2. Chaves do Google Maps e do Firebase — feito (2026-07-23)

Usuário já tinha rotacionado as duas chaves; migrei `app.json` → `app.config.js` e `Firebase.js` para lerem do `.env` de verdade (antes era só um placeholder morto que não funcionava), e adicionei `.env` ao `.gitignore` (não estava, risco de commitar as chaves novas de novo). Detalhe em [[Ambiente de Desenvolvimento (Mac)]].

## ✅ 3. `react-native-fast-image` removido (2026-07-23)

Foi removido durante o ajuste do item 1 (travava a instalação por incompatibilidade com React 19). As outras dependências sem uso continuam pendentes:

Remover sem risco (zero uso confirmado em `src/`): `fix`, `uuid`, `react-native-elements`, `react-native-mail`, `htmlparser2`, `node-html-parser`, `react-native-html-parser`, `react-native-htmlview`, `domhandler`. Reduz peso do bundle e superfície de manutenção.

## 4. Corrigir a navegação quebrada dos formulários de Alerta

Trocar `navigation.navigate('home')` (rota inexistente) pela rota real (`'Home'` dentro da tab navigator, ou o fluxo correto de retorno). Decidir também se esses formulários devem realmente persistir os relatos (Firestore, como o SOS de `Home.js` já faz) ou se são intencionalmente só uma casca de UI a ser removida/simplificada.

## 5. Reconectar o menu de Perfil às telas reais

8 dos 9 itens do menu de Perfil apontam para "Em Construção" mesmo quando a tela já existe (`LocaisSalvos`, `Configuracoes`). Simplesmente trocar o destino da navegação já destrava funcionalidade existente sem escrever código novo.

## 6. Decidir o destino do fluxo de Passagens/Pagamento

Hoje é 100% mockado (sem gateway real). Decidir: vira feature real (integrar um provedor de pagamento) ou permanece como protótipo visual claramente sinalizado como tal.

## 7. Adicionar tooling básico

ESLint + Prettier (ex: `eslint-config-expo`) e Jest (`jest-expo`) — hoje não há nenhum, o que dificulta pegar os problemas listados em [[Problemas Conhecidos]] automaticamente no futuro.

## 8. Faxina de código morto

Remover `src/pages/menu/menu.js` e `src/pages/padrao/index.js` (confirmados órfãos, sem import em lugar nenhum), e os blocos grandes de JSX/comentários mortos em `stackRoutes.js` e `Home/index.js`.

## 9. Migrar Firebase v10 → v12

Não urgente isoladamente, mas vale planejar junto com a decisão do item 1 (upgrade de SDK), já que ambos tocam o mesmo tipo de superfície (dependências nativas/JS core do app). Aproveitar para unificar em torno da API modular só (remover os imports `firebase/compat/*`).

## 10. (Feature nova, não urgente) Monitoramento de ônibus em tempo real no mapa

Arquitetura já pesquisada e desenhada em [[Monitoramento em Tempo Real]]: viável para ônibus via SPTrans Olho Vivo, exige criar um backend (Firebase Cloud Function) que hoje não existe no projeto. **Não é viável** para trens/metrô com dados públicos atuais — só dá para trocar a fonte de status já usada em `Operacao/index.js` pela API Trilhos oficial da ARTESP. Não depende dos itens 1-9, mas faz mais sentido priorizar depois de resolver o bloqueio de SDK (item 1).

## Notas relacionadas
- [[Visão Geral]]
- [[Funcionalidades]]
- [[Dependências e Riscos]]
- [[Problemas Conhecidos]]
- [[Monitoramento em Tempo Real]]
