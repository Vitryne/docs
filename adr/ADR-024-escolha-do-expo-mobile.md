---
id: ADR-024
titulo: Base do app mobile com Expo em vez de React Native CLI
status: Proposta
data: 2026-09-19
autor: Leonardo Xavier
decisores: Leonardo Xavier e Bruno Valerio
area: Mobile
substitui: —
substituida-por: —
---

# ADR-024 — Base do app mobile com Expo em vez de React Native CLI

## Contexto e problema

O app mobile começou com o React Native CLI. Com ele, rodar o app em um iPhone exige um computador
com macOS e Xcode, e rodar no Android exige instalar e configurar o Android Studio, o SDK e o JDK
em cada máquina.

Parte do time desenvolve no Windows, e quem usava iPhone não conseguia rodar o app no próprio
celular. Com isso, só parte da equipe conseguia testar o que desenvolvia.

A troca para o Expo já foi feita, mas nunca foi registrada. Esta ADR registra essa escolha e define
como o Expo é usado no projeto.

## Critérios de decisão

- Qualquer integrante consegue rodar e testar o app, no Windows, Linux ou macOS, com Android ou
  iPhone
- Ambiente de desenvolvimento simples de configurar
- TypeScript e React, como no front web
- Acesso a recursos do celular (arquivos, câmera) sem escrever código nativo
- Geração do instalador do app para a entrega

## Alternativas consideradas

1. **Expo, com Expo Go no dia a dia** — escolhida.
2. **React Native CLI, usado no início do projeto** — *perdeu porque:* rodar no iPhone exige macOS
   e Xcode, o que deixava de fora quem usa Windows, e o ambiente Android é trabalhoso de montar em
   cada máquina. Em troca, teria controle total sobre o código nativo.
3. **Flutter** — *perdeu porque:* usa Dart, diferente do TypeScript e do React usados no front web,
   e o time teria que aprender uma linguagem nova.
4. **Nativo (Kotlin e Swift)** — *perdeu porque:* exige dois códigos separados, um por plataforma,
   o que não cabe em um time de dois.
5. **Expo só com development build, sem Expo Go** — *perdeu porque:* exige gerar um build novo a
   cada biblioteca nativa adicionada, e instalar no iPhone exige conta paga de desenvolvedor da
   Apple.

## Decisão

App criado com Expo, SDK 56, em TypeScript.

No dia a dia:

- O app roda no celular físico pelo Expo Go, lendo o QR code gerado por `npx expo start`
- Bibliotecas instaladas com `npx expo install`, que escolhe a versão compatível com o SDK
- Preferência por bibliotecas do próprio Expo (`expo-*`) ou que já funcionam no Expo Go
- Configuração do app (nome, ícone, permissões) feita no `app.json`
- Navegação com React Navigation, sem Expo Router, conforme a ADR-017

Hoje o app não usa nenhuma biblioteca com código nativo fora do Expo Go, então o Expo Go atende o
desenvolvimento. Se alguma entrar, ela só poderá ser testada em um development build.

O app é entregue apenas para Android, em um APK gerado pelo EAS Build. O iOS fica como melhoria
futura: o time não tem as ferramentas nem o tempo necessários para tratar a publicação na
plataforma. O iPhone continua servindo para desenvolver e testar pelo Expo Go.

## Consequências

### O que ganhamos

- Todo o time consegue rodar e testar o app, independente do sistema operacional e do celular
- Ambiente simples: Node, o projeto e o Expo Go instalado no celular
- Alterações aparecem no celular ao salvar o arquivo
- Acesso a câmera, arquivos e localização pelas bibliotecas do Expo, sem código nativo
- Build do app na nuvem, sem precisar de Android Studio ou Xcode instalados

### O que aceitamos em troca

- O Expo Go só roda as bibliotecas que já vêm com ele; qualquer outra biblioteca nativa exige
  development build, e no iPhone físico isso ainda exige conta paga de desenvolvedor da Apple
- A escolha da biblioteca passa a depender de ela funcionar no Expo Go, o que pode descartar
  opções melhores
- O app sai só para Android nesta versão, mesmo o Expo servindo às duas plataformas
- Versões das bibliotecas presas ao SDK do Expo; atualizar o SDK exige atualizar as dependências
  juntas
- O plano gratuito do EAS Build tem fila e limite de builds
- A migração do CLI para o Expo gerou retrabalho, como a troca de `@react-native-vector-icons` por
  `@expo/vector-icons`

## Validação

Todo integrante consegue rodar o app no próprio celular a partir do próprio computador, toda
dependência foi instalada com `npx expo install`, e `npx expo-doctor` não aponta erros.

## Evidências

- `mobile/package.json` — dependência `expo` (SDK 56) e `@expo/vector-icons`
- `mobile/app.json` — configuração do app
- Histórico do repositório `mobile` — migração do React Native CLI para o Expo
