---
id: ADR-055
titulo: Representar valores monetários em reais com duas casas decimais no banco e na API, sem arredondamento implícito
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: João Ehlers e Henrique Pacheco
area: Back-end
substitui: —
substituida-por: —
---

# ADR-055 — Representar valores monetários em reais com duas casas decimais no banco e na API, sem arredondamento implícito

## Contexto e problema

Preço de produto, preço promocional, valor previsto do carrinho, preço unitário, subtotal, frete,
total do pedido e valor do pagamento são valores em dinheiro. Até a ETI-267, os campos de valor do
carrinho tinham precisão diferente da dos campos de produto, e um mesmo valor podia ser gravado de
formas diferentes em tabelas diferentes.

Um erro de representação em dinheiro aparece como centavo a mais ou a menos no total do pedido, e
o valor passa pelo banco, pela API, pelo portal do lojista e pelo app do cliente. A pergunta é como
representar esses valores em cada camada.

## Critérios de decisão

- Nenhuma perda de precisão em soma e multiplicação de valores
- Mesmo formato em todas as tabelas e em todos os campos da API
- Nenhum arredondamento silencioso: valor inválido é recusado, não ajustado
- Leitura direta pelo front-end e pelo mobile, sem conversão no contrato da API

## Alternativas consideradas

1. **`NUMERIC(12,2)` no banco, `BigDecimal` com escala 2 no back-end e número decimal em reais na
   API, recusando valores com mais de duas casas** — escolhida.
2. **Inteiro em centavos no banco e na API** — *perdeu porque:* elimina a precisão decimal, mas
   obriga todos os clientes a converter para exibir, e o banco e a API já estavam em reais.
3. **`double` no back-end** — *perdeu porque:* ponto flutuante binário não representa valores como
   0,10 de forma exata, e a soma dos itens pode divergir do total em centavos.
4. **Arredondar automaticamente para duas casas (`HALF_UP`)** — *perdeu porque:* esconde um erro
   de quem enviou o valor e pode gravar um preço diferente do digitado.

## Decisão

- **Banco:** todo valor monetário é `NUMERIC(12,2)`, até dez dígitos inteiros e duas casas
  decimais. A ETI-267 alinhou os campos do carrinho a esse formato.
- **Back-end:** todo valor monetário é `BigDecimal`. Na entrada, o valor é normalizado com
  `setScale(2, RoundingMode.UNNECESSARY)`: um valor com mais de duas casas, ou com mais de dez
  dígitos inteiros, é recusado com erro de validação, nunca arredondado. Cálculos de preço final,
  subtotal e total acontecem no back-end, e a API entrega esses valores prontos (ADR-007).
- **API:** os valores trafegam como número decimal em reais (por exemplo, `149.90`).
- **Clientes:** o front-end converte para centavos inteiros no mapper ao receber
  (`Math.round(valor * 100)`) e de volta para reais ao enviar, e faz contas de exibição em
  inteiros. O mobile exibe o valor em reais como recebido, formatado com duas casas.

## Consequências

### O que ganhamos

- Totais calculados sem erro de ponto flutuante no back-end
- Um único formato de valor em todas as tabelas e em todos os campos da API
- Valor fora do formato é recusado com uma mensagem clara, em vez de gravado diferente

### O que aceitamos em troca

- O número decimal em JSON é lido como ponto flutuante pelo JavaScript; os clientes não devem
  somar valores em reais diretamente, e o front-end converte para centavos por isso
- Front-end e mobile tratam os valores de forma diferente (centavos e reais); uma regra de cálculo
  no cliente precisa considerar o formato de cada um
- O limite de dez dígitos inteiros vale para qualquer valor, inclusive totais de pedido

## Validação

As colunas de valor do banco são `NUMERIC(12,2)`; enviar um preço com três casas decimais devolve
erro de validação; o total do pedido é igual à soma dos subtotais mais o frete, sem diferença de
centavos.

## Evidências

- `backend/src/main/java/com/vitryne/api/product/entity/Product.java` — normalização com `RoundingMode.UNNECESSARY`
- `backend/src/main/java/com/vitryne/api/order/entity/Order.java` e `OrderItem.java` — valores do pedido
- `backend/src/main/java/com/vitryne/api/payment/entity/Payment.java` — valor do pagamento
- Migration `ETI-267-ALIGN_CART_MONEY_PRECISION.SQL` — alinhamento dos campos do carrinho
- `frontend/src/lib/product-mapper.ts` — conversão entre reais e centavos
- `mobile/src/Customer/Hooks/useProduct.ts` — exibição em reais
- ETI-267 — revisão do carrinho
- ADR-007 — valores prontos na resposta da API
