---
id: ADR-023
titulo: Padrão de chamadas de API do app mobile com Axios e conversão de DTO centralizada
status: Proposta
data: 2026-09-19
autor: Leonardo Xavier
decisores: Leonardo Xavier e Bruno Valerio
area: Mobile
substitui: —
substituida-por: —
---

# ADR-023 — Padrão de chamadas de API do app mobile com Axios e conversão de DTO centralizada

## Contexto e problema

O app mobile consome a API do back-end em Java/Spring Boot, que devolve DTOs com campos em
português (`usuarioId`, `quantidade`, `precoUnitario`). O código do app é escrito em inglês,
então toda resposta precisa ser convertida antes de chegar aos hooks e às telas.

Na primeira versão das chamadas do carrinho, essa conversão estava repetida em cada função. Uma
mudança de campo no back exigiria corrigir o mesmo trecho em vários lugares. Além disso, os DTOs
do back ainda não estão fechados, então mudanças de campo são esperadas.

Esta ADR define como as chamadas de API são escritas.

## Critérios de decisão

- Mudança de campo no back corrigida em um lugar só
- Hooks e telas trabalhando só com tipos do app, em inglês
- Timeout e tratamento de erro sem repetir código
- Chamadas fáceis de conferir com os endpoints do back
- Sem dependência além do necessário

## Alternativas consideradas

1. **Axios com instância única, uma função de conversão por DTO e funções `async` nomeadas que só
   fazem a requisição** — escolhida.
2. **Conversão repetida dentro de cada função** — *perdeu porque:* um campo alterado no back exige
   corrigir várias funções iguais, e é fácil esquecer uma.
3. **Devolver o DTO direto para o hook** — *perdeu porque:* os campos em português se espalham
   pelo app, e qualquer mudança no back quebra hooks e telas.
4. **`fetch` nativo** — *perdeu porque:* não tem timeout nem interceptor prontos, e exige converter
   o JSON e checar o status à mão em cada chamada.
5. **TanStack Query** — *perdeu porque:* adiciona dependência e conceitos novos (cache, chaves de
   consulta), e os hooks do domínio já cuidam do estado das telas.

## Decisão

Chamadas feitas com Axios, sempre pela instância `api` de `src/Shared/Api/apiConfig`, que
concentra a URL base, o timeout e o interceptor de erro.

Cada recurso do back tem um arquivo na pasta `Api` do domínio que o usa, com os tipos do DTO e do
app na pasta `Types` do mesmo domínio. O arquivo leva o nome do recurso em inglês, em camelCase e
com extensão `.ts`, conforme a ADR-022 — por exemplo, `cart.ts` para o carrinho.

Dentro do arquivo:

- **Uma função de conversão por DTO**, no topo, com o nome `map<Recurso>` — por exemplo,
  `mapCart`. Ela é o único lugar que conhece os campos em português
- **Valores numéricos com padrão** (`?? 0`) na conversão, para evitar `NaN` em cálculos
- **Funções `async` nomeadas**, que só fazem a requisição, passam o `response.data` para a
  conversão e devolvem o tipo do app
- **Nomes pelo que a função faz**: `fetch` para buscar, `add` para criar, `update` para alterar,
  `remove` para excluir um item e `clear` para excluir todos
- **Cabeçalho de comentário em cada função**, com o método e o endpoint — por exemplo,
  `GET /carrinho/{userId}`
- **Export único no final do arquivo**, com todas as funções

Hooks e telas nunca importam o Axios nem os tipos de DTO: usam só as funções da pasta `Api` e os
tipos do app.

## Consequências

### O que ganhamos

- Campo alterado no back corrigido só na função de conversão
- Hooks e telas isolados do formato do back
- Funções curtas, fáceis de conferir com a documentação da API
- Timeout e tratamento de erro configurados uma vez na instância `api`

### O que aceitamos em troca

- Dois tipos por recurso para manter: o DTO e o tipo do app
- O `?? 0` pode esconder um erro do back, mostrando zero em vez de falhar
- Os campos dos DTOs ainda não são os definitivos e precisam ser conferidos com o back quando ele
  estiver pronto
- A URL base aponta para o ambiente de desenvolvimento e precisa ser trocada pelo domínio do
  Render quando o back for publicado

## Validação

Nenhum hook ou tela importa `axios` ou um tipo de DTO, toda chamada usa a instância `api`,
nenhuma função de API converte campos fora da função `map` do arquivo, e nenhum arquivo da pasta
`Api` usa nome em português ou extensão `.tsx`.

## Evidências

- `mobile/src/Shared/Api/apiConfig` — instância `api` do Axios
- `mobile/src/Customer/Api/cart.ts` — `mapCart`, `fetchCart`, `addItem`, `updateItemQuantity`,
  `removeItem`, `clearCart`
- `mobile/src/Customer/Types/cartMenu` — `Cart`, `CartItem` e `CartResponseDTO`
- `mobile/package.json` — dependência `axios`
