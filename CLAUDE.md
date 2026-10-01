# CLAUDE.md — BestSellersSection / ProductCarousel

## Objetivo

Implementar a seção `BestSellersSection` com fidelidade ao Figma e ao `index.html` deste pacote, preservando o `ProductCarousel` como comportamento compartilhado para outras prateleiras de produtos com títulos, temas, CTAs, categorias e conteúdos diferentes.

Este arquivo é um contrato. Não simplifique nem “melhore” os componentes por conta própria.

---

## 1. Hierarquia de fontes

Use as fontes nesta ordem de responsabilidade:

- **Figma:** visual, estrutura, componentes, variantes, tokens, assets, Fill/Hug/Wrap, gaps, paddings, alinhamento e truncamento.
- **`index.html`:** comportamento executável validado — sizing do carrossel, drag, momentum, rubber-band, snap, estados e breakpoints web.
- **Este `CLAUDE.md`:** resolve ambiguidades de implementação e registra decisões aprovadas.

Nunca transforme uma dimensão resolvida do Figma em largura fixa quando o nó estiver em Fill/Hug. Exemplo: `ContentContainer` resolve em 1368px no Desktop, mas o contrato é `width:100%` + `max-width:1368px`.

Não altere componentes compartilhados existentes além do necessário para reproduzir o contrato aprovado. Reuse tokens, componentes, variantes e assets existentes antes de criar qualquer coisa.

---

## 2. Estrutura da seção

Composição esperada:

```text
BestSellersSection
└─ ContentContainer
   ├─ SectionHeader
   │  ├─ HeaderRow
   │  │  ├─ TitleGroup
   │  │  │  ├─ Eyebrow
   │  │  │  └─ SectionTitle
   │  │  └─ RankingCTA
   │  └─ CategoryChips
   └─ ProductCarousel
      ├─ Items / viewport
      │  └─ ProductCard[]
      └─ CarouselControls
         ├─ CarouselButton / previous
         └─ CarouselButton / next
```

### Layout base

- Section: `width:100%`, altura Hug/fit-content.
- `ContentContainer`: Fill horizontal, `max-width:1368px`, padding horizontal 32px, gap vertical 36px, Hug vertical, overflow/clipping conforme Figma.
- `SectionHeader`: Fill/Hug, coluna, gap 36px.
- Chips: Fill/Hug, `flex-wrap:wrap`, gap 12px.
- `ProductCarousel`: Fill horizontal, coluna, gap 32px entre viewport e controles.
- Viewport de cards: altura 520px, clipping horizontal.

Não adicionar Header/Nav/Footer ou qualquer elemento externo à seção.

---

## 3. Breakpoints validados

Os media queries abaixo fazem parte do protótipo final.

### Desktop — acima de 768px

- padding vertical da seção: 64px;
- `HeaderRow`: horizontal, `space-between`, alinhado ao centro;
- `TitleGroup`: Fill, padding-right 16px;
- eyebrow: 21px / line-height 24px;
- título principal: 32px / 115%;
- referência 1920px → conteúdo útil do carrossel 1304px.

### Tablet — 557px a 768px

- padding vertical: 48px;
- `HeaderRow`: vertical, alinhado à esquerda, gap 24px;
- `TitleGroup`: Fill;
- eyebrow: 21px / 24px;
- título: 28px / 125%;
- referência 768px → conteúdo útil 704px.

### Mobile — até 556px

- preserva padding vertical 48px e header vertical;
- `TitleGroup` sem padding-right;
- eyebrow: 18px / 20px;
- título: 24px / 125%;
- referência 390px → conteúdo útil 326px.

**Observação sobre spacing tipográfico:** o Figma pode reportar `itemSpacing` diferente do `gap` CSS do protótipo por causa das métricas de line box/fontes. Não copie cegamente o número resolvido entre ferramentas. Preserve a aparência validada; para a implementação web desta entrega, use os valores do protótipo final quando ele explicita o espaçamento CSS.

---

## 4. ProductCarousel é genérico e reutilizável

