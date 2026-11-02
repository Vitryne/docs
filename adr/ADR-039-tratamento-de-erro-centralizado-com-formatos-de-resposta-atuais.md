---
id: ADR-039
titulo: Centralizar o tratamento de erro em um RestControllerAdvice, registrando os formatos de resposta que existem hoje
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: ADR-009
substituida-por: —
---

# ADR-039 — Centralizar o tratamento de erro em um RestControllerAdvice, registrando os formatos de resposta que existem hoje

## Contexto e problema

A ADR-009 decidiu centralizar o tratamento de erro em um `@RestControllerAdvice`, com exceções
próprias do pacote `com.vitryne.api.exception` e um corpo único com `timestamp`, `status`, `error`
e `message`. Ela assumiu que a implementação estava incompleta, com apenas
`ProductNotFoundException` tratada.

O estado atual é outro. O `GlobalExceptionHandler`, em `shared/exception`, tem 9 métodos
`@ExceptionHandler`. As exceções de domínio ficam em `shared/exception` e nas funcionalidades
(`product/exception`, `store/exception`). Mas o corpo de erro não é único, e o código usa
`ResponseStatusException` nos services, que a ADR-009 havia descartado.

A pergunta é o que está em vigor hoje no tratamento de erro. Esta ADR substitui a ADR-009 e
registra o estado como ele é, sem declarar padronizado o que não está.

## Critérios de decisão

- Falha de negócio sai com o status HTTP correto, não com 500
- Controller não repete código de tratamento
- Cliente web e mobile conseguem tratar o erro sem interpretar texto livre

## Alternativas consideradas

1. **`@RestControllerAdvice` global, com exceções próprias e `ResponseStatusException` onde o
   service já a usa** — escolhida, por ser o que o código faz.
2. **`ProblemDetail` (RFC 7807) como formato único** — já usado em um handler
   (`handleInvalidInput`), mas não em todos. A ADR-009 a apontou como a alternativa mais forte,
   e a decisão de adotá-la como padrão não foi tomada.
3. **`try/catch` em cada controller** — descartada na ADR-009, e esta ADR não a reabre.
4. **Só exceções próprias, sem `ResponseStatusException` nos services** — era a posição da
   ADR-009. O código a contradiz: há 21 ocorrências de `ResponseStatusException` em cinco
   arquivos (`RegistrationService`, `AuthService`, `CartService`, `StockService` e o próprio
   handler, contando imports). A motivação de ter passado a usá-la não está registrada —
   confirmar com o time.

## Decisão

Um único `@RestControllerAdvice`, o `GlobalExceptionHandler` em `shared/exception`, traduz
exceções em resposta HTTP. Ele trata `ProductNotFoundException`, as exceções de promoção e data,
`ResponseStatusException`, `InvalidInputException`, `MethodArgumentNotValidException`,
`IllegalArgumentException`, `ArithmeticException`, `OptimisticLockingFailureException` e
`DataIntegrityViolationException`.

Status em vigor:

- 404 para produto não encontrado
- 400 para entrada inválida, falha de validação e promoção ou data inválida
- 409 para atualização concorrente de estoque e para violação de integridade
- o status da própria exceção, nos casos de `ResponseStatusException`

**O corpo de erro não é único.** Há quatro formatos: `Map` com `timestamp`, `status`, `error` e
`message` (produto não encontrado e promoção inválida); `Map` só com `message`
(`ResponseStatusException`, atualização concorrente e violação de integridade); `Map` com a
mensagem por campo (falha de validação); e `ProblemDetail` (`InvalidInputException`). O record
`ApiErrorResponseDTO`, em `shared/dto`, é usado na documentação das respostas de erro do
`RegistrationController`. Padronizar o corpo fica na fila de decisões do README.

## Consequências

### O que ganhamos

- Um ponto único de tradução de exceção em resposta HTTP
- Os erros de negócio conhecidos saem com o status certo, em vez de 500
- Controller sem tratamento repetido

### O que aceitamos em troca

- O cliente precisa lidar com quatro formatos de corpo de erro
- O corpo montado com `Map` não é tipado e não aparece na especificação da API
- Ainda há 11 ocorrências de `IllegalArgumentException` no código principal (ETI-157), e o
  handler as devolve como erro 400 genérico
- Erro de negócio nasce de dois mecanismos, exceção própria e `ResponseStatusException`, com
  decisão de status em lugares diferentes
- Mensagens de erro são texto livre em português, sem código de erro estável para o cliente

## Validação

Toda exceção própria tem handler, e `grep -rn "ResponseStatusException" src/main/java` mostra
onde o mecanismo alternativo ainda é usado.

## Evidências

- `backend/src/main/java/com/vitryne/api/shared/exception/GlobalExceptionHandler.java` — 9
  métodos `@ExceptionHandler`, e `ProblemDetail` em `handleInvalidInput`
- `backend/src/main/java/com/vitryne/api/product/exception/`,
  `backend/src/main/java/com/vitryne/api/store/exception/`
- `backend/src/main/java/com/vitryne/api/shared/dto/ApiErrorResponseDTO.java`
- ETI-157 — Revisão de código, produto: substituir `IllegalArgumentException` genérica
- ADR-009 — decisão substituída
