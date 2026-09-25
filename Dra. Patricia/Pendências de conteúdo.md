---
tags: [pendencias, conteudo, dra-patricia-vieira]
projeto: dra-patricia-vieira
atualizado: 2026-08-13
status: aguardando confirmação da clínica
---

# Pendências de conteúdo — aguardando confirmação da clínica

> [!info] Contexto
> Em 2026-08-13, removemos do site **todo texto visível de "a confirmar com a clínica"** — a pedido explícito do cliente, pra página não parecer inacabada. O conteúdo real (o que já tínhamos, escrito por nós com base em fontes confirmadas — Instagram, Google Maps) foi mantido; o que não tinha conteúdo real nenhum foi **removido da tela**, não inventado.
>
> Este arquivo é o mapeamento privado do que ficou pendente — não fica em lugar nenhum do site, é só para o time saber o que ainda precisa buscar com a clínica antes do lançamento (ou numa próxima rodada de revisão de conteúdo).

Ver também: [[Índice]] · [[Design System — Clínica Patrícia Vieira]] · [[Conteúdo do Site]]

---

## 🔴 Removido do site — precisa de conteúdo real antes de voltar

### FAQ (`lib/content.ts` → `faq`)
A página `/faq` tinha 5 perguntas; 4 não tinham resposta real (só o placeholder) e foram **removidas da lista** — hoje a página mostra só 1 pergunta. Isso deixa a página bem enxuta; vale priorizar.

- [ ] **Vocês atendem convênio odontológico?**
- [ ] **Quanto tempo dura o tratamento de lentes de contato dental?**
- [ ] **As lentes de contato desgastam o dente?**
- [ ] **Quais formas de pagamento são aceitas?**

> Sugestão: mandar essas 4 perguntas direto pra Dra. Patricia/recepção, são perguntas típicas de paciente e provavelmente já têm resposta pronta de tanto repetir no dia a dia.

---

## 🟡 Mantido no site, mas com informação parcial

### Horário de funcionamento (seção "Localização")
Hoje o site mostra só: **"Fecha às 18h."** (confirmado via Google Maps).

Falta:
- [ ] Horário de abertura (todos os dias)
- [ ] Funciona aos sábados? Em que horário?
- [ ] Fecha pra almoço?
- [ ] Domingo/feriado: fechado?

### "Nossa história" (seção "A Clínica")
Texto atual, **escrito por nós** com base em fatos já confirmados (bio do Instagram, perfil institucional) — não é texto oficial da clínica:

> *"A Clínica Patrícia Vieira nasceu do compromisso da Dra. Patricia Vieira com uma odontologia estética que respeita a naturalidade de cada sorriso. Hoje reúne uma equipe multiprofissional em Atibaia, SP, oferecendo desde clínica geral até procedimentos estéticos avançados."*

- [ ] Validar esse texto com a clínica, ou substituir pela história real de fundação (ano, motivação pessoal da Dra. Patricia, marco de crescimento da equipe etc.)

---

## 🟢 Baixo risco — só validar antes do lançamento, não bloqueia nada

- [ ] **Legendas da seção Tecnologia** ("Scanner intraoral", "Equipamentos", "Precisão") — descrevem o que aparece na foto, mas não citam marca/modelo do equipamento. Perguntar se vale a pena nomear (ex: "Scanner intraoral iTero").
- [ ] **Especialidade da fundadora** no card da Equipe ("Odontologia Estética · Lentes de Contato Dental") — vem da bio do Instagram, alto grau de confiança, mas nunca foi confirmada literalmente pela clínica.
- [ ] **Legendas dos casos de Resultados** ("Resultado de lentes de contato dental — caso 01"...) — assumem que todo caso é de lentes de contato. Confirmar se algum dos 4 casos é de outro procedimento (implante, faceta) pra ajustar o texto alternativo (`alt`) da imagem.

---

## ⚪ Aguardando arquivo (não é texto, mas está na mesma fila)

- [ ] **`caso-4.png`** — já está referenciado em `lib/content.ts` (`resultCases`), só falta o arquivo em `public/images/resultados/caso-4.png`. Aparece sozinho assim que for enviado (ver padrão `assetExists` no [[Design System — Clínica Patrícia Vieira#9. Stack técnica|design system]]).

---

## Como usar este arquivo

Quando a clínica confirmar um item: marcar o checkbox, atualizar o texto correspondente em `lib/content.ts` (ou direto no JSX de `app/page.tsx`, quando o texto for muito específico de uma seção), e mover a linha pra baixo de um `~~riscado~~` ou simplesmente apagar daqui — este arquivo deve refletir só o que **ainda** está pendente.