Não criar `BestSellersCarousel`, `FavoritesCarousel`, `NewReleasesCarousel` etc. A lógica deve existir uma única vez em `ProductCarousel`.

O pai/shelf fornece:

- eyebrow;
- título;
- tema/background/tokens contextuais;
- CTA;
- chips/categorias e estado selecionado;
- lista/ordem de produtos;
- conteúdo das instâncias de ProductCard.

`ProductCarousel` fornece apenas:

- sizing responsivo dos cards;
- viewport/track;
- overflow;
- drag/swipe/momentum;
- snap;
- tratamento das extremidades;
- controles anterior/próximo.

Não acoplar textos, categorias, cores de tema da shelf ou conteúdo de produto ao código do carrossel.

---

## 5. Algoritmo obrigatório de cardsPerView

Constantes aprovadas para este ProductCarousel:

```text
minCardWidth = 230px
gap = 32px
```

A quantidade de cards visíveis é calculada pela largura útil real da viewport, não por breakpoints fixos de quantidade:

```js
cardsPerView = Math.max(
  1,
  Math.min(
    itemCount,
    Math.floor((availableWidth + gap) / (minCardWidth + gap))
  )
)
```

Depois, todos os cards irmãos recebem a mesma largura e preenchem exatamente 100% da viewport útil:

```js
cardWidth = Math.max(
  minCardWidth,
  (availableWidth - gap * (cardsPerView - 1)) / cardsPerView
)
```

Regras obrigatórias:

- nunca comprimir ProductCard abaixo de 230px;
- se N cards não couberem, reduzir `cardsPerView`;
- cards podem crescer acima de 230px;
- todos os cards do mesmo carousel/contexto têm a mesma largura;
- sempre mostrar um número inteiro de cards;
- não criar peek intencional;
- não adicionar padding/espaço artificial no final do track;
- recalcular ao redimensionar a viewport.

Referências aprovadas:

- 1304px → 5 × 235,2px, gap 32px;
- 704px → 2 × 336px, gap 32px;
- 492px → 2 × 230px, gap 32px;
- 326px → 1 × 326px.

Com gutters de página de 32px, 556px de viewport é o ponto em que cabem 2 cards mínimos: `32 + 230 + 32 + 230 + 32 = 556`.

Não transformar 235,2 / 336 / 326 em variantes de ProductCard; são resultados do contexto.

---

## 6. Navegação e snap

### Controles

- Sem loop.
- Usar `CarouselButton` existente.
- Mostrar controles somente quando houver overflow.
- Previous/Next avançam **exatamente 1 card / 1 snap position por clique**, nunca uma página inteira.
- Desabilitar Previous no início físico e Next no fim físico.

### Posições de snap

Calcular snap positions usando offsets reais dos cards, limitados por:

```text
0 <= scrollLeft <= maxScroll
maxScroll = track.scrollWidth - viewport.clientWidth
```

O início `0` e o `maxScroll` físico são autoritativos. O último estado deve deixar o último card encostado à extremidade direita da viewport, sem vazio artificial.

Use `scroll-snap-type: x proximity` como apoio, mas o comportamento aprovado inclui settling customizado após drag/momentum.

---

## 7. Drag, momentum, settling e rubber-band

O `index.html` é a referência executável. O comportamento aprovado é:

### Mouse

- drag livre enquanto o pointer estiver pressionado;
- não iniciar drag se `pointerdown` começou em `a`, `button`, `input`, `select`, `textarea` ou elemento com `role=button`;
- threshold aproximado de 5px para considerar movimento;
- suavização de velocidade no drag: `velocity = velocity * 0.72 + instantVelocity * 0.28`;
- após soltar, continuar um pequeno momentum antes do snap;
- decay de momentum: aproximadamente `0.92^(dt / 16.67)`;
- ao ficar lento ou atingir limite, aguardar ~50ms e assentar no snap válido mais próximo.

### Touch / pen

- usar scroll/inércia nativos;
- após o movimento terminar, assentar no snap válido mais próximo quando a plataforma expuser `scrollend` ou mecanismo equivalente.

