---
id: ADR-005
titulo: Organizar o backend em camadas técnicas com regra de negócio nas entidades
status: Aceita
data: 2026-09-13
autor: João Ehlers
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-005 — Organizar o backend em camadas técnicas com regra de negócio nas entidades

## Contexto e problema

O backend está organizado em pacotes por camada técnica — controller, service, repository,
entity, dto, exception — sem fronteira de módulo por domínio. Ao mesmo tempo, o termo
"monólito modular com DDD" circula na documentação do projeto, descrevendo uma arquitetura que o
código não implementa.

Ao mesmo tempo, algumas entidades não são anêmicas: Product concentra cálculo de preço final,
configuração e verificação de promoção, e verificação de disponibilidade de estoque. Outras
entidades do domínio, no entanto, são e devem continuar sendo simples portadoras de estado, sem
regra de negócio própria.

Esta ADR existe para o time decidir qual das duas é a arquitetura real — camadas técnicas ou
monólito modular com DDD — e, dentro da arquitetura escolhida, estabelecer um critério explícito
para quando uma entidade concentra regra de negócio e quando ela permanece anêmica, em vez de
tratar "entidade rica" como padrão aplicado uniformemente a todo o domínio.

## Critérios de decisão

- Legível para sete pessoas com níveis diferentes de experiência
- Consistente entre o que está escrito e o que está implementado
- Regra de negócio em lugar previsível, não espalhada entre service e entidade
- Critério de aplicação replicável, para que qualquer pessoa do time decida sem precisar
- perguntar caso a caso onde uma nova regra deve morar
- Custo de mudança compatível com o tempo restante do projeto

## Alternativas consideradas

1. Camadas técnicas, com regra de negócio aplicada seletivamente nas entidades conforme
   critério explícito — escolhida.
2. Monólito modular com DDD, como está documentado — perdeu porque: exigiria reorganizar
   todo o código por bounded context, com agregados, repositórios de domínio e camada de
   aplicação, num momento em que o time ainda está fechando o primeiro caso de uso completo.
3. Continua sendo o destino possível, e reverter esta ADR é barato enquanto o código é pequeno.
4. Arquitetura hexagonal — perdeu porque: introduz portas, adaptadores e inversão de
   dependência que só se pagam quando há mais de uma fonte de dados ou mais de um cliente de
   entrada.
5. Entidades anêmicas com toda a regra nos services, aplicada uniformemente — perdeu
   porque: deixaria as regras de preço e estoque duplicadas em cada service que as consome.
6. Entidades ricas com toda a regra nas entidades, aplicada uniformemente — perdeu porque:
   entidades majoritariamente de leitura/escrita sem regra de transição de estado (ex: endereço,
   notificação) não ganham nada em encapsular comportamento, e ainda pagam o custo de acoplar
   domínio a Hibernate sem benefício correspondente.

## Decisão

O backend fica organizado por camada técnica. controller recebe e responde, service
orquestra e controla transação, repository persiste, entity guarda o estado e, quando o
critério abaixo se aplica, as regras que pertencem ao próprio objeto.

Critério de quando a regra mora na entidade: a lógica fica na entidade quando atende a pelo
menos um dos dois pontos a seguir:

(a) é consumida por mais de um service, ou
(b) protege um invariante cuja violação causaria inconsistência de dado (ex: preço promocional
maior que o preço normal, estoque negativo).

Fora esses dois casos — entidades majoritariamente de leitura/escrita, sem regra de transição de
estado (ex: endereço, notificação) — a entidade permanece anêmica, e qualquer lógica associada
fica no service. O resultado esperado não é "todas as entidades ricas" nem "todas anêmicas", e
sim um mix onde cada entidade é classificada individualmente por esse critério.

Injeção de dependência por construtor, via @RequiredArgsConstructor. Serviços anotados com
@Transactional, e readOnly = true em leitura.

## Consequências

### O que ganhamos

- Estrutura previsível: qualquer pessoa acha a classe pelo nome do pacote
- Nenhum trabalho de reorganização agora, com o time em ritmo de entrega

### O que aceitamos em troca
- Documentação de arquitetura precisa ser corrigida: hoje ela promete DDD e entrega camadas
- Sem fronteira de módulo, nada impede um service de alcançar qualquer repository
- Entidades classificadas como ricas pelo critério acima acoplam domínio a Hibernate: mudar de
- ORM significa mexer nessas regras junto — custo localizado às entidades que atendem ao
  critério, não espalhado pelo domínio inteiro
- A pasta service tende a crescer sem divisão por domínio conforme novos casos de uso entram
- O critério exige julgamento na aplicação (é preciso avaliar se uma regra nova atende a (a) ou
  (b)), diferente de uma regra universal que não exige avaliação caso a caso

## Validação

Não existe classe de regra de negócio fora de entity ou service, a documentação de
arquitetura do projeto descreve a arquitetura que está no código, e toda entidade classificada
como rica atende a pelo menos um dos dois critérios da seção Decisão — toda entidade anêmica foi
verificada como não atendendo a nenhum.

## Evidências

- backend/src/main/java/com/vitryne/api/ — pacotes controller, service, repository,
   entity, dto, exception
- entity/Product.java — calculateFinalPrice, configureSale, isItInSale, checkAvailability
   (rico por critério (a): consumido por mais de um service — catálogo e checkout)
- service/CartService.java — orquestração com @Transactional
- ETI-154 Refatoração do Back-end — escopo por classe, sem menção a módulo de domínio
