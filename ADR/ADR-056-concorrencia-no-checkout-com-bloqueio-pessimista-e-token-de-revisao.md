---
id: ADR-056
titulo: Confirmar o checkout com bloqueio pessimista do usuário e das variações, e com um token que garante os valores da revisão
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-056 — Confirmar o checkout com bloqueio pessimista do usuário e das variações, e com um token que garante os valores da revisão

## Contexto e problema

A ADR-041 decidiu controlar a concorrência no estoque com bloqueio otimista (`@Version` em
`Stock`): uma escrita concorrente é recusada com 409, e quem recebeu o erro tenta de novo. Na seção
de alternativas, a ADR-041 registra que o bloqueio pessimista não foi avaliado.

O checkout (ETI-269) tem exigências que o bloqueio otimista sozinho não atende:

- Vários clientes podem comprar a última unidade da mesma variação ao mesmo tempo, e só um pode
  levar
- O mesmo cliente pode confirmar duas vezes com cliques repetidos, ou editar o carrinho em outra
  aba durante a confirmação
- Entre a revisão (`GET /orders/checkout/review`) e a confirmação, o preço, o estoque ou o endereço
  podem mudar, e o cliente não pode pagar um valor diferente do que viu

Recusar com 409 e pedir nova tentativa no meio de um pagamento seria uma experiência ruim e
arriscaria registrar um pagamento sem pedido.

A pergunta é como garantir que a confirmação use o estoque e os valores corretos, uma única vez.
Esta ADR complementa a ADR-041.

## Critérios de decisão

- Nunca vender mais unidades do que o estoque tem
- Nunca criar dois pedidos para a mesma confirmação
- O cliente paga exatamente o valor que viu na revisão, ou é avisado para revisar de novo
- Sem deadlock entre checkouts que compartilham itens
- Sem infraestrutura nova

## Alternativas consideradas

1. **Bloqueio pessimista do usuário e das variações dentro da transação do checkout, com um token
   de revisão que garante os valores** — escolhida.
2. **Só o bloqueio otimista da ADR-041, com nova tentativa automática** — *perdeu porque:* a
   recusa só acontece na escrita, depois de o pagamento ter sido processado, e não impede dois
   pedidos do mesmo cliente nem garante o valor da revisão.
3. **`UPDATE` condicional no estoque (`quantity = quantity - n WHERE quantity >= n`)** — *perdeu
   porque:* resolve a baixa de estoque, mas não serializa o carrinho do cliente nem garante os
   valores da revisão.
4. **Reserva de estoque ao revisar, com expiração** — *perdeu porque:* exige guardar reservas,
   expirá-las e liberá-las, e prende estoque de quem desiste da compra.
5. **Chave de idempotência enviada pelo cliente** — *perdeu porque:* impede pedido duplicado, mas
   exige que todos os clientes gerem e reenviem a chave, e não resolve a disputa pelo estoque.

## Decisão

O `OrderService.checkout` roda numa única transação, com isolamento `READ COMMITTED`, nesta ordem:

1. **Bloqueia o usuário** com `SELECT ... FOR UPDATE` na linha dele em `users`. É o mesmo bloqueio
   usado pelas operações que alteram o carrinho, então checkout e edição do carrinho do mesmo
   cliente acontecem um de cada vez, e um segundo clique espera o primeiro terminar.
2. **Bloqueia as variações** do carrinho com `PESSIMISTIC_WRITE`, ordenadas por id. A ordem fixa
   evita deadlock entre checkouts com itens em comum.
3. **Recalcula a compra** com os dados já bloqueados: itens, preços atuais, estoque, endereço de
   entrega, frete e total. Qualquer problema que impede a compra devolve 409 com o motivo.
4. **Compara o token de revisão.** A revisão devolve um `reviewToken`, um hash SHA-256 da loja, do
   endereço, de cada item (variação, quantidade e preço unitário), do frete e do total. A
   confirmação recalcula o hash; se for diferente do enviado, devolve 409 pedindo nova revisão.
5. **Processa o pagamento** (ADR-054) e, se aprovado, cria o pedido, baixa o estoque e esvazia o
   carrinho, tudo antes de liberar os bloqueios.

A revisão não bloqueia nada, não baixa estoque e não altera o carrinho. O `@Version` de `Stock`
continua valendo para as demais escritas no estoque, como a reposição feita pelo lojista (ADR-041).

## Consequências

### O que ganhamos

- Duas compras da última unidade nunca são confirmadas juntas: a segunda espera e encontra o
  estoque atualizado
- Cliques repetidos não criam pedidos duplicados
- O valor pago é sempre o valor revisado, ou o cliente revisa de novo
- Tudo resolvido no PostgreSQL, sem nova infraestrutura

### O que aceitamos em troca

- Checkouts que disputam a mesma variação esperam um pelo outro; com muito acesso simultâneo ao
  mesmo item, o tempo de resposta aumenta
- O pagamento é processado com as linhas bloqueadas; com um gateway real, mais lento e externo, os
  bloqueios vão durar mais e o desenho precisará ser revisto
- A concorrência no estoque passa a ter dois mecanismos, pessimista no checkout e otimista nas
  demais escritas, e quem altera o estoque precisa saber qual vale em cada caso
- O token depende de os valores serem calculados da mesma forma na revisão e na confirmação; uma
  mudança nesse cálculo exige mudar a versão do token (`v1`)

## Validação

Duas confirmações simultâneas para a última unidade da mesma variação resultam em um pedido e uma
recusa por estoque; duas confirmações seguidas do mesmo carrinho criam um único pedido; alterar o
preço de um item entre a revisão e a confirmação devolve 409 pedindo nova revisão.

## Evidências

- `backend/src/main/java/com/vitryne/api/order/service/OrderService.java` — transação e ordem dos passos do checkout
- `backend/src/main/java/com/vitryne/api/cart/repository/CartRepository.java` — `lockUserForCartMutation`
- `backend/src/main/java/com/vitryne/api/stock/repository/StockRepository.java` — `lockAllByIdForUpdate`, ordenado por id
- `backend/src/main/java/com/vitryne/api/order/service/CheckoutEvaluator.java` — recálculo e geração do `reviewToken`
- `backend/src/main/java/com/vitryne/api/cart/service/CartService.java` — mesmo bloqueio nas alterações do carrinho
- ETI-269 — checkout com pagamento simulado
- ADR-041 — bloqueio otimista no estoque, que esta ADR complementa
- ADR-054 — pagamento simulado
  