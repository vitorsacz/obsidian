#projeto #dentist-system #roadmap #agenda #freelancer

Ver [[Visão Geral]] pro contexto, [[Roadmap]] (seção "Feito") pro estado
geral, [[Arquitetura]] (seção "Padrão mock primeiro, backend depois") pro
que existe hoje. **Status: implementado em 2026-09-22 (PR #12, commit
`69bbff8`) — plano aprovado sem mudança de desenho.** Roster real nos dois
eixos (`Clinic`/`User`), `PaletteColorToken` no schema, endpoint novo
`GET organization/dentists`. Agendamentos/calendários em si continuam
mockados nesta rodada — só o roster mudou. Ver [[Arquitetura]] e
[[Funcionalidades e Endpoints]] pro desenho técnico final.

## Escopo desta rodada (e o que fica de fora, de propósito)

**Dentro do escopo**: trocar a fonte de dado da **sidebar de calendários**
da Agenda (o roster — lista de consultórios ou de dentistas, com cor) de
mockado pra real, nos dois eixos:
- Freelancer → `Clinic` reais do próprio tenant.
- Clínica, admin/recepcionista → `User` reais com `role: DENTIST` da
  organização.
- Clínica, dentista → sem sidebar (já decidido, sem mudança).

**Fora do escopo, não mexe agora**: os **agendamentos/calendários em si**
(`MockAppointment`/`MockCalendar`) continuam mockados — essa rodada troca só
quem aparece na lista de "que calendário eu posso ligar/desligar", não o
conteúdo dos agendamentos dentro de cada um. `Location`/`Calendar` reais
(ver [[Implementação da Agenda Multi-Consultório — Plano Técnico]]) seguem
como trabalho futuro separado.

## 1. Campo de cor: existe hoje?

**Não existe em nenhum dos dois.** Hoje cor é 100% derivada no front, nunca
persistida:
- `Clinic` (freelancer): `location-colors.ts`,
  `locationColorForIndex(index)` — cicla os 5 tokens oficiais pela posição
  no array, sem gravar nada.
- Dentista (clínica): `dentist-store.ts`, `MockDentist.colorToken` — mock
  completo em `localStorage`, comentário no próprio código já diz "quando o
  backend ganhar esse conceito, isso vira um GET/POST real" (PR #7).

**Migration proposta** (não aplicar ainda): mesmo enum de cor pros dois
models, criado uma vez em `packages/shared-types` (pra Zod e Prisma
compartilharem os 5 valores) e no schema:

```prisma
enum PaletteColorToken {
  BRAND
  SUCCESS
  WARNING
  ERROR
  INFO
}

model Clinic {
  // ...campos existentes
  colorToken PaletteColorToken?
}

model User {
  // ...campos existentes
  colorToken PaletteColorToken?
}
```

Nullable nos dois — linhas já existentes não têm valor (sem backfill
necessário). Onde `colorToken` for `null`, o front continua caindo no
fallback por índice (`locationColorForIndex`), sem regressão visual pra
consultório/dentista já cadastrado antes da migration. Na criação
(`POST /clinics`, `POST /users`), `colorToken` vira opcional no input — se
omitido, o **serviço** (não mais o front) atribui a próxima cor por índice
dentro daquele tenant, mesma lógica que hoje mora em
`paletteColorForIndex(current.length)` no front.

**Ponto fora do escopo, sinalizar e não resolver agora**: o mock de
dentista também carrega `croUf` (CRO+UF), que não tem campo equivalente no
`User` real hoje. Essa integração não propõe adicionar `croUf` ao `User` —
é uma decisão separada (o roster real, por enquanto, mostra nome + cor, sem
CRO+UF, até decidirmos se/onde esse dado entra no modelo real).

## 2. Onde fica a decisão de qual eixo mostrar

Um hook novo, único ponto de decisão — `useAgendaCalendarAxis()` (ou
incorporado diretamente na reescrita de `useAgendaMockData`, que passa a se
chamar algo como `useAgendaData`). Ele já teria motivo pra existir mesmo sem
essa mudança, porque **hoje a Agenda nem chama `GET /organization`** — só
`clinicsApi.list()`. Esse hook passa a:

