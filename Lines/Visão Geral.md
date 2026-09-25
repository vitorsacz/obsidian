---
tags: [projeto, lines, react-native, expo, firebase]
data: 2026-07-23
---

#projeto #lines

## O que é

**LinesApp** — app mobile (Expo/React Native) companion de transporte público da região metropolitana de São Paulo: mapa com estações e linhas de Metrô/CPTM/ViaMobilidade, alertas de incidentes reportados pela comunidade, botão de SOS, "compra de passagens" (protótipo), notícias sobre a rede, e uma tela informativa de integração com Alexa.

Pacote Android: `com.loctime.LinesApp`. Organização no GitHub: `loctimegroup`.

Repositório local: `/Users/MAC/VITOR/LinesV1`.

## Status geral

⚠️ **Projeto dormente.** Último commit real de desenvolvimento: `f157b63`/`6c47282` ("update para sdk 52"), **abril/2025** — mais de um ano sem atividade até a data desta análise (2026-07-23).

⚠️ **Working tree com edição não commitada e quebrada**: `package.json` foi editado localmente subindo `"expo"` para `^57.0.8`, mas **nenhuma outra dependência foi atualizada junto** — todo o resto do projeto (`react-native@0.76.9`, `react@18.3.1`, `react-native-reanimated@~3.16.1`, `expo-image-picker`, `expo-location` etc.) continua na geração do **Expo SDK 52**. O SDK 57 real exige React Native 0.86 e React 19.2. Isso deixa o projeto **inconsistente e não instalável/buildável como está** — precisa de uma decisão antes de qualquer trabalho novo (ver [[Dependências e Riscos]]).

## Stack técnica

- **Framework**: Expo (managed, com pasta `android/` prebuild também presente) + React Native
- **Navegação**: React Navigation (native-stack + bottom-tabs), com um menu hambúrguer lateral customizado implementado à mão dentro do próprio arquivo de rotas
- **Backend**: Firebase (Auth + Firestore + Storage) — é o único backend real do app
- **Mapa**: `react-native-maps` (Google Maps provider), com todas as coordenadas de estações/linhas hardcoded no código
- **Dados externos**: 1 API não-oficial (`diretodostrens.com.br`) + 2 scrapers HTML client-side (`cheerio` sobre MetroCPTM e Exame)
- **Sem TypeScript, sem testes, sem ESLint/Prettier configurados**

## Estrutura de pastas

```
LinesV1/
├── App.js                    # entrada — escuta auth state do Firebase
├── app.json                  # config Expo (contém chave do Google Maps hardcoded)
├── src/
│   ├── Routes/
│   │   ├── index.js           # NavigationContainer
│   │   ├── stackRoutes.js      # 553 linhas — stack navigator + menu lateral custom inline
│   │   └── tabRoutes.js        # bottom tabs: Home, Operação, Notícias, Perfil
│   ├── Componentes/
│   │   └── Firebase/Firebase.js  # init do Firebase (config hardcoded)
│   ├── pages/                  # ~35 telas, uma pasta por feature (ver [[Funcionalidades]])
│   └── img/                    # assets estáticos (ícones, pins do mapa, mapas em PNG/PDF)
└── android/                    # projeto Android prebuild (gradle)
```

Ponto de atenção estrutural: `src/pages/Home/index.js` tem **16.409 linhas** (~778 KB) — a tela do mapa concentra sozinha todas as coordenadas de estações e polylines de todas as linhas de trem/metrô de SP hardcoded inline, além da lógica de pings em tempo real. É o arquivo mais crítico e mais difícil de manter do projeto.

## Notas relacionadas

- [[Funcionalidades]] — o que cada tela faz de verdade, e o que é mock/placeholder/código morto
- [[Dependências e Riscos]] — auditoria de dependências, chaves expostas, o problema do Expo SDK
- [[Problemas Conhecidos]] — bugs concretos encontrados no código
- [[Próximos Passos]] — lista priorizada do que resolver primeiro
- [[Monitoramento em Tempo Real]] — pesquisa e arquitetura para plotar ônibus/trens em tempo real no mapa (feature futura)
- [[Ambiente de Desenvolvimento (Mac)]] — o que falta instalar para rodar o app na máquina do usuário, caminho recomendado (Expo Go no iPhone)
