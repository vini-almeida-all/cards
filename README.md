# Handoff — BestSellersSection / ProductCarousel

Este pacote contém o protótipo final validado da seção **BestSellersSection** e o contrato de implementação do **ProductCarousel** reutilizável.

## Fonte de verdade

Use as três fontes em conjunto:

1. **Figma** — fonte de verdade visual, estrutural, tokens, componentes, variantes e assets.
2. **`index.html`** — referência executável do comportamento responsivo e das interações validadas.
3. **`CLAUDE.md`** — contrato de implementação e regras anti-ambiguidade para a reimplementação.

Não reinterpretar componentes existentes. Se houver diferença entre um pixel resolvido e a regra de Auto Layout, preserve primeiro a regra: Fill, Hug, Wrap, gap, padding, alinhamento e min/max-width.

## Arquivos

- `index.html` — protótipo final responsivo, com CSS e JS no próprio arquivo.
- `assets/` — somente as seis capas realmente usadas pelo protótipo.
- `CLAUDE.md` — instruções detalhadas para implementação.
- `README.md` — este resumo de handoff.

O pacote não inclui arquivos de fonte. Em produção, use as fontes/tokens já disponíveis no projeto. Bogart deve vir do ambiente/design system do projeto; o protótipo usa fallback quando ela não está instalada.

## Figma sincronizado

Arquivo: `[NOVO] LEITURA | E-COMMERCE`  
File key: `OtFihfNvBMeg5ITRk3ry2t`

Frames de referência:

- Desktop — `2557:4933` `BestSellersSection - Desktop`
- Tablet — `2796:2233` `BestSellersSection - Tablet`
- Mobile — `2797:5382` `BestSellersSection - Mobile`

Componentes principais:

- `2742:1349` `ProductCarousel`
- `2701:1446` `ProductCard`
- `2697:1359` `ProductTag`
- `2279:821` `Button`
- `2764:2238` / `2764:2239` `CategoryChip`
- `2001:3531` `CarouselButton`

A sincronização final no Figma inclui:

- Desktop: viewport útil de 1304px representada com **5 cards de 235,2px**.
- Tablet: viewport útil de 704px representada com **2 cards de 336px**, sem peek do terceiro.
- Mobile: viewport útil de 326px com **1 card de 326px**.
- `ProductCarousel` documentado como componente comportamental reutilizável para diferentes prateleiras/títulos/temas.
- `ProductCard` documentado com o estado persistente `ADICIONADO`.
- annotations Desktop/Tablet/Mobile atualizadas; as notas antigas de `SeLigaSection` no Mobile foram removidas.

Essas larguras de card são **overrides do contexto** e não novas variantes de ProductCard.

## Reuso em outras prateleiras

O carrossel não deve ser duplicado por tema. O componente compartilhado cuida apenas de viewport, cards, sizing responsivo, drag/swipe, snap e controles.

A seção/pai é responsável por conteúdo e aparência contextual: eyebrow, título, background/tema, CTA, chips/categorias e conjunto de produtos. Uma prateleira com outro nome ou tema deve reutilizar a mesma implementação de `ProductCarousel`.

## Pontos de QA

Validar pelo menos em 1920px, 768px, 556px, 555px e 390px:

- nenhum card abaixo de 230px;
- número inteiro de cards visíveis, sem peek intencional;
- primeiro e último estados alinhados às extremidades reais;
- setas avançam 1 card;
- drag não bloqueia links/botões internos;
- rubber-band funciona somente nas extremidades;
- após o drift, o snap é suave;
- `ADICIONADO` persiste sem timeout;
- Desktop/Tablet/Mobile preservam os layouts e tipografia definidos no Figma/protótipo.
