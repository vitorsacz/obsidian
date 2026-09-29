---
projeto: dentist-system
tipo: roadmap
status: em execução
atualizado: 2026-09-26
tags: [roadmap, permissoes, financeiro, seguranca]
---

# Roadmap: papéis, permissões e financeiro

> Status: **em execução**. Fase 0 e R1 concluídos. Próximo: **R3 → P3**. Cada item roda com um prompt próprio no Claude Code: branch, testes, teste no Chrome, PR e merge com CI verde (ou só com aprovação, quando o prompt pedir). Ver [[#Progresso]].

## Legenda

- **P0**: bloqueia uso com clínica real. Fazer primeiro.
- **P1**: necessário para o produto ficar completo.
- **P2**: melhoria. Pode esperar.

## Diagnóstico de partida

O que a análise do código mostrou e que motiva este roadmap:

- O freelancer não tem papel próprio. É um `DENTIST` comum numa organização `FREELANCER`, e o tipo de organização só muda regra ao criar consultório e nos termos financeiros.
- O dentista da clínica tem poderes de dono (catálogo, preços, consultórios, financeiro da organização inteira).
- O admin da clínica não tem poderes de gestor (não vê financeiro, não edita catálogo nem consultórios, não acessa a agenda real).
- `reports/financial` soma todos os atendimentos da organização sem filtrar por dentista: um dentista vê o faturamento do outro.
- O relatório desconta aluguel do repasse do dentista de clínica, mas na clínica quem paga aluguel é a clínica.
- `Attendance` não tem campo de dentista, só `createdByUserId` (quem lançou, que pode ser a recepcionista).
- Não existe módulo de recebimentos: não há registro de pagamentos, parcelas nem forma de pagamento.
- Front com dados mock: dashboard inteiro, catálogo de procedimentos (a API já existe) e agendamentos da agenda.
- ~~Segurança: `/auth/lookup` público expõe nomes de organizações, não há rate limit no login, o refresh token não é revogável e o RBAC libera rotas sem `@Roles`.~~ Resolvido na Fase 0 (2026-09-26).

## Decisões

### Fechadas

| ID | Decisão |
|---|---|
| D1 | Um usuário pode ter mais de um papel (`roles: Role[]`). Freelancer = `[ADMIN, DENTIST]` numa organização `FREELANCER`. Permissões vêm dos papéis; o tipo de organização só muda regras de negócio. |
| D3 | Todos os dentistas da clínica veem todos os prontuários. Contrapartida: auditoria de acesso (Fase 5). |
| D4 | Freelancer = exatamente 1 dentista + até 1 usuário não dentista. Limites guardados na organização (`maxDentists`, `maxStaff`), com padrão por tipo e editáveis pelo Super Admin. |
| D5 | Admin vê o financeiro por paciente (valores e status de pagamento), sem acesso a dados clínicos, a menos que também seja `DENTIST`. Auditoria de tudo e histórico financeiro do paciente. |
| D6 | O admin da clínica cadastra os consultórios. O Super Admin só cria a organização e o primeiro admin. |

### Em aberto

| ID | Decisão | Onde entra |
|---|---|---|
| D2 | Modelo de repasse: modelo, base de cálculo, momento, exceções e fechamento | Fase 7 (descoberta) |
| D7 | A recepcionista acessa orçamentos? Hoje acessa; o README diz que ela nunca vê dados financeiros | Item P4 |

## Ordem de execução

```
Fase 0 (segurança) → Fase 1 (papéis) → Fase 2 (permissões)
                                     ↘ A1 (dentistId) → Fase 3 (correções financeiras)
Depois, em paralelo: Fase 4 (agenda) · Fase 5 (auditoria) · Fase 6 (freelancer)
Fase 7 começa após a conversa sobre financeiro (D2)
```

Ordem combinada em 2026-09-26 para a sequência atual: **R1 → R3 → P3**, cada um em seu PR (o P3 depende do R1 e do R3).

## Progresso

| Item | PR | Estado |
|---|---|---|
| S5. Cobertura do isolamento | #13 | ✅ na `main` |
| S4. RBAC negando por padrão | #14 | ✅ na `main` |
| S1. Login sem vazamento | #15 | ✅ na `main` |
| S2. Rate limit + helmet | #16 | ✅ na `main` |
| S3. Refresh token revogável | #17 | ✅ na `main` |
| Matriz `ACCESS` centralizada (preparação da Fase 2) | #18 | ✅ na `main` |
| R1. Múltiplos papéis — parte 1 (expandir) | #19 | ✅ na `main` |
| R1. Múltiplos papéis — parte 2 (remove `User.role`) | #20 | ✅ na `main` (mergeado depois do deploy do #19 ficar Live) |
| Manutenção pós-R1: confirmação ao dar/tirar Admin | #21 | ✅ na `main` |

Suíte e2e: 10 suítes, 82 testes na `main`. Detalhes técnicos em [[Arquitetura]] e [[Funcionalidades e Endpoints]]; avisos de deploy em [[Infraestrutura e Deploy]].

### Decisões tomadas durante a execução

- **Refresh com 30/min em vez de 5/min** (S2): o front faz refresh a cada carregamento de página (2× em dev) e a clínica sai por um IP só; com 5/min a sessão cairia ao recarregar algumas vezes. Login ficou com 5/min.
- **Web Locks entre abas** (S3): a trava de um refresh por vez precisa valer entre abas, não só na mesma aba; sem isso duas abas abrindo juntas pareciam reuso de token e derrubavam a sessão.
- **Rotação sem transação interativa** (S3): o Prisma 5.22 falha com P2028 quando duas transações interativas disputam a mesma linha — ver [[Problemas Conhecidos]].
- **Capacidades só de tela** na matriz: `agenda.view` e `dashboard.view`. A página de Pacientes usa `patients.write` (a leitura da API inclui ADMIN só por causa da Agenda). `GET /organization` segue `@AllowAuthenticated`.
- **Capitania mantida** (R1): o prompt dizia "um admin não edita outro admin", mas o Vitor decidiu manter o comportamento de hoje — o fundador edita outros admins.
- **Pelo menos um ADMIN ativo** por organização (R1): regra nova, checada antes da regra "não remover o próprio ADMIN".
- **Migration em dois PRs** (R1): expandir (#19) → contrair (#20), porque o Render aplica a migration enquanto a versão anterior ainda atende.
- **Texto da confirmação de Admin** (#21) descreve o que o ADMIN pode **hoje** (gerenciar usuários e ver a agenda de todos os dentistas), não "consultórios e catálogo" — isso só vale depois do P1.

---

## Fase 0: segurança base (P0)

- [x] **S1. Login sem vazamento de organizações** · P0 · ✅ PR #15 (2026-09-26)
  - Remover o `/auth/lookup` público. O login valida identificador + senha primeiro; só se houver mais de uma conta válida, devolve a lista de organizações para escolha.
  - Aceite: nenhuma rota pública revela se um e-mail existe ou em quais organizações está.
  - Feito: login responde sempre 200 em sucesso; e-mail inexistente e senha errada dão o mesmo 401; o teste de política trava a lista de rotas públicas (login, refresh, health).
- [x] **S2. Rate limit e headers de segurança** · P0 · ✅ PR #16 (2026-09-26)
  - `@nestjs/throttler` no login e no refresh. `helmet` no `main.ts`.
  - Aceite: tentativas repetidas de login recebem 429; headers de segurança presentes nas respostas.
  - Feito: login 5/min, refresh 30/min, global 100/min, `/health` livre; `trust proxy` por `TRUST_PROXY_HOPS`; setup em `app.setup.ts`, usado também pelos testes.
- [x] **S3. Refresh token revogável** · P0 · ✅ PR #17 (2026-09-26)
  - Guardar hash do refresh token no banco, gerar token novo a cada refresh (rotação) e invalidar o anterior. Logout invalida no servidor. Opção "sair de todos os dispositivos".
  - Aceite: token usado após logout ou após rotação é rejeitado; reuso de token antigo invalida a sessão inteira.
  - Feito: `RefreshSession` (hash SHA-256, famílias); `POST /auth/logout-all` (ainda sem botão no front); desativar usuário revoga sessões; `JWT_REFRESH_SECRET` removido. **Usuários logados precisam entrar de novo após o deploy.**
- [x] **S4. RBAC negando por padrão** · P0 · ✅ PR #14 (2026-09-26)
  - Rota sem `@Roles` fica bloqueada. Rotas liberadas para qualquer autenticado recebem um decorator explícito.
  - Aceite: teste que percorre todos os controllers e falha se alguma rota não tiver política declarada.
  - Feito: decorator `@AllowAuthenticated()`; 56 rotas com a mesma política efetiva de antes.
- [x] **S5. Teste de cobertura do isolamento** · P0 · ✅ PR #13 (2026-09-26)
  - Teste que lê o schema Prisma e garante que todo model com `organizationId` está em `TENANT_SCOPED_MODELS`.
  - Aceite: adicionar um model com `organizationId` fora da lista quebra o CI.

## Fase 1: identidade, papéis e Super Admin (P0)

- [x] **R1. Múltiplos papéis por usuário** · P0 · depende de S4 · ✅ PRs #19 + #20 (2026-09-26)
  - Migrar `role` para `roles: Role[]`, com migration de dados. Atualizar JWT, `RolesGuard` (libera se tiver qualquer papel exigido), `shared-types` e front.
  - Aceite: usuário `[ADMIN, DENTIST]` acessa rotas de ambos; testes existentes continuam passando.
  - Feito: tela de usuários com checkboxes de papéis; rótulo "Admin · Dentista"; regras (pelo menos um ADMIN ativo, não remover o próprio ADMIN, capitania, todo dentista com cor); estatísticas contam cada papel e o total conta pessoas; teste da migration pela cadeia real. Nenhum usuário mudou de papel (freelancer continua só DENTIST até o P3).
- [ ] **R2. Limites por organização** · P0 · depende de R1
  - Campos `maxDentists` e `maxStaff` na `Organization`. Padrão: `FREELANCER` = 1 dentista + 1 não dentista; `CLINIC` = sem limite ou limite alto.
  - Aceite: criar usuário acima do limite retorna erro claro; o Super Admin consegue alterar os limites.
- [ ] **R3. Criação completa de organizações** · P0 · depende de R1
  - Super Admin cria organização `CLINIC` ou `FREELANCER` já com o primeiro admin. No freelancer, o primeiro usuário nasce `[ADMIN, DENTIST]`.
  - Aceite: fluxo completo pela tela do Super Admin, sem seed.
- [ ] **R4. Suspender e reativar organização** · P0 · depende de S3
  - Usar o `OrganizationStatus` que já existe. Login e refresh bloqueados para organização suspensa; sessões ativas encerradas.
  - Aceite: usuários de organização suspensa não acessam nada; reativar devolve o acesso.
- [ ] **R5. Recuperar acesso do admin** · P0 · depende de S3
  - Super Admin gera senha temporária ou link de redefinição para o admin de uma organização. Troca obrigatória no primeiro acesso.
  - Aceite: o admin recupera o acesso sem intervenção no banco; a ação fica registrada.
- [ ] **R6. Converter freelancer em clínica** · P2 · depende de R2
  - Mudar o tipo de `FREELANCER` para `CLINIC` mantendo todos os dados e ajustando os limites.

## Fase 2: matriz de permissões (P0)

- [x] **Preparação: matriz `ACCESS` centralizada** · ✅ PR #18 (2026-09-26)
  - `packages/shared-types/src/access.ts` mapeia capacidade → papéis; a API usa `@Roles(...ACCESS["x"])` e o front `can(user, "x")`. Sem mudança de acesso (56 rotas e 21 checagens do front conferidas).
  - Consequência: P1, P2 e P4 passam a ser, em boa parte, **mudanças de papéis dentro do `ACCESS`** (mais testes). Regras por tipo de organização continuam fora da matriz.
- [ ] **P1. Admin com poderes de gestor** · P0 · depende de R1
  - Admin gerencia dados da clínica, consultórios, catálogo, preços e usuários, e vê o financeiro da organização.
- [ ] **P2. Dentista da clínica sem poderes de dono** · P0 · depende de R1
  - Dentista tem acesso clínico (anamnese, prontuário, odontograma) e à própria agenda e ao próprio financeiro. Não edita catálogo, preços nem consultórios.
- [ ] **P3. Freelancer com acesso total** · P0 · depende de R1 e R3
  - Com `[ADMIN, DENTIST]`, acessa tudo da própria organização sem regra especial por tipo.
  - Prompt já existe (migration idempotente DENTIST → `[ADMIN, DENTIST]` só em organização `FREELANCER`); ficou para depois do R3, que ainda não existe (hoje `POST /platform/organizations` só cria `CLINIC`).
- [ ] **P4. Recepcionista conforme a matriz** · P1 · depende de D7
  - Agenda, cadastro de pacientes, recalls e materiais. Orçamentos conforme a decisão D7. Nunca dados clínicos.
- [ ] **P5. Testes e2e por papel** · P0 · depende de P1–P3
  - Suíte que percorre a matriz inteira (papel × rota × resultado esperado), no mesmo estilo da suíte de isolamento entre organizações.

## Fase 3: correções urgentes do financeiro atual (P0)

Não dependem do D2: são falhas, não decisões de produto.

- [ ] **A1. Dentista em agendamento e atendimento** · P0 · depende de R1
  - Adicionar `dentistId` em `Appointment` e `Attendance`, com migration e backfill (usar `createdByUserId` quando ele for dentista). Validar que o dentista pertence à organização.
- [ ] **F1. Dentista vê só o próprio financeiro** · P0 · depende de A1
  - `reports/financial` filtra por `dentistId` quando o usuário é só `DENTIST`.
  - Aceite: teste com 2 dentistas na mesma clínica, cada um vendo apenas os próprios valores.
- [ ] **F2. Aluguel só para freelancer** · P0
  - O desconto de aluguel do consultório só se aplica em organização `FREELANCER`.
- [ ] **F3. Visão do admin** · P1 · depende de F1
  - Admin vê o financeiro da organização inteira, com filtros por dentista e por consultório.

## Fase 4: agenda real (P1)

- [ ] **A2. Agenda com agendamentos reais** · P1 · depende de A1
  - Substituir o `buildAppointments` mock pela API. Trocar os tipos `Mock*` pelos tipos de `shared-types`.
- [ ] **A3. Agenda por papel** · P1 · depende de A2, P4
  - Recepcionista agenda para qualquer dentista; dentista vê e gerencia a própria agenda; admin vê todas.
- [ ] **A4. Catálogo de procedimentos real** · P1
  - Conectar a tela ao `/procedures`, que já existe, removendo o `useMockProcedureCatalog`.

## Fase 5: auditoria e LGPD (P1)

- [ ] **L1. Tabela de auditoria** · P1
  - `AuditLog` só de inserção: quem, ação, entidade, id, paciente relacionado, data, IP. A role de runtime do banco (`app_user`) sem `UPDATE`/`DELETE` nessa tabela.
- [ ] **L2. O que registrar** · P1 · depende de L1
  - Todas as escritas. Leituras de anamnese, prontuário e odontograma.
- [ ] **L3. Rastro financeiro** · P1 · depende de L1
  - Valor antes e depois em toda alteração de dado financeiro.
- [ ] **L4. Tela de auditoria** · P2 · depende de L2
  - Admin consulta o log por paciente, por usuário e por período.
- [ ] **L5. Infra de produção** · P1
  - Plano pago (sem hibernação), banco e API em região no Brasil, backups automáticos testados.

## Fase 6: freelancer (P1)

- [ ] **FL1. Tela dos termos por consultório** · P1 · depende de P3
  - Aluguel fixo (diário, semanal, mensal), comissão ou valor por atendimento, usando o `ClinicFinancialTerms` que já existe.
- [ ] **FL2. Relatório com os termos** · P1 · depende de FL1
  - O relatório do freelancer considera aluguel e comissão de cada consultório.
  - Obs.: pode ser reescrito na Fase 7 se o D2 adotar um motor único de divisão entre duas partes.
- [ ] **FL3. Secretária do freelancer** · P1 · depende de R2
  - Freelancer cria um usuário `RECEPTIONIST` dentro do limite.

## Fase 7: financeiro completo (descoberta do D2)

- [ ] **D2.1. Sessão de refinamento do repasse** · P1
  - Decidir: modelo (percentual, tabela, fixo mensal), base (bruto ou líquido de laboratório, taxa de cartão, material), momento (produzido ou recebido), exceções por procedimento, regra com data de vigência e fechamento do período.
  - Ponto de partida: esboço de regra e extrato já discutido; freelancer como o mesmo motor com fluxo invertido.
- [ ] **D2.2. Módulo de recebimentos** · P1 · depende de D2.1
  - Pagamentos, parcelas e forma de pagamento, ligados a orçamento e atendimento.
- [ ] **D2.3. Histórico financeiro do paciente** · P1 · depende de D2.2
  - Total gasto na clínica, gasto por procedimento e valores em aberto.
- [ ] **D2.4. Motor de repasse** · P2 · depende de D2.1, D2.2
  - Regra com vigência, exceções, extrato com status (aberto, fechado, pago) e ajustes.
- [ ] **D2.5. Dashboard real** · P2 · depende de D2.2
  - Substituir o mock do dashboard por dados reais.

---


