---
tags: [projeto, linesapp, lint, refactor]
criado: 2026-07-30
---

# LinesApp — Auditoria do ESLint

← [[LinesApp - Visão Geral]] · relacionado: [[Plano de Melhorias]], [[Execução - Rodada 1 (2026-07-30)]], [[Execução - Rodada 2, correção do lint (2026-07-30)]]

Levantamento do que o ESLint (configurado na [[Execução - Rodada 1 (2026-07-30)]]) estava acusando no projeto. Reproduzir com `npm run lint` (ou `npx eslint src App.js` na raiz do projeto).

## ✅ Status: corrigido em 2026-07-30

Todos os 207 achados (43 erros / 164 avisos) listados abaixo foram corrigidos na mesma sessão — ver [[Execução - Rodada 2, correção do lint (2026-07-30)]] para o detalhe do que foi feito em cada categoria. `npm run lint` agora retorna 0 problemas. O conteúdo abaixo é o registro histórico do que foi encontrado, mantido como referência.

## Resumo

| Regra | Qtd | Severidade | Arquivos |
|---|---|---|---|
| `no-unused-vars` | 123 | warning | 31 |
| `no-dupe-keys` | 36 | 🔴 error | 14 |
| `import/no-duplicates` | 25 | warning | 9 |
| `eqeqeq` | 6 | warning | 2 |
| `no-undef` | 5 | 🔴 error | 2 |
| `react-hooks/exhaustive-deps` | 4 | warning | 3 |
| `import/no-named-as-default` | 4 | warning | 4 |
| `react/no-unescaped-entities` | 2 | 🔴 error | 1 |
| `import/no-named-as-default-member` | 1 | warning | 1 |
| `no-unused-expressions` | 1 | warning | 1 |
| **Total** | **207** | 43 erros / 164 avisos | 48 de 91 arquivos |

## 🔴 Bugs reais (prioridade para quando for corrigir)

Diferente do resto (estilo de código), estes três grupos são bugs de verdade — código que quebra ou se comporta errado em runtime, não só "sujeira":

### `no-dupe-keys` (36×) — estilos duplicados, um sobrescreve o outro silenciosamente

Quase todos em `src/pages/Alertas/**/*.styles.js` — evidência de que essas telas foram copiadas por copy-paste sem limpar. Mesmo padrão do bug que já corrigi em `telaAjuda/style.js` na rodada anterior.

```
src/pages/Alertas/Alertas.styles.js:53,122 - fontWeight
src/pages/Alertas/SOS/SOS.styles.js:93,138 - fontWeight, backgroundColor
src/pages/Alertas/acidente/acidente.styles.js:82 - fontWeight
src/pages/Alertas/acidente/climatico/climatico.styles.js:85,87,197 - fontWeight, fontSize, backgroundColor
src/pages/Alertas/acidente/colisao/colisao.styles.js:85,87,197 - fontWeight, fontSize, backgroundColor
src/pages/Alertas/acidente/descarrilamento/descarrilamento.styles.js:85,195 - fontWeight, backgroundColor
src/pages/Alertas/acidente/tecnico/tecnico.styles.js:85,87,197 - fontWeight, fontSize, backgroundColor
src/pages/Alertas/eletrica/eletrica.styles.js:85,87,197 - fontWeight, fontSize, backgroundColor
src/pages/Alertas/lentidao/lentidao.styles.js:85,87,197 - fontWeight, fontSize, backgroundColor
src/pages/Alertas/movimento/movimento.styles.js:94,140 - fontWeight, backgroundColor
src/pages/Alertas/obras/obras.styles.js:85,87,197 - fontWeight, fontSize, backgroundColor
src/pages/Home/Home.styles.js:334,363,374,394 - fontWeight (x2), backgroundColor, borderRadius
src/pages/alexa/alexa.styles.js:41 - borderRadius
src/pages/compartilharLocalizacao/compartilharLocalizacao.styles.js:35,36,59,72 - borderRadius, padding, marginBottom, marginTop
```

