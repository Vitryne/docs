---
id: ADR-036
titulo: Organizar o backend em pacotes por funcionalidade, mantendo o critério de entidade rica ou anêmica
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: ADR-005
substituida-por: —
---

# ADR-036 — Organizar o backend em pacotes por funcionalidade, mantendo o critério de entidade rica ou anêmica

## Contexto e problema

A ADR-005 decidiu organizar o backend por camada técnica — `controller`, `service`, `repository`,
`entity`, `dto`, `exception` — com pacotes globais. O código atual não segue mais isso. Sob
`com.vitryne.api`, os pacotes são por funcionalidade: `address`, `auth`, `cart`, `client`,
`product`, `stock` e `store`, mais `shared` e `documentation`. Cada funcionalidade tem suas
camadas por dentro, e nem todas têm todas: `controller`, `dto`, `entity`, `enums`, `exception`,
`repository` e `service`. Dentro de `auth/dto` há ainda a divisão `client` e `store`.

A ADR-005 também registrou, como consequência aceita, que "a pasta service tende a crescer sem
divisão por domínio". A organização atual muda o que a ADR-005 afirma sobre a estrutura, mas não
muda o critério que ela fixou para decidir onde mora a regra de negócio. A pergunta é como o
backend se organiza hoje, sem perder esse critério.

Esta ADR substitui a ADR-005 e repete o critério dela, para que ele não se perca com a
substituição.

## Critérios de decisão

- Achar o código de uma funcionalidade em um lugar só
- Regra de negócio em lugar previsível
- Estrutura compatível com um monólito único (ADR-004)

## Alternativas consideradas

1. **Pacotes por funcionalidade, com as camadas dentro de cada pacote** — escolhida.
2. **Camadas técnicas em pacotes globais, como na ADR-005** — substituída. A motivação da troca
   não está registrada — confirmar com o time. A ADR-005 já apontava como custo o crescimento da
   pasta `service` sem divisão por domínio.
3. **Monólito modular com DDD, com bounded contexts, agregados e camada de aplicação** — a ADR-005
   a descartou, e esta ADR não a reabre. A organização por funcionalidade não implementa DDD.

## Decisão

O código do backend fica organizado por funcionalidade em `com.vitryne.api`:

| Pacote | Conteúdo |
|---|---|
| `auth` | login e cadastro de cliente e de lojista, com `dto/client` e `dto/store` |
| `cart` | carrinho do cliente |
| `client` | entidade e repositório do cliente |
| `product` | catálogo e gestão de produto, com `exception` próprio |
| `stock` | estoque por tamanho de produto |
| `store` | loja do lojista, com `exception` próprio |
| `address` | entidade de endereço |
| `shared` | `config`, `dto`, `entity`, `enums`, `exception`, `repository` e `security`, usados por mais de uma funcionalidade |
| `documentation` | exemplos usados na documentação da API |

`User` e `UserRole` ficam em `shared`.

O critério da ADR-005 continua valendo. A regra de negócio mora na entidade quando:

- (a) é consumida por mais de um service, ou
- (b) protege um invariante cuja violação causaria inconsistência de dado (por exemplo, preço
  promocional maior que o preço normal, estoque negativo).

Fora desses dois casos, a entidade permanece anêmica e a lógica fica no service. O resultado
esperado é um mix, com cada entidade classificada individualmente por esse critério.

Service orquestra e controla a transação. Injeção por construtor, com `@RequiredArgsConstructor`
(ADR-008).

## Consequências

### O que ganhamos

- Todo o código de uma funcionalidade fica no mesmo pacote
- A pasta `service` deixa de ser global e de crescer sem divisão por domínio
- O critério de entidade rica ou anêmica segue como regra única

### O que aceitamos em troca

- Sem fronteira de módulo: nada impede um service de alcançar qualquer repository, de qualquer
  funcionalidade, e `shared` pode virar depósito do que não tem dono
- Nem toda funcionalidade tem todas as subpastas (`client` e `address` só têm `entity`, e
  `client` também `repository`), então a estrutura não é uniforme
- O critério exige julgamento em cada regra nova, como na ADR-005
- Mais de uma funcionalidade usa `shared` e `Product`, `Stock` e `CartItem` se cruzam: as
  dependências entre pacotes de funcionalidade não têm regra escrita

## Validação

`find src/main/java/com/vitryne/api -maxdepth 1 -type d` lista os pacotes por funcionalidade, e
não existe `controller`, `service`, `repository` ou `dto` na raiz de `com.vitryne.api`.

## Evidências

- `backend/src/main/java/com/vitryne/api/` — pacotes `address`, `auth`, `cart`, `client`,
  `documentation`, `product`, `shared`, `stock`, `store`
- `backend/src/main/java/com/vitryne/api/shared/` — `config`, `dto`, `entity`, `enums`,
  `exception`, `repository`, `security`
- ADR-005 — decisão substituída e origem do critério de entidade rica
