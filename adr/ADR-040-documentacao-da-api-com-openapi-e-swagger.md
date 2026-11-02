---
id: ADR-040
titulo: Documentar a API com OpenAPI e Swagger UI, gerados pelo springdoc a partir do código
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-040 — Documentar a API com OpenAPI e Swagger UI, gerados pelo springdoc a partir do código

## Contexto e problema

O front-end e o mobile consomem a API, e o mobile tem DTOs ainda não fechados (ADR-021). Até a
ETI-261, a API não tinha documentação de contrato publicada. Essa ETI entrou com o fluxo de
cadastro e o Swagger, sem ADR correspondente: o `pom.xml` tem
`springdoc-openapi-starter-webmvc-ui`, e há uma classe `OpenApiConfig` e um pacote
`documentation`.

A pergunta é como o contrato da API é documentado e quem o mantém atualizado.

## Critérios de decisão

- Contrato conferível por quem consome, sem ler o código do back-end
- Documentação que acompanha o código, em vez de um documento separado
- Teste manual dos endpoints autenticados

## Alternativas consideradas

1. **springdoc-openapi, com anotações no código e Swagger UI** — escolhida.
2. As demais alternativas não estão registradas — confirmar com o time se outra foi considerada.

## Decisão

O contrato é gerado pelo springdoc a partir do código.

- Swagger UI em `/swagger-ui.html` e especificação em `/v3/api-docs` (e `.yaml`), configuradas em
  `application.properties`, com ordenação alfabética de tags e operações.
- `OpenApiConfig` declara o título "Vitryne API", a versão "v1-mvp", a descrição dos fluxos
  disponíveis e o esquema de segurança `bearerAuth`, do tipo HTTP bearer com JWT. A descrição
  orienta a fazer login em `POST /auth/login` e colar apenas o token.
- Os controllers descrevem operações e respostas com anotações do OpenAPI. Exemplos de payload
  ficam no pacote `documentation` (`RegistrationExamples`).
- As rotas da documentação são públicas (ADR-038).

## Consequências

### O que ganhamos

- Contrato publicado e atualizado junto com o código
- Teste dos endpoints autenticados pelo próprio Swagger UI, com o esquema `bearerAuth`
- Quem consome (front-end e mobile) confere campos e respostas sem perguntar ao back-end

### O que aceitamos em troca

- As rotas de documentação são públicas em qualquer ambiente, sem restrição por perfil
- A documentação depende de anotações escritas à mão; um endpoint sem anotação aparece pela
  estrutura do código, sem descrição
- A versão "v1-mvp" é só um rótulo: os endpoints não têm prefixo de versão (fila do README)
- Os DTOs de erro documentados não cobrem os quatro formatos de resposta reais (ADR-039)

## Validação

`GET /swagger-ui.html` e `GET /v3/api-docs` respondem sem autenticação, e todo endpoint listado em
`SecurityConfig` aparece na especificação.

## Evidências

- `backend/pom.xml` — `springdoc-openapi-starter-webmvc-ui`
- `backend/src/main/resources/application.properties` — `springdoc.swagger-ui.*`
- `backend/src/main/java/com/vitryne/api/shared/config/OpenApiConfig.java`
- `backend/src/main/java/com/vitryne/api/documentation/RegistrationExamples.java`
- ETI-261 — Fluxo de cadastro e Swagger da aplicação
