#projeto #hubassistent #infra

Ver [[Visão Geral]] para contexto, [[Arquitetura]] para o código por trás disso.

## Onde cada coisa roda

| Componente | Serviço | URL/detalhe |
|---|---|---|
| Frontend (`apps/web`) | Vercel | build a partir da raiz do monorepo via Turborepo, `VITE_API_URL` embutido no build (precisa redeploy ao trocar) |
| API (`apps/api`) | Render (free) | `https://hubassistent-back.onrender.com` — "dorme" após inatividade, cold-start de alguns segundos |
| Postgres | Supabase | projeto `mmcmpsjyvrtbwwdtxxvp`, região `aws-1-us-west-2` |

Tudo 100% gratuito por escolha deliberada (sem custo mensal fixo).

## Pegadinha: conexão direta do Supabase não funciona

O host de conexão direta do Supabase (`db.<ref>.supabase.co:5432`) não é alcançável nem da máquina local do Vitor nem (provavelmente) do Render — parece ser só IPv6, sem caminho IPv4.

**Solução aplicada**: usar os hosts do *pooler* pros dois usos:
- `DATABASE_URL` (runtime da API) → **Transaction pooler**, porta `6543`, com `?pgbouncer=true` no final.
- `DIRECT_URL` (usado só pelo Prisma pra migrations) → **Session pooler**, mesmo host, porta `5432` em vez do host de conexão direta.

Ambos IPv4-compatíveis. Isso está configurado no `schema.prisma` (`directUrl = env("DIRECT_URL")`) e nas env vars do Render/local.

## Pegadinha: Prisma Client vazio no build do Render

Ver detalhe em [[Arquitetura]] — resumo: faltava `"postinstall": "prisma generate"` no `apps/api/package.json`. Sem isso, o build passava localmente (porque alguém já tinha rodado `prisma generate` manualmente antes) mas quebrava num ambiente limpo com `Namespace 'Prisma' has no exported member 'X'`.

## Variáveis de ambiente necessárias

`apps/api` (Render + local `.env`):
- `DATABASE_URL` — pooler de transação (ver acima)
- `DIRECT_URL` — pooler de sessão (ver acima)
- `JWT_ACCESS_SECRET` / `JWT_REFRESH_SECRET` — strings aleatórias, geradas uma vez (usar o botão "Generate" do próprio Render, não reusar as de dev local)
- `CORS_ORIGIN` — URL da Vercel
- `GEMINI_API_KEY` — chave do Google AI Studio, usada pro parsing de fatura em PDF (ver [[Arquitetura]]). **Pendente**: só está no `.env` local, ainda não foi adicionada nas env vars do Render — vai quebrar o boot em produção até isso ser feito, porque a validação de env (`env.validation.ts`) exige a variável.

`apps/web` (Vercel):
- `VITE_API_URL` — URL do Render

## Cuidado: dev local usa o mesmo banco de produção

Não existe banco separado pra desenvolvimento — o `.env` local do `apps/api` aponta pro **mesmo** projeto Supabase de produção (`mmcmpsjyvrtbwwdtxxvp`). Confirmado em 2026-07-22 ao testar a feature de faturas.

**Implicação prática**: qualquer teste manual em dev local (criar cartão/fatura/transação de teste pra clicar no fluxo) grava direto no banco real. Pra não sujar os dados do Vitor: usar uma conta de teste com nome óbvio (ex. `*.qa@hubassistent.local`) em vez de logar na conta real, e apagar tudo que foi criado ao final (transações → faturas → cartões, nessa ordem por causa das foreign keys). Não existe endpoint pra apagar a própria conta ainda, então uma linha de usuário de teste é o único resíduo que fica.

Também vale saber: o pooler de transação do Supabase tem um limite de conexões modesto. Testes automatizados batendo muitas requisições em sequência (ou reiniciar o servidor de dev várias vezes seguidas) pode esgotar esse pool temporariamente, aparecendo como erro 500 (`Timed out fetching a new connection from the connection pool`) até em rotas sem nada a ver com o que está sendo testado (o próprio guard de autenticação JWT já dá esse erro quando o pool está cheio). Isso se resolve sozinho depois de alguns segundos — não é bug de código.