### `no-undef` (5×) — variável/função usada sem existir no escopo

Isso quebra (`ReferenceError`) se o caminho de código rodar:

```
src/pages/Home/Home.js:162 - 'Alert' is not defined (falta import { Alert } from 'react-native')
src/pages/configuracoes/configuracoes.js:19 - 'setTema' is not defined
src/pages/configuracoes/configuracoes.js:23 - 'setGeral' is not defined
src/pages/configuracoes/configuracoes.js:31 - 'setPrivacidade' is not defined
src/pages/configuracoes/configuracoes.js:35 - 'setSobre' is not defined
```

### `no-unused-expressions` (1×) — expressão solta que não faz nada

```
src/pages/perfil/perfil.js:483 - Expected an assignment or function call and instead saw an expression.
```

Suspeita de código incompleto ou erro de digitação (ex. um `if` sem chamar a função, ou um `&&` que devia ser um `if`). Vale olhar o contexto quando for mexer.

## Imports duplicados (`import/no-duplicates`, 25×)

Inclui o caso do **`webViewTwitter` importado 4 vezes** em `stackRoutes.js` — já registrado em [[Nomenclatura - Auditoria]] e [[Plano de Melhorias]] como a duplicação das rotas `MetroSp`/`MetroCPTM`/`MetroNoticiando`/`DiarioDoTransporte` (mesmo componente, 4 nomes — decisão de produto pendente, não é só limpar o import). O resto é reimport de imagens (`trem.png`, `chuvaT.png`, `engrenagemT.png`) e de `react`/`@react-navigation/native` em `Home.js` — esses são seguros de consolidar num único `import` a qualquer momento.

```
src/Routes/stackRoutes.js:34-37 - webViewTwitter.js importado 4x (MetroSp, MetroCPTM, MetroNoticiando, DiarioDoTransporte)
src/pages/Alertas/acidente/acidente.js:6,10 - img/trem.png
src/pages/Alertas/acidente/climatico/climatico.js:7,9 - img/chuvaT.png
src/pages/Alertas/acidente/colisao/colisao.js:6,10 - img/trem.png
src/pages/Alertas/acidente/tecnico/tecnico.js:7,9 - img/engrenagemT.png
src/pages/Alertas/movimento/movimento.js:6,10 - img/trem.png
src/pages/Home/Home.js:1,2 - react (node_modules/react)
src/pages/Home/Home.js:37,39 - @react-navigation/native
src/pages/Home/Home.js:43,50,54 - img/trem.png (3x)
src/pages/politicaPrivacidade/politicaPrivacidade.js:2,3 - react-native
src/pages/sobreNos/sobreNos.js:2,3 - react-native
```

## Variáveis/imports não usados (`no-unused-vars`, 123×)

Muito volume pra listar linha a linha aqui — contagem por arquivo, do maior pro menor, pra saber por onde começar:

