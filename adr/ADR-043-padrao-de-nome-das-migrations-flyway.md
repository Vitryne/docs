---
id: ADR-043
titulo: PostgreSQL com schema versionado em Flyway e versão das migrations em data e hora
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: ADR-006
substituida-por: —
---

# ADR-043 — PostgreSQL com schema versionado em Flyway e versão das migrations em data e hora

## Contexto e problema

A ADR-006 decidiu usar PostgreSQL com migrations Flyway, `ddl-auto=none`, e nome de arquivo
`V<timestamp>__ETI-NNN-DESCRICAO.SQL`. O diretório `db/migration` tem hoje 13 migrations, e os
nomes seguem mais de uma forma: as cinco primeiras usam um timestamp em milissegundos
(`V1781471246721`), as seguintes usam data e hora (`V20261022203000`), e a descrição mudou de
maiúsculas com hífen (`CREATE-USER`) para minúsculas com sublinhado (`add_active_to_users`).
Há ainda dois arquivos fora de qualquer dessas formas.

A decisão de usar Flyway e de nunca deixar o Hibernate alterar o schema segue valendo. O que
precisa de decisão é o padrão de nome. Esta ADR substitui a ADR-006 e repete o que dela continua
valendo.

## Critérios de decisão

- Integridade referencial garantida pelo banco, não só pelo código
- Mesmo schema em qualquer máquina, a partir do zero
- Histórico de alterações auditável e ligado à tarefa que o motivou
- Versão que não colida entre branches paralelas
- Nome de arquivo legível, que mostre a data e a tarefa

## Alternativas consideradas

1. **Flyway com `ddl-auto=none` e versão em data e hora (`yyyyMMddHHmmss`)** — escolhida.
2. **Versão em milissegundos desde a época (`V1781471246721`), como nas cinco primeiras** — foi o
   primeiro padrão. A motivação de trocar para data e hora não está registrada — confirmar com o
   time. Na prática, a data e hora é legível sem conversão.
3. **`ddl-auto=update` do Hibernate** — descartada na ADR-006, e esta ADR não a reabre: não
   versiona nada e diverge entre máquinas.
4. **Liquibase e banco não relacional** — descartados na ADR-006, e esta ADR não os reabre.

## Decisão

PostgreSQL como único banco. Schema criado e evoluído exclusivamente por migrations Flyway em
`src/main/resources/db/migration`, com `spring.jpa.hibernate.ddl-auto=none`.

Nome do arquivo de uma migration nova:

```
V<yyyyMMddHHmmss>__ETI-<NNN>-<descricao_em_snake_case>.sql
```

Por exemplo, `V20261022203000__ETI-158-add_active_to_users.sql`. A versão é a data e a hora de
criação, para não colidir entre branches paralelas. A chave do Jira liga a alteração à tarefa. A
descrição fica em minúsculas, com sublinhado, e a extensão é `.sql` em minúsculas.

Os arquivos já aplicados não se renomeiam nem se alteram; correção entra como migration nova. Os
que fogem do padrão ficam como estão:

- as cinco migrations `V17814...__ETI-127-*`, com versão em milissegundos e descrição em
  maiúsculas
- `V20261024230000__expand_client_and_store_registration.sql`, sem chave do Jira
- `V20261031233400__ETI-267-ALIGN_CART_MONEY_PRECISION.SQL`, com a extensão e a descrição em
  maiúsculas

Invariantes que o banco garante: chave estrangeira, `CHECK` de faixa e de valor permitido, e
`UNIQUE` onde a duplicação não faz sentido. Hoje existem, entre outras, `uq_users_email`,
`uq_users_username`, `uq_client_cpf`, `uq_store_cnpj`, `uq_stock_product_size` e
`uq_cart_item_stock`.

## Consequências

### O que ganhamos

- Banco reproduzível do zero, igual em todas as máquinas
- Regra crítica protegida em dois lugares: aplicação e schema
- Cada alteração de schema rastreável até a tarefa que a pediu
- Nome do arquivo mostra a data da alteração

### O que aceitamos em troca

- Toda mudança de entidade exige migration escrita à mão, e esquecer disso quebra a subida
- Migration já aplicada não se altera: três arquivos seguem fora do padrão, e o padrão novo não
  os alcança
- `spring.flyway.baseline-on-migrate=true` está ligado, e aceita aplicar migrations sobre um
  banco que já tem tabelas sem histórico do Flyway
- Nada verifica o padrão de nome automaticamente; a conferência depende de revisão de PR

## Validação

A aplicação sobe com banco vazio e aplica todas as migrations sem erro, nenhuma tabela existe sem
migration correspondente, e todo arquivo novo em `db/migration` casa com
`V[0-9]{14}__ETI-[0-9]+-[a-z0-9_]+\.sql`.

## Evidências

- `backend/src/main/resources/application.properties` — `ddl-auto=none`, `flyway.enabled=true`,
  `flyway.baseline-on-migrate=true`
- `backend/src/main/resources/db/migration/` — 13 migrations
- `V20261022203000__ETI-158-add_active_to_users.sql` — exemplo do padrão em vigor
- `V1781471530531__ETI-127-CREATE-PRODUCT.sql`, `V1781471629971__ETI-127-CREATE-SUPPLY.sql` —
  `CHECK` e `UNIQUE`
- ETI-158, ETI-265 e ETI-267 — migrations de loja, de datas do produto e do estoque, e de
  precisão monetária do carrinho
- ADR-006 — decisão substituída
