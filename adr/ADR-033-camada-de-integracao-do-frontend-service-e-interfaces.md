---
id: ADR-033
titulo: Separar a integração do front-end com a API em src/service e os contratos de dados em src/interfaces
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: —
substituida-por: —
---

# ADR-033 — Separar a integração do front-end com a API em src/service e os contratos de dados em src/interfaces

## Contexto e problema

A ADR-014 registrou que o acesso à API fica isolado no hook `useHttp` e nos arquivos de
`src/service`, e que nenhuma tela chama Axios direto. Ela não registrou onde ficam os tipos dos
dados que trafegam nessas chamadas. Com a integração de login e cadastro em andamento
(ETI-324), esses tipos passaram a existir no código, em `src/interfaces/`, sem decisão
correspondente.

Hoje o código está assim: a instância Axios é criada em `src/components/hooks/use-http.ts`, e o
interceptor de resposta só faz `console.error`. `LoginService` chama `useHttp()` e retorna um
objeto vazio. `orders-service.ts` não usa Axios: mantém os pedidos em memória a partir de
`src/mocks/orders-mock.ts`. Em `src/interfaces/` existe apenas `login-interface.ts`, com o DTO de
request `LoginRequestDTO`. Não existe DTO de resource/response. A pergunta em aberto é como
organizar a integração para que o ponto de contato com a API e o contrato de dados tenham lugar
definido antes que os demais recursos sejam integrados.

Esta ADR complementa a ADR-014; não a substitui.

## Critérios de decisão

- Um único ponto onde o front conversa com a API
- Contrato de dados tipado e identificável, nos dois sentidos (envio e recebimento)
- Trocar mock por API real sem alterar componentes e páginas
- Estrutura simples o bastante para um time de duas pessoas no front

## Alternativas consideradas

1. **Services em `src/service`, DTOs em `src/interfaces`, componentes e páginas só consomem
   services** — escolhida.
2. **Chamadas Axios direto nos componentes e nas páginas** — *perdeu porque:* espalha URL,
   cabeçalho e tratamento de erro pelas telas e impede trocar mock por API real sem editar cada
   componente. Também contradiz a ADR-014 e a ADR-027, que já vedam chamada direta.
3. **Tipos junto de cada feature, sem pasta central** — *perdeu porque:* o mesmo contrato é usado
   pelo service e pela tela que o consome; guardá-lo dentro de uma feature obriga o service a
   importar de `components/features`, invertendo a dependência. O tipo `Order`, que hoje vive em
   `components/features/panel/orders/types`, é o exemplo desse acoplamento.

## Decisão

A integração do front com a API fica em duas pastas com responsabilidades separadas.

**`src/service/`** guarda os services de integração. Cada service concentra os endpoints de um
recurso e é o único ponto onde o front conversa com a API. O arquivo se chama `<recurso>-service.ts`
(`login-service.ts`, `orders-service.ts`). Os services obtêm a instância Axios pelo hook `useHttp`
(ADR-014). Componentes e páginas não chamam Axios: consomem os services.

**`src/interfaces/`** guarda os DTOs usados pelos services, nos dois sentidos. O arquivo se chama
`<recurso>-interface.ts` (`login-interface.ts`) e os tipos levam o sufixo `DTO`:

- **Request**: dados enviados à API, com sufixo `RequestDTO` (`LoginRequestDTO`).
- **Resource/Response**: dados recebidos da API, com sufixo `ResponseDTO` (por exemplo,
  `LoginResponseDTO`).

Os services tipam suas chamadas com esses DTOs.

O que falta para a decisão estar implementada por inteiro:

- Nenhum DTO de resource/response existe ainda; só `LoginRequestDTO`.
- `LoginRequestDTO` ainda não é usado por nenhum service, pois `LoginService` está vazio.
- `orders-service.ts` ainda usa mocks em memória, não a API real, e exporta funções soltas, ao
  passo que `LoginService` é uma factory exportada como default. A ADR não escolhe um desses
  formatos.
- As telas de produtos e o passo de dados bancários do cadastro ainda importam de `src/mocks`
  diretamente, sem passar por service.

## Consequências

### O que ganhamos

- Ponto único de integração: URL base, cabeçalho e tratamento de erro ficam em `useHttp`, e cada
  recurso tem um arquivo próprio em `src/service`
- Contratos tipados e em lugar previsível, separados dos componentes
- Trocar o mock pela API real é uma mudança dentro do service, sem tocar nas telas

### O que aceitamos em troca

- O interceptor só registra o erro com `console.error`; não há tratamento de token nem de sessão
- DTOs de resource/response ainda não existem, então o retorno das chamadas não tem tipo
- Mocks continuam em uso em pedidos, produtos e dados bancários
- Os services seguem dois formatos de exportação diferentes, até alguém padronizar
- `useHttp` cria uma instância Axios nova a cada chamada do hook
- Sem teste automatizado, a regra de que ninguém chama Axios fora de `src/service` depende de
  revisão de PR

## Validação

`grep -rn "axios" src` retorna apenas `src/components/hooks/use-http.ts`, e toda chamada à API
parte de um arquivo de `src/service` que tipa request e response com tipos de `src/interfaces`.

## Evidências

- `frontend/src/components/hooks/use-http.ts` — criação da instância Axios (linhas 3-9) e
  interceptor que só loga o erro (linhas 11-17)
- `frontend/src/service/login-service.ts` — `LoginService` chamando `useHttp()` e retornando
  objeto vazio
- `frontend/src/service/orders-service.ts` — pedidos em memória a partir de
  `@/mocks/orders-mock`, sem Axios
- `frontend/src/interfaces/login-interface.ts` — `LoginRequestDTO`, único DTO existente
- `frontend/src/mocks/` — `banks-mock.ts`, `orders-mock.ts`, `products-mock.ts`
- ADR-014 — decisão original de isolar o Axios em `useHttp` e `src/service`
- ETI-324 — Integrar APIs de login e cadastro
