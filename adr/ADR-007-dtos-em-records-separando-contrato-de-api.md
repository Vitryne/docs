---
id: ADR-007
titulo: Separar contrato da API das entidades JPA usando DTOs em records
status: Aceita
data: 2026-09-13
autor: Equipe
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-007 — Separar contrato da API das entidades JPA usando DTOs em records

## Contexto e problema

Entidade JPA exposta direto no endpoint carrega o modelo de persistência para dentro do
contrato público: relação bidirecional que serializa em laço infinito, carregamento preguiçoso
que estoura fora da transação, e campo interno visível para o cliente.

Além disso, web e mobile precisam de dados que não estão na entidade — preço final já calculado,
subtotal do item, nome do produto dentro do item de carrinho.

## Critérios de decisão

- Cliente não enxerga campo que não é dele
- Mudança de modelo de persistência não quebra o contrato da API
- Resposta traz valor derivado pronto, sem cálculo no cliente
- Objeto de transporte imutável

## Alternativas consideradas

1. **DTOs de request e response como `record`, montados na camada de service** — escolhida.
2. **Expor as entidades diretamente** — *perdeu porque:* acopla contrato a schema, expõe campo
   interno e quebra na serialização de relação bidirecional.
3. **DTOs como classes com Lombok** — *perdeu porque:* `record` já entrega imutabilidade,
   `equals`, `hashCode` e `toString` sem anotação.
4. **Biblioteca de mapeamento automático como MapStruct** — *perdeu porque:* o mapeamento hoje é
   pequeno e explícito; gerar mapper esconderia onde o valor derivado é calculado.

## Decisão

Todo endpoint recebe e devolve DTO, nunca entidade. DTOs ficam em `com.vitryne.api.dto`,
declarados como `record`, com `@Builder` do Lombok onde a montagem tem muitos campos.

Nomenclatura: `<Ação><Recurso>RequestDTO` para entrada e `<Recurso>ResponseDTO` para saída.

A conversão é responsabilidade do service, que é quem tem transação aberta e acesso aos
repositórios necessários para compor o valor derivado.

## Consequências

### O que ganhamos

- Contrato estável, independente do schema
- Resposta com valor pronto: `finalPrice`, `subtotal`, nome e foto do produto no item
- Objeto de transporte imutável, sem efeito colateral entre camadas

### O que aceitamos em troca

- Código de conversão manual em cada service, que cresce com o número de campos
- Campo novo exige alteração em dois lugares — entidade e DTO
- Sem validação declarada nos DTOs hoje: a checagem de entrada está espalhada nos services

## Validação

Nenhuma assinatura de método em `controller` recebe ou devolve tipo do pacote `entity`.

## Evidências

- `backend/src/main/java/com/vitryne/api/dto/` — `ProductResponseDTO`, `CartResponseDTO`,
  `AddItemRequestDTO`, `UpdateItemRequestDTO`, `CartItemResponseDTO`, `AvailableSizeDTO`
- `service/CartService.java` — `toResponseDTO` e `toItemResponseDTO`
- `controller/ProductController.java` — `ResponseEntity<List<ProductResponseDTO>>`
