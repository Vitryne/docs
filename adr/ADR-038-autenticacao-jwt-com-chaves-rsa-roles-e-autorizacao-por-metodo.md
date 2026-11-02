---
id: ADR-038
titulo: Autenticar com JWT assinado por par de chaves RSA e autorizar por perfil nos métodos
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: ADR-011
substituida-por: —
---

# ADR-038 — Autenticar com JWT assinado por par de chaves RSA e autorizar por perfil nos métodos

## Contexto e problema

A ADR-011 decidiu autenticar por JWT validado por um filtro do Spring Security, com o segredo de
assinatura em variável de ambiente e três perfis de acesso. Na época, a API estava aberta e o
carrinho era identificado por um `userId` na URL. A implementação veio na ETI-262, e difere da
ADR-011: a assinatura usa par de chaves RSA lido de arquivos, a validação usa o OAuth2 Resource
Server do Spring, há quatro perfis e a autorização é feita por método.

A pergunta é como a API autentica e autoriza hoje. Esta ADR substitui a ADR-011.

## Critérios de decisão

- Backend sem estado de sessão (regulamento, requisito (a))
- Mesmo mecanismo para web e mobile
- Autorização por perfil
- Identidade do usuário vinda do token, não da URL

## Alternativas consideradas

1. **JWT assinado com RS256 por par de chaves RSA, validado pelo OAuth2 Resource Server** —
   escolhida.
2. **JWT assinado com segredo simétrico em variável de ambiente, validado por filtro próprio** —
   a ADR-011 descrevia isto. A motivação da troca para RSA e Resource Server não está registrada —
   confirmar com o time.
3. **Sessão com cookie de servidor, OAuth 2.0 com provedor externo e chave de API** — descartadas
   na ADR-011, pelos mesmos motivos: estado no servidor, dependência de terceiro e identificar a
   aplicação em vez da pessoa. Esta ADR não as reabre.

## Decisão

**Token.** `POST /auth/login` valida e-mail e senha e devolve um JWT, gerado pelo `TokenService`
com algoritmo RS256. Claims: `iss` = `vitryne-api`, `sub` = id do usuário, `iat`, `exp`, `email` e
`role`. A expiração vem de `jwt.expiration-seconds`, com padrão de 3600 segundos.

**Chaves.** A chave pública e a privada são lidas de `jwt.public.key=classpath:app.pub` e
`jwt.private.key=classpath:app.key`, arquivos em `src/main/resources`. O `app.pub` é versionado. O
`app.key` está no `.gitignore` e não é versionado.

**Validação e sessão.** Spring Security com `oauth2ResourceServer` e `JwtDecoder` pela chave
pública. Sessão `STATELESS`, CSRF desabilitado. Senha armazenada com `BCryptPasswordEncoder`.

**Perfis.** `UserRole` tem `CONSUMER`, `STORE`, `DELIVERER` e `ADMIN`. O perfil vem do claim
`role`, convertido para autoridade com prefixo `ROLE_`. A autorização é por método, com
`@EnableMethodSecurity` e `@PreAuthorize`: `hasRole('STORE')` em `ProductManagementController` e
`StockController`, e `hasRole('CONSUMER')` em `CartController`.

**Rotas públicas.** Swagger e OpenAPI (ADR-040), `POST /auth/register/client`,
`POST /auth/register/merchant`, `POST /auth/login`, `GET /product` e `GET /product/client/*`. Todo
o resto exige autenticação.

**Identidade.** O controller obtém o id do usuário do token por `AuthenticatedUser.id(jwt)`. O
`userId` não vem mais da URL.

## Consequências

### O que ganhamos

- Backend sem estado: qualquer instância com a chave pública valida qualquer token
- Um único mecanismo para web e mobile
- A identidade vem do token, não de um parâmetro manipulável
- Quem valida só precisa da chave pública, e só quem emite precisa da privada

### O que aceitamos em troca

- Sem refresh token e sem revogação: um token válido continua válido até expirar
- A expiração de 3600 s é o padrão do código, e não há decisão registrada sobre esse valor
- O `app.key` fica fora do Git, e o repositório não documenta como cada pessoa o obtém nem como
  ele chega ao ambiente publicado
- O `Dockerfile` copia `src/` inteiro; se o `app.key` existir na máquina que gera a imagem, ele
  entra no JAR e na imagem
- Vazamento da chave privada permite forjar token de qualquer perfil
- Autorização por perfil só existe nos métodos anotados; rota nova sem `@PreAuthorize` aceita
  qualquer usuário autenticado
- O gatilho de revisão do CORS (ADR-012) disparou com a entrada do JWT e segue sem decisão

## Validação

Rota que exige autenticação responde 401 sem token válido, `POST /auth/login` devolve token com os
claims `email` e `role`, e `git ls-files` lista `app.pub` e não lista `app.key`.

## Evidências

- `backend/src/main/java/com/vitryne/api/shared/config/SecurityConfig.java` — filtro, rotas
  públicas, `JwtDecoder`, `JwtEncoder` e conversor de autoridades
- `backend/src/main/java/com/vitryne/api/auth/service/TokenService.java` — claims e RS256
- `backend/src/main/java/com/vitryne/api/shared/security/AuthenticatedUser.java`
- `backend/src/main/java/com/vitryne/api/shared/enums/UserRole.java`
- `backend/src/main/resources/application.properties` — `jwt.public.key`, `jwt.private.key`
- `backend/.gitignore` — `src/main/resources/app.key`
- ETI-262 — Implementar login e controle de acesso por perfil
- ADR-011 — decisão substituída
