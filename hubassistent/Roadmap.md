#projeto #hubassistent #roadmap

Ver [[Visão Geral]] para contexto completo.

- [x] **Fase 0** — scaffold do monorepo (pnpm + Turborepo)
- [x] **Fase 1** — autenticação (registro, login, refresh token)
- [x] **Fase 2** — núcleo financeiro (contas, cartões, categorias, transações — backend + frontend)
- [x] **Identidade visual** — Navy Duotone aplicada em todo o app (ver [[Design System]])
- [x] **Fase 3a** — faturas de cartão de crédito: CRUD, importação de PDF com revisão antes de gravar, filtro por mês/banco, vínculo transação↔fatura. Testado localmente ponta a ponta (ainda não publicado em produção).
- [ ] **Fase 3b** — contas a pagar gerais (não ligadas a cartão) + dashboard/extrato mensal detalhado
- [ ] **Fase 4** — módulo de rotina/calendário (agendamentos, reuniões, exercícios)
- [ ] **Fase 5** — importação Open Finance (Pluggy/Belvo, opção gratuita)

Tudo até a Fase 2 + identidade visual está publicado em produção e testado de ponta a ponta (ver [[Infraestrutura e Deploy]]). A Fase 3a ainda não foi deployada — falta rodar a migration em produção e fazer o deploy do Render/Vercel.

## Fase 3a — detalhes

- **Schema novo**: `Card.bank` (texto livre, nome do banco/emissor) e `Transaction.installmentNumber`/`installmentTotal` (metadados de parcela — cada mês só lança o valor daquela parcela, sem gerar transações futuras).
- **Importação de PDF**: fluxo em duas etapas — `POST /invoices/parse-pdf` (multipart, não grava nada) extrai texto do PDF com `pdf-parse` e manda pro **Gemini** (`gemini-flash-lite-latest`) estruturar em JSON (banco, mês/ano de referência, vencimento, transações com parcela); a tela mostra uma prévia **editável** (nunca importa direto sem revisão); ao confirmar, `POST /invoices/import` cria/atualiza a fatura do mês e cria as transações em lote com `source: IMPORT`.
- O parsing por IA substituiu uma primeira versão por regex (heurística, sensível ao formato de cada banco). O prompt foi portado do projeto `~/VITOR/invoice-reader` (Python/Streamlit, que joga pro Google Sheets) — só a extração foi reaproveitada, a persistência continua 100% no Postgres/Prisma do hubassistent e a visualização é a própria tela `/invoices` (filtro por mês/banco, resumo de gastos), por pedido explícito do Vitor de não depender do Sheets.
- Requer `GEMINI_API_KEY` no `.env` do `apps/api` — **ainda falta adicionar essa variável no Render** antes do próximo deploy, senão o boot quebra (validação de env é obrigatória).
- Ver [[Arquitetura]] para os detalhes técnicos do módulo e [[Infraestrutura e Deploy]] para o cuidado de testar contra o banco de produção.
