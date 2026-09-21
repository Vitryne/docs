---
id: ADR-018
titulo: Design System no Figma como base de componentes do protótipo
status: Aceita
data: 2026-09-13
autor: Hugo Zuin
decisores: Hugo Zuin
area: Produto e UI
substitui: —
substituida-por: —
---

# ADR-018 — Design System no Figma como base de componentes do protótipo

## Contexto e problema

O produto tem quatro âmbitos de aplicação — Web-Portal-Lojista, Mobile-Consumidor,
Mobile-Entregador e Web-Consumidor-Final — e dezenas de telas a desenhar para o protótipo de alta
fidelidade.

O arquivo do Figma tem um único operador. O problema, então, não é coordenar várias pessoas
desenhando: é sustentar o volume. Desenhar tela a tela, do zero, faz o mesmo botão nascer com três
espaçamentos diferentes ao longo de dezenas de telas, e qualquer ajuste de estilo vira varredura
manual pelo arquivo inteiro.

Quem lê o resultado depois é o desenvolvimento — três pessoas que não editam o Figma e precisam
saber, olhando o componente, o que é estado, o que é dado e o que o Figma não representa.

## Critérios de decisão

- Montar dezenas de telas sem redesenhar o mesmo elemento
- Ajuste em um componente propagando para todas as telas que o usam
- Consistência interna do protótipo, que é o que o desenvolvimento vai ler
- Componente legível por quem não edita o arquivo
- Ferramenta que já está em uso

## Alternativas consideradas

1. **Design System próprio no Figma, em Atomic Design, com componentes em conjuntos de variantes e
   contrato de uso escrito na descrição** — escolhida.
2. **Desenhar cada tela do protótipo sem biblioteca de componentes** — *perdeu porque:* com um
   único operador e dezenas de telas, o custo aparece na primeira rodada de ajuste, e a
   inconsistência interna do protótipo chega ao produto pela mão de quem implementa.
3. **Biblioteca de componentes pronta, como Material ou shadcn** — *perdeu porque:* entregaria
   velocidade com identidade visual de terceiro, e a marca é parte do produto. Continua disponível
   como base técnica no código, desde que os valores venham do Design System.
4. **Valores definidos primeiro no código** — *perdeu porque:* o protótipo precisa existir antes da
   implementação, e é ele que orienta a tela.
5. **Um componente solto por estado, em vez de conjunto de variantes** — *perdeu porque:*
   multiplica o número de componentes, impede troca de estado pela instância e espalha o mesmo
   conceito por várias entradas da biblioteca.
6. **Contrato de uso em página separada do Figma ou no Confluence** — *perdeu porque:* a regra fica
   longe de quem lê o componente, e as duas fontes divergem.

## Decisão

O arquivo `Design System - Vitryne ` no Figma reúne os componentes e os valores
que sustentam o protótipo, organizado em Atomic Design: tokens, átomos, moléculas, organismos e
documentação. Cor, tipografia, espaçamento e raio ficam documentados no arquivo.

Tela do protótipo é montada a partir de componente existente; componente novo nasce no Design
System antes de entrar na tela.

Cada conceito de interface é um conjunto de variantes, com os estados e tamanhos como propriedades
de variante. A descrição do componente carrega o contrato de uso, escrita para quem vai implementar
e não edita o arquivo, e registra:

- Onde o componente aparece e a qual caso de uso atende
- O que cada estado significa e quando usar
- Valores concretos, em hexadecimal e em pixel, para o desenvolvimento
- O que é variante e o que é dado — texto que muda por contexto não vira propriedade
- O que o Figma não representa e o desenvolvimento precisa implementar, como animação
- Exceções deliberadas ao sistema, como cor de marca de terceiro
- Origem: o que veio do protótipo anterior e o que é extensão nova do sistema

O Design System é insumo, não entrega. O que vale como referência de interface para o
desenvolvimento é o protótipo montado a partir dele — ADR-019.

Como levar os valores do Design System para o código, sem que divirjam entre Figma, Tailwind e
`commonStyles`, continua na fila de decisões.

## Consequências

### O que ganhamos

- Dezenas de telas montadas por composição, não por redesenho
- Ajuste em um componente propaga para todas as telas do protótipo
- Consistência interna do protótipo garantida por construção, e não por revisão
- Desenvolvimento lê estado, valor e comportamento sem precisar medir a tela
- Rastro entre componente e caso de uso, que é exatamente o que a avaliação pede ao perguntar qual
  decisão de interface está por trás de uma tela

### O que aceitamos em troca

- Operador único no arquivo é gargalo e ponto único de falha: tela travada espera uma pessoa, e o
  conhecimento do sistema não está distribuído pelo time
- Construir o componente antes da tela é investimento adiantado, que só se paga a partir da segunda
  ou terceira reutilização
- Design System só vale enquanto for mantido: componente criado direto na tela sem passar por ele
  quebra a base
- Descrição é texto livre: ninguém garante que ela continue verdadeira quando o componente mudar
- Contrato em português dentro do Figma, código em inglês — a tradução fica por conta de quem
  implementa
- Nada liga automaticamente componente do Figma a componente do código: a correspondência é manual

## Validação

Toda tela do protótipo é montada a partir de componentes da biblioteca, sem elemento solto
redesenhado, e todo componente publicado tem descrição preenchida com estados, valores e o que o
desenvolvimento precisa implementar.

## Evidências

- Figma — [arquivo](https://www.figma.com/design/aS6KcYO6MWteEDkmedZPBa/Vitryne-%E2%80%94-Design-System?node-id=166-500&t=fXsUKLfE4JcT7ZCM-1), publicado como biblioteca `Vitryne — Design System`
- Páginas do arquivo organizadas em Atomic Design: Tokens, Átomos, Moléculas, Organismos e
  Documentação, mais Capa, Como usar e Changelog
- Conjuntos de variantes `Botão Google`, `Spinner` e `Bloco de pagamento PIX`
- Descrição do `Bloco de pagamento PIX` — vínculo com o UC29, distinção entre variante e dado
- Descrição do `Spinner` — animação descrita para implementação em código, e o que veio do
  protótipo anterior
- Descrição do `Botão Google` — exceção de cor de marca de terceiro
- Páginas `Como usar` e `Nomenclatura` no próprio arquivo do Design System
