---
id: ADR-020
titulo: Padrão de estilos do app mobile com StyleSheet, tokens compartilhados e nomes em snake_case
status: Proposta
data: 2026-09-19
autor: Leonardo Xavier
decisores: Leonardo Xavier e Bruno Valerio
area: Mobile
substitui: —
substituida-por: —
---

# ADR-020 — Padrão de estilos do app mobile com StyleSheet, tokens compartilhados e nomes em snake_case

## Contexto e problema

O app mobile tem várias telas, nas áreas geral, entregador e consumidor, que repetem os mesmos
elementos visuais: fundo de tela, cards, botões, rótulos de seção. Sem um padrão, cada tela acaba
redefinindo esses estilos e escrevendo cores direto no código, o que gera diferenças visuais
entre telas e dificulta mudar a identidade do app.

Os nomes dos estilos também não tinham regra, o que deixava as telas inconsistentes entre si.

Esta ADR define como os estilos são escritos, nomeados e compartilhados.

## Critérios de decisão

- Identidade visual consistente em todas as telas
- Trocar uma cor ou espaçamento em um lugar só
- Evitar estilos duplicados entre telas
- Nomes de estilo previsíveis e fáceis de ler
- Sem dependência extra, usando o que o React Native já oferece

## Alternativas consideradas

1. **API `StyleSheet` do React Native, com tokens e estilos comuns em
   `src/Shared/Styles/commonStyles.ts` e `styles.tsx` colocalizado em cada tela** — escolhida.
2. **NativeWind (Tailwind para React Native)** — *perdeu porque:* adiciona dependência e
   configuração de build, e exige que o time aprenda as classes utilitárias. Em troca, teria
   estilos mais curtos direto no JSX.
3. **styled-components** — *perdeu porque:* adiciona dependência e muda a forma de escrever
   componentes, sem ganho claro para o tamanho do app.
4. **Estilos inline em cada componente** — *perdeu porque:* recria os objetos a cada render,
   espalha cores e medidas pelo código e impede reaproveitamento.
5. **Nomes de estilo em camelCase, padrão mais comum na comunidade** — *perdeu porque:* o time já
   adotou snake_case em `commonStyles` e nas telas, e trocar agora geraria retrabalho sem ganho.

## Decisão

Estilos escritos com a API `StyleSheet` do React Native.

`src/Shared/Styles/commonStyles.ts` concentra:

- `colors`: paleta do app (primária, texto, fundo, borda, estados de erro e sucesso)
- `spacing`: espaçamentos de `xs` (4) a `xl` (32)
- `radius`: arredondamentos de `sm` (8) a `full` (999)
- `commonStyles`: só os estilos que aparecem em várias telas, como `screen`, `card`, `row`,
  `section_label`, `button` e `button_text`

Regras:

- Nomes de estilo sempre em minúsculo, com `_` separando as palavras — por exemplo,
  `scroll_content`, `section_label` e `button_text`. Os tokens (`colors`, `spacing`, `radius`)
  seguem o padrão de objeto JavaScript, como `primarySoft` e `textMuted`
- Cores sempre vindas de `colors`, nunca escritas em hexadecimal nas telas
- Espaçamentos e arredondamentos sempre vindos de `spacing` e `radius`
- Antes de criar um estilo, verificar se já existe em `commonStyles`; se existir, usar o comum
- Estilos específicos de uma tela ficam no `styles.tsx` da própria tela
- Um estilo só vai para `commonStyles` quando uma segunda tela precisar dele
- Para ajustar um estilo comum em uma tela, combinar os dois em vez de copiar:
  `style={[commonStyles.card, styles.product_card]}`

## Consequências

### O que ganhamos

- Visual consistente entre as telas
- Mudar a cor primária ou um espaçamento afeta o app inteiro com uma edição
- Menos código repetido nos `styles.tsx`
- Nomes de estilo com o mesmo padrão em todo o projeto
- Nenhuma biblioteca extra

### O que aceitamos em troca

- Dois padrões de nome convivendo: snake_case nos estilos e camelCase nos tokens, que precisa
  ficar claro para quem entra no projeto
- snake_case foge do padrão mais comum em exemplos e bibliotecas de React Native
- Tamanhos de fonte ainda são escritos direto nos estilos, porque não há token de tipografia
- Sem tema escuro: as cores são fixas, e suportar tema exigiria reorganizar `colors`
- Decidir o que é "comum" depende de revisão; sem ela, estilos duplicados voltam a aparecer

## Validação

Nenhum `styles.tsx` contém cor em hexadecimal, nenhum nome de estilo usa camelCase, e nenhum
estilo de tela repete um estilo que já existe em `commonStyles`.

## Evidências

- `mobile/src/Shared/Styles/commonStyles.ts` — `colors`, `spacing`, `radius` e `commonStyles`
- `mobile/src/Delivery/Navigation/` — `screenOptions` usando `colors.background`
- `styles.tsx` colocalizado nas pastas das telas
