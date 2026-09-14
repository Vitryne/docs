---
id: ADR-014
titulo: Migrar valores monetários de Double para BigDecimal
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Back-end
substitui: —
substituida-por: —
---

# ADR-014 — Migrar valores monetários de Double para BigDecimal

## Contexto e problema

O banco armazena dinheiro corretamente, em `NUMERIC(10,2)`. O código Java lê esse valor como
`Double`, que é ponto flutuante binário e não representa valores decimais com exatidão.

O carrinho já faz aritmética sobre esses valores: multiplica preço unitário por quantidade,
soma subtotais e calcula o total. Cada operação acumula um erro pequeno, e a diferença aparece
justamente onde é mais visível — o valor final que o consumidor vê antes de pagar.

O projeto vai integrar API de pagamento, exigência do regulamento. Divergência de centavo entre
o total calculado e o valor cobrado é o tipo de defeito que só aparece em produção.

## Critérios de decisão

- Valor exibido igual ao valor cobrado, sempre
- Coerência entre o tipo do banco e o tipo da aplicação
- Arredondamento explícito, decidido pelo código e não pelo hardware
- Custo de correção proporcional ao tamanho atual do código

## Alternativas consideradas

1. **`BigDecimal` com escala e modo de arredondamento definidos** — escolhida.
2. **Manter `Double`** — *perdeu porque:* é errado por construção para dinheiro, e o erro cresce
   com o número de operações — exatamente o que um carrinho faz.
3. **Inteiro em centavos** — *perdeu porque:* exige conversão em toda fronteira de entrada e
   saída, e o banco já está em `NUMERIC`, que mapeia direto para `BigDecimal`.
4. **Tipo de dinheiro de biblioteca (moeda e valor juntos)** — *perdeu porque:* a plataforma opera
   em uma moeda só; complexidade sem retorno.

## Decisão

Valores monetários passam a ser `BigDecimal` em entidades, DTOs e cálculos: `price`,
`promotionalPrice`, `unitPrice`, `estimatedTotalValue`, subtotal e frete.

Escala de duas casas, arredondamento definido explicitamente em toda operação que divide ou
aplica percentual — o caso do desconto percentual em `Product.applyDiscount`.

Comparação por `compareTo`, nunca por `equals`.

Enquanto a migração não acontecer, esta ADR fica como Proposta e o `Double` permanece registrado
como dívida conhecida, não como decisão.

## Consequências

### O que ganhamos

- Total calculado igual ao total cobrado, sem diferença de centavo
- Tipo da aplicação coerente com o `NUMERIC(10,2)` do banco
- Arredondamento decidido no código, visível na revisão

### O que aceitamos em troca

- Aritmética mais verbosa: `add` e `multiply` no lugar de `+` e `*`
- Migração toca entidades, DTOs, services e os testes de carrinho e produto já escritos
- Quanto mais tarde, mais caro: depois que web e mobile consumirem o contrato e o pagamento
  estiver integrado, a mudança atravessa os três repositórios

## Validação

Nenhum campo monetário declarado como `Double` ou `double`, e o total do carrinho fecha com a
soma dos subtotais em cenário com preço quebrado e desconto percentual.

## Evidências

- `entity/Product.java` — `Double price`, `Double promotionalPrice`
- `entity/CartItem.java` — `Double unitPrice`
- `service/CartService.java` — `calculateSubtotal` e `calculateTotal` com `mapToDouble`
- `V1781471530531__ETI-127-CREATE-PRODUCT.SQL` — `price NUMERIC(10,2)`
- Regulamento Escola de TI 2026, requisito arquitetural (f): integração com API de pagamento
