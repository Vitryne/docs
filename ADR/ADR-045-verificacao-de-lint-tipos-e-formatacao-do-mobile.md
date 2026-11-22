---
id: ADR-045
titulo: Verificar lint, tipos e formatação de todo PR do mobile, com as regras do React Compiler como aviso
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Gabriel Rodrigues e Leonardo Xavier
area: Mobile
substitui: —
substituida-por: —
---

# ADR-045 — Verificar lint, tipos e formatação de todo PR do mobile, com as regras do React Compiler como aviso

## Contexto e problema

Até novembro, o mobile validava apenas a governança do PR. O `package.json` tinha o script
`lint` chamando `expo lint`, mas não havia configuração de ESLint, script de verificação de tipos,
formatador nem `.nvmrc`. O front-end, também em TypeScript, já tinha lint, tipos e formatação
obrigatórios em todo PR (ADR-034).

Ao configurar as ferramentas, o código atual apresentou 14 erros de lint e 25 erros de tipo em 14
arquivos. Os 14 erros de lint vinham das regras do React Compiler (`react-hooks/refs`,
`react-hooks/purity` e `react-hooks/set-state-in-effect`), trazidas pelo `eslint-config-expo` e
relevantes porque o `app.json` ativa o compilador (`reactCompiler: true`). Eles se concentravam em
4 arquivos, 10 deles no hook de animação `useRevealOnScrollUp`.

A pergunta é qual padrão de qualidade o mobile adota e como colocá-lo no CI sem reescrever código de interface.

## Critérios de decisão

- Mesmo padrão do front-end: lint, tipos e formatação obrigatórios em todo PR
- Usar a configuração oficial do Expo, sem regras próprias
- Não alterar o comportamento das telas para fazer o CI passar
- Reaproveitar o `node-quality.yml` sem mudar o workflow central

## Alternativas consideradas

1. **ESLint com `eslint-config-expo`, as três regras do React Compiler rebaixadas para aviso,
   verificação de tipos com `tsc --noEmit`, Prettier igual ao do front-end e correção dos erros de
   tipo** — escolhida.
2. **Manter as regras do React Compiler como erro e corrigir os 4 arquivos antes de ligar o CI** —
   *perdeu porque:* exigiria reescrever o hook de animação e telas do fluxo de cadastro e pagamento
   às vésperas da apresentação, com risco de regressão visual. Os componentes funcionam; o
   compilador só deixa de otimizá-los. Deve ser reaberta depois da apresentação.
3. **Mobile sem Prettier, com uma opção no `node-quality.yml` para desligar o job de formatação** —
   *perdeu porque:* exigiria mudar o workflow central e deixaria o mobile com padrão mais fraco que
   o front-end. Com o hook de pre-commit (ADR-046), formatar não custa nada a quem desenvolve.
4. **Deixar a verificação de tipos fora do CI** — *perdeu porque:* o `node-quality.yml` sempre
   roda o typecheck, e 23 dos 25 erros eram só de declaração de tipo, sem mudança de comportamento.

## Decisão

O mobile adota:

- `.nvmrc` com Node 24, a mesma versão do front-end
- ESLint com `eslint-config-expo` em `eslint.config.js`, com `react-hooks/refs`,
  `react-hooks/purity` e `react-hooks/set-state-in-effect` como `warn`
- Script `typecheck` (`tsc --noEmit`)
- Prettier `3.9.10` com versão fixa e `tabWidth: 2`, scripts `format` e `format:check`
- `eslint-config-expo`, `prettier` e as demais ferramentas em `devDependencies`

O workflow `.github/workflows/quality.yml` chama `node-quality.yml@main` com `build: false`, porque
o build do aplicativo é feito pelo Expo, fora do GitHub. Os checks `qualidade / Qualidade` (lint e
tipos) e `qualidade / Formatação` são obrigatórios nos rulesets de `dev` e `main`, junto com
`governanca / Governança`. O `ci.yml` de governança ganhou `permissions` e `concurrency`.

Para o código passar nas verificações, foram feitas correções de tipo: declaração de módulo para
`.svg`, `ref` no `FormInput` do entregador, tipos da pilha de navegação e imports de tipos que
faltavam. Três mudanças tocam a interface ou o fluxo:

- `fontWeight: "470"` (valor inválido) passou a `"500"` no botão do Google, em 3 telas
- A prop `icon` foi removida de dois usos do `PrimaryButton` do cliente, que nunca a exibiu
- No fim do cadastro do entregador, a navegação para `RegisterSuccess`, uma tela que não existe,
  passou a levar ao `Login`

## Consequências

### O que ganhamos

- Nenhum PR entra no mobile com erro de lint, erro de tipo ou fora da formatação
- Front-end e mobile seguem o mesmo padrão de qualidade e as mesmas ferramentas
- Os pontos que impedem o React Compiler de otimizar componentes ficam visíveis como aviso

### O que aceitamos em troca

- 17 avisos de lint permanecem (14 do React Compiler e 3 que já existiam), e os componentes desses
  avisos não são otimizados pelo compilador
- O destino do fim do cadastro do entregador (`Login`) precisa ser confirmado pelo time
- A primeira formatação alterou 75 arquivos de `src/`, o que torna o histórico desses arquivos mais
  difícil de ler naquele commit
- Não há teste automatizado no mobile

## Validação

No mobile, `npm run lint` termina com 0 erros, `npm run typecheck` termina sem erros e todo PR para
`dev` ou `main` mostra os checks `qualidade / Qualidade` e `qualidade / Formatação`.

## Evidências

- `mobile/eslint.config.js` — configuração do Expo e regras do React Compiler como aviso
- `mobile/package.json` — scripts `lint`, `typecheck`, `format` e `format:check`
- `mobile/.prettierrc` e `mobile/.nvmrc`
- `mobile/.github/workflows/quality.yml` — chamada ao `node-quality.yml@main` com `build: false`
- `mobile/src/Shared/Types/svg.d.ts` — declaração de módulo para `.svg`
- ETI-383 — qualidade e CI do mobile
- ADR-034 — padrão equivalente no front-end
