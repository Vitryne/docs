---
id: ADR-005
titulo: Organizar o backend em camadas técnicas com regra de negócio nas entidades
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Back-end
substitui: —
substituida-por: —
---

# ADR-005 — Organizar o backend em camadas técnicas com regra de negócio nas entidades

## Contexto e problema

O backend está organizado em pacotes por camada técnica — `controller`, `service`, `repository`,
`entity`, `dto`, `exception` — sem fronteira de módulo por domínio. Ao mesmo tempo, o termo
"monólito modular com DDD" circula na documentação do projeto, descrevendo uma arquitetura que o
código não implementa.

Ao mesmo tempo, as entidades não são anêmicas: `Product` concentra cálculo de preço final,
aplicação e remoção de desconto, verificação de disponibilidade e baixa de estoque.

Esta ADR existe para o time decidir qual das duas é a arquitetura real: ou o código passa a
seguir a intenção que a documentação declara, ou a documentação passa a descrever o código.

## Critérios de decisão

- Legível para sete pessoas com níveis diferentes de experiência
- Consistente entre o que está escrito e o que está implementado
- Regra de negócio em lugar previsível, não espalhada entre service e entidade
- Custo de mudança compatível com o tempo restante do projeto

## Alternativas consideradas

1. **Camadas técnicas com entidades ricas, como já está implementado** — escolhida para
   registro, sujeita à validação do time.
2. **Monólito modular com DDD, como está documentado** — *perdeu porque:* exigiria reorganizar
   todo o código por bounded context, com agregados, repositórios de domínio e camada de
   aplicação, num momento em que o time ainda está fechando o primeiro caso de uso completo.
   Continua sendo o destino possível, e reverter esta ADR é barato enquanto o código é pequeno.
3. **Arquitetura hexagonal** — *perdeu porque:* introduz portas, adaptadores e inversão de
   dependência que só se pagam quando há mais de uma fonte de dados ou mais de um cliente de
   entrada.
4. **Entidades anêmicas com toda a regra nos services** — *perdeu porque:* deixaria as regras de
   preço e estoque duplicadas em cada service que as consome.

## Decisão

O backend fica organizado por camada técnica. `controller` recebe e responde, `service`
orquestra e controla transação, `repository` persiste, `entity` guarda o estado e as regras
que pertencem ao próprio objeto.

Injeção de dependência por construtor, via `@RequiredArgsConstructor`. Serviços anotados com
`@Transactional`, e `readOnly = true` em leitura.

Regra prática para decidir onde a lógica mora: o que depende só do estado da própria entidade
fica na entidade; o que depende de mais de um agregado ou de repositório fica no service.

## Consequências

### O que ganhamos

- Estrutura previsível: qualquer pessoa acha a classe pelo nome do pacote
- Regras de produto e estoque ficam junto do dado que elas protegem
- Nenhum trabalho de reorganização agora, com o time em ritmo de entrega

### O que aceitamos em troca

- Documentação de arquitetura precisa ser corrigida: hoje ela promete DDD e entrega camadas
- Sem fronteira de módulo, nada impede um service de alcançar qualquer repository
- Entidade JPA com regra de negócio acopla domínio a Hibernate: mudar de ORM significa mexer nas
  regras junto
- A pasta `service` tende a crescer sem divisão por domínio conforme novos casos de uso entram

## Validação

Não existe classe de regra de negócio fora de `entity` ou `service`, e a documentação de
arquitetura do projeto descreve a arquitetura que está no código.

## Evidências

- `backend/src/main/java/com/vitryne/api/` — pacotes `controller`, `service`, `repository`,
  `entity`, `dto`, `exception`
- `entity/Product.java` — `calculateFinalPrice`, `applyDiscount`, `decreaseStock`
- `service/CartService.java` — orquestração com `@Transactional`
- ETI-154 Refatoração do Back-end — escopo por classe, sem menção a módulo de domínio
