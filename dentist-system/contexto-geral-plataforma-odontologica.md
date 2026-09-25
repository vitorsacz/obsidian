---
tags: [projeto, dentist-system, contexto-geral, arquitetura, mvp]
status: em desenvolvimento — MVP focado em clínicas; suporte a tenant Freelancer começou a virar real em 2026-09-22 (schema + regras de negócio pontuais, ver nota abaixo), mas onboarding/self-service self ainda é fase futura
relacionado: "[[Visão Geral]], [[Roadmap]], [[Arquitetura]], [[Implementação da Agenda Multi-Consultório — Plano Técnico]], [[feature - painel-admin-rbac-lgpd]], [[design-system-plataforma-odontologica]]"
---

# Plataforma Odontológica — Contexto Geral do Projeto

Nota de consolidação — reúne as decisões de produto/arquitetura fixas e um
resumo do estado atual, para dar contexto rápido a quem (ou o que) entrar no
projeto depois. Termina com um bloco pronto pra colar no início de uma sessão
nova do Claude Code (seção final).

**Diferença pra [[Visão Geral]]**: aquela nota é o hub técnico (linha do
tempo de decisões, dia a dia, link pra tudo); esta é o resumo de "decisões de
produto já fechadas" + estado atual, pensado pra reler rápido ou colar em uma
sessão nova sem re-explicar tudo do zero. As notas relacionadas acima têm o
detalhamento completo de cada tema.

## Visão do produto

