---
tags: [projeto, linesapp, roadmap]
criado: 2026-07-30
---

# LinesApp — Plano de Melhorias

← [[LinesApp - Visão Geral]]

Checklist priorizado, consolidando [[Arquitetura]], [[Infraestrutura]] e [[Nomenclatura - Auditoria]]. Progresso registrado em [[Execução - Rodada 1 (2026-07-30)]].

## 🔴 Urgente (segurança / risco de produção)

- [ ] Restringir ou rotacionar a chave do Google Maps exposta em texto puro em `android/app/src/main/AndroidManifest.xml:18` (commitada no histórico do git em `loctimegroup/LinesV1`). **Não feito** — exige acesso ao Google Cloud Console do usuário.
- [ ] Criar keystore de release própria para Android — hoje o build de `release` reusa a keystore de `debug` (`android/app/build.gradle`). **Não feito** — decisão de alto risco, melhor via `eas credentials` com o usuário.
- [x] Mudar o perfil `production` do `eas.json` de APK para **AAB** antes de qualquer submissão à Play Store. ✅ 2026-07-30

## 🟠 Alto (débito técnico visível)

- [x] Remover código morto: `src/pages/noticiasExame/`, `src/pages/sair/`, `src/pages/locaisSalvos/` (não roteados). ✅ 2026-07-30
- [ ] Resolver a duplicação de rota: `MetroSp`, `MetroCPTM`, `MetroNoticiando`, `DiarioDoTransporte` em `stackRoutes.js` apontam pro mesmo componente `webViewTwitter` — decidir se cada um deveria ter conteúdo próprio ou consolidar em uma rota parametrizada. **Não feito** — decisão de produto, não técnica.
- [x] Remover dependência `react-native-paper` (não usada). ✅ 2026-07-30 — `fix@0.0.3` já não existia mais no branch atual.
- [x] Consolidar ícones: escolher só `@expo/vector-icons` e remover o uso direto de `react-native-vector-icons` em `src/pages/perfil/perfil.js`. ✅ 2026-07-30
- [x] Criar `README.md` com setup do projeto, variáveis de ambiente (`.env`) necessárias e comandos (`expo start`, `expo run:android`). ✅ 2026-07-30 (+ `.env.example`)

## 🟡 Médio (arquitetura e nomenclatura)

- [x] Adotar a convenção de nomenclatura única proposta em [[Nomenclatura - Auditoria]] para os itens 2, 3 e 5 (arquivo de entrada, arquivo de estilo, `Componentes` → `components`). ✅ 2026-07-30 — 39 telas renomeadas, ver [[Execução - Rodada 1 (2026-07-30)]]. **Itens 1, 4, 6, 7 (casing de pasta, idioma, assets) continuam pendentes.**
- [ ] Quebrar `src/pages/Home/Home.js` (2497 linhas, 13 `useState`) em subcomponentes (mapa, markers, painel de alertas) + hooks customizados. **Não feito** — refactor de comportamento, precisa QA manual sem suíte de testes.
- [ ] Criar uma camada de dados para Firebase (Auth/Firestore) em vez de chamadas inline nas telas (`perfil`, `Login`, `Home`, `Cadastro`), seguindo o padrão já bom de `src/services/directionsService.js`/`placesService.js`. **Não feito.**
- [ ] Introduzir `AuthContext` para eliminar o prop drilling de `loggedIn`/`loading` (`App.js` → `Routes` → `StackRoutes`). **Não feito.**
- [ ] Migrar componentes de classe legados (`src/pages/Alertas/**`) para function components + hooks. **Não feito.**
- [ ] Reavaliar se `noticias`/scraping via `cheerio` deveria virar uma função server-side (Cloud Function) em vez de scraping client-side. **Não feito.**

## 🟢 Baixo (qualidade de longo prazo)

- [x] Adicionar ESLint + Prettier com config mínima. ✅ 2026-07-30
- [x] Corrigir achados do lint (43 erros / 164 avisos → 0). ✅ 2026-07-30 — detalhe em [[Execução - Rodada 2, correção do lint (2026-07-30)]], inventário original em [[Lint - Auditoria]].
- [ ] Adicionar testes básicos (Jest) começando por `src/services/` e `src/utils/` (partes mais isoladas/testáveis hoje). **Não feito.**
- [x] Configurar CI simples (GitHub Actions) rodando lint em PRs. ✅ 2026-07-30 (`.github/workflows/lint.yml` — só lint por enquanto, build check pode entrar depois).
- [ ] Avaliar migração incremental para TypeScript, começando pelos arquivos novos/já limpos (`src/services/`, `src/utils/`). **Não feito.**

## Ordem sugerida de execução

1. Segurança (🔴) — não depende de nada, faz sozinho.
2. Limpeza de código morto + dependências (🟠) — reduz superfície antes de qualquer refactor maior.
3. Nomenclatura em massa (🟡, primeiro item) — fazer **antes** dos outros itens médios, já que eles vão tocar nos mesmos arquivos; renomear depois só geraria retrabalho de merge.
4. Refactors de arquitetura (🟡, demais itens) — feitos, um por vez, no novo esqueleto de nomes.
5. Qualidade (🟢) — contínuo, pode começar em paralelo a qualquer momento.
