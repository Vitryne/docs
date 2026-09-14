---
id: ADR-011
titulo: Autenticação stateless com JWT e filtro do Spring Security
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Back-end
substitui: —
substituida-por: —
---

# ADR-011 — Autenticação stateless com JWT e filtro do Spring Security

## Contexto e problema

A API está aberta: qualquer requisição alcança qualquer endpoint, e o carrinho é identificado por
um `userId` que chega na URL sem nenhuma prova de que o solicitante é aquele usuário.

O regulamento exige backend stateless, o que descarta sessão em memória do servidor. A plataforma
tem três perfis de acesso — consumidor, lojista e entregador — e dois clientes diferentes, web e
mobile, sem cookie compartilhado entre eles.

## Critérios de decisão

- Backend sem estado de sessão entre requisições
- Mesmo mecanismo servindo web e mobile
- Autorização por perfil, já que os três papéis veem recursos diferentes
- Suportado pelo ecossistema que o projeto já usa

## Alternativas consideradas

1. **JWT assinado, validado por filtro do Spring Security a cada requisição** — escolhida.
2. **Sessão com cookie de servidor** — *perdeu porque:* guarda estado no servidor, o que o
   regulamento veta, e complica o app mobile, que não tem o mesmo ciclo de cookie do navegador.
3. **OAuth 2.0 com provedor externo** — *perdeu porque:* depende de serviço de terceiro e de
   configuração que o time não tem tempo de operar; pode voltar depois como login social.
4. **Chave de API por cliente** — *perdeu porque:* identifica a aplicação, não a pessoa, e o
   domínio precisa saber quem é o usuário.

## Decisão

Autenticação por JWT: login valida credencial e devolve token assinado; um filtro do Spring
Security valida assinatura e expiração em toda requisição e popula o contexto de segurança. O
perfil do usuário viaja no token e sustenta a autorização por rota.

Segredo de assinatura fica em variável de ambiente, nunca no repositório. Senha armazenada como
hash, gerado na camada de autenticação.

Decisões desta ADR que ainda não têm resposta e vão para a fila: tempo de expiração, existência
de refresh token, e o que fazer para revogar token antes do vencimento.

## Consequências

### O que ganhamos

- Backend sem estado: qualquer instância atende qualquer requisição
- Um único mecanismo para web e mobile
- Identidade do usuário deixa de vir como parâmetro de URL manipulável

### O que aceitamos em troca

- Token válido não se revoga sem infraestrutura extra: quem sai continua com acesso até expirar
- Segredo de assinatura vira ponto único de falha, e vazamento compromete todos os tokens
- Endpoints que hoje recebem `userId` na URL precisam mudar de contrato quando o usuário passar a
  vir do token — quanto mais tarde, mais caro

## Validação

Rota protegida sem token válido responde 401, e o carrinho retornado corresponde ao usuário do
token, não ao identificador da URL.

## Evidências

- ETI-70 Implementar token JWT e Security Filter — em andamento
- ETI-125 Criar entidade Usuario e validar mock de usuário — pré-requisito, com campo de senha
  previsto para receber o hash
- `controller/CartController.java` — `@GetMapping("/{userId}")`, identificação pela URL
- Regulamento Escola de TI 2026, requisito arquitetural (a): backend stateless
