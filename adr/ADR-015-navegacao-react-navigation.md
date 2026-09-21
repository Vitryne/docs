---
id: ADR-015
titulo: Navegação do app mobile com React Navigation em vez de Expo Router
status: Aceita
data: 2026-09-19
autor: Leonardo Xavier
decisores: Leonardo Xavier e Bruno Abrahim
area: Mobile
substitui: —
substituida-por: —
---

# ADR-015 — Navegação do app mobile com React Navigation em vez de Expo Router

## Contexto e problema

O app mobile atende consumidores e entregadores, com fluxos distintos que compartilham
componentes. O projeto nasceu do template do Expo, que vem com Expo Router — navegação baseada
em estrutura de arquivos.

O código atual não usa Expo Router: a navegação é declarada explicitamente com o stack nativo do
React Navigation, dividida em três áreas — geral, entregador e consumidor. A troca nunca foi
registrada, e a documentação do projeto ainda descreve o app com Expo Router.

Esta ADR existe para fechar essa divergência com uma decisão, em vez de deixá-la como pendência.

## Critérios de decisão

- Rotas de cada área visíveis em um lugar só, com parâmetros tipados
- Cabeçalho e tema padronizados em todas as telas, sem repetição
- Separação entre os fluxos de consumidor e de entregador
- Estrutura que o time de dois consiga manter

## Alternativas consideradas

1. **React Navigation com `createNativeStackNavigator`, uma navegação por área em
   `src/Shared/Navigation`, `src/Delivery/Navigation` e `src/Customer/Navigation`** — escolhida.
2. **Expo Router, padrão do template** — *perdeu porque:* roteamento por arquivo espalha a
   definição das rotas pela árvore de pastas, e o time preferiu enxergar o fluxo de cada área em
   um arquivo. Em troca, abre mão de deep linking pronto e de divisão automática de código.
3. **Um único stack para o app inteiro** — *perdeu porque:* mistura telas de consumidor e de
   entregador no mesmo arquivo e na mesma lista de parâmetros.
4. **Navegação manual por estado** — *perdeu porque:* reimplementa pilha, botão voltar e transição
   que a biblioteca já entrega testados.

## Decisão

Navegação com React Navigation, stack nativo, dividida em três áreas, cada uma com sua pasta
`Navigation`: `src/Shared/Navigation` para a navegação geral, `src/Delivery/Navigation` para o
entregador e `src/Customer/Navigation` para o consumidor.

Cada área tipa seu stack com uma lista de parâmetros própria — por exemplo,
`DeliveryStackParamList` em `src/Delivery/Types/navigation.ts`. Opções de tela compartilhadas —
cabeçalho nativo oculto e cor de fundo vinda de `src/Shared/Styles/commonStyles` — definidas uma
vez em `screenOptions`.

Organização por domínio: cada área tem `Navigation`, `Screens`, `Hooks` e `Types`. A pasta de cada
tela contém `index.tsx` e `styles.tsx` colocalizado, usando a API `StyleSheet`.

A separação entre os fluxos de consumidor e de entregador, iniciada na ETI-190, se apoia nessa
estrutura.

## Consequências

### O que ganhamos

- Fluxo de cada área legível em um arquivo
- Parâmetros de rota checados pelo compilador
- Consumidor e entregador evoluem separados, sem mexer nas rotas um do outro
- Cabeçalho e tema padronizados sem repetir configuração por tela

### O que aceitamos em troca

- Contra a corrente do Expo: documentação e exemplos novos assumem Expo Router, e o próprio
  `AGENTS.md` do repositório manda consultar a documentação versionada antes de escrever código
- Deep linking e navegação por URL no modo web ficam por conta do time
- Rota nova exige duas edições: o arquivo de rotas e o de tipos da área
- Navegar entre áreas exige navegação aninhada, passando pela navegação geral
- A documentação de arquitetura precisa ser corrigida, porque ainda descreve Expo Router

## Validação

Nenhuma tela navega fora dos stacks declarados nas três pastas `Navigation`, e nenhum `navigate`
usa nome de rota que não esteja na lista de parâmetros da própria área.

## Evidências

- `mobile/src/Delivery/Navigation/` — `DeliveryRoutes` com
  `createNativeStackNavigator<DeliveryStackParamList>`
- `mobile/src/Customer/Navigation/` — rotas do consumidor
- `mobile/src/Shared/Navigation/` — navegação geral
- `mobile/package.json` — `@react-navigation/native` e `@react-navigation/native-stack`
- ETI-190 Separação entregador e consumidor