### Snap suave

Não usar apenas `scrollTo({behavior:'smooth'})` se isso produzir o snap agressivo que foi rejeitado durante a validação.

O protótipo aprovado usa:

```js
smootherStep(t) = t*t*t * (t * (t*6 - 15) + 10)
```

Duração adaptativa:

```js
duration = clamp(360ms, 360ms + abs(distance) * 0.55, 560ms)
```

A animação começa e termina com velocidade zero para que a passagem do drift para o snap não dê um “tranco”.

### Rubber-band nas bordas

Quando o usuário tenta ultrapassar o início/fim, não mover o `scrollLeft` além do limite. Deslocar visualmente o track com resistência:

```js
rubberBandDistance(distance) = tanh(max(0, distance) / 190) * 128
```

- alcance visual máximo aproximado: 128px;
- ao soltar, retornar a `transform:0`;
- referência de spring-back: 220ms, `cubic-bezier(.22,.61,.36,1)`;
- depois do retorno, permanecer no início/fim físico válido.

### Reduced motion

Respeitar `prefers-reduced-motion: reduce`: não executar settling/momentum animado desnecessário; posicionar diretamente quando adequado.

---

## 8. ProductCard — contrato

Reutilizar o `ProductCard` aprovado no Figma. Não recriar localmente.

Propriedades importantes:

- altura no shelf atual: 520px;
- padding: 20px;
- border/radius conforme componente;
- ProductMedia: Fill horizontal, 210px de altura, imagem `contain/FIT`, sem crop;
- ranking/editorial tags usam ProductTag;
- título máximo 2 linhas;
- autor máximo 1 linha;
- linhas opcionais desaparecem e o conteúdo colapsa naturalmente;
- não reservar espaços vazios para tag/título/desconto ausente;
- o espaço flexível restante fica antes do CTA para manter o CTA no fundo.

A largura é propriedade do contexto ProductCarousel; não alterar o componente para “forçar” Desktop/Tablet/Mobile.

### Links

- imagem e título navegam para PDP na mesma aba;
- no protótipo o `preventDefault` existe apenas porque não há rotas reais;
- não reproduzir esse bloqueio em produção.

---

## 9. Estado de adicionar ao carrinho

`availability=available` começa com o Button primário:

- ícone ShoppingCart;
- label `ADICIONAR`;
- adiciona exatamente 1 unidade sem navegar para outra página.

Depois de sucesso, o estado deve ser persistente e refletir o estado do carrinho — **não usar timeout**:

- label: `ADICIONADO`;
- fundo branco;
- outline/border 1px brand blue;
- texto brand blue;
- ícone brand blue;
- mesma geometria/padding/radius;
- permanece assim enquanto o item estiver no estado “adicionado”.

O protótipo torna cliques subsequentes neste estado inertes. Preserve isso em v1, salvo contrato de carrinho posterior explicitamente diferente.

**Não usar a variante Secondary atual do Button para simular esse estado.** A Secondary existente possui fill azul-claro e não corresponde ao estado aprovado. No Figma, o comportamento/visual pós-add está documentado na descrição de ProductCard; o Button global não foi expandido com uma nova variante sem uma decisão mais ampla do design system.

Outros estados:

- `availability=notify` → `AVISE-ME`, secondary, sem ícone;
- `availability=unavailable` → `INDISPONÍVEL`, primary disabled, sem ícone.

Não criar modal/quickview para v1.

---

## 10. CategoryChip e CTA

Reutilizar `CategoryChip` existente.

- seleção é single-select;
- somente um chip tem `aria-pressed=true`;
- seleção deve atualizar/filtrar a lista quando a fonte de dados estiver integrada;
- não alterar labels/ordem/estado inicial deste frame sem nova especificação.

Categorias desta referência:

`Geral`, `Ficção`, `Não-ficção`, `Infantojuvenil`, `HQ & Mangás`, `Filosofia & Ensaios`.