- Dois públicos possíveis: **Clínica** (B2B, CNPJ, múltiplos usuários) e **Dentista freelancer** (B2C, CPF, self-service, usuário único). **O foco de desenvolvimento continua sendo clínica** — self-service/onboarding de freelancer não é prioridade —, mas desde 2026-09-22 (PRs #9–#12) `TenantType.FREELANCER` existe de verdade no schema e já tem regra de negócio própria implementada: só Freelancer cria novos consultórios (clínica é fixa), só Freelancer tem `ClinicFinancialTerms` (relação financeira por consultório), e a sidebar da Agenda distingue os dois eixos com dado real. Criar um tenant Freelancer hoje só é possível via script (`seed-test-personas.ts`), não pelo painel `platform/*` — ver [[Roadmap]] e [[Funcionalidades e Endpoints]].
- Diferenciação frente à concorrência (Clínica Experts, Clinicorp, Codental): transparência total de custo variável, sem implementação obrigatória paga, IA acessível mais cedo que os concorrentes, NF-e nativa em todos os planos pagos. Ver [[Roadmap]], seção "Visão de produto mais ampla", pro comparativo completo — ainda não reconciliado com o MVP em produção.

## Identidade e multi-tenancy

- `TENANT` = Clínica (CNPJ, criado manualmente pelo Super Admin, venda B2B) ou Freelancer (CPF, self-service, B2C).
- `USER` é **isolado por tenant** — não existe identidade global nem tabela de vínculo entre tenants. A mesma pessoa física atuando em 3 lugares tem **3 contas completamente independentes**, sem nenhuma referência cruzada no banco. Isso é uma regra de privacidade (a clínica não pode saber que o dentista atua em outro lugar), não só técnica. **Implementado de verdade** — ver [[Arquitetura]], seção Multi-tenancy.
- Domínio: `clinica.plataforma.com.br` — um único domínio compartilhado por todas as clínicas (não um subdomínio por clínica). `freelancer.plataforma.com.br` — domínio separado, exclusivo do modelo self-service. *(Ainda não implementado — hoje é um único domínio Vercel pra tudo, sem distinção de rota por tipo de tenant.)*
- **Fluxo de login**: e-mail → se mais de uma conta usa aquele e-mail, mostra tela de seleção de tenant **antes** de pedir a senha (nunca depois — senhas de tenants diferentes podem coincidir). Nickname opcional e único globalmente permite pular essa etapa. **Implementado** (`POST auth/lookup` → `POST auth/login`).
- Criação de usuário pela clínica (convite) nunca pode revelar se aquele e-mail já existe em outro tenant — checagem de duplicidade sempre escopada ao próprio tenant. **Implementado.**
- **Soft delete em tudo** — `TENANT` e `USER` sempre têm campo `status`, nunca são apagados de fato. Reativação sempre possível, histórico preservado para auditoria. **Implementado** (`Organization.status`, `User.active`).

## RBAC — hierarquia de usuários

- **Super Admin** (Vitor, owner da plataforma) — visão global de todos os tenants, cria/edita/suspende clínicas manualmente. `isSuperAdmin` é uma flag booleana no `User`. **Implementado**, com dashboard próprio (`/platform`, ver [[Roadmap]]).
- **Tenant Admin** — pode haver múltiplos por clínica. `Organization.foundingAdminUserId` marca o "capitão": só ele gerencia (edita/remove) outros Tenant Admins da mesma clínica. Transferência de capitania **não é self-service** — só via Super Admin, por suporte. **Implementado.**
- **Staff** (dentista/recepcionista) — operacional, permissões limitadas à função. Ver tabela completa de acesso por módulo em [[Arquitetura]] — **ADMIN ganhou leitura em `/clinics`, `/patients`, `/procedures` em 2026-09-22** (pra Agenda multi-dentista funcionar), ainda sem acesso a `/appointments` de verdade (agenda do admin é mockada hoje).

## Cadastro progressivo e LGPD (documentado, 🟡 P1 — não implementado)

- **Fase 1** (libera acesso): nome completo, CRO+UF, telefone, e-mail qualquer (sem exigência de domínio corporativo) — com verificação obrigatória do e-mail.
- **Fase 2** (prazo de 7 dias): CPF, endereço completo, foto, documentos digitalizados, especialidades, dados bancários. Modal de boas-vindas com CTA + opção de "lembrar mais tarde"; **a partir do 7º dia, obrigatório** para manter acesso às funcionalidades completas.
- **Dois momentos de consentimento**: aceite geral na Fase 1, aceite específico na Fase 2 (quando CPF/dados bancários/documentos são de fato coletados) — princípio de consentimento por finalidade da LGPD.
- Retenção de prontuário: Resolução CFO nº 91/2009 — mínimo 10 anos físico, permanente para o digital. O "direito ao esquecimento" não se aplica ao prontuário clínico em si.
- Toda ação sensível (login, edição de CPF/documentos, impersonation do Super Admin, transferência de capitania) gera entrada em `AUDIT_LOG`. *(Não implementado ainda — ver [[Roadmap]].)*

## Agenda

- Visão multi-calendário estilo Google Calendar. **Implementado visualmente** (FullCalendar, mockado) em 2026-09-21/22 — ver [[Roadmap]] e [[Arquitetura]] (seção "Padrão mock primeiro, backend depois").
- Duas granularidades de cor coexistindo: por **consultório/local** (contexto freelancer — cor por índice na paleta oficial de 5 tokens) e por **dentista** (contexto clínica — cada dentista da equipe tem cor própria, editável pelo admin via seletor simples). **Roster + cor virou real em 2026-09-22** (PR #12): `Clinic.colorToken`/`User.colorToken` persistidos de verdade no schema, `GET organization/dentists` real. O que ainda não existe é o backend do calendário em si (`Location`/`Calendar`, agendamentos/eventos) — esses continuam mockados, ver [[Implementação da Agenda Multi-Consultório — Plano Técnico]].
- RBAC aplicado de verdade tanto no roster (backend real desde PR #12 — `DENTIST` recebe `403` em `GET organization/dentists`) quanto, ainda, só na camada de dados do front pro **conteúdo** do calendário: **dentista vê só a própria agenda**; admin/recepcionista veem a agenda de **todos os dentistas da clínica**, com sidebar de checklist para ligar/desligar cada um. Quando o backend de agendamentos existir de verdade, essa regra pro conteúdo do calendário precisa ser reimplementada no servidor.
- Interação: clique em horário vazio abre **modal** de criação (paciente, dentista/consultório, horário real de início/fim, procedimento); clique em evento existente abre **painel lateral** de detalhe com ações rápidas (Confirmar, Remarcar, Cancelar) — diferente do modal de edição que a referência visual usa. **Implementado.**
- Agendamentos sobrepostos (dentistas diferentes, mesmo horário/consultório) renderizam **lado a lado em colunas**, não empilhados (`slotEventOverlap={false}` do FullCalendar — não precisou de plugin de resource view).
- A referência visual (TailAdmin `/calendar`) só demonstra eventos "all-day" (sem horário) — o agendamento com horário real na grade foi construído do zero, não copiado da referência.

## Design System

- Extraído do template **TailAdmin** (`react-demo.tailadmin.com`) via Claude in Chrome, documentado e publicado como Artifact **"Plataforma Odontológica"**. Ver [[design-system-plataforma-odontologica]] (nota única, já incorpora a análise de extração + os tokens aplicados) + `design-system-plataforma-odontologica-tokens.json` ao lado.
- Fonte **Outfit**; cor de ação `brand-500` (#465FFF); estados semânticos (success/error/warning/info) com 3 variantes cada; visual apoiado em borda fina (`border-default`), não em sombra pesada; raios: 16px cards, 8px controles, pill para badges.
- **Aplicado de verdade no produto** desde 2026-09-21 (Tailwind config re-hexado pros tokens oficiais, `Badge` component, `PageHeader`, sidebar fixa) — não é mais só documentação de referência.

## Estado atual da implementação (2026-09-22)

- Stack: React + Vite + Tailwind CSS (front), NestJS + Prisma + Postgres (back), monorepo pnpm/Turborepo.
- **CI verde de ponta a ponta** (lint/typecheck/build/testes e2e) desde a correção completa do pipeline em 2026-09-21. `main` só aceita merge via PR (GitHub Repository Ruleset), nunca push direto.
- Rotas existentes: Início, Agenda, Pacientes, Financeiro, Estoque, Procedimentos, Consultórios, Minha Clínica, Usuários (admin), Plataforma (super admin).
- **Sidebar de navegação lateral fixa** substituiu o menu horizontal (2026-09-21) — colapsável, grupos por seção, rodapé com avatar/nome/papel/sair. Bug de scroll (sidebar rolando junto com a página) corrigido em 2026-09-22 — sidebar trava em 100% da viewport, só o conteúdo rola. Ver [[Problemas Conhecidos]].
- **Agenda redesenhada** (FullCalendar, grade semanal/dia/mês, "linha de agora", criação por arrastar-e-soltar, painel lateral de detalhe) — 100% mockada exceto Locations (consultórios reais) e os pickers de paciente/procedimento no modal de criação.
- **Múltiplos dentistas na Agenda** (2026-09-22) — admin/recepcionista agrupam e coloram por dentista (não só consultório), sidebar de checklist de dentistas, cadastro mínimo (nome/CRO+UF/cor, mockado em `localStorage`), agendamentos sobrepostos lado a lado. Dentista logado continua vendo a agenda por consultório, sem mudança. RBAC real do backend estendido (`GET /clinics`, `/patients`, `/procedures` agora leem com ADMIN) só o suficiente pra essa tela funcionar.
- **Home Dashboard redesenhada** (header, stat cards, gráficos) — mockada.
- **Módulo de Procedimentos** — catálogo redesenhado (filtros, categoria, paginação, modal de criar/editar) — mockado, schema já pensado pra suportar um futuro `PROCEDURE_LOG` (registro do que foi de fato realizado).
- Ver [[Roadmap]] pra lista completa "Feito"/"Em planejamento" e [[Arquitetura]] pro padrão técnico "mock primeiro, backend depois" usado nas telas acima.

## Backlog conhecido, ainda não iniciado

- **Backend real** das três telas redesenhadas acima — migrar de `useMock*`/`localStorage` pra endpoints de verdade (`Location`/`Calendar` da agenda multi-consultório, procedimentos, dashboard).
- **Financeiro (visão geral)** — vai consumir KPIs de procedimentos mais realizados, quantidade por procedimento e valor consolidado por procedimento (depende do `PROCEDURE_LOG` acima).
- Gestão de equipe completa (convite formal com e-mail, edição de papel, reenvio de convite) — hoje o cadastro de dentista é via `/admin/users` (real, mas simples: nome/email/senha/papel).
- Emissão de NF-e, CRM completo, campanhas automáticas — Incrementos 2 e 3 do roadmap geral de produto.
- Compartilhamento de agenda entre freelancer e clínica — futuro, envolve dois planos pagos coexistindo, tratado como Incremento, não MVP.
- Painel Super Admin completo (observabilidade, add-ons, análise regional) — mockado visualmente, implementação adiada até existir uso real para validar as métricas.

---

## Prompt pronto pra colar no início de uma sessão nova

Cole isto no início de uma sessão do Claude Code para dar contexto de projeto
sem precisar reexplicar tudo (mantido atualizado junto com o resto desta
nota — se algo aqui divergir do resto do arquivo, o resto do arquivo é a
fonte de verdade mais recente):

```
Contexto de projeto: estou construindo uma plataforma SaaS multi-tenant de gestão odontológica (React + Vite + Tailwind no front, NestJS + Prisma no back). Antes de qualquer tarefa visual, leia design-system-plataforma-odontologica.md e design-system-plataforma-odontologica-tokens.json (ou onde eu indicar) — todo estilo (cor, tipografia Outfit, espaçamento, raio) vem de lá, nunca invente valor fora desses arquivos.

Decisões de arquitetura já fechadas — trate como contexto fixo, não como pergunta em aberto:

IDENTIDADE E MULTI-TENANCY (implementado)
- TENANT é Clínica (CNPJ, B2B, criado pelo Super Admin) ou Freelancer (CPF, B2C, self-service). O foco de desenvolvimento agora é 100% clínica.
- USER é isolado por tenant — não existe identidade global nem tabela de vínculo entre tenants. A mesma pessoa em 3 lugares tem 3 contas totalmente independentes, sem referência cruzada.
- Login: e-mail → se >1 conta usa aquele e-mail, mostra seleção de tenant ANTES de pedir a senha (nunca depois). Nickname opcional e único globalmente pula essa etapa.
- Soft delete em tudo (TENANT, USER) — nunca exclusão física, sempre campo status + possibilidade de reativação.

RBAC (implementado)
- Super Admin (owner da plataforma): global, cria/edita/suspende tenants, dashboard em /platform.
- Tenant Admin: pode haver múltiplos por clínica; Organization.foundingAdminUserId é o único que gerencia outros Tenant Admins; transferência de capitania só via Super Admin/suporte.
- Staff (dentista/recepcionista): operacional, permissão limitada à função.
- Regra de agenda: dentista vê SÓ a própria agenda; admin e recepcionista veem a agenda de TODOS os dentistas da clínica. Enforcement sempre no backend/serviço de dados, nunca só escondido na UI — o ROSTER (quem aparece na sidebar) já é real desde 2026-09-22 (GET organization/dentists, 403 de verdade pra DENTIST); o CONTEÚDO do calendário (agendamentos em si) continua mockado, esse RBAC ainda simulado no front pra essa parte.

CADASTRO E LGPD (documentado, não implementado — P1, fora do escopo atual)
- Fase 1 (libera acesso): nome, CRO+UF, telefone, e-mail qualquer (verificado). Fase 2 (prazo de 7 dias, depois obrigatório): CPF, endereço, foto, documentos, especialidades, dados bancários.
- Dois momentos de consentimento: termos gerais na Fase 1, aceite específico na Fase 2.

AGENDA (redesenhada, mockada)
- Estilo Google Calendar, FullCalendar. Cor por CONSULTÓRIO (contexto freelancer) e cor por DENTISTA (contexto clínica) coexistem — dentista logado vê por consultório, admin/recepcionista veem por dentista com sidebar de checklist.
- Clique em horário vazio → modal de criação (paciente, dentista, horário real de início/fim, procedimento). Clique em evento existente → painel lateral de detalhe com ações rápidas (Confirmar/Remarcar/Cancelar), não reabre o modal.
- Agendamentos sobrepostos (dentistas diferentes, mesmo horário) renderizam lado a lado em colunas, não empilhados.
- Agendamentos/eventos do calendário em si continuam 100% mockados — backend real (Location/Calendar) ainda não existe. Locations/dentistas/pickers de paciente/procedimento já são reais; desde 2026-09-22 o roster da sidebar (quem aparece, com cor) também virou real nos dois eixos (Clinic para freelancer, User role=DENTIST para clínica).

PADRÕES GERAIS
- Sombra pesada nunca — visual apoiado em borda fina (border-default).
- Toda ação de "remover" é soft delete (desativar), nunca exclusão física.
- Toda lista segue o mesmo padrão de tabela (avatar/nome + subtítulo, badge de status, ações).
- Telas novas: construir a camada visual/UX primeiro com dado mockado (useMock* + useState local), plugar backend de verdade depois — reaproveitar endpoint real onde já existe e funciona (ex.: clínicas, pacientes, procedimentos), mockar só o que é conceito genuinamente novo.
- main só aceita merge via PR (GitHub Ruleset) — nunca commit/push direto.

Estado atual do projeto (2026-09-22): sidebar de navegação lateral fixa (bug de scroll corrigido), Home Dashboard, Agenda (multi-dentista com cor própria) e catálogo de Procedimentos redesenhados e mockados. CI verde ponta a ponta.

Quando eu pedir uma tarefa nova, ela se soma a este contexto — não repita nem redesenhe o que já está descrito acima, a menos que eu peça explicitamente para mudar algo aqui.
```
