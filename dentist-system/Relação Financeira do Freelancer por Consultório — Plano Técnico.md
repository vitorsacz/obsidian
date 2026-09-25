#projeto #dentist-system #roadmap #freelancer #financeiro

Ver [[Visão Geral]] pro contexto do produto, [[Roadmap]] (seção "Feito") pro
estado geral, [[Arquitetura]] pro schema atual. **Status: implementado em
2026-09-22 (PR #11, commit `0aeed35`) — plano aprovado sem mudança de
desenho.** Decisões das "Perguntas em aberto" (seção abaixo) confirmadas
como propostas: opcional na criação, `RentPeriodicity` sem `YEARLY`,
`ownerLabel` texto livre, `ClinicProcedureRate` fora do v1. Ver
[[Arquitetura]] e [[Funcionalidades e Endpoints]] pro desenho técnico final
(model `ClinicFinancialTerms`, endpoint `GET`/`PUT
clinics/:clinicId/financial-terms`).

## Contexto e o que já existe hoje

O `Clinic` real (consultório) já tem `type: ClinicType` (`OWN`/`RENTED`) +
`dailyRentValue: Decimal?` — mas esses campos foram desenhados pro contexto
de **tenant Clínica** (uma clínica que paga aluguel por um espaço que ela
mesma opera, alimentando o `rentCost` do relatório financeiro hoje — ver
[[Funcionalidades e Endpoints]], `reports/financial`). Isso é uma relação
diferente da que este plano cobre: aqui é o **próprio freelancer** que tem
uma relação financeira pessoal com cada consultório onde atende (paga
aluguel, paga comissão, ou é pago pelo espaço) — não faz sentido reaproveitar
`ClinicType`/`dailyRentValue` pra isso, é um conceito novo e paralelo, não
uma extensão do existente.

O termo "Location" usado no pedido é o conceito visual/mockado que já existe
hoje só no front (`MockLocation { id, name, colorToken }`, ver
[[Arquitetura]] seção "Padrão mock primeiro, backend depois") — no banco
real ainda é `Clinic`. Este plano assume que a relação financeira pendura em
cima do `Clinic` real (não espera o rename futuro pra `Location` do plano da
Agenda, ver [[Implementação da Agenda Multi-Consultório — Plano Técnico]]) —
se esse rename acontecer antes desta feature ser implementada, é só
substituir a FK.

## 1. Modelo de dados proposto

### Tabela nova: `ClinicFinancialTerms` (1:1 com `Clinic`)

Proposta: **tabela separada**, não colunas soltas no `Clinic`. Motivo:

- Hoje ~100% dos `Clinic` reais pertencem a tenant tipo Clínica — se os
  campos de relação financeira do freelancer virassem colunas do `Clinic`,
  seriam colunas **sempre nulas** pra quase todas as linhas do sistema, pra
  sempre (cheiro de coluna esparsa/mal modelada). Numa tabela separada, ela
  simplesmente **não tem linha** pra um `Clinic` de tenant Clínica — o
  isolamento fica óbvio pela ausência do relacionamento, não por uma
  checagem de "esses campos estão todos null mesmo".
- Também considerei um campo `Json` único (`relationshipDetails: Json`) em
  vez de colunas tipadas — mais flexível pra adicionar tipos de relação no
  futuro sem migration nova, mas **descartei**: os KPIs da seção 3 abaixo
  dependem de `SUM()`/`GROUP BY` em SQL sobre esses valores (ex.: somar
  aluguel fixo de todos os consultórios `RENTED_FIXED` no período) — colunas
  tipadas fazem isso de graça, JSON exigiria path queries mais caras e menos
  óbvias de auditar. Com só 3 tipos de relação e ~2 campos cada, a
  "esparsidade" de colunas nulas dentro desta tabela nova é pequena e
  aceitável (bem diferente do problema de colocar isso direto no `Clinic`).

```prisma
enum LocationRelationshipType {
  RENTED_FIXED   // "Alugado"
  COMMISSION     // "Comissionado"
  PER_SERVICE    // "Recebe por serviço"
}

enum RentPeriodicity {
  DAILY
  WEEKLY
  MONTHLY
}

model ClinicFinancialTerms {
  id     String @id @default(cuid())
  clinicId String @unique
  clinic Clinic @relation(fields: [clinicId], references: [id], onDelete: Cascade)

  // Redundante com clinic.organizationId, mas necessário direto na tabela —
  // é o mesmo padrão já usado em BudgetItem/MaterialBatch/MaterialUsage
  // (ver Arquitetura, seção Multi-tenancy: "todo model de negócio... mesmo
  // filhos alcançáveis via relação, porque são consultados diretamente").
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  relationshipType LocationRelationshipType

  // --- RENTED_FIXED ---
  rentValue       Decimal?          @db.Decimal(10, 2)
  rentPeriodicity RentPeriodicity?

  // --- COMMISSION ---
  commissionPercentage Decimal? @db.Decimal(5, 2) // 0–100

  // --- PER_SERVICE ---
  defaultServiceRate Decimal? @db.Decimal(10, 2) // valor fixo por atendimento (fallback)

  // Referência livre, não modelada como entidade — ver observação abaixo.
  ownerLabel String? // "proprietário"/contato, texto livre

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([organizationId])
}
```

**Extensão futura, não incluída neste v1** (registrar aqui pra não esquecer,
não implementar agora): `ClinicProcedureRate` (`financialTermsId`,
`procedureId`, `rate`) — permitiria `PER_SERVICE` variar por procedimento em
vez de valor fixo (ver pergunta do Vitor no pedido original). Proposta: **v1
só com `defaultServiceRate` fixo** — é o suficiente pro caso de uso mais
comum (consultório paga X por atendimento, sem diferenciar procedimento), e
a tabela de rate por procedimento entra depois, sem quebrar nada, se aparecer
demanda real (mesmo princípio de incremento usado no resto do projeto — ver
[[Arquitetura]]).

### Os três tipos são mutuamente exclusivos (recomendação)

Um `Clinic` tem **no máximo uma** linha em `ClinicFinancialTerms`, com um
único `relationshipType`. Não recomendo campos híbridos (ex.: aluguel fixo
reduzido + comissão acima de um teto) — é um caso real que existe no mundo,
mas não foi pedido e adicionaria uma camada de regra de cálculo bem mais
complexa (qual comissão incide sobre o quê, teto, etc.) sem um caso de uso
concreto puxando isso agora. Se aparecer necessidade real de um modelo
híbrido no futuro, a recomendação é adicionar um `relationshipType` novo
explícito (ex.: `HYBRID_RENT_PLUS_COMMISSION`) com seus próprios campos,
não tentar generalizar os três atuais pra permitir combinação livre.

**Validação de campos por tipo** (camada de aplicação, Zod — mesmo padrão já
usado em `createClinicSchema` hoje, que exige `dailyRentValue` só quando
`type === "RENTED"`): union discriminada por `relationshipType`, cada branch
exigindo só os campos daquele tipo e proibindo os dos outros (evita salvar
`commissionPercentage` preenchido num registro `RENTED_FIXED`, por exemplo).

## 2. Isolamento pro contexto Freelancer

`ClinicFinancialTerms` **não tem** um campo tipo "só se aplica quando
tenant.type = FREELANCER" — o isolamento não é um campo, é uma **regra de
criação**: o serviço que cria/edita um `ClinicFinancialTerms` verifica
`organization.type` antes de permitir, exatamente como o `ClinicsService`
já faz hoje pra bloquear criação de `Clinic` novo em tenant Clínica (ver PR
#9 mergeado — `if (organization?.type === "CLINIC") throw new
ForbiddenException(...)`). Aqui a regra é a **oposta**: só permite criar/
editar `ClinicFinancialTerms` quando `organization.type === "FREELANCER"`
— pra tenant Clínica, o endpoint correspondente responde 403 (ou nem existe
rota exposta pro papel ADMIN/DENTIST de clínica acessar).

Consequência prática: um `Clinic` de tenant Clínica **nunca tem** uma linha
em `ClinicFinancialTerms` — não porque um campo está vazio, mas porque a
linha simplesmente nunca é criada (nenhum código no caminho de tenant
Clínica sabe que essa tabela existe). Isso já é consistente com o padrão
estabelecido no PR #9 (regra de negócio no serviço, não só escondida na UI)
e com o padrão de isolamento por `organizationId` de todo o resto do sistema
(ver [[Arquitetura]], seção Multi-tenancy) — `ClinicFinancialTerms` entra na
lista `TENANT_SCOPED_MODELS` da extension do Prisma normalmente, sem
tratamento especial.

**Front-end**: mesmo padrão já usado em `clinics-page.tsx` (`canCreateClinic
= myClinicQuery.data?.type !== "CLINIC"`, ver PR #9) — a UI de editar relação
financeira de um consultório só aparece/é chamável quando `GET /organization`
devolve `type: "FREELANCER"`.

## 3. KPIs que ficam calculáveis

Cruzando `ClinicFinancialTerms` com `Attendance` (`clinicId`, `date`,
`grossValue`, `materialCost` — já indexado por `[clinicId, date]`, ver
schema atual) e `Appointment` (`clinicId`, `startsAt`, `status`). Lista só —
nenhum destes está implementado, é o que **se torna possível** calcular.

### Por consultório `RENTED_FIXED`
- Custo de aluguel no período (valor × nº de ciclos da periodicidade dentro
  do range — ex.: mensal conta 1x por mês corrido; diário conta por dia com
  atendimento, mesmo cálculo já usado hoje pro `rentCost` de Clínica).
- Receita bruta gerada no consultório no período (`SUM(grossValue)` por
  `clinicId`).
- Receita líquida do freelancer naquele consultório (bruta − aluguel do
  período − custo de material).
- Ticket médio por atendimento no consultório.
- Ponto de equilíbrio: quantos atendimentos/qual receita mínima no período
  cobre o aluguel pago.

### Por consultório `COMMISSION`
- Valor de comissão devido ao proprietário no período (`grossValue × %`
  por atendimento, somado).
- Receita líquida do freelancer (bruta − comissão − custo de material).
- Comparação de rentabilidade efetiva entre consultórios com percentuais de
  comissão diferentes, dado volume/ticket parecidos.

### Por consultório `PER_SERVICE`
- Valor recebido do consultório no período (nº de atendimentos ×
  `defaultServiceRate`, ou soma por procedimento se a extensão de rate por
  procedimento entrar).
- Aqui não há "receita bruta do paciente" relevante pro freelancer (quem
  paga é o consultório, não o paciente a ele) — o KPI é direto: quanto
  ganhou daquele local, sem dedução de comissão/aluguel.
- Produtividade (nº de atendimentos realizados ali) — métrica operacional,
  não financeira, mas informa "vale a pena continuar atendendo aqui".

### Cross-consultório (comparativos, qualquer tipo)
- Ranking de consultórios por receita líquida no período (precisa de uma
  função "líquido" que abstrai por tipo: bruto relevante − custo do tipo).
- Nº de atendimentos por consultório no período — **já calculável hoje**
  sem nenhuma mudança de schema (só `GROUP BY clinicId` em `Attendance`).
- Receita bruta total do freelancer, segmentada por consultório e por tipo
  de relação — visão "de onde vem o dinheiro".
- Custo total de "ocupação" no período (soma de aluguéis fixos pagos +
  comissões pagas), cross-consultório.
- % de ocupação da agenda por consultório (cruza com `Appointment`, não só
  `Attendance` realizado — inclui agendado não realizado ainda).
- Simulação "e se": pra `RENTED_FIXED`, cada atendimento a mais é margem
  quase pura (custo fixo não cresce com volume); pra `COMMISSION`, cada
  atendimento a mais tem custo marginal proporcional — dá pra comparar
  "onde vale mais a pena investir tempo/agenda" entre dois consultórios de
  tipos diferentes.

## Perguntas em aberto pra validar antes de implementar

1. `ClinicFinancialTerms` é **opcional** na criação do consultório (o
   freelancer pode cadastrar o local primeiro, definir a relação financeira
   depois) ou **obrigatório** desde já? Recomendo opcional — os KPIs da
   seção 3 simplesmente ignoram/sinalizam consultórios sem termos definidos,
   sem bloquear o fluxo de "adicionar consultório" que já existe.
2. `RentPeriodicity`: proponho `DAILY`/`WEEKLY`/`MONTHLY` — falta `YEARLY`
   ou outra granularidade?
3. `ownerLabel` (texto livre pra identificar o proprietário/comissionado) é
   suficiente, ou algum dia isso precisa virar uma entidade de verdade
   (contato com telefone/e-mail, pra gerar recibo de pagamento por
   exemplo)? Proponho texto livre por enquanto — não modelar uma entidade
   nova sem um caso de uso puxando isso.
4. A extensão de rate por procedimento (`ClinicProcedureRate`) entra já
   junto com o v1, ou fica mesmo pra depois como sugerido acima?
