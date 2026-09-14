---
id: ADR-017
titulo: Navegação do app mobile com React Navigation em vez de Expo Router
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Mobile
substitui: —
substituida-por: —
---

# ADR-017 — Navegação do app mobile com React Navigation em vez de Expo Router

## Contexto e problema

O app mobile atende consumidores e entregadores, com fluxos distintos que compartilham
componentes. O projeto nasceu do template do Expo, que vem com Expo Router — navegação baseada
em estrutura de arquivos.

O código atual não usa Expo Router: a navegação é declarada explicitamente com o stack nativo do
React Navigation, em um arquivo único de rotas. A troca nunca foi registrada, e a documentação do
projeto ainda descreve o app com Expo Router.

Esta ADR existe para fechar essa divergência com uma decisão, em vez de deixá-la como pendência.

## Critérios de decisão

- Rotas visíveis em um lugar só, com parâmetros tipados
- Cabeçalho e tema padronizados em todas as telas, sem repetição
- Separação futura entre os fluxos de consumidor e de entregador
- Estrutura que o time de dois consiga manter

## Alternativas consideradas

1. **React Navigation com `createNativeStackNavigator`, rotas declaradas em `src/Navigation`** —
   escolhida para registro, sujeita à validação.
2. **Expo Router, padrão do template** — *perdeu porque:* roteamento por arquivo espalha a
   definição das rotas pela árvore de pastas, e o time preferiu enxergar o fluxo inteiro em um
   arquivo. Em troca, abre mão de deep linking pronto e de divisão automática de código.
3. **Navegação manual por estado** — *perdeu porque:* reimplementa pilha, botão voltar e transição
   que a biblioteca já entrega testados.

## Decisão

Navegação com React Navigation, stack nativo, declarada em `src/Navigation/rotas.tsx`. Parâmetros
tipados por `RootStackParamList` em `src/Types/navigation.ts`. Opções de tela compartilhadas —
cor de fundo, tipografia do título, botão de voltar próprio — definidas uma vez em
`screenOptions`.

Organização por funcionalidade: cada tela é uma pasta com `Screens`, `Components` e `index.tsx`
como ponto de entrada, com `styles.tsx` colocalizado usando a API `StyleSheet` e tokens de
`src/Styles/commonStyles`.

A separação entre os fluxos de consumidor e de entregador, iniciada na ETI-190, se apoia nessa
estrutura.

## Consequências

### O que ganhamos

- Fluxo de navegação inteiro legível em um arquivo
- Parâmetros de rota checados pelo compilador
- Cabeçalho e tema padronizados sem repetir configuração por tela

### O que aceitamos em troca

- Contra a corrente do Expo: documentação e exemplos novos assumem Expo Router, e o próprio
  `AGENTS.md` do repositório manda consultar a documentação versionada antes de escrever código
- Deep linking e navegação por URL no modo web ficam por conta do time
- Rota nova exige duas edições: o arquivo de rotas e o de tipos
- A documentação de arquitetura precisa ser corrigida, porque ainda descreve Expo Router

## Validação

Nenhuma tela navega fora do stack declarado em `rotas.tsx`, e nenhum `navigate` usa nome de rota
que não esteja em `RootStackParamList`.

## Evidências

- `mobile/src/Navigation/rotas.tsx` — `createNativeStackNavigator<RootStackParamList>`
- `mobile/package.json` — `@react-navigation/native` e `@react-navigation/native-stack`, sem
  `expo-router`
- `mobile/src/Screens/` — pastas por funcionalidade com `styles.tsx` colocalizado
- ETI-190 Separação entregador e consumidor
