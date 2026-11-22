---
id: ADR-037
titulo: DTOs em records dentro de cada funcionalidade, com validação de entrada por Bean Validation
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: ADR-007
substituida-por: —
---

# ADR-037 — DTOs em records dentro de cada funcionalidade, com validação de entrada por Bean Validation

## Contexto e problema

A ADR-007 decidiu que todo endpoint recebe e devolve DTO em `record`, nunca entidade, e que os
DTOs ficam em `com.vitryne.api.dto`. Ela registrou também, como custo, que não havia validação
declarada nos DTOs e que a checagem de entrada estava espalhada nos services.

O código mudou nos dois pontos. Os DTOs ficam dentro de cada funcionalidade (ADR-036), e há
validação declarada neles: o `pom.xml` tem `spring-boot-starter-validation`, e DTOs como
`LoginRequestDTO` trazem `@NotBlank`, `@Email` e `@Size`. A pergunta é onde ficam os DTOs e quem
valida a entrada, mantendo a regra de nunca expor entidade.

Esta ADR substitui a ADR-007 e repete o que dela continua valendo.

## Critérios de decisão

- Cliente não enxerga campo que não é dele
- Mudança de modelo de persistência não quebra o contrato da API
- Objeto de transporte imutável
- Entrada inválida recusada antes de chegar à regra de negócio

## Alternativas consideradas

1. **DTOs como `record` na pasta `dto` de cada funcionalidade, com Bean Validation nos campos de
   request** — escolhida.
2. **DTOs em um pacote único `com.vitryne.api.dto`, como na ADR-007** — substituída. A motivação
   da troca não está registrada — confirmar com o time. A ADR-036 já tirou o pacote global de
   camadas.
3. **Expor as entidades diretamente** — descartada na ADR-007, e esta ADR não a reabre: acopla
   contrato a schema e expõe campo interno.
4. **Validar só dentro dos services** — situação anterior, registrada na ADR-007 como custo. A
   motivação de passar a declarar nos DTOs não está registrada — confirmar com o time.

## Decisão

Todo endpoint recebe e devolve DTO, nunca entidade.

- DTOs são `record`, na pasta `dto` da funcionalidade a que pertencem (`auth/dto`, `cart/dto`,
  `product/dto`, `stock/dto`). Em `auth/dto`, os de cadastro se dividem em `client` e `store`.
  DTOs usados por mais de uma funcionalidade ficam em `shared/dto`, como `ApiErrorResponseDTO`.
- Nomes seguem `<Ação><Recurso>RequestDTO` para entrada e `<Recurso>ResponseDTO` para saída, como
  em `CreateProductRequestDTO`, `ReplenishStockRequestDTO`, `CartResponseDTO` e
  `StockManagementResponseDTO`. `LoginRequestDTO` e `AddressRequestDTO` omitem a ação.
- Campos de request levam anotações do Jakarta Validation (`@NotBlank`, `@Email`, `@Size`). Uma
  entrada inválida é recusada com `MethodArgumentNotValidException`, tratada pelo handler global
  (ADR-039).
- A conversão entre entidade e DTO é responsabilidade do service, como na ADR-007. Esta auditoria
  não reverificou esse ponto em todos os services.

## Consequências

### O que ganhamos

- Contrato estável, independente do schema
- A entrada é validada na borda, com a regra declarada no próprio DTO
- Objeto de transporte imutável, sem efeito colateral entre camadas

### O que aceitamos em troca

- Código de conversão manual em cada service, que cresce com o número de campos
- Campo novo exige alteração em dois lugares, a entidade e o DTO
- A validação do DTO e a validação da entidade (por exemplo, o construtor de `Stock`, que lança
  `IllegalArgumentException`) coexistem e podem divergir
- A convenção de nome tem exceções (`LoginRequestDTO`, `AddressRequestDTO`) e há DTOs de mesmo
  nome em pastas diferentes de `auth` e `shared`

## Validação

Nenhuma assinatura de método em `controller` recebe ou devolve tipo de uma pasta `entity`, e todo
`*RequestDTO` com campo obrigatório declara a restrição no próprio campo.

## Evidências

- `backend/pom.xml` — `spring-boot-starter-validation`
- `backend/src/main/java/com/vitryne/api/auth/dto/LoginRequestDTO.java` — `@NotBlank`, `@Email`,
  `@Size`
- `backend/src/main/java/com/vitryne/api/cart/dto/`, `product/dto/`, `stock/dto/`
- `backend/src/main/java/com/vitryne/api/shared/dto/ApiErrorResponseDTO.java`
- ADR-007 — decisão substituída
