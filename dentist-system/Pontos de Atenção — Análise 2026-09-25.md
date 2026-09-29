---
tags: [projeto, dentist-system, analise, bugs, roadmap]
status: análise — virou itens do roadmap-papeis-permissoes-financeiro
relacionado: "[[roadmap-papeis-permissoes-financeiro]], [[Roadmap]], [[Problemas Conhecidos]], [[Funcionalidades e Endpoints]], [[Arquitetura]]"
---
	
# Pontos de Atenção — Análise de 2026-09-25

> **Atualização 2026-09-26.** Os pontos abaixo viraram itens do
> [[roadmap-papeis-permissoes-financeiro]]. Até agora, os PRs #13–#19 (Fase 0
> de segurança, matriz `ACCESS`, múltiplos papéis) **não resolveram nenhum dos
> pontos 1–11**. Eles continuam valendo, com os itens do roadmap que vão
> tratá-los:
> - 1 e 2: R3 e P1. **Atenção:** a decisão D6 põe o cadastro de consultórios
>   com o admin da clínica.
> - 3, 9 e 10: Fase 4 (agenda real).
> - 4: sem item ainda.
> - 5: D2.5.
> - 11: R4.
>
> O ponto 12 (notas desatualizadas) foi corrigido nesta data. Arquitetura,
> Funcionalidades, Infraestrutura, Problemas Conhecidos e Visão Geral
> refletem o estado de 2026-09-26.

