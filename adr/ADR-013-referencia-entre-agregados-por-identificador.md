---
id: ADR-013
titulo: Referenciar agregados por identificador em vez de relação JPA
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Back-end
substitui: —
substituida-por: —
---

# ADR-013 — Referenciar agregados por identificador em vez de relação JPA

## Contexto e problema

O modelo tem duas formas de ligação convivendo no mesmo código. Dentro do carrinho, `Cart` e
`CartItem` se relacionam por `@OneToMany` e `@ManyToOne`, com cascata e remoção de órfãos. Já a
ligação entre `CartItem` e `Stock` é feita por um `Long stockId` solto, sem relação mapeada,
embora a chave estrangeira exista no banco.

A consequência é visível no `CartService`: para montar a resposta, ele carrega os estoques em
lote a partir dos identificadores e monta o DTO manualmente.

Esta ADR existe para o time confirmar se isso é intencional — fronteira de agregado — ou
inconsistência a corrigir.

## Critérios de decisão

- Limite claro do que é carregado junto e do que não é
- Sem carregamento em cascata inesperado ao ler um carrinho
- Consulta previsível, sem consulta extra por item

## Alternativas consideradas

1. **Relação JPA apenas dentro do agregado; referência por identificador entre agregados** —
   escolhida para registro, sujeita à validação.
2. **Mapear tudo com `@ManyToOne`, inclusive `CartItem` para `Stock`** — *perdeu porque:* traria
   o produto inteiro junto de cada item e abriria caminho para consulta em cascata a cada leitura
   do carrinho.
3. **Referência por identificador em todas as relações, inclusive `Cart` e `CartItem`** —
   *perdeu porque:* item de carrinho não existe sem carrinho, e cascata com remoção de órfãos é
   exatamente o comportamento desejado ali.

## Decisão

`@OneToMany` e `@ManyToOne` só dentro do mesmo agregado, onde o ciclo de vida é compartilhado.
Entre agregados, guarda-se o identificador e o carregamento é feito explicitamente pelo service,
em lote, quando a resposta precisar do dado.

A integridade dessa ligação continua garantida pelo banco, via chave estrangeira.

## Consequências

### O que ganhamos

- Leitura de carrinho com número previsível de consultas, sem cascata surpresa
- Fronteira de responsabilidade explícita: carrinho não gerencia estoque
- Caminho aberto para separar os módulos depois, se o monólito precisar de fronteiras internas

### O que aceitamos em troca

- Composição manual no service: o DTO do item de carrinho é montado juntando duas fontes
- Sem relação mapeada, o compilador não ajuda: `stockId` apontando para lugar nenhum só falha em
  tempo de execução, ou na chave estrangeira
- Duas convenções no mesmo modelo exigem que a regra esteja escrita, senão vira preferência
  individual

## Validação

Nenhuma entidade tem relação JPA mapeada para entidade de outro agregado, e a leitura de um
carrinho com vários itens não gera uma consulta por item.

## Evidências

- `entity/CartItem.java` — `@Column(name = "stock_id") private Long stockId`, sem `@ManyToOne`
- `entity/Cart.java` — `@OneToMany(cascade = ALL, orphanRemoval = true)` para os itens
- `service/CartService.java` — `stockRepository.findAllById(stockIds)` e montagem do DTO
- `V1781471891900__ETI-127-CREATE-CART-ITEM.SQL` — `fk_cart_item_stock` no banco
