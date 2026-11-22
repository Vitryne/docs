---
id: ADR-054
titulo: Simular o pagamento no checkout, com um único ponto de troca para a integração com gateway
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-054 — Simular o pagamento no checkout, com um único ponto de troca para a integração com gateway

## Contexto e problema

O regulamento exige integração com um meio de pagamento em ambiente de homologação, e os artefatos
do projeto preveem um gateway no checkout: o Mapa de Fluxos cita Mercado Pago ou Stripe, e o
diagrama de sequência do checkout mostra o pedido aguardando o resultado do gateway. O gateway
ainda não foi escolhido (ver "Integração de pagamento" na fila).

O checkout do back-end (ETI-269) precisava de uma etapa de pagamento para converter o carrinho em
pedido, e o mobile já tem as telas de pagamento por cartão e por PIX.

A pergunta é como entregar o checkout completo, do carrinho ao pedido, sem um gateway integrado e
sem dificultar a integração quando ele for escolhido.

## Critérios de decisão

- Checkout funcionando de ponta a ponta já no MVP
- A troca pelo gateway real concentrada em um único ponto do código
- Cada tentativa de pagamento registrada, aprovada ou recusada
- Os caminhos de aprovação e de recusa testáveis de forma previsível
- Nenhuma conta, credencial ou dado de cartão envolvido

## Alternativas consideradas

1. **Simulador interno, em que o cliente informa o resultado desejado e o pagamento é registrado
   como se viesse de um gateway** — escolhida.
2. **Integrar um gateway em modo sandbox agora** — *perdeu porque:* o gateway não foi escolhido, e a
   integração exige conta, credenciais e, no PIX, o recebimento assíncrono da confirmação; não cabe
   no prazo do MVP.
3. **Confirmar o pedido sem etapa de pagamento** — *perdeu porque:* foge do fluxo dos diagramas,
   obrigaria a reestruturar o checkout na integração e não permite testar a recusa.
4. **Simulador com resultado aleatório** — *perdeu porque:* torna testes e demonstrações
   imprevisíveis.

## Decisão

O módulo `payment` tem a entidade `Payment` (usuário, valor em `NUMERIC(12,2)`, método, resultado e
data de criação, nenhum deles alterável depois de gravado), com os métodos `CREDIT_CARD`,
`DEBIT_CARD` e `PIX` e os resultados `APPROVED` e `DECLINED`.

O `POST /orders/checkout` recebe o `paymentMethod` e um campo opcional `simulatedResult`. O
`OrderService` chama o `PaymentSimulator` com o total do pedido, e o simulador grava o pagamento
com o resultado informado, ou `APPROVED` quando o campo não vem.

- **Pagamento aprovado:** o pedido é criado com status `CONFIRMED` e ligado ao pagamento, o estoque
  é baixado e o carrinho é esvaziado.
- **Pagamento recusado:** o pagamento fica registrado, nenhum pedido é criado, e o carrinho e o
  estoque continuam como estavam. A resposta traz o pagamento recusado, sem pedido.

O checkout não recebe nem guarda dados de cartão. Para integrar um gateway, a troca acontece no
`PaymentSimulator`, que é o único ponto do checkout que conhece o pagamento.

## Consequências

### O que ganhamos

- Checkout completo, do carrinho ao pedido, demonstrável nos dois resultados
- Histórico de tentativas de pagamento desde já
- Nenhum dado de cartão trafega ou é armazenado
- A integração futura troca uma classe, sem mudar o fluxo do pedido

### O que aceitamos em troca

- O requisito do regulamento de integrar um meio de pagamento em homologação continua pendente
  (fila)
- O resultado do pagamento é escolhido pelo cliente; o campo `simulatedResult` faz parte do contrato
  da API e precisa sair quando o gateway entrar
- O pagamento é síncrono; com um gateway real, o PIX é confirmado depois, o que vai exigir um status
  de pedido aguardando pagamento e o recebimento da confirmação pelo back-end

## Validação

Um checkout com `simulatedResult: DECLINED` grava um pagamento recusado e não cria pedido, mantendo
carrinho e estoque; sem o campo, o pagamento é aprovado e o pedido é criado com status `CONFIRMED`.

## Evidências

- `backend/src/main/java/com/vitryne/api/payment/service/PaymentSimulator.java` — simulador e ponto de troca
- `backend/src/main/java/com/vitryne/api/payment/entity/Payment.java` e `payment/enums/`
- `backend/src/main/java/com/vitryne/api/order/dto/CheckoutRequestDTO.java` — `paymentMethod` e `simulatedResult`
- `backend/src/main/java/com/vitryne/api/order/service/OrderService.java` — tratamento da aprovação e da recusa
- Migration `ETI-269-create_order_and_payment.sql`
- ETI-269 — checkout com pagamento simulado
