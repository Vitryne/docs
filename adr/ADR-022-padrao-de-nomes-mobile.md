---
id: ADR-022
titulo: Nomes em inglês, com PascalCase em pastas e componentes e camelCase em arquivos comuns
status: Proposta
data: 2026-09-19
autor: Leonardo Xavier
decisores: Leonardo Xavier e Bruno Valerio
area: Mobile
substitui: —
substituida-por: —
---

# ADR-022 — Nomes em inglês, com PascalCase em pastas e componentes e camelCase em arquivos comuns

## Contexto e problema

Com várias telas, componentes e hooks sendo criados em paralelo, cada um tendia a nomear do seu
jeito: misturando português e inglês, maiúscula e minúscula, com e sem prefixo. Isso dificulta
encontrar arquivos e deixa o código inconsistente.

Esta ADR define como pastas, arquivos e identificadores são nomeados. A organização das pastas
está na ADR-021, e o nome dos estilos, na ADR-020.

## Critérios de decisão

- Nome previsível: saber como chamar algo sem precisar perguntar
- Diferenciar pelo nome o que é pasta, componente, arquivo comum e estilo
- Seguir as convenções do React e do TypeScript sempre que possível
- Código em um idioma só

## Alternativas consideradas

1. **Código em inglês, pastas e componentes em PascalCase, arquivos comuns em camelCase e
   `index.tsx`/`styles.tsx` fixos** — escolhida.
2. **Código em português** — *perdeu porque:* mistura com os nomes em inglês do React, do React
   Native e das bibliotecas, gerando nomes híbridos.
3. **Tudo em kebab-case (`primary-button/`, `common-styles.ts`)** — *perdeu porque:* foge do padrão
   do ecossistema React, em que componentes usam PascalCase.
4. **Arquivo com o nome do componente (`PrimaryButton.tsx`, `PrimaryButton.styles.tsx`)** —
   *perdeu porque:* o time preferiu a pasta com `index.tsx`, que deixa o import mais curto.

## Decisão

- **Idioma:** pastas, arquivos, funções, variáveis e tipos em inglês. Texto que aparece para o
  usuário em português do Brasil
- **Pastas:** PascalCase — `Customer`, `Delivery`, `Components`, `PrimaryButton`, `StepIndicator`
- **Telas e componentes:** o nome fica na pasta; dentro dela, sempre `index.tsx` e `styles.tsx`
- **Telas:** nome descritivo igual ao nome da rota, sem numeração de etapa —
  `RegisterPersonalData`, `RegisterDocuments`, `RegisterVehicle`, `RegisterBankAccount`
- **Arquivos comuns (sem componente):** camelCase — `commonStyles.ts`, `navigation.ts`,
  `apiConfig.ts`
- **Hooks:** camelCase com prefixo `use`, um por tela — por exemplo, `useVehicleForm.ts`
- **Componentes e tipos:** PascalCase — `PrimaryButton`, `DeliveryStackParamList`
- **Navegação:** componente de rotas `<Domínio>Routes` e tipos `<Domínio>StackParamList` —
  `DeliveryRoutes`, `DeliveryStackParamList`
- **Funções e variáveis:** camelCase
- **Estilos:** minúsculo com `_`, conforme a ADR-020
- **Extensão:** `.tsx` para arquivos com JSX e para `styles.tsx`; `.ts` para o resto

## Consequências

### O que ganhamos

- Pelo formato do nome dá para saber o que é: pasta ou componente (PascalCase), arquivo comum ou
  hook (camelCase), estilo (snake_case)
- Nomes alinhados com o React e as bibliotecas usadas
- Menos dúvida e menos discussão na hora de criar arquivos

### O que aceitamos em troca

- Três padrões convivendo (PascalCase, camelCase e snake_case), que precisam ser aprendidos por
  quem entra no projeto
- Código em inglês e interface em português exigem atenção para não misturar
- `styles.tsx` usa `.tsx` mesmo sem JSX, diferente de `commonStyles.ts`
- Arquivos que já estão fora do padrão precisam ser renomeados

## Validação

Nenhuma pasta fora de PascalCase, nenhum hook sem prefixo `use`, nenhum identificador em português
e nenhuma pasta de tela ou componente sem `index.tsx` e `styles.tsx`.

## Evidências

- `mobile/src/Delivery/Screens/Register/Components/` — `PrimaryButton`, `RegisterHeader`,
  `StepBadge`, `StepIndicator`
- `mobile/src/Delivery/Navigation/` — `DeliveryRoutes`
- `mobile/src/Delivery/Types/navigation.ts` — `DeliveryStackParamList`
- `mobile/src/Shared/Styles/commonStyles.ts`
- `mobile/src/Delivery/Screens/Register/Screens/` — `RegisterPersonalData`,
  `RegisterDocuments`, `RegisterVehicle` e `RegisterBankAccount`
