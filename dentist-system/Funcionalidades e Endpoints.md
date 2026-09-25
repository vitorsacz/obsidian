#projeto #dentist-system #api

Ver [[Visão Geral]] pro contexto do produto, [[Arquitetura]] pro desenho técnico por trás.
Esta nota é o **norte funcional**: o que o sistema faz de verdade hoje, endpoint por
endpoint, com exemplo real de request/response — pra consultar antes de decidir o que
falta implementar ou mudar.

Convenções globais (valem pra tudo abaixo, não repetidas em cada endpoint):
- Sem prefixo `/api` — o path do `@Controller` já é o path final (ex.: `auth/login`).
- Toda rota exige JWT (Bearer access token) exceto as marcadas **público**.
- Todo body é validado por Zod (`packages/shared-types`) via `ZodValidationPipe` —
  erro de validação retorna `400` com `{ message, error }` (formato `.flatten()` do Zod).
- Todo `Decimal` do Prisma (dinheiro, quantidade) já chega como `number` no JSON
  (interceptor global — ver [[Problemas Conhecidos]]).

---

## auth — `auth/*`

| Rota | Quem acessa |
|---|---|
| `POST auth/lookup` | público |
| `POST auth/login` | público |
| `POST auth/refresh` | público (lê cookie) |
| `POST auth/logout` | qualquer autenticado |
| `GET auth/me` | qualquer autenticado |

**Identidade é isolada por tenant** (2026-09-17, revisado no mesmo dia da
multi-tenancy — ver [[Arquitetura]]): o mesmo e-mail pode existir em mais de
uma organização, como contas completamente independentes. Login é em 2 passos:

```json
// POST auth/lookup
{ "identifier": "dra.marina@sorrisosaudavel.com.br" }
// → 201, e-mail existe em só 1 organização (ou nenhuma)
{ "requiresOrganizationSelection": false, "accounts": [] }
// → 201, e-mail existe em mais de uma organização
{ "requiresOrganizationSelection": true, "accounts": [{ "organizationId": "clx0...", "organizationName": "Clínica A" }, { "organizationId": "clx1...", "organizationName": "Clínica B" }] }
```
`identifier` aceita e-mail **ou** `nickname` (apelido único globalmente,
opcional) — logar por nickname nunca pede escolha de organização, mesmo que o
e-mail associado se repita em outro tenant.

```json
// POST auth/login
{ "identifier": "dra.marina@sorrisosaudavel.com.br", "password": "SenhaForte123", "organizationId": "clx0..." }
// → 200
{ "accessToken": "<jwt>" }
```
`organizationId` só é obrigatório quando o `lookup` anterior devolveu
`requiresOrganizationSelection: true` — mandar sem ele nesse caso dá `401`
(mensagem genérica, igual pra e-mail/senha errados — nunca revela qual dos
casos aconteceu). Gera access token (15min, em memória no front) + refresh
token (30 dias, cookie httpOnly).

**Refresh** — sem body, lê o cookie `refresh_token`; reemite os dois tokens (rotação).
Não existe endpoint público de cadastro — o primeiro usuário/organização nascem do
`prisma/seed.ts`; Super Admin cria clínicas novas por `platform/*`, admin da clínica
cria dentista/recepcionista por `users/*`.

---

## users — `users/*` (só ADMIN, a nível de tenant)

Painel de administração de usuários **da própria organização** — opera direto
em `User` (não existe mais `Membership` no meio, removida no mesmo dia em que
foi criada — ver [[Arquitetura]]). `:id` nas rotas abaixo é o id do próprio
`User`. Checagem de e-mail já existente roda só **dentro do tenant** — nunca
revela se aquele e-mail já existe em outra organização.

