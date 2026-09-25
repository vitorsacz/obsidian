---
tags: [projeto, lines, arquitetura, feature-futura]
data: 2026-07-23
---

#projeto #lines

Pesquisa de viabilidade e arquitetura para uma futura feature de monitoramento em tempo real de veículos (ônibus/trens) no mapa do LinesApp. Ver [[Visão Geral]] para contexto do projeto. **Isto é só pesquisa + desenho de arquitetura — nada foi implementado.**

## Ponto de partida

O Google Maps (usado hoje via `react-native-maps`) fornece o mapa e ferramentas de roteamento, mas **não** fornece posições de veículos em tempo real — isso precisa vir de uma fonte de dados de trânsito, ser processado em um backend, e só então ser plotado como marcador no mapa.

## 🚌 Ônibus — SPTrans API Olho Vivo (viável, dados reais existem)

- Fornece posição GPS real de toda a frota de ônibus municipais de SP: `GET /Posicao` (frota inteira), `GET /Posicao/Linha?codigoLinha=` (por linha), além de previsão de chegada (`/Previsao*`). Resposta em JSON com lat/long por veículo.
- **Autenticação é stateful (sessão/cookie)**, não um bearer token simples: `POST /Login/Autenticar?token={token}` estabelece uma sessão que precisa ser mantida entre chamadas. O token é gratuito, obtido cadastrando um "aplicativo" no site da SPTrans (área Desenvolvedores).
- Nenhum limite de taxa documentado oficialmente, mas a API foi desenhada para consumo server-side (não é prática para um app mobile chamar diretamente).
- Fonte: [Documentação API Olho Vivo — SPTrans](https://www.sptrans.com.br/desenvolvedores/api-do-olho-vivo-guia-de-referencia/documentacao-api/)

## 🚆 Metrô/CPTM — achado importante: a premissa do GTFS-Realtime não se sustenta para SP

Pesquisei especificamente se existe um feed GTFS-Realtime público com posição de trens/composições para Metrô ou CPTM em São Paulo — **não existe**. Os dados GTFS abertos da região (SMTU/EMTU) **excluem explicitamente sistemas ferroviários/metroviários**, cobrindo só ônibus.

O que existe de fato para trilhos são APIs de **status de linha / ocorrências**, não posição de veículo:

- **API Trilhos (ARTESP)** — oficial, cobre Metrô, CPTM, ViaMobilidade e ViaQuatro. Requer API key mediante cadastro (`Authorization: Api-Key cci_metro_status_live_<key>`), limite de **12 requisições/hora**, recomendação de no máximo 1 chamada a cada 5 min. Retorna status operacional por linha + histórico de ocorrências — **não** latitude/longitude de composições. Fonte: [API Trilhos — ARTESP](https://ccm.artesp.sp.gov.br/metroferroviario/api/docs/)
- **`diretodostrens.com.br`** — não-oficial, gratuita, mesmo tipo de dado (status por linha). **Já é a fonte usada hoje** em `src/pages/Operacao/index.js`.

### Conclusão prática

A tela "Operação" que já existe no app está, essencialmente, **no teto do que é publicamente possível para trilhos hoje**. Dá para trocar a fonte não-oficial pela API Trilhos oficial (mais confiável, mas com cadastro e rate limit), mas **não é possível montar um mapa com trens se movendo em tempo real** como se faria com ônibus — essa granularidade de dado não é aberta ao público por Metrô/CPTM/ViaMobilidade atualmente.

## Arquitetura recomendada (para quando for implementar)

Só faz sentido implementar "veículos se movendo no mapa" para **ônibus** (via Olho Vivo). Para trilhos, a evolução possível é só trocar/reforçar a fonte de status da tela Operação já existente.

1. **É necessário um backend próprio** — hoje o LinesApp não tem nenhum (só Firebase Auth/Firestore/Storage). Motivos:
   - A sessão/cookie do Olho Vivo não é prática de manter direto num app mobile.
   - O token pessoal não deve ir embarcado no app publicado (risco de abuso, violação de ToS, esgotamento de cota compartilhada entre todos os usuários do app).
   - Milhares de instalações do app batendo direto na API da prefeitura a cada ~15s não é uma prática saudável nem confiável.
2. **Onde**: como o projeto já usa Firebase, o caminho de menor atrito é uma **Firebase Cloud Function agendada** (Cloud Scheduler) que autentica no Olho Vivo, busca `/Posicao` — filtrado por linhas/bounding box relevante, não a frota inteira de ~15 mil veículos — a cada 15-20s, e mantém um snapshot em cache.
3. **Entrega ao app**: **não** replicar o padrão de "pings" (1 documento Firestore por marcador, como em `Home/index.js`) para isso — o volume de atualização (milhares de veículos a cada ~15s) tornaria o custo de escrita do Firestore proibitivo. Melhor caminho: expor um endpoint HTTPS simples (a própria Cloud Function) que devolve o snapshot em cache, e o app faz **polling a cada 15-20s via `axios`** — mesmo padrão já usado em `Operacao/index.js`. Renderizar os veículos como `Marker`s no `MapView` já existente em `Home/index.js`, reaproveitando o padrão visual já usado para estações/pings.
4. **Para trilhos**: avaliar trocar `diretodostrens.com.br` pela **API Trilhos oficial** em `Operacao/index.js` — ganha confiabilidade/oficialidade, mas exige cadastro de API key e respeitar 12 req/hora (compatível com o uso atual, que já é só status de linha). Não resolve/permite veículos em movimento no mapa.

## Escopo — o que fica de fora por enquanto

Ônibus **metropolitanos** (EMTU, fora do município de SP) têm GTFS + API própria separada da Olho Vivo — fora do escopo desta pesquisa, que focou no pedido original (ônibus municipais + trens/metrô). Vale revisitar se o app expandir cobertura para a região metropolitana.

## Notas relacionadas
- [[Visão Geral]]
- [[Funcionalidades]] (tela Operação atual)
- [[Próximos Passos]]
