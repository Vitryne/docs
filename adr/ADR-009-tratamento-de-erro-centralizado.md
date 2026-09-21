---
id: ADR-009
titulo: Centralizar tratamento de erro em RestControllerAdvice com exceções próprias
status: Aceita
data: 2026-09-13
autor: Equipe
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-009 — Centralizar tratamento de erro em RestControllerAdvice com exceções próprias

## Contexto e problema

Sem tratamento central, cada controller decide sozinho o código de status e o formato da
resposta de erro, e o cliente recebe stack trace do Spring quando algo escapa.

O projeto já criou exceções próprias para as falhas de negócio — produto não encontrado,
estoque indisponível, quantidade inválida, carrinho não encontrado — mas o handler global
trata apenas uma delas. As demais chegam ao cliente como erro 500, e parte das validações ainda
lança `IllegalArgumentException` genérica.

Esta ADR registra a decisão e assume que a implementação está incompleta.

## Critérios de decisão

- Um único formato de erro para toda a API
- Falha de negócio sai com o status HTTP correto, não com 500
- Cliente web e mobile conseguem tratar erro sem interpretar texto livre
- Controller não repete código de tratamento

## Alternativas consideradas

1. **`@RestControllerAdvice` global com uma exceção própria por falha de negócio** — escolhida.
2. **`try/catch` em cada controller** — *perdeu porque:* duplica o tratamento e garante
   divergência de formato entre endpoints.
3. **`ResponseStatusException` do Spring nos services** — *perdeu porque:* mistura decisão de
   transporte HTTP com regra de negócio dentro do domínio.
4. **`ProblemDetail` (RFC 7807), nativo no Spring** — *perdeu por enquanto:* é a alternativa mais
   forte, padroniza o corpo do erro e o time já usa Spring Boot 4. Ficou de fora porque o formato
   atual já estava escrito. Vale reabrir: reverter é barato agora, caro depois que os clientes
   dependerem do formato.

## Decisão

Falhas de negócio lançam exceções próprias do pacote `com.vitryne.api.exception`. Um único
`@RestControllerAdvice` traduz cada exceção em resposta HTTP com corpo padronizado contendo
`timestamp`, `status`, `error` e `message`.

Trabalho pendente para a decisão valer na prática:

- Registrar handler para todas as exceções próprias já existentes
- Substituir as `IllegalArgumentException` restantes por exceção própria, conforme ETI-157
- Mapear os status: 404 para não encontrado, 409 para conflito de estoque, 400 para entrada inválida

## Consequências

### O que ganhamos

- Formato de erro único em toda a API, previsível para os clientes
- Controller limpo, sem tratamento repetido
- Nome da exceção documenta a falha de negócio

### O que aceitamos em troca

- Uma classe de exceção por falha, e o pacote cresce rápido
- Enquanto os handlers não estiverem completos, a API mente: devolve 500 para erro de negócio
- Corpo de erro montado como `Map` não é tipado nem documentado pela especificação da API

## Validação

Toda exceção do pacote `exception` tem handler correspondente, e nenhuma requisição com entrada
inválida conhecida devolve 500.

## Evidências

- `backend/src/main/java/com/vitryne/api/exception/` — oito exceções próprias
- `exception/GlobalExceptionHandler.java` — apenas `ProductNotFoundException` tratada
- `service/CartService.java` — `IllegalArgumentException` em `validateQuantity`
- ETI-157 Revisão de código — Produto: substituir `IllegalArgumentException` genérica