`Ver ranking completo` usa o Button compartilhado e deve navegar para a experiência de ranking completa definida pelo produto.

---

## 11. Assets e tipografia

Use os assets reais do projeto/Figma. As capas incluídas neste pacote são a referência do protótipo:

- `assets/verity.jpg`
- `assets/cromos.jpg`
- `assets/eu-duvido-voce-terminar.jpg`
- `assets/album-copa.jpg`
- `assets/hipotese-do-amor.jpg`
- `assets/flores-para-algernon.jpg`

Não versionar/font-embed arquivos de fonte deste pacote. Use o sistema tipográfico já existente no projeto.

- Nunito: usar estilos/tokens existentes.
- Bogart: usar Bogart Medium/token correspondente do projeto para eyebrow. Não substituir Bogart por Nunito na implementação final.

---

## 12. Figma — referências finais

Arquivo: `OtFihfNvBMeg5ITRk3ry2t`

Frames:

- `2557:4933` — BestSellersSection - Desktop
- `2796:2233` — BestSellersSection - Tablet
- `2797:5382` — BestSellersSection - Mobile

Componentes:

- `2742:1349` — ProductCarousel
- `2701:1446` — ProductCard
- `2670:979` — ProductCard / availability=available
- `2701:1360` — ProductCard / availability=notify
- `2701:1403` — ProductCard / availability=unavailable
- `2697:1359` — ProductTag
- `2279:821` — Button
- `2764:2238` / `2764:2239` — CategoryChip
- `2001:3531` — CarouselButton

Os frames e descrições foram sincronizados no handoff final. Não usar screenshots antigos onde Tablet mostrava um terceiro card parcial.

---

## 13. Reuso futuro de shelves

Ao receber outra seção com “o mesmo tipo de carrossel”:

1. reutilizar `ProductCarousel` sem fork;
2. reutilizar `ProductCard` e suas variantes;
3. trocar apenas conteúdo/ordem dos produtos via dados/props;
4. deixar eyebrow, título, CTA, chips e tema no componente pai da shelf;
5. aplicar mudanças estruturais só quando o novo Figma explicitamente exigir comportamento diferente;
6. não criar uma variante de ProductCarousel apenas por mudança de cor/título/tema.

Se uma shelf não tiver chips, CTA ou eyebrow, isso deve ser definido pelo Figma daquela shelf no nível do pai; não remover esses elementos do ProductCarousel porque eles nunca pertencem a ele.

---

## 14. Checklist de aceite

### Estrutura

- [ ] Fill/Hug/Wrap preservados.
- [ ] ContentContainer com max-width 1368 e gutters 32.
- [ ] Sem alturas de seção artificiais / `100vh`.
- [ ] Componentes existentes reutilizados.

### Carousel

- [ ] minCardWidth 230.
- [ ] gap 32.
- [ ] integer cardsPerView.
- [ ] cards iguais e Fill do espaço útil.
- [ ] sem peek.
- [ ] sem trailing space artificial.
- [ ] sem loop.
- [ ] setas = 1 card.
- [ ] controles somente com overflow.
- [ ] first/last flush às bordas físicas.
- [ ] drag não captura links/botões.
- [ ] momentum + snap suave.
- [ ] rubber-band ~128px nas bordas.
- [ ] reduced-motion respeitado.

### Breakpoints

- [ ] 1920px: 5 cards de ~235,2px na viewport útil 1304.
- [ ] 768px: 2 cards de 336px na viewport útil 704.
- [ ] 556px: 2 cards de 230px na viewport útil 492.
- [ ] 555px: cai para 1 card Fill.
- [ ] 390px: 1 card de 326px.

### ProductCard

- [ ] mídia contain/FIT sem crop.
- [ ] título/autor truncam conforme contrato.
- [ ] CTA no fundo sem espaços reservados falsos.
- [ ] imagem/título clicáveis para PDP.
- [ ] `ADICIONADO` persistente, sem timeout.
- [ ] estado pós-add branco + outline/texto/ícone brand blue, não Secondary.