1. Buscar `GET /organization` (já existe, já devolve `type` desde o PR #9)
   — única fonte de verdade sobre o tenant.
2. Calcular o eixo, puro e testável, uma função sem componente:

```ts
type CalendarAxis = "location" | "dentist" | "none";

function resolveCalendarAxis(organizationType: TenantType, role: Role | null): CalendarAxis {
  if (organizationType === "FREELANCER") return "location";
  if (role === "DENTIST") return "none";
  return "dentist"; // ADMIN ou RECEPTIONIST em tenant CLINIC
}
```

3. Conforme o eixo, buscar só o roster necessário (`GET /clinics` pra
   `"location"`, o endpoint novo da seção 3 pra `"dentist"`, nada pra
   `"none"`) e devolver pro componente da página um shape já normalizado
   (`{ axis, roster: {id, name, colorToken}[] }`), sem o componente da
   Agenda nem a sidebar precisarem saber a regra de tenant/papel por trás —
   eles só recebem "aqui está o eixo e o roster", igual já fazem hoje com
   `mock.dentists`/`mock.locations`. `agenda-page.tsx` e as duas sidebars
   (`calendar-sidebar.tsx`/`dentist-sidebar.tsx`) deixam de checar
   `role`/`type` diretamente — hoje isso já está parcialmente centralizado
   (`viewByDentist = mock.dentists.length > 0` em `agenda-page.tsx`), essa
   mudança só empurra a checagem de fato pra dentro do hook, que é a
   intenção original documentada em [[Arquitetura]].

## 3. Endpoints envolvidos

| Eixo | Endpoint | Situação |
| --- | --- | --- |
| Freelancer → consultórios | `GET /clinics` | **Já existe e já é usado** pela Agenda hoje (via `clinicsApi.list()`) — só precisa passar a devolver `colorToken` (campo novo, seção 1) |
| Clínica → dentistas | — | **Não existe rota que sirva pra isso** — ver detalhe abaixo |
| Cor: editar de um consultório | `PATCH /clinics/:id` | Existe (`updateClinicSchema`) — precisa aceitar `colorToken?` novo |
| Cor: editar de um dentista | `PATCH /users/:id` | Existe (`updateTenantUserSchema`, `@Roles("ADMIN")`) — precisa aceitar `colorToken?` novo |
| Cadastro de consultório | `POST /clinics` | Existe, freelancer-only (PR #9) — `colorToken?` novo, opcional |
| Cadastro de dentista | `POST /users` | Já existe (`/admin/users`, ADMIN) — fora do escopo mexer aqui além de aceitar `colorToken?` opcional, se fizer sentido no momento da criação |

### Por que nenhuma rota existente serve pro roster de dentistas

Duas candidatas óbvias, as duas erradas pra esse uso — é o ponto mais
importante deste plano, ver seção 5:

- `GET /organization` — devolve `members` (todos os papéis da org), mas é
  **deliberadamente permissivo**: "qualquer papel autenticado", incluindo
  `DENTIST` (é o que alimenta "Minha Clínica" hoje, onde o dentista **deve**
  ver os colegas — comportamento correto e intencional daquela tela). Se a
  Agenda reaproveitasse esse endpoint pro roster de dentistas, um dentista
  logado conseguiria pedir a lista de colegas por essa rota mesmo que a
  sidebar não apareça pra ele na UI — viola a exigência "não é só filtro de
  UI".
- `GET /users` (`UsersController`) — é `@Roles("ADMIN")` na classe inteira,
  **RECEPTIONIST não acessa**. A Agenda precisa que admin **e**
  recepcionista vejam o roster de dentistas — essa rota exclui metade de
  quem precisa.

**Proposta**: rota nova `GET organization/dentists`, no mesmo módulo
`organization` (já é o módulo de "informação sobre a minha org"), com
`@Roles` próprio no método sobrepondo a ausência de `@Roles` da classe —
mesmo padrão já usado em `ClinicsController` (classe sem restrição de
leitura, métodos de escrita com `@Roles("DENTIST")` específico):

```ts
@Controller("organization")
export class OrganizationController {
  @Get()
  getMine(...) { ... } // como já é hoje, sem @Roles = qualquer autenticado

  @Roles("ADMIN", "RECEPTIONIST")
  @Get("dentists")
  getDentists(@CurrentUser() user: AuthenticatedUser) {
    return this.organizationService.findDentists(user.organizationId);
  }
}
```

```ts
// organization.service.ts
findDentists(organizationId: string) {
  return this.prisma.user.findMany({
    where: { organizationId, role: "DENTIST", active: true },
    select: { id: true, name: true, colorToken: true },
    orderBy: { name: "asc" },
  });
}
```

Contrato de resposta:

```json
// GET organization/dentists  (200, só ADMIN/RECEPTIONIST)
[
  { "userId": "clx1...", "name": "Dra. Exemplo", "colorToken": "BRAND" },
  { "userId": "clx2...", "name": "Dr. Segundo Dentista", "colorToken": null }
]
```

```json
// GET organization/dentists  (403, papel DENTIST)
```

## 4. Nomenclatura na resposta da API

Manter o vocabulário do modelo real — **`clinic`/`clinics`**, não
`location`/`locations`. O rename `Clinic → Location` ainda não aconteceu
(fica pra quando a camada `Location`/`Calendar` da Agenda entrar de verdade,
ver [[Implementação da Agenda Multi-Consultório — Plano Técnico]]) —
inventar `location` na API agora criaria um campo que não corresponde a
nenhuma tabela, e teria que ser desfeito/rebatizado quando o rename real
acontecer. `GET /clinics` continua devolvendo `Clinic[]`, sem mudança de
nome de campo, só o `colorToken` novo.

Pro roster de dentistas não tem ambiguidade nenhuma pra resolver — é
`dentist(s)` mesmo, não concorre com o nome de nenhum outro conceito.

No **front**, os nomes de interface locais (`MockLocation`, `MockDentist`)
podem continuar existindo como estão ou perder o prefixo `Mock` — decisão
de implementação, não de contrato de API; o wire format (`Clinic`/o roster
de dentistas) é o que importa manter estável e sem ambiguidade.

## 5. RBAC — confirmação e onde a garantia realmente mora

A exigência é: **dentista de clínica não recebe do backend a lista de
outros dentistas nem seus agendamentos** — nunca só escondido na UI. Com o
desenho da seção 3:

- `GET organization/dentists` tem `@Roles("ADMIN", "RECEPTIONIST")` — um
  dentista autenticado que chame essa rota direto (fora da UI, via curl)
  recebe **403**, não um array vazio nem dado filtrado depois. A garantia
  mora no guard, não em lógica de negócio que poderia ter um caminho
  esquecido.
- O hook do front (`useAgendaCalendarAxis`, seção 2) nem tenta chamar essa
  rota quando `role === "DENTIST"` — mas isso é só UX (evitar uma chamada
  que daria 403 de qualquer forma), a garantia de verdade é o `@Roles` no
  backend, exatamente como já vale pra `/appointments`,
  `/attendances`/`reports/financial` etc. hoje.
- **"Nem seus agendamentos"**: agendamentos continuam mockados nesta rodada
  (ver escopo no topo) — não existe hoje um endpoint real tipo
  `GET /calendars` que vaze isso, porque `Calendar` real ainda não existe.
  Fica registrado aqui como requisito pra quando essa camada virar real: o
  endpoint que servir "agendamentos de todos os dentistas da clínica" pro
  admin/recepcionista precisa do mesmo tratamento — `@Roles("ADMIN",
  "RECEPTIONIST")`, dentista nunca alcança a lista de agendamentos alheios
  por essa rota, só os próprios (via `/appointments` já existente, que já é
  `DENTIST`/`RECEPTIONIST`, sem ADMIN — ver [[Arquitetura]]).

## Resumo do que muda, por camada

- **Schema**: `PaletteColorToken` (enum novo) + `Clinic.colorToken` +
  `User.colorToken`, ambos nullable. Uma migration aditiva.
- **Backend**: `GET organization/dentists` (rota nova, `ADMIN`/
  `RECEPTIONIST`); `colorToken?` opcional em `createClinicSchema`/
  `updateClinicSchema`/`createTenantUserSchema`/`updateTenantUserSchema`;
  serviços de `Clinic`/`User` passam a atribuir cor por índice quando
  omitida na criação (hoje isso é feito no front).
- **Front**: `useAgendaMockData` → `useAgendaData` (ou hook irmão
  `useAgendaCalendarAxis` por trás dele) decide o eixo e busca o roster
  certo; `dentist-store.ts` (mock/localStorage) é aposentado — sidebar de
  dentista passa a receber dado real; `location-colors.ts` perde a
  responsabilidade de *gerar* cor por índice como única fonte (vira só
  fallback quando `colorToken` vier `null` do backend).

## Perguntas em aberto pra validar antes de implementar

1. `croUf` do dentista (hoje só no mock) fica de fora desta integração —
   confirma, ou entra junto como campo novo em `User`?
2. O botão "+ Novo dentista"/"+ Novo consultório" que hoje escreve direto
   no mock/localStorage passa a chamar `POST /users`/`POST /clinics` reais
   na mesma tela da Agenda, ou esses botões somem da Agenda e a orientação
   vira "cadastre em /admin/users ou na tela de Consultórios, a Agenda só
   lê"? Os dois fazem sentido, é decisão de UX, não técnica.
3. Dentista inativo (`User.active: false`) — confirma que some do roster
   de `GET organization/dentists` (proposta acima), inclusive de
   agendamentos passados que ele tinha na visão mockada?
