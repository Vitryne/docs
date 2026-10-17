---
id: ADR-033
titulo: Agrupar em subpasta de src/components os componentes exclusivos de uma página
status: Proposta
data: 2026-10-15
autor: Henrique Pacheco
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: —
substituida-por: —
---

# ADR-033 — Agrupar em subpasta de src/components os componentes exclusivos de uma página

## Contexto e problema

A ADR-024 define que os componentes de interface do front-end ficam em `src/components`, mas não
diz como organizar essa pasta. Com o cadastro do lojista, `src/components` passou a misturar
componentes genéricos (`Button`, `Input`, `Checkbox`, `SelectUf`, `MaskedInput`, `ButtonGoogle`)
com componentes que só o cadastro usa (`SidebarRegister`, `Header` e uma `Step*` por etapa).
Quanto mais telas surgirem, mais difícil fica saber, olhando a pasta, o que é reutilizável e o que
pertence a uma tela. A pergunta em aberto é como separar os dois grupos.

## Critérios de decisão

- Distinguir de relance componente reutilizável de componente exclusivo de uma página
- Manter os componentes de uma mesma página juntos, para achar e apagar de uma vez
- Não criar rotas sem querer, já que o Pages Router transforma em rota todo arquivo de `src/pages`
- Custo baixo de adoção: sem nova camada de pastas nem mudança de imports nos componentes genéricos

## Alternativas consideradas

1. **Subpasta por página dentro de `src/components`** (`src/components/steps-register/`) —
   escolhida.
2. **Todos os componentes soltos na raiz de `src/components`** (estado anterior) — *perdeu
   porque:* a raiz mistura genéricos com exclusivos e cresce a cada tela nova; não há como saber,
   sem abrir o código, quem usa cada componente.
3. **Componentes ao lado da página, dentro de `src/pages`** — *perdeu porque:* no Pages Router
   cada arquivo de `src/pages` vira uma rota, então componentes ali exigiriam configuração extra
   para não serem publicados como página.

## Decisão

Dentro de `src/components`, a regra é:

- Componente usado em mais de um lugar fica solto na raiz de `src/components`.
- Componente usado por uma única página ou fluxo fica em uma subpasta de `src/components` com o
  nome dessa página ou fluxo, e todos os componentes que pertencem a ela ficam dentro da subpasta.
  Exemplo: `src/components/steps-register/` reúne `header.tsx`, `sidebar-register.tsx` e as
  etapas `step-identity.tsx`, `step-store.tsx`, `step-category.tsx`, `step-business-hours.tsx` e
  `step-bank-details.tsx`.
- Quando um componente de subpasta passar a ser necessário em outra página, ele sobe para a raiz
  de `src/components`.

Esta ADR complementa a ADR-024 e não a substitui. Os caminhos de `sidebar-register.tsx` citados
nas evidências da ADR-027 refletem a organização anterior à subpasta.

## Consequências

### O que ganhamos

- A raiz de `src/components` passa a conter só o que é reutilizável
- Tudo o que pertence a uma página está em um único lugar
- Remover ou refatorar uma página não deixa componentes órfãos espalhados

### O que aceitamos em troca

- Mover componente entre subpasta e raiz muda o caminho de import em quem o usa
- Decidir se um componente é "exclusivo" depende de julgamento: um componente pode nascer em uma
  subpasta e só ser reaproveitado meses depois, e a regra de subir para a raiz depende de alguém
  lembrar de aplicá-la
- Não há verificação automática da regra, ela depende de revisão de PR

## Validação

Todo componente que só uma página importa está em uma subpasta de `src/components`, e todo
componente da raiz de `src/components` é importado por mais de um lugar.

## Evidências

- `frontend/src/components/steps-register/` — `header.tsx`, `sidebar-register.tsx`,
  `step-bank-details.tsx`, `step-business-hours.tsx`, `step-category.tsx`, `step-identity.tsx`,
  `step-store.tsx`
- `frontend/src/components/` — `button.tsx`, `button-google.tsx`, `checkbox.tsx`, `input.tsx`,
  `masked-input.tsx`, `select-uf.tsx` na raiz
- ETI-171 Criar etapa de dados da loja no cadastro
- ETI-198 Criar sidebar da tela de registro
