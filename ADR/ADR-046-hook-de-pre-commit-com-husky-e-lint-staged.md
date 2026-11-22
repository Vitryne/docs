---
id: ADR-046
titulo: Formatar e corrigir o lint a cada commit com husky e lint-staged no front-end e no mobile
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Gabriel Rodrigues
area: Infraestrutura
substitui: —
substituida-por: —
---

# ADR-046 — Formatar e corrigir o lint a cada commit com husky e lint-staged no front-end e no mobile

## Contexto e problema

O check `qualidade / Formatação` é obrigatório no front-end (ADR-034) e passou a ser no mobile
(ADR-045). Na prática, o check falhava com frequência porque o código era enviado sem rodar o
`npm run format`: o PR ficava vermelho, a pessoa formatava, fazia um commit só de formatação e
esperava o CI de novo.

A pergunta é como garantir que o código já chegue formatado ao PR, sem depender de alguém lembrar
de rodar um comando.

## Critérios de decisão

- Nenhum passo manual a cada commit
- Atuar só nos arquivos que estão no commit, para não reformatar o projeto inteiro
- Não depender de configuração de editor de cada pessoa
- Manter o check do CI como garantia final

## Alternativas consideradas

1. **Hook de pre-commit com `husky` e `lint-staged`, instalado pelo `npm install`** — escolhida.
2. **Workflow que formata o código e faz commit na branch do PR** — *perdeu porque:* um commit
   feito com o `GITHUB_TOKEN` não dispara os workflows de novo, então os checks obrigatórios ficam
   pendentes e o PR trava. Contornar exigiria um token pessoal ou um GitHub App, e o histórico
   passaria a ter commits de bot.
3. **Formatar ao salvar, configurado no editor** — *perdeu porque:* depende do editor e da
   configuração de cada pessoa, e não há como garantir que todos usem.
4. **Remover o Prettier e o check de formatação** — *perdeu porque:* eliminaria a falha do CI ao
   custo de voltar a ter cada arquivo num estilo, com mudanças de espaço misturadas às de código
   nos diffs.

## Decisão

O front-end e o mobile têm `husky` e `lint-staged` em `devDependencies` e o script
`"prepare": "husky"`, que instala o hook a cada `npm install`. O `.husky/pre-commit` executa
`npx lint-staged`, configurado no `package.json`:

- Arquivos `.ts` e `.tsx` de `src/`: `eslint --fix` e depois `prettier --write`
- Arquivos `.css` de `src/` (só no front-end): `prettier --write`

Os arquivos entram no commit já corrigidos e formatados. Se sobrar algum erro de lint que o
`--fix` não resolve, o commit é bloqueado. Avisos não bloqueiam.

O check `qualidade / Formatação` continua obrigatório no CI.

## Consequências

### O que ganhamos

- O código chega formatado ao PR sem nenhum comando manual
- O check de formatação deixa de falhar por esquecimento
- Erros de lint aparecem no momento do commit, antes de chegar ao CI

### O que aceitamos em troca

- O hook só existe depois do `npm install`; quem não rodar o comando após clonar ou puxar a
  mudança fica sem ele
- O hook pode ser ignorado com `git commit --no-verify` e não roda em commits feitos pela interface
  do GitHub; o check do CI cobre esses casos
- Cada commit fica alguns segundos mais lento

## Validação

O arquivo `.husky/pre-commit` existe no front-end e no mobile, e um commit com um arquivo `.tsx`
fora da formatação entra no histórico já formatado.

## Evidências

- `frontend/.husky/pre-commit` e `mobile/.husky/pre-commit` — execução do `lint-staged`
- `frontend/package.json` e `mobile/package.json` — script `prepare` e bloco `lint-staged`
- ETI-386 — hook de pre-commit no front-end
- ETI-383 — hook de pre-commit no mobile
- ADR-034 e ADR-045 — checks de formatação no CI
