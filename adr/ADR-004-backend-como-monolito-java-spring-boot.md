---
id: ADR-004
titulo: Backend como monólito único em Java 21 com Spring Boot 4
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Back-end
substitui: —
substituida-por: —
---

# ADR-004 — Backend como monólito único em Java 21 com Spring Boot 4

## Contexto e problema

A plataforma precisa de uma API REST stateless com controle transacional e persistência,
exigência arquitetural explícita do regulamento. O time de back-end tem três pessoas, e a
disciplina cobre um período letivo — não há espaço para operar infraestrutura distribuída.

O domínio envolve catálogo, estoque, carrinho, pedido, pagamento e entrega, todos com
transações que cruzam mais de uma entidade.

## Critérios de decisão

- Controle transacional entre entidades, sem coordenação distribuída
- Stack que a maioria do time de back-end já conhece
- Tempo de setup baixo e ambiente local simples de subir
- Um único artefato de deploy

## Alternativas consideradas

1. **Monólito único em Java 21 com Spring Boot 4, artefato `com.vitryne.api`** — escolhida.
2. **Microsserviços por domínio** — *perdeu porque:* transação distribuída, service discovery e
   observabilidade custam mais do que o problema que resolvem nesta escala, e multiplicariam o
   trabalho de infraestrutura por um time de três pessoas.
3. **Node.js com NestJS** — *perdeu porque:* o time de back-end tem Java como linguagem comum, e
   trocar de stack por preferência individual custaria semanas de curva.
4. **Backend as a service** — *perdeu porque:* o regulamento não permite frameworks que escondam
   grande parte da construção do software, e o objetivo da disciplina é justamente a construção.

## Decisão

Um único serviço Spring Boot 4.0.6 sobre Java 21, expondo API REST stateless, empacotado como
JAR executável e publicado em imagem Docker. Pacote base `com.vitryne.api`.

Dependências de partida: Spring Web MVC, Spring Data JPA, Flyway, driver PostgreSQL e Lombok.

## Consequências

### O que ganhamos

- Transação local resolve consistência entre carrinho, estoque e produto
- Um repositório, um build, um deploy
- Ecossistema Spring cobre segurança, validação e documentação sem trazer stack nova

### O que aceitamos em troca

- Escala é do processo inteiro: não dá para escalar só a leitura de catálogo sem escalar o resto
- Acoplamento interno cresce sem barreira física entre módulos — o limite é disciplina de código
- Spring Boot 4 é versão recente, com menos material de apoio e menos respostas prontas quando
  algo quebra

## Validação

A aplicação sobe com `docker compose up --build` e responde aos endpoints de produto, estoque e
carrinho sem depender de estado em memória entre requisições.

## Evidências

- `backend/pom.xml` — `spring-boot-starter-parent` 4.0.6, `java.version` 21
- `backend/src/main/java/com/vitryne/api/ApiApplication.java`
- `backend/Dockerfile` — build multi-stage Maven e runtime `eclipse-temurin:21-jre-alpine`
- ETI-154 Refatoração do Back-end — stack declarada na epic
- Regulamento Escola de TI 2026, requisito arquitetural (a): backend stateless REST com controle
  transacional e persistência
