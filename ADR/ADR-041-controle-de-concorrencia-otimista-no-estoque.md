---
id: ADR-041
titulo: Controlar a concorrência no estoque com bloqueio otimista por versão
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-041 — Controlar a concorrência no estoque com bloqueio otimista por versão

## Contexto e problema

O estoque é alterado por mais de um fluxo: o lojista cadastra, repõe e ajusta a quantidade por
tamanho, e o carrinho do cliente consulta e reserva itens. Duas alterações simultâneas na mesma
linha de estoque podem sobrescrever uma à outra. O regulamento pede controle transacional, e a
ADR-004 aponta a transação local como razão de ser do monólito.

A entidade `Stock` já tem um campo `@Version`, e o handler global já trata
`OptimisticLockingFailureException`, sem ADR que registre a decisão.

## Critérios de decisão

- Nenhuma alteração concorrente de estoque se perde em silêncio
- Custo baixo em leitura, sem bloquear a linha enquanto o usuário decide
- Resposta clara para o cliente quando houver conflito

## Alternativas consideradas

1. **Bloqueio otimista com `@Version` na entidade `Stock`** — escolhida.
2. As demais alternativas (por exemplo, bloqueio pessimista na leitura ou `UPDATE` condicional)
   não estão registradas — confirmar com o time se alguma foi considerada.

## Decisão

`Stock` tem um campo `@Version` (`Long version`), e o JPA recusa a escrita se a linha mudou desde
a leitura. O `GlobalExceptionHandler` traduz `OptimisticLockingFailureException` em resposta 409
com a mensagem "O estoque foi alterado por outra operação. Atualize os dados e tente novamente.".

Em complemento, a combinação de produto e tamanho é única no banco (`uq_stock_product_size`), e a
violação de integridade vira 409 (ADR-039). Só `Stock` usa `@Version`; `Product` e `CartItem` não.

## Consequências

### O que ganhamos

- Atualização concorrente de estoque detectada, em vez de sobrescrita em silêncio
- Sem bloqueio de linha durante a leitura
- O cliente recebe 409 com uma mensagem que orienta a tentar de novo

### O que aceitamos em troca

- Quem perde a corrida precisa repetir a operação, e a API não tenta de novo sozinha
- O controle vale só para `Stock`; outras entidades alteradas por mais de um fluxo seguem sem
  proteção equivalente
- Nenhuma migration em `db/migration` cria uma coluna `version` na tabela `stock`, e o schema só
  muda por migration (ADR-043, `ddl-auto=none`). Confirmar se a coluna existe no banco, porque a
  entidade depende dela
- Não há teste de concorrência, apenas testes unitários e de contexto

## Validação

Duas alterações simultâneas sobre o mesmo estoque resultam em uma delas com resposta 409, e a
tabela `stock` tem a coluna usada por `@Version`.

## Evidências

- `backend/src/main/java/com/vitryne/api/stock/entity/Stock.java` — campo `@Version` (linha 51)
- `backend/src/main/java/com/vitryne/api/shared/exception/GlobalExceptionHandler.java` — handler
  de `OptimisticLockingFailureException` (409)
- `backend/src/main/resources/db/migration/V1781471629971__ETI-127-CREATE-SUPPLY.sql` —
  `uq_stock_product_size` e `chk_stock_quantity`
- ETI-265 — Cadastro de produtos e manipulação de estoque
