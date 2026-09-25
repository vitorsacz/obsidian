#projeto #doorcor #conteúdo

Ver [[Arquitetura]] para como isso se conecta aos componentes. Tudo abaixo
vive em `src/lib/content.ts` — nenhum componente tem texto hardcoded, exceto
microcopy estrutural (labels de UI como "Scroll", títulos de seção fixos).

## Identidade da marca (`brand`)

| Campo | Valor |
|---|---|
| Nome | DoorCor |
| Sufixo | ACM |
| Telefone (exibição) | +55 11 93215-2858 |
| Telefone (WhatsApp, formato E.164 sem +) | 5511932152858 |
| Instagram | [@doorcor.acm](https://www.instagram.com/doorcor.acm/) |
| Threads | @doorcor.acm |

**Origem dos dados**: extraídos do perfil público do Instagram em
2026-08-13, via análise por browser automation. O comentário no topo de
`content.ts` avisa explicitamente: *"Substitua pelos dados oficiais da
empresa (endereço, CNPJ, textos definitivos) antes de publicar."* — ou seja,
nada aqui foi confirmado diretamente com a DoorCor real, é inferido do
Instagram. Ver [[Próximos Passos]].

## WhatsApp — o único canal de contato do site

```ts
export const whatsappMessages = {
  quote: "Olá! Gostaria de solicitar um orçamento para portas em ACM da DoorCor.",
  seeWork: "Olá! Quero ver os produtos em ACM da DoorCor.",
};

export function buildWhatsAppUrl(message: string) {
  return `https://wa.me/${brand.phoneWhatsapp}?text=${encodeURIComponent(message)}`;
}
```

Duas intenções, duas mensagens diferentes:
- **`quote`** ("orçamento") — usada em Header (desktop + mobile), Hero
  ("Solicitar Orçamento") e Footer.
- **`seeWork`** ("ver produtos") — usada só no Contact, que é a seção final
  de conversão.

Toda chamada de `buildWhatsAppUrl` abre em nova aba (`target="_blank"
rel="noreferrer"`). Não existe nenhum outro canal de contato no site — decisão
explícita do Vitor em 2026-09-02, sem formulário de contato.

## Navegação (`navLinks`)

Sobre → Produtos → Projetos → Diferenciais → Contato. Nota: a ordem visual
das seções no `App.tsx` difere ligeiramente (Diferenciais vem antes de
Projetos desde 2026-09-02), mas a ordem do menu de navegação **não** foi
atualizada para refletir isso — ainda lista "Projetos" antes de
"Diferenciais". Não é um bug funcional (os links `#anchor` continuam
corretos), mas é uma inconsistência de ordem entre menu e página real. Ver
[[Próximos Passos]].

## Produtos & Serviços (`services`, seção 02)

01. Portas Externas — pivotantes/entrada, sob medida
02. Portas Internas — ACM + madeira laminada, foco em silêncio de operação
03. Tecnologia Embarcada — fechaduras eletrônicas, biometria, automação
04. Fachadas em ACM — revestimento comercial, acabamento espelhado/fosco/texturizado

## Diferenciais (`differentials`, seção 03)

01. Alto Padrão — cada projeto como peça única
02. Produção 100% Brasileira — fabricação nacional ponta a ponta
03. Tecnologia Integrada — automação pensada no projeto, não adaptada depois
04. Atendimento Direto — contato direto com quem executa, do orçamento à instalação

## Fotos da seção Sobre (`aboutPhotos`)

3 fotos com legenda, grid de 3 colunas:
1. `doorcor-01.jpg` — "Porta pivotante — vista interna"
2. `doorcor-09.jpg` — "Painéis em ACM sob medida"
3. `doorcor-17.jpg` — "Fachada em pedra e ACM"

## Galeria de Projetos (`galleryItems`, seção 04)

8 itens, grid responsivo (1/2/3 colunas), cada um com tag de categoria:

| # | Título | Tag | Arquivo |
|---|---|---|---|
| 01 | Porta de Correr Espelhada | Interno | doorcor-02.jpg |
| 02 | Porta Alta em Closet | Interno | doorcor-07.jpg |
| 03 | Fechadura Eletrônica | Tecnologia | doorcor-08.jpg |
| 04 | Porta Embutida em ACM | Interno | doorcor-13.jpg |
| 05 | Fachada em Pedra e ACM | Residencial | doorcor-14.jpg |
| 06 | Porta Pivotante em Madeira | Residencial | doorcor-16.jpg |
| 07 | Fachada Externa em ACM | Residencial | doorcor-18.jpg |
| 08 | Porta com Puxador Cromado | Residencial | doorcor-20.jpg |

Paginação exibida como "01 / 08" com setas "← Prev" / "Next →" —
**decorativa/estática hoje**, não filtra nem pagina de fato (só 8 itens,
todos exibidos de uma vez no grid). Ver [[Próximos Passos]] se o Vitor quiser
paginação funcional.

## Foto do Hero

`doorcor-30.jpg`, alt: "Porta pivotante DoorCor em ACM, aberta para vista
externa" — única foto fora do array `content.ts` (referenciada direto em
`Hero.tsx` por ser uso único).
