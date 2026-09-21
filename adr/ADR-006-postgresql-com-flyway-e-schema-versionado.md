---
id: ADR-006
titulo: PostgreSQL como banco único com schema versionado em Flyway
status: Aceita
data: 2026-09-13
autor: Equipe
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-006 — PostgreSQL como banco único com schema versionado em Flyway

## Contexto e problema

O domínio é relacional por natureza: produto tem estoque por tamanho, carrinho tem itens,
pedido referencia loja, consumidor e entregador. O regulamento pede persistência com controle
transacional, e o time já produziu um diagrama entidade-relacionamento como artefato.

Com sete pessoas subindo o banco em máquinas diferentes, o schema precisa ser reproduzível.
Deixar o Hibernate gerar as tabelas a partir das entidades cria divergência silenciosa entre
ambientes e apaga o histórico de como o schema chegou ao estado atual.

## Critérios de decisão

- Integridade referencial garantida pelo banco, não só pelo código
- Mesmo schema em qualquer máquina, a partir do zero
- Histórico de alterações auditável e vinculado à tarefa que as motivou
- Nada de alteração automática de schema em tempo de execução

## Alternativas consideradas

1. **PostgreSQL com migrations Flyway e `ddl-auto=none`** — escolhida.
2. **`ddl-auto=update` do Hibernate** — *perdeu porque:* não versiona nada, diverge entre
   máquinas, não remove coluna e é perigoso em qualquer ambiente que não seja descartável.
3. **Liquibase** — *perdeu porque:* XML ou YAML a mais para aprender; o time escreve SQL direto,
   e o Flyway aceita SQL puro.
4. **Banco não relacional** — *perdeu porque:* o modelo é relacional, com constraints que o banco
   pode garantir, e o time já modelou o DER como artefato.

## Decisão

PostgreSQL como único banco. Schema criado e evoluído exclusivamente por migrations Flyway em
`src/main/resources/db/migration`, com `spring.jpa.hibernate.ddl-auto=none`.

Nome do arquivo: `V<timestamp>__ETI-NNN-DESCRICAO.SQL`, com versão em timestamp para evitar
colisão entre branches paralelas e a chave do Jira ligando a alteração à tarefa.

Invariantes que o banco garante: chave estrangeira, `CHECK` de faixa e de valor permitido, e
`UNIQUE` onde a duplicação não faz sentido.

## Consequências

### O que ganhamos

- Banco reproduzível do zero, igual em todas as máquinas
- Regra crítica protegida em dois lugares: aplicação e schema
- Cada alteração de schema rastreável até a tarefa que a pediu

### O que aceitamos em troca

- Toda mudança de entidade exige migration escrita à mão, e esquecer disso quebra a subida
- Migration já aplicada não se altera: correção entra como migration nova
- Renomeações em massa, como a da ETI-196, obrigam a recriar bancos locais

## Validação

Aplicação sobe com banco vazio e aplica todas as migrations sem erro, e nenhuma tabela existe
sem migration correspondente.

## Evidências

- `backend/src/main/resources/application.properties` — `ddl-auto=none`, `flyway.enabled=true`
- `backend/src/main/resources/db/migration/` — sete migrations no padrão `V<timestamp>__ETI-NNN`
- `V1781471530531__ETI-127-CREATE-PRODUCT.SQL` — `CHECK` de status, rating e preço promocional
- `V1781471891900__ETI-127-CREATE-CART-ITEM.SQL` — chave estrangeira e `UNIQUE (cart_id, stock_id)`