Análise do estado real do código (`main` em `f8146c5`, PR #12) cruzado com as
notas do vault. Cada ponto foi conferido direto no código, não só na
documentação. Ordenado por gravidade: 🔴 bloqueia uso real, 🟠 perda de
funcionalidade/dado, 🟡 risco latente, ⚪ documentação.

## Resumo

| # | Ponto | Gravidade |
|---|---|---|
| 1 | Clínica nova não tem consultório e não consegue criar nenhum | 🔴 |
| 2 | Clínica nova não tem procedimento e a tela de Procedimentos é mock | 🔴 |
| 3 | Agenda não salva nada (regrediu de real para mock) | 🟠 |
| 4 | Recall (retorno) só existe no backend, sem tela | 🟠 |
| 5 | Home Dashboard 100% mock | 🟡 |
| 6 | `GET budgets` / `GET attendances` sem `patientId` retornam tudo | 🟡 |
| 7 | Update de consultório aceita `RENTED` sem diária | 🟡 |
| 8 | Status sem máquina de estado (orçamento, agendamento, retorno) | 🟡 |
| 9 | Agendamento sem checagem de conflito de horário | 🟡 |
| 10 | ADMIN sem acesso a `/appointments` | 🟡 |
| 11 | `Organization.status` não é checado no login | 🟡 |
| 12 | Notas do vault desatualizadas | ⚪ |

**Ordem sugerida de correção**: 1 e 2 juntos (destravam uso por qualquer
clínica nova) → 3 (agenda real, já planejada) → 4 → 6/7 (baratos, 1 linha
cada) → resto quando a área for mexida.

---

## 🔴 1. Clínica nova não tem consultório — e não consegue criar

**O que acontece**: `POST platform/organizations` cria só a `Organization` +
o admin fundador (`platform.service.ts`, `createOrganization`). O tipo sai
com o default do schema, `CLINIC`. Nenhum consultório (`Clinic`) é criado. E
desde o PR #9, `POST /clinics` responde **403 sempre** para tenant `CLINIC`
(`clinics.service.ts`, `create()` — "consultórios de uma clínica são
fixos"). O `seed.ts` também não cria consultório.

**Impacto**: `attendances` e `appointments` exigem `clinicId`
(`shared-types/attendance.ts`, `appointment.ts`). Uma clínica criada pelo
painel `/platform` **não consegue lançar atendimento nem agendamento**, por
nenhum caminho da interface. Só o `seed-test-personas.ts` e os testes e2e
criam consultório, direto no banco.

**Causa raiz**: o PR #9 decidiu que "consultório de clínica é fixo", mas
ninguém ficou responsável por criar esses consultórios fixos.

**Correção sugerida** (escolher uma):
- Super Admin cadastra os consultórios da clínica: campo no
  `POST platform/organizations` (lista de consultórios) e/ou endpoint
  `POST platform/organizations/:id/clinics`. Mais coerente com o modelo B2B
  (clínica é criada manualmente por você).
- Ou liberar `POST /clinics` para o **ADMIN** de tenant `CLINIC` (continua
  bloqueado para DENTIST/RECEPTIONIST). Mais simples, mas contradiz a regra
  "fixo".

---

## 🔴 2. Clínica nova não tem procedimento — e a tela é mock

**O que acontece**: o catálogo padrão (6 procedimentos: consulta, limpeza,
restauração, extração, canal, clareamento) só é criado pelo `seed.ts`, para a
organização padrão. `createOrganization` não cria nenhum. A página
`/procedures` usa `useMockProcedureCatalog()` (`procedures/mock-data.ts`) e
nenhum arquivo do front chama `proceduresApi.create`/`update`, apesar de
existirem em `procedures/api.ts` e o backend (`POST/PATCH /procedures`) estar
pronto.

**Impacto**: `procedureId` é obrigatório em `BudgetItem` e `Attendance`
(`shared-types/budget.ts`, `attendance.ts`). Orçamento e atendimento
**leem o catálogo real** (`budgets-tab.tsx`, `attendance-form.tsx` usam
`proceduresApi.list`), que fica vazio numa clínica nova — os seletores
aparecem sem opção. E o que a dentista cadastra na tela de Procedimentos
some no reload e nunca aparece no orçamento. Pior: a tela parece funcionar,
então o problema passa despercebido.

**Correção sugerida**:
- Criar o catálogo padrão dentro da transação de `createOrganization`
  (extrair `DEFAULT_PROCEDURES` do `seed.ts` pra um lugar compartilhado).
- Ligar a página redesenhada ao `proceduresApi` real. O mock tem campos que
  o schema não tem (categoria, etc.) — decidir se adiciona no
  `ProcedureCatalog` ou se a tela esconde por enquanto.

---

## 🟠 3. Agenda não salva nada

**O que acontece**: a Agenda original era real (`/appointments`). O
redesenho estilo Google Calendar (PR #4) trocou por estado local:
`agenda-page.tsx` guarda os agendamentos em
`useState<MockAppointment[]>`, semeado de `agenda/mock-data.ts`. Nenhum
arquivo do front chama `/appointments`. Só o roster da sidebar (consultórios
e dentistas, com cor) é real desde o PR #12.

**Impacto**: quem cria, remarca ou cancela um agendamento vê funcionar, mas
tudo some no reload. O backend de agendamentos continua pronto e sem uso. O
vault trata isso como "mock primeiro, backend depois", mas **não registra
que foi uma regressão** em relação a uma funcionalidade que já estava em
produção.

**Correção sugerida**: é o item "Agenda multi-consultório (backend real)" do
[[Roadmap]] / [[Implementação da Agenda Multi-Consultório — Plano Técnico]].
Um passo intermediário mais barato: ligar a tela ao `/appointments` que já
existe (tem `clinicId`, `startsAt`, status) antes de modelar
`Location`/`Calendar`. Vai precisar de: campo de dentista no `Appointment`
(para o modo multi-dentista), `endsAt`/duração, e ajustar o `@Roles` (ver
ponto 10).

---

## 🟠 4. Recall (retorno) sem tela

**O que acontece**: o módulo `recalls` existe no backend (`GET`/`POST`/
`PATCH status`), mas nenhum arquivo do front usa `recalls`. O [[Roadmap]]
marca "Controle de retorno (recall)" como feito.

**Impacto**: funcionalidade inexistente para o usuário. Detalhe extra:
`GET recalls` só devolve `PENDING` (filtro fixo em `recalls.service.ts`),
então não dá pra ver histórico nem pela API.

**Correção sugerida**: aba "Retornos" no detalhe do paciente + lista de
pendentes (bom candidato a card no Home Dashboard, ponto 5). Corrigir o
[[Roadmap]] para "backend pronto, sem tela".

---

## 🟡 5. Home Dashboard 100% mock

`dashboard-page.tsx` usa `useMockDashboardOverview()` (dado fixo com
`setTimeout`). É a primeira tela que DENTIST/RECEPTIONIST veem ao logar —
números falsos podem confundir num uso real. Sugestão: até ter endpoint
próprio, montar os cards com o que já existe (`reports/financial` do mês,
`materials` com `lowStock`, `recalls` pendentes) ou mostrar um aviso de
"dados de exemplo".

---

## 🟡 6. `GET budgets` / `GET attendances` sem `patientId` retornam tudo

`@Query("patientId") patientId: string` sem validação
(`budgets.controller.ts`, `attendances.controller.ts`). Se vier vazio, o
Prisma recebe `where: { patientId: undefined }`, que significa "sem
filtro". A extension de tenant continua valendo, então **não vaza entre
clínicas** — mas devolve todos os orçamentos/atendimentos da organização, e
o RBAC por paciente fica só na disciplina do front. Já estava documentado
em [[Funcionalidades e Endpoints]]. Correção: validar com Zod
(`patientId` obrigatório) ou `400` se ausente — uma linha.

---

## 🟡 7. Update de consultório aceita `RENTED` sem diária

`createClinicSchema` tem o `.refine()` ("alugado precisa de diária"), mas
`updateClinicSchema` não (`shared-types/clinic.ts`). Um `PATCH` com
`type: "RENTED"` num consultório sem `dailyRentValue` passa, e o relatório
financeiro calcula `rentCost: 0` em silêncio — o resultado líquido da
dentista fica **maior do que o real**. Correção: checar no service,
combinando com o valor já gravado (o refine sozinho não resolve, porque o
PATCH pode trazer só `type`).

---

## 🟡 8. Status sem máquina de estado

`Budget`, `Appointment` e `Recall` aceitam qualquer transição
(`budgets.service.ts`, `recalls.service.ts` gravam `input.status` direto).
Ex.: orçamento `COMPLETED` volta para `PENDING`; agendamento `DONE`
(marcado automaticamente pelo atendimento) volta para `SCHEDULED`. Baixa
prioridade enquanto é uma dentista só; vira problema com relatório/KPI em
cima de status. Correção: mapa de transições permitidas por enum, checado
no service.

---

## 🟡 9. Agendamento sem checagem de conflito

Dois agendamentos no mesmo consultório e horário são aceitos sem erro. A
Agenda nova até **renderiza** sobreposição lado a lado
(`slotEventOverlap={false}`), o que é certo pra dentistas diferentes —
mas o mesmo dentista em dois lugares ao mesmo tempo deveria ser bloqueado
ou ao menos avisado. Já está listado como ponto aberto no plano técnico da
agenda; resolver junto com o ponto 3.

---

## 🟡 10. ADMIN sem acesso a `/appointments`

`appointments.controller.ts` continua `@Roles("DENTIST", "RECEPTIONIST")`.
Hoje não quebra nada porque a Agenda é mock (ponto 3), mas o modo
multi-dentista foi feito justamente para ADMIN/RECEPTIONIST. **Tem que
entrar no mesmo PR** que ligar a Agenda ao backend, senão o ADMIN abre uma
agenda vazia com 403. Já anotado em [[Problemas Conhecidos]].

---

## 🟡 11. `Organization.status` não é checado no login

`Organization` tem `status` (`ACTIVE`/`SUSPENDED`/`DELETED`), mas
`auth.service.ts` e `jwt.strategy.ts` só checam `user.active`. Uma clínica
`SUSPENDED` ou `DELETED` continua logando e usando tudo. Hoje é latente:
não existe endpoint para mudar o status (o `PlatformController` só tem
criar/listar/detalhar/transferir capitania). Quando entrar "suspender
clínica" no painel, o login e o `jwt.strategy` precisam checar o status da
organização junto — senão suspender não suspende nada.

---

## ⚪ 12. Notas do vault desatualizadas

- [[Visão Geral]] — "Primeiro commit feito localmente, ainda sem push": já
  tem 12 PRs mergeados.
- [[Infraestrutura e Deploy]] — diz que o CI roda "lint + typecheck +
  build"; roda também `pnpm test` (e2e com Postgres). A seção "Credenciais
  de teste" ainda cita `@example.com` como principal.
- [[Arquitetura]] — árvore do front lista `features/budgets` (não existe;
  orçamento está em `features/patients/tabs/budgets-tab.tsx`) e não lista
  `platform`, `my-clinic`, `financeiro`.
- [[Problemas Conhecidos]] — "Limitações conhecidas" ainda fala de
  cadastro/cor de dentista em `localStorage` (removido no PR #12).
- [[Roadmap]] — recall marcado como feito (ver ponto 4); Procedimentos
  descrito como redesenho, sem registrar que o cadastro real ficou sem tela
  (ponto 2).
