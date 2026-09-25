---
tags: [plataforma-odontologica, painel-admin, rbac, lgpd, arquitetura]
status: decisões fechadas — pronto para virar especificação técnica
relacionado: "[[Roadmap]], [[Implementação da Agenda Multi-Consultório — Plano Técnico]]"
---

# Painel Administrativo — RBAC, Multi-tenant e LGPD

Consolidação de todas as decisões tomadas na sessão de análise crítica do painel admin — hierarquia de usuários, modelo de tenants, autenticação e conformidade LGPD. Organizado por ordem de prioridade de implementação.

## Contexto

Vitor é o Super Admin (owner) da plataforma. A plataforma atende dois tipos de tenant: **Clínica** (CNPJ, modelo B2B, criado manualmente pelo Super Admin) e **Freelancer** (CPF, modelo B2C, self-service, tenant de usuário único).

---

## 🔴 P0 — Decisões estruturais (bloqueiam qualquer especificação técnica)

Essas são decisões de modelo de dados e de identidade — mudar qualquer uma delas depois de começar a construir é caro. Precisam estar fechadas antes do primeiro schema.

### Identidade é isolada por tenant, não global
- Não existe um `USER` global com múltiplos vínculos (`Membership`) entre tenants.
- Cada tenant tem seus próprios usuários, completamente independentes — mesmo que seja a mesma pessoa física.
- Exemplo: um dentista que atua como freelancer e em 2 clínicas tem **3 contas separadas**, 3 senhas separadas, sem nenhuma referência cruzada no banco.
- Motivo: a clínica não pode saber que o dentista atua em outro lugar (nem como freelancer, nem em outra clínica concorrente) — isolamento é uma regra de privacidade, não só técnica.

### Dois tipos de tenant, dois modelos comerciais
| | Clínica | Freelancer |
| --- | --- | --- |
| Documento | CNPJ | CPF |
| Criação | Manual, pelo Super Admin (venda B2B, com contato comercial) | Self-service (compra direta, sem fricção, B2C) |
| Usuários | Múltiplos (Tenant Admin, dentistas, recepcionistas) | Sempre 1 único usuário |
| "Locations" na agenda | Consultórios reais do tenant | Só rótulos soltos (nome + cor) para organização pessoal — nunca uma referência real a outro tenant, mesmo que a clínica citada também use a plataforma |

### Domínio de acesso
- `clinica.plataforma.com.br` — **um único domínio compartilhado** por todas as clínicas (não um subdomínio por clínica).
- `freelancer.plataforma.com.br` — domínio separado, exclusivo do modelo self-service.
- Consequência direta: e-mail deixa de ser identificador único global — pode se repetir em contas de tenants diferentes.

### Fluxo de login (ordem corrigida — importante)
1. Usuário digita e-mail (ou nickname).
2. Sistema verifica quantas contas usam aquele e-mail.
3. Se só 1 → pede a senha direto.
4. Se mais de 1 → mostra lista de tenants (nome/logo) **antes** de pedir a senha.
5. Usuário escolhe o tenant → só então digita a senha → validada contra a conta específica escolhida.
- **Por que a ordem importa**: senhas de contas diferentes podem coincidir. Perguntar "qual tenant" só depois de validar a senha é logicamente quebrado — não dá pra saber contra qual conta comparar.
- **Nickname**: campo opcional, único globalmente, permite pular a etapa de seleção de tenant. Não é obrigatório (evita fricção no cadastro).
- Freelancer nunca vê essa tela — e-mail já é 1:1 com a conta por natureza do tenant.

### Criação de usuário pela clínica nunca pode vazar dado de outro tenant
- Ao convidar um dentista por e-mail, a checagem de "e-mail já existe" só pode rodar **dentro do próprio tenant**.
- A resposta da tela de convite deve ser sempre idêntica, exista ou não aquele e-mail em outro tenant — nenhuma variação de mensagem pode denunciar indiretamente que a pessoa já está em outro lugar.
- Implementação: são dois serviços de backend separados (busca para login vs. busca para convite), nunca a mesma função reaproveitada.

### Soft delete em tudo — nunca exclusão física
- `TENANT` e `USER` sempre têm um campo `status` (ex.: active/suspended), nunca são apagados de fato.
- Ao remover um dentista da clínica: suspende o acesso, mantém prontuários e histórico intactos (o paciente pertence ao tenant, não ao profissional).
- Reativação é possível a qualquer momento; histórico preservado para auditoria.
- Rótulo correto na UI: "Desativar acesso" — nunca "Excluir usuário".

