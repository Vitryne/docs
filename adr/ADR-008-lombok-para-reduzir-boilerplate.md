---
id: ADR-008
titulo: Adotar Lombok para reduzir código repetitivo no backend
status: Aceita
data: 2026-09-13
autor: João Ehlers
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-008 — Adotar Lombok para reduzir código repetitivo no backend

## Contexto e problema

Entidade JPA em Java pede construtor, getters, setters, builder e log. Escrito à mão, isso
enterra a regra de negócio sob dezenas de linhas sem conteúdo, e revisão de pull request
vira leitura de getter.

## Critérios de decisão

- Revisão focada na regra, não no acessor
- Injeção de dependência por construtor, sem escrever o construtor
- Padrão único de log em toda a aplicação

## Alternativas consideradas

1. **Lombok com `@Getter`, `@Setter`, `@Builder`, `@RequiredArgsConstructor` e `@Slf4j`** —
   escolhida.
2. **Java puro, com tudo escrito à mão** — *perdeu porque:* multiplica o tamanho das classes
   sem acrescentar informação, e getter escrito à mão erra tanto quanto o gerado.
3. **`record` também para entidades** — *perdeu porque:* JPA exige construtor sem argumentos e
   campos mutáveis; `record` serve para DTO, não para entidade gerenciada.

## Decisão

Lombok como dependência opcional, com processador de anotação configurado no
`maven-compiler-plugin` e excluído do artefato final pelo `spring-boot-maven-plugin`.

Uso padronizado:

- Entidades: `@Getter`, `@Setter`, `@Builder`, `@AllArgsConstructor`, `@NoArgsConstructor`
- Services e controllers: `@RequiredArgsConstructor` para injeção por construtor
- Controllers: `@Slf4j` para log de entrada e saída de requisição

## Consequências

### O que ganhamos

- Classes curtas, com a regra de negócio visível
- Injeção por construtor sem custo de escrita, o que mantém os campos `final`
- Log uniforme, sem declaração manual de `Logger`

### O que aceitamos em troca

- Dependência de processador de anotação: quem não habilitar o plugin na IDE vê erro de
  compilação que não existe
- `@Setter` em entidade abre a porta para mudar estado sem passar pela regra de negócio
- `@Builder` em entidade JPA permite construir objeto em estado inválido se o campo for esquecido

## Validação

O projeto compila com `mvn clean package` sem configuração adicional de IDE, e o JAR final não
contém classes do Lombok.

## Evidências

- `backend/pom.xml` — dependência `org.projectlombok:lombok` como `optional`, `annotationProcessorPaths`
  e exclusão no `spring-boot-maven-plugin`
- `entity/Product.java`, `service/CartService.java`, `controller/CartController.java`
