---
id: ADR-025
titulo: Ícones do app mobile com @expo/vector-icons e SVG próprio, sem ícones em PNG
status: Proposta
data: 2026-09-19
autor: Leonardo Xavier
decisores: Leonardo Xavier e Bruno Valerio
area: Mobile
substitui: —
substituida-por: —
---

# ADR-025 — Ícones do app mobile com @expo/vector-icons e SVG próprio, sem ícones em PNG

## Contexto e problema

O app usa o `@expo/vector-icons` para os ícones, mas a biblioteca não tem todos os ícones que as
telas precisam. Para os que faltam, a saída mais comum é usar uma imagem PNG.

Ícone em PNG perde qualidade quando muda de tamanho, precisa de versões em várias resoluções e
não troca de cor com facilidade. Além disso, as cores do app ficam em `colors` (ADR-020), e um
ícone com a cor presa no arquivo foge desse padrão.

Esta ADR define como os ícones são usados e quando usar SVG, PNG ou JPG.

## Critérios de decisão

- Ícone nítido em qualquer tamanho e tela
- Trocar a cor do ícone pelo código, usando os tokens de `colors`
- Poder usar ícones que não existem no `@expo/vector-icons`
- Funcionar no Expo Go, sem development build (ADR-024)
- Pouca configuração extra

## Alternativas consideradas

1. **`@expo/vector-icons` primeiro e, quando não houver o ícone, SVG próprio com
   `react-native-svg`, importado como componente** — escolhida.
2. **Ícones em PNG** — *perdeu porque:* perde nitidez ao mudar de tamanho, precisa de arquivos
   `@2x` e `@3x`, e cada cor diferente acaba virando outro arquivo.
3. **Só `@expo/vector-icons`** — *perdeu porque:* não tem todos os ícones de que o app precisa, e
   o time ficaria limitado ao que a biblioteca oferece.
4. **SVG escrito como texto no código, com `SvgXml`** — *perdeu porque:* o SVG vira uma string
   longa dentro de um arquivo `.ts`, difícil de ler e de trocar. Em troca, não exigiria configurar
   o Metro.
5. **Fonte de ícones própria (IcoMoon)** — *perdeu porque:* exige gerar a fonte de novo a cada
   ícone adicionado.
6. **SVG também para imagens coloridas e ilustrações** — *perdeu porque:* SVG com muitos detalhes
   pesa na renderização, e foto não é vetor.

## Decisão

Ordem de escolha para ícones:

1. Ícone do `@expo/vector-icons`, quando existir
2. SVG próprio, quando o ícone não existir na biblioteca

Imagens coloridas, fotos e ilustrações usam PNG ou JPG. Nenhum ícone é feito em PNG ou JPG.

Regras para os SVGs:

- Renderizados com `react-native-svg` e importados como componente com
  `react-native-svg-transformer`
- Só ícones de uma cor, com `fill="currentColor"` (ou `stroke="currentColor"`) no arquivo, para a
  cor vir do código
- Cor sempre vinda de `colors` (ADR-020), nunca escrita no arquivo nem na tela
- Tamanho passado por `width` e `height` no uso do componente
- Arquivos guardados em `assets/Icons`, na raiz do projeto, junto com as outras imagens
- Nome do arquivo em camelCase (ADR-022), e o componente importado em PascalCase com sufixo
  `Icon` — por exemplo, `hanger.svg` importado como `HangerIcon`
- Bibliotecas instaladas com `npx expo install`, para ficarem compatíveis com o SDK

Exemplo de uso:

`<HangerIcon width={24} height={24} color={colors.primary} />`

## Consequências

### O que ganhamos

- Ícones nítidos em qualquer tamanho e tela
- Cor do ícone trocada pelo código, seguindo a paleta do app
- Liberdade para usar ícones que não existem no `@expo/vector-icons`
- Funciona no Expo Go, porque o `react-native-svg` já vem incluído nele
- Nenhum ícone em várias resoluções para manter

### O que aceitamos em troca

- Configuração extra: `metro.config.js` para o transformer e um arquivo de tipos para o
  TypeScript aceitar o import de `.svg`
- SVG exportado de sites e do Figma costuma vir com cor fixa, que precisa ser trocada por
  `currentColor` à mão
- Dois jeitos de usar ícone no app, o que exige conferir primeiro se o ícone existe na biblioteca
- Ícones de mais de uma cor não seguem essa regra e precisam de outra solução, se aparecerem

## Validação

Nenhum ícone em PNG ou JPG, nenhum SVG com cor escrita no arquivo, e nenhum SVG próprio para um
ícone que já existe no `@expo/vector-icons`.

## Evidências

- `mobile/package.json` — `react-native-svg` e `react-native-svg-transformer`
- `mobile/metro.config.js` — configuração do transformer
- `mobile/assets/Icons` — arquivos `.svg`
- `mobile/package.json` — `@expo/vector-icons`