| Arquivo | Qtd |
|---|---|
| `src/pages/Home/Home.js` | 14 |
| `src/pages/perfil/perfil.js` | 12 |
| `src/pages/Alertas/acidente/climatico/climatico.js` | 7 |
| `src/pages/Alertas/acidente/colisao/colisao.js` | 7 |
| `src/pages/alexa/contaVinculada/contaVinculada.js` | 7 |
| `src/pages/Alertas/acidente/tecnico/tecnico.js` | 6 |
| `src/pages/introducao/introducao.js` | 5 |
| `src/pages/Alertas/eletrica/eletrica.js` | 4 |
| `src/pages/Alertas/lentidao/lentidao.js` | 4 |
| `src/pages/Alertas/movimento/movimento.js` | 4 |
| `src/pages/Alertas/obras/obras.js` | 4 |
| `src/pages/Cadastro/Cadastro.js` | 4 |
| `src/pages/Login/Login.js` | 4 |
| `src/pages/extrato/extrato.js` | 4 |
| `src/pages/telaAjuda/telaAjuda.js` | 4 |
| `src/pages/valida/valida.js` | 4 |
| `src/pages/Alertas/acidente/descarrilamento/descarrilamento.js` | 3 |
| `src/pages/aprovado/aprovado.js` | 3 |
| `src/pages/compartilharLocalizacao/compartilharLocalizacao.js` | 3 |
| `src/pages/passagens/passagens.js` | 3 |
| `src/pages/suaPassagem/suaPassagem.js` | 3 |
| `src/pages/Alertas/Alertas.js` | 2 |
| `src/pages/alexa/alexa.js` | 2 |
| `src/pages/politicaPrivacidade/politicaPrivacidade.js` | 2 |
| `src/pages/webview/twitter/twitter.js` | 2 |
| `src/pages/Alertas/acidente/acidente.js` | 1 |
| `src/pages/construcao/construcao.js` | 1 |
| `src/pages/pagamento/pagamento.js` | 1 |
| `src/pages/sobreNos/sobreNos.js` | 1 |
| `src/services/directionsService.js` | 1 |
| `src/services/placesService.js` | 1 |

`npx eslint --fix` **não** remove `no-unused-vars` sozinho de forma seguraem todos os casos (às vezes a variável não usada é sintoma de um bug, tipo os `no-undef` acima onde o setter esperado nunca foi criado) — precisa passar o olho arquivo por arquivo, não só rodar fix cego.

## Resto (regras menores, estilo de código)

**`eqeqeq`** (usar `===`/`!==` em vez de `==`/`!=`, 6×):
```
src/Routes/stackRoutes.js:77,78
src/pages/perfil/perfil.js:60 (x2), 104, 263
```

**`react-hooks/exhaustive-deps`** (dependência faltando no array do `useEffect`, 4×):
```
src/Routes/stackRoutes.js:88 - falta 'navigation'
src/pages/Home/Home.js:148 - falta 'db'
src/pages/perfil/perfil.js:183,208 - falta 'firestore'
```
⚠️ Merece mais atenção que as outras "menores" — dependência faltando pode causar dado desatualizado (stale closure), não é só estilo.

**`import/no-named-as-default`** (importando `styles` como default de um módulo que também exporta nomeado, 4×):
```
src/pages/Alertas/Alertas.js:3
src/pages/Cadastro/Cadastro.js:17
src/pages/Login/Login.js:13
src/pages/politicaPrivacidade/politicaPrivacidade.js:4
```

**`react/no-unescaped-entities`** (aspas não escapadas em JSX, 2×):
```
src/pages/alexa/contaVinculada/contaVinculada.js:18
```

**`import/no-named-as-default-member`** (1×):
```
src/pages/noticias/noticias.js:28 - cheerio também exporta 'load' nomeado, conferir se não é isso que era pra usar
```

## Próximos passos sugeridos (para quando for atuar)

1. **Bugs reais primeiro**: os 36 `no-dupe-keys`, os 5 `no-undef`, o 1 `no-unused-expressions`. Baixo risco de regressão (são keys duplicadas/variáveis inexistentes — corrigir só torna o código correto), alto valor.
2. **`react-hooks/exhaustive-deps`** (4×) — exige mais cuidado, pode mudar comportamento de quando um efeito reroda; testar cada tela depois.
3. **Imports duplicados** (25×) — consolidar os de imagem/lib são triviais; o caso `webViewTwitter` depende da decisão de produto já registrada em [[Plano de Melhorias]].
4. **`no-unused-vars`** (123×) — maior volume, menor risco individual; ir arquivo por arquivo pela tabela acima, começando pelos de maior contagem (`Home.js`, `perfil.js`).
5. **Resto** (`eqeqeq`, `no-named-as-default`, `no-unescaped-entities`) — cosmético, pode ir junto de qualquer PR que já esteja tocando o arquivo.
