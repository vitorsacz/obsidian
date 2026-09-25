#projeto #hubassistent

## O que é

App pessoal do Vitor para:
1. Controlar finanças — gastos, entradas, faturas, cartões, Pix, extrato.
2. (Futuro) Organizar rotina — calendário, agendamentos, reuniões, exercícios.

Hoje é um projeto solo, mas a arquitetura foi pensada desde o início para virar produto multiusuário com pouco retrabalho — isso não é suposição, foi um requisito explícito do Vitor na conversa de planejamento inicial. Toda decisão de arquitetura ou design deve priorizar "barato de estender depois" em vez de "gambiarra mais rápida pra um usuário só".

Repositório: [GitHub — vitorsacz/hubassistent](https://github.com/vitorsacz/hubassistent)

## Notas relacionadas
- [[Arquitetura]] — stack, monorepo, padrões de código
- [[Design System]] — identidade visual Navy Duotone, tokens, tipografia
- [[Infraestrutura e Deploy]] — URLs de produção, pegadinhas já resolvidas
- [[Roadmap]] — status de cada fase

## Linha do tempo de decisões
- **2026-07-20** — scaffold do monorepo, autenticação, núcleo financeiro (contas/cartões/categorias/transações) implementados e publicados em produção (Vercel + Render + Supabase), testados de ponta a ponta.
- **2026-07-20/21** — identidade visual genérica ("cara de IA") foi apontada como problema antes de continuar com features. Foram exploradas 3 direções visuais (Ledger, Sol, Signal), depois 3 variações de azul dentro da Ledger. Escolhida: **Navy Duotone**. Aplicada em todo o app.
