---
tags: [projeto, react-native, expo, linesapp]
criado: 2026-07-30
---

# LinesApp — Visão Geral

Nota-índice (MOC) do projeto **LinesApp**, app de transporte público (Expo / React Native).

## Stack

- **Expo SDK 54** · React Native 0.81.5 · React 19.1.0
- Workflow **prebuild/bare**: pasta `android/` nativa commitada no git (sem pasta `ios/`)
- Navegação: `@react-navigation` (native-stack + bottom-tabs; drawer instalado mas não usado)
- Backend: **Firebase** (Auth + Firestore, SDK JS)
- Mapas: `react-native-maps` + Google Directions/Places API (via `src/services/`)
- 100% JavaScript (sem TypeScript)
- Repositório: `github.com/loctimegroup/LinesV1`

## Status atual (2026-07-30)

- ✅ Migração para **Expo SDK 54** concluída (estava preso no SDK 52 por `node_modules` desatualizado em relação ao `package.json`; corrigido com reinstalação limpa + `expo prebuild --clean`).
- ✅ Rodada 1 de melhorias executada (código morto, dependências, README, ESLint/Prettier, CI, AAB, nomenclatura de ~39 telas) — ver [[Execução - Rodada 1 (2026-07-30)]].
- ✅ Rodada 2: 100% dos achados do ESLint corrigidos (207 → 0 problemas) — ver [[Execução - Rodada 2, correção do lint (2026-07-30)]]. Mudanças ainda não commitadas.
- ⚠️ Itens de segurança (chave do Maps exposta, keystore de release) e refactors maiores (monólito `Home`, camada de dados Firebase) continuam pendentes — ver [[Plano de Melhorias]].

## Notas relacionadas

- [[Arquitetura]] — estrutura de pastas, navegação, estado, camada de dados, código morto
- [[Infraestrutura]] — segredos, Firebase, build/EAS, projeto nativo, dependências, testes
- [[Nomenclatura - Auditoria]] — inventário de inconsistências de nomes + convenção proposta
- [[Plano de Melhorias]] — checklist priorizado de ações
- [[Execução - Rodada 1 (2026-07-30)]] — o que já foi de fato aplicado no código
- [[Lint - Auditoria]] — inventário completo dos achados do ESLint (histórico; já corrigido)
- [[Execução - Rodada 2, correção do lint (2026-07-30)]] — correção de 100% dos achados do lint (207 → 0 problemas)

## Resumo executivo

O app funciona e a stack está atualizada, mas foi crescendo sem convenções fixas: nomes de pasta/arquivo em português e inglês misturados, sem padrão de casing, sem TypeScript/lint/testes, com lógica de Firebase e scraping de HTML espalhada dentro das telas em vez de uma camada de dados, e uma tela (`Home`) com quase 2500 linhas. O ponto mais urgente não é nomenclatura — é uma **chave de API do Google Maps commitada em texto puro** no projeto nativo Android (ver [[Infraestrutura]] e [[Plano de Melhorias]]).