### Capitania da clínica
- `TENANT.founding_admin_user_id` — o Tenant Admin fundador é o único com permissão de gerenciar (editar/remover) outros Tenant Admins da mesma clínica.
- Pode haver múltiplos Tenant Admins, mas só o fundador administra os demais.
- Transferência de capitania **não é self-service** — só acontece via Super Admin, por canal de suporte, em caso de saída do fundador ou disputa.

---

## 🟡 P1 — Regras de negócio (necessárias antes do lançamento, não bloqueiam o desenho do schema)

### Cadastro progressivo com prazo
- **Fase 1** (libera acesso): nome completo, CRO + UF, telefone/WhatsApp, e-mail (qualquer um, sem exigência de domínio corporativo) — **com verificação obrigatória** do e-mail.
- **Fase 2** (em até 7 dias corridos): CPF, endereço completo, foto, documentos digitalizados, especialidades, dados bancários.
- Modal de boas-vindas após o primeiro login, com CTA direto para completar o cadastro e opção de "lembrar mais tarde".
- **A partir do 7º dia, o preenchimento da Fase 2 é obrigatório** — sem ele, o usuário perde acesso às funcionalidades completas da plataforma.

### Consentimento LGPD em dois momentos
- Aceite geral de termos de uso e política de privacidade na Fase 1.
- Aceite específico e adicional na Fase 2, no momento em que CPF, dados bancários e documentos são de fato coletados (pode ser um checkbox dentro do próprio fluxo, sem tela dedicada).
- Motivo: consentimento específico por finalidade (princípio da LGPD) — evita um "aceito tudo" genérico assinado antes desses dados existirem no sistema.

### Auditoria obrigatória
- Toda ação sensível gera entrada em log: login, edição de CPF/documentos/dados bancários, impersonation do Super Admin, transferência de capitania.
- Impersonation do Super Admin (entrar como um Tenant Admin para suporte) deve notificar o Tenant Admin de que houve acesso.

### Retenção de prontuário
- Resolução CFO nº 91/2009: guarda mínima de 10 anos para prontuário físico, guarda permanente para o digital.
- Consequência prática: o "direito ao esquecimento" do paciente não se aplica ao prontuário clínico em si — só a dados de outras finalidades (marketing, prospecção). **Validar esse ponto com jurídico antes de travar a regra de exclusão de conta no produto.**

---

## ⚪ Descartado / fora do escopo por enquanto

- **"Quem está online" em tempo real** — decidido que não faz sentido para o contexto atual. Dashboard do Tenant Admin fica só com métricas de consulta direta (volume de atendimentos, alertas de cadastro incompleto), sem infraestrutura de presença.
- **Freelancer compartilhar agenda/usuário com uma clínica** — ideia validada como incremento futuro (aparece naturalmente como parte do Incremento 2 — Escala do roadmap geral), não algo a resolver agora. Envolve decisão comercial de planos coexistentes (freelancer pago + clínica paga) que só faz sentido depois de escalar.
- **Segundo Super Admin (sócio)** — arquitetura já suporta via flag `is_super_admin` no `USER`, sem nenhuma mudança estrutural necessária. Não é uma ação para hoje, só registro de que não é um bloqueio futuro.

---

## Estrutura de dados (resumo pós-decisões)

| Tabela | Observação após as decisões desta sessão |
| --- | --- |
| `TENANT` | +`founding_admin_user_id`; status active/suspended/deleted (soft delete) |
| `USER` | Sempre escopado a 1 tenant; sem tabela `MEMBERSHIP` many-to-many; +`nickname` (opcional, único global); +`status` |
| `PROFESSIONAL_PROFILE` | +`profile_deadline_at` (7 dias a partir da criação); campos nulos até completar a Fase 2 |
| `CONSENT_RECORD` | Duas linhas possíveis por usuário: consentimento Fase 1 e consentimento Fase 2, cada um com sua própria versão/timestamp |
| `AUDIT_LOG` | Cobre login, impersonation, transferência de capitania e edição de dado sensível |
| `LOCATION` (do freelancer) | Sem `tenant_id` de referência a outra organização — sempre um rótulo solto (nome + cor) |

---

## Próximo passo

Transformar os itens 🔴 P0 em especificação técnica (schema definitivo + fluxos de tela), antes de tocar nos itens 🟡 P1.
