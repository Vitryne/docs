---
id: ADR-010
titulo: Nomenclatura em inglês no código e no banco, documentação em português
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Back-end
substitui: —
substituida-por: —
---

# ADR-010 — Nomenclatura em inglês no código e no banco, documentação em português

## Contexto e problema

O backend nasceu com nomenclatura misturada: classes e endpoints em português — `Produto`,
`CarrinhoService`, `/carrinho` — convivendo com termos técnicos em inglês vindos do framework.
Tabelas e colunas seguiam o mesmo padrão híbrido.

Mistura de idioma no mesmo projeto obriga cada pessoa a lembrar em que língua está escrito o
nome que ela procura, e gera nomes como `CarrinhoRepository` e `findByUsuarioId` na mesma linha.

## Critérios de decisão

- Um idioma só dentro do código
- Alinhamento com a convenção do ecossistema Java e Spring
- Documentação legível pelo time e pela banca, que fala português
- Custo de migração pago uma vez, cedo

## Alternativas consideradas

1. **Inglês em todo o código e banco; português em documentação, comentários e artefatos** —
   escolhida.
2. **Português em tudo** — *perdeu porque:* colide com a convenção dos frameworks e com o próprio
   Spring Data, que deriva consulta a partir do nome do método; `findByUsuarioId` já mostrava a
   costura aparecendo.
3. **Manter a mistura** — *perdeu porque:* é o pior dos dois mundos, e o custo de corrigir só
   aumenta com o tamanho do código.

## Decisão

Classes, métodos, variáveis, pacotes, endpoints, tabelas e colunas em inglês. Colunas e tabelas
em `snake_case`. Documentação, comentários, descrição de issue e artefatos formais em português.

Migração executada na ETI-196: `Carrinho` virou `Cart`, `ItemCarrinho` virou `CartItem`,
`Produto` virou `Product`, `Estoque` virou `Stock`, com controllers, services, repositories, DTOs
e exceções renomeados junto. As migrations foram reescritas mantendo as versões originais, o que
obriga a recriar bancos locais.

## Consequências

### O que ganhamos

- Um idioma só dentro do código, sem tradução mental a cada leitura
- Nome de método do Spring Data coerente com o nome do campo
- Alinhamento com a documentação oficial das ferramentas

### O que aceitamos em troca

- Todo mundo teve que recriar o banco local após a migração
- Termos de negócio traduzidos podem perder precisão — "lojista", "entregador" e "loja"
  precisam de tradução acordada antes de virarem classe
- O diagrama de classes e o DER estão em português: as duas fontes deixam de casar nome a nome

## Validação

Nenhum identificador em português em `src/main/java` nem nas migrations, e a aplicação sobe com
o schema novo aplicado do zero.

## Evidências

- ETI-196 Padronizar nomenclatura do back-end para inglês — mergeada na `dev`
- `backend/src/main/java/com/vitryne/api/entity/` — `Cart`, `CartItem`, `Product`, `Stock`
- `backend/src/main/resources/db/migration/` — tabelas `product`, `stock`, `cart`, `cart_item`
- ETI-154 Refatoração do Back-end — "Idioma: pt-BR em documentação e comentários"