```json
// POST users
{ "email": "recepcao@sorrisosaudavel.com.br", "password": "Recep2026!", "name": "Juliana Costa", "role": "RECEPTIONIST" }
// → 201
{ "userId": "clx2...", "email": "recepcao@sorrisosaudavel.com.br", "name": "Juliana Costa", "role": "RECEPTIONIST", "active": true, "colorToken": null, "createdAt": "..." }
```
**`colorToken` (2026-09-22, PR #12)** — opcional em `POST`/`PATCH users/:id`,
só relevante pra `role: "DENTIST"` (identidade na sidebar da Agenda). Se
omitido na criação de um dentista, o serviço cicla a paleta de 5 tokens por
índice (nº de dentistas já cadastrados no tenant) — mesma lógica que antes
vivia no front (`dentist-store.ts`, removido).
```json
// PATCH users/:userId
{ "active": false }
```
Duas regras de proteção:
- Admin não consegue desativar ou rebaixar a **própria** conta (`400`) — evita
  se trancar fora do sistema.
- **Capitania da clínica**: se o alvo for `ADMIN` e não for quem está agindo,
  só o admin **fundador** da organização (`Organization.foundingAdminUserId`)
  pode editar/desativar (`403` senão). Transferir a capitania não é
  self-service — só o Super Admin faz isso, via `platform/*`.

```json
// PATCH users/:userId/password — reset de senha (2026-08-03)
{ "password": "NovaSenhaForte123" }
// → 200, mesmo shape do usuário (nunca retorna hash)
```
Sem restrição de auto-reset (admin pode redefinir a própria senha).

⚠️ `User`/`Organization` ficam de fora da extension automática de tenant (são
o próprio mecanismo de resolução — login precisa buscar `User` por e-mail sem
saber a organização ainda) — todo filtro por `organizationId` neste módulo é
manual no `UsersService`, sem a rede de segurança que o resto da API tem por
padrão. Ver [[Arquitetura]].

---

## organization — `organization` (qualquer papel autenticado, 2026-09-17)

Rota de leitura simples pra "minha clínica" — nome da organização + lista de
quem faz parte dela. Diferente de `users/*`, **não é restrita a ADMIN**: um
dentista ou recepcionista também pode ver quem trabalha na mesma clínica.
Sempre resolve a organização do próprio token (`user.organizationId` do JWT),
nunca aceita um id de organização por parâmetro — não existe rota tipo
`GET /organization/:id`.

```json
// GET organization
{
  "id": "clx0...",
  "name": "Consultório Padrão",
  "type": "CLINIC",
  "members": [
    { "userId": "clx1...", "name": "Dra. Exemplo", "role": "DENTIST", "active": true },
    { "userId": "clx2...", "name": "Recepcionista Teste", "role": "RECEPTIONIST", "active": true }
  ]
}
```
Não retorna email nem dado sensível dos colegas — só nome/papel/ativo. Pra
gerenciar de verdade (criar, desativar, resetar senha) continua sendo
`users/*`, só ADMIN. Sem endpoint de edição do nome da clínica ainda — só
leitura por enquanto. **`type` (2026-09-22, PR #9)** — expõe
`organization.type` (`CLINIC`/`FREELANCER`) pro front decidir quando mostrar
"Novo consultório" (só Freelancer).

```json
// GET organization/dentists (2026-09-22, PR #12 — ADMIN, RECEPTIONIST; NUNCA DENTIST)
[
  { "userId": "clx1...", "name": "Dra. Exemplo", "colorToken": "BRAND" },
  { "userId": "clx4...", "name": "Dr. Outro Dentista", "colorToken": "SUCCESS" }
]
```
Roster real pra sidebar "por dentista" da Agenda (tenant `CLINIC`,
admin/recepcionista). Rota separada de `GET /organization` de propósito:
aquela é permissiva pra qualquer papel; esta é `@Roles("ADMIN",
"RECEPTIONIST")` — `DENTIST` recebe `403` de verdade, não filtro de UI. Só
lista dentista `active: true` (inativo some do roster, mas continua
vinculado ao histórico de agendamentos que já existir).

---

## platform — `platform/*` (só Super Admin)

Painel da plataforma — Vitor, fora de qualquer organização
(`User.isSuperAdmin: true`, `organizationId: null`). Guardado por um
`SuperAdminGuard` próprio, não pelo `RolesGuard`/`@Roles(...)` global (Super
Admin não tem `role` de tenant). Escopo desta rodada: criar/listar/detalhar
clínicas, transferir capitania, e um dashboard de estatísticas cross-tenant
(2026-09-18) — cadastro progressivo, LGPD, auditoria, billing e add-ons
ficam pra depois (ver [[feature - painel-admin-rbac-lgpd]] e a seção
"Painel do Super Admin — próximas fases" em [[Roadmap]]). `TenantType.FREELANCER`
já existe de verdade desde 2026-09-22 (PR #10) mas `platform/*` ainda só
cria organização tipo `CLINIC` (`POST platform/organizations` não aceita
`type`) — criar um tenant Freelancer hoje só é possível via script/seed
(`seed-test-personas.ts`), não pelo painel.

```json
// POST platform/organizations
{ "name": "Clínica Sorriso Saudável", "foundingAdminEmail": "dra.marina@sorrisosaudavel.com.br", "foundingAdminName": "Dra. Marina", "foundingAdminPassword": "SenhaForte123" }
// → 201 — cria a Organization + o User admin fundador numa transação
{ "id": "clx0...", "name": "Clínica Sorriso Saudável", "type": "CLINIC", "status": "ACTIVE", "foundingAdminUserId": "clx1...", "createdAt": "..." }
```
```json
// GET platform/organizations
// → lista todas as organizações da plataforma
```
```json
// GET platform/organizations/:id
// → dado cadastral de uma organização + nome/email do admin fundador
{ "id": "clx0...", "name": "Clínica Sorriso Saudável", "type": "CLINIC", "status": "ACTIVE", "foundingAdminUserId": "clx1...", "createdAt": "...", "foundingAdmin": { "name": "Dra. Marina", "email": "dra.marina@sorrisosaudavel.com.br" } }
```
```json
// PATCH platform/organizations/:id/founding-admin
{ "userId": "clx3..." }
// → transfere a capitania pra outro ADMIN já existente na mesma organização
```
```json
// GET platform/stats/overview
// → visão geral cross-tenant pro dashboard: organizações por status,
// usuários por papel, dentistas (ativos vs. total cadastrado), atendimentos
// no mês corrente, crescimento de novos tenants nos últimos 6 meses (zero-
// filled) e ranking das top 10 clínicas por atendimentos do mês corrente
{
  "organizationsByStatus": { "ACTIVE": 2, "SUSPENDED": 0, "DELETED": 0, "total": 2 },
  "usersByRole": { "ADMIN": 2, "DENTIST": 3, "RECEPTIONIST": 1, "total": 6 },
  "totalDentistsRegistered": 3,
  "totalDentistsActive": 3,
  "newTenantsByMonth": [{ "month": "2026-04", "count": 0 }, { "month": "2026-09", "count": 2 }],
  "attendancesThisMonth": 0,
  "topOrganizationsByAttendance": []
}
```
Transferência de capitania nunca é self-service — só existe por este caminho,
não tem equivalente em `users/*` pro próprio admin fundador se substituir.

`GET platform/stats/overview` é o único endpoint do backend inteiro que usa
o client Prisma sem a extension de isolamento de tenant (token DI
`PRISMA_UNSCOPED_SERVICE`, ver [[Arquitetura]]) — necessário porque o Super
Admin não tem `organizationId`, então o `AsyncLocalStorage` de tenant nunca é
populado pra ele, e as queries agregadas em `Attendance` (que é tenant-scoped)
precisam atravessar todas as organizations de propósito.

---

## patients — `patients/*` (leitura: ADMIN+DENTIST+RECEPTIONIST desde 2026-09-22; escrita: DENTIST, RECEPTIONIST)

CRUD de paciente. `birthDate` aceita `""` do `<input type="date">` vazio sem quebrar
(preprocess `optionalCoercedDate`).
```json
// POST patients
{ "name": "Carlos Eduardo Ferreira", "phone": "(11) 98765-4321", "birthDate": "1985-03-14", "address": "Rua das Flores, 120, São Paulo - SP" }
```

### Anamnese — `patients/:patientId/anamnesis` (só DENTIST)
Um registro por paciente (upsert). Recepcionista não acessa — dado clínico sensível.
```json
// PUT patients/:patientId/anamnesis
{
  "allergies": "Penicilina",
  "medications": "Losartana 50mg",
  "healthConditions": "Hipertensão controlada",
  "dentalHistory": "Extração do dente 38 em 2019",
  "chiefComplaint": "Sensibilidade ao frio no quadrante superior direito"
}
```

### Prontuário/evolução — `patients/:patientId/clinical-records` (só DENTIST)
Histórico de atendimentos em texto livre, uma linha por consulta.
```json
// POST patients/:patientId/clinical-records
{ "date": "2026-08-02", "procedureNote": "Restauração em resina composta no dente 26 (face oclusal)", "observations": "Sensibilidade leve pós-procedimento." }
```

### Odontograma — `patients/:patientId/tooth-records` (só DENTIST)
Notação FDI validada contra whitelist fechada de 32 valores (`11`–`18`, `21`–`28`,
`31`–`38`, `41`–`48`). Cada POST **cria** um novo registro de histórico pro dente —
não sobrescreve o anterior (apesar do nome do schema ser "upsert"), por isso o front
sempre pega o mais recente por `updatedAt` pra decidir a cor do dente no mapa
(ver [[Roadmap]]).
```json
// POST patients/:patientId/tooth-records
{ "toothNumber": 26, "procedure": "Restauração", "status": "PLANNED", "notes": "Cárie oclusal moderada" }
```

```json
// PATCH patients/:patientId/tooth-records/:id/status (2026-08-03)
{ "status": "DONE" }
```
Transiciona um registro existente (ex.: "Planejado" → "Realizado") sem criar uma
entrada nova no histórico — resolve o caso comum de "procedimento planejado foi
feito", que antes só dava pra registrar criando outra linha duplicada. Botão
"Marcar como realizado" no front, visível só em registros ainda `PLANNED`.

---

## clinics — `clinics/*` (leitura: ADMIN+DENTIST+RECEPTIONIST desde 2026-09-22; escrita: só DENTIST)

Consultório físico (`OWN`/`RENTED`). Se `type: "RENTED"`, `dailyRentValue` é
obrigatório na criação (regra `.refine()` do Zod) — mas **não** é reforçada no update,
então dá pra trocar um consultório pra `RENTED` sem diária, e o relatório financeiro
silenciosamente calcula `rentCost: 0` pra ele.
```json
// POST clinics
{ "name": "Consultório Vila Mariana", "type": "RENTED", "dailyRentValue": 180.00, "colorToken": "WARNING" }
```

**ADMIN ganhou leitura (2026-09-22)** — necessário pro modo multi-dentista da
Agenda redesenhada (mockada, ver [[Arquitetura]]) buscar a lista real de
consultórios/Locations. Escrita continua só DENTIST.

**Criação bloqueada pra tenant `CLINIC` (2026-09-22, PR #9)** — `POST
clinics` responde `403` quando `organization.type === "CLINIC"`, sempre,
independente de quem pede. Consultório de uma clínica é fixo (dentista de
equipe trabalha nos locais que já existem, não cadastra os próprios) — só
tenant `FREELANCER` cria novos consultórios, que representam os locais onde
ele mesmo atende.

**`colorToken` (2026-09-22, PR #12)** — opcional em `POST`/`PATCH
clinics/:id`. Se omitido na criação, o serviço cicla a paleta de 5 tokens
(`BRAND`/`SUCCESS`/`WARNING`/`ERROR`/`INFO`) por índice (nº de consultórios
já cadastrados no tenant).

---

## clinic-financial-terms — `clinics/:clinicId/financial-terms` (só DENTIST, 2026-09-22, PR #11)

Relação financeira do dentista Freelancer com um consultório específico —
1:1 opcional com `Clinic`, mesmo escopo de acesso de
`attendances`/`reports/financial` (dado financeiro sensível, ADMIN e
RECEPTIONIST ficam de fora). Só existe pra tenant `FREELANCER`: `403` pra
tenant `CLINIC` mesmo no próprio consultório (regra de negócio no serviço,
não só escondida na UI).

```json
// GET clinics/:clinicId/financial-terms
// → sem termos definidos ainda é estado válido (não bloqueia "adicionar
// consultório") — devolve null, não 404
null
```
```json
// PUT clinics/:clinicId/financial-terms
{ "relationshipType": "COMMISSION", "commissionPercentage": 40, "ownerLabel": "Dr. Proprietário" }
// → 200
{ "id": "clx9...", "clinicId": "clx7...", "relationshipType": "COMMISSION", "rentValue": null, "rentPeriodicity": null, "commissionPercentage": 40, "defaultServiceRate": null, "ownerLabel": "Dr. Proprietário", "updatedAt": "..." }
```
`relationshipType` é discriminated union no Zod — cada ramo só aceita os
campos daquele tipo:
- `RENTED_FIXED`: `rentValue` (positivo) + `rentPeriodicity`
  (`DAILY`/`WEEKLY`/`MONTHLY`, sem `YEARLY` por decisão consciente no v1).
- `COMMISSION`: `commissionPercentage` (0–100).
- `PER_SERVICE`: `defaultServiceRate` (positivo, valor fixo por atendimento —
  rate por procedimento é extensão futura, não implementada neste v1).

`ownerLabel` é texto livre (referência ao proprietário/comissionado, não uma
entidade). Trocar `relationshipType` (ex.: `RENTED_FIXED` → `COMMISSION`)
zera de verdade os campos do tipo anterior no `upsert`, não só ignora. Ver
[[Relação Financeira do Freelancer por Consultório — Plano Técnico]] e
[[Arquitetura]].

## procedures — `procedures/*` (leitura: ADMIN+DENTIST+RECEPTIONIST desde 2026-09-22; escrita: só DENTIST)

Catálogo de procedimentos reutilizáveis com valor padrão (referenciado por orçamento e
atendimento — diferente do texto livre do odontograma).
```json
// POST procedures
{ "name": "Limpeza (Profilaxia)", "defaultValue": 150.00, "active": true }
```

---

## budgets — `budgets/*` (DENTIST, RECEPTIONIST)

⚠️ `GET budgets?patientId=...` não valida o query param — se `patientId` for omitido,
retorna **todos** os orçamentos de todos os pacientes (Prisma trata `undefined` como
"sem filtro"). Sempre passar `patientId` no front.
```json
// POST budgets
{
  "patientId": "clx3...",
  "items": [
    { "procedureId": "clx8...", "toothNumber": 26, "value": 165.00, "notes": "Restauração em resina" },
    { "procedureId": "clxp2...", "value": 150.00 }
  ]
}
// → total é calculado na hora (soma dos itens), não fica gravado
```
```json
// PATCH budgets/:id/status
{ "status": "APPROVED" }
```
Status (`PENDING`/`APPROVED`/`IN_PROGRESS`/`COMPLETED`) não tem máquina de estado —
qualquer transição é aceita, inclusive "voltar" de `COMPLETED` pra `PENDING`.

**Sugestões a partir do odontograma (2026-08-06, front only)** — o formulário de
novo orçamento (`budgets-tab.tsx`) busca os `tooth-records` do paciente, pega o
mais recente por dente e mostra os que estão `PLANNED` como chips clicáveis.
Clicar num chip dá `append()` num item do orçamento já preenchido com
`toothNumber` e `notes` (texto do odontograma), e tenta casar `procedure`
(texto livre) com um nome do catálogo de procedimentos por **substring** (não
só igualdade exata) pra puxar `procedureId`/`value` automaticamente — se não
achar match, item fica com procedimento vazio e valor `0`, exige preenchimento
manual. Sem vínculo gravado no banco entre `ToothRecord` e `BudgetItem` — é só
atalho de UI, o dentista sempre revisa antes de salvar. Chip some da lista
depois de usado (controlado por `addedRecordIds` em estado local, não
persistido).

---

## appointments — `appointments/*` (DENTIST, RECEPTIONIST — ADMIN ainda NÃO)

Agenda por consultório e período. Sem checagem de conflito de horário (dois
agendamentos podem se sobrepor no mesmo consultório sem erro). ⚠️ Diferente de
`clinics`/`patients`/`procedures`, este endpoint **não** ganhou `ADMIN` em
2026-09-22 — a tela de Agenda que o admin usa é mockada (ver [[Arquitetura]]),
nunca chama `GET /appointments` de verdade. Se a Agenda virar real, lembrar de
revisar este `@Roles` também.
```json
// GET appointments?from=2026-08-01&to=2026-08-31&clinicId=clx7...
```
```json
// POST appointments
{ "patientId": "clx3...", "clinicId": "clx7...", "startsAt": "2026-08-05T13:00:00-03:00", "notes": "Primeira consulta - avaliação" }
// status nasce sempre "SCHEDULED"
```
Status também muda automaticamente pra `DONE` quando um atendimento é lançado
referenciando esse agendamento (ver `attendances` abaixo).

---

## attendances — `attendances/*` (só DENTIST)

**O endpoint com mais regra de negócio do sistema.** Registra o atendimento financeiro
e dá baixa de estoque, tudo em uma transação — se faltar material, tudo é revertido
(inclusive a criação do atendimento).
```json
// POST attendances
{
  "patientId": "clx3...", "appointmentId": "clxapp1...", "clinicId": "clx7...", "procedureId": "clx8...",
  "date": "2026-08-05", "grossValue": 165.00, "repassePercentage": 50, "materialCost": 12.50,
  "materialUsages": [ { "materialId": "clxmat1...", "quantity": 2 } ]
}
```
O que acontece por trás, em ordem:
1. Cria o `Attendance` com os valores como enviados (repasse **não** é recalculado
   aqui, só no relatório).
2. Pra cada item de `materialUsages`, dá baixa por **FEFO** (usa primeiro o lote que
   vence mais cedo; lote sem validade é consumido por último). Estoque insuficiente →
   `400` e desfaz tudo.
3. Grava um `MaterialUsage` por material, vinculado ao atendimento.
4. Se veio `appointmentId`, marca aquele agendamento como `DONE` automaticamente.

---

## reports — `reports/financial` (só DENTIST)

O motor de cálculo financeiro. Agrupa por consultório num período.
```json
// GET reports/financial?from=2026-08-01&to=2026-08-31
// → 200
{
  "from": "2026-08-01T00:00:00.000Z", "to": "2026-08-31T00:00:00.000Z",
  "clinics": [
    { "clinicId": "clx7...", "clinicName": "Consultório Vila Mariana",
      "grossRevenue": 4200.00, "dentistRepasse": 2100.00, "materialCost": 310.00, "rentCost": 1440.00, "netResult": 350.00 }
  ],
  "totals": { "grossRevenue": 4200.00, "dentistRepasse": 2100.00, "materialCost": 310.00, "rentCost": 1440.00, "netResult": 350.00 }
}
```
Regras:
- `dentistRepasse = soma(grossValue * repassePercentage/100)` de cada atendimento.
- `rentCost` só existe pra consultório `RENTED`: conta **dias distintos** com pelo
  menos um atendimento no período × `dailyRentValue` (não é por atendimento, é por
  dia com movimento).
- `netResult = dentistRepasse - materialCost - rentCost` — é o líquido que **sobra
  pra dentista**, não o resultado da clínica como um todo (`grossRevenue` não entra
  nessa conta, só é exibido como referência).

---

## materials — `materials/*` (DENTIST, RECEPTIONIST)

Estoque com lotes e alertas calculados em toda leitura (não fica gravado, é derivado
na hora): `currentStock` (soma dos lotes), `lowStock` (`currentStock < minimumStock`),
`expiringSoon` (algum lote com validade nos próximos 30 dias).
```json
// GET materials/:id
{
  "id": "clxmat1...", "name": "Luva de Látex M", "unit": "par", "minimumStock": 20,
  "batches": [ { "id": "clxb1...", "quantity": 48, "expiryDate": "2026-12-01T00:00:00.000Z", "receivedAt": "2026-06-01T00:00:00.000Z" } ],
  "currentStock": 48, "lowStock": false, "expiringSoon": false
}
```
```json
// POST materials/:id/batches — entrada de estoque (reposição)
{ "quantity": 100, "expiryDate": "2027-03-01" }
```
Lotes só são decrementados (nunca apagados) pela baixa FEFO do módulo `attendances`.

---

## recalls — `recalls/*` (DENTIST, RECEPTIONIST)

Controle de retorno/lembrete por paciente. `GET recalls` **só retorna os pendentes**
(`status: "PENDING"` fixo no filtro) — não tem como listar os já feitos/cancelados
por essa rota hoje.
```json
// POST recalls
{ "patientId": "clx3...", "dueDate": "2027-02-05", "reason": "Retorno de limpeza semestral" }
```
```json
// PATCH recalls/:id/status
{ "status": "DONE" }
```

---

## health — `health` (público)

`GET health` → `{ "status": "ok" }`. Health check simples, sem checar conexão com o
banco — usado pra keep-alive/monitoramento em produção (Render).

---

## Coisas a lembrar ao construir em cima disso

- **`patientId` sempre explícito** em `GET budgets`/`GET attendances` — omitir retorna
  tudo, não dá erro (ver ⚠️ acima).
- **Status sem máquina de estado** em `Budget`, `Recall`, `Appointment` — qualquer
  transição passa. Se precisar de regra (ex.: não permitir `COMPLETED → PENDING`),
  é trabalho novo, não existe hoje.
- **Odontograma é histórico, não estado** — sempre pegar o registro mais recente por
  dente pra saber o status atual (é isso que o front já faz).
- **RBAC por módulo inteiro, não por campo** — RECEPTIONIST nunca vê anamnese,
  prontuário, odontograma, atendimento financeiro ou relatório (tabela completa em
  [[Arquitetura]]).
