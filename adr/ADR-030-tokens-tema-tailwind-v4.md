---
id: ADR-030
titulo: Tokens de tema no Tailwind CSS v4 via @theme no CSS global
status: Aceita
data: 2026-09-20
autor: Henrique Pacheco
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: —
substituida-por: —
---

# ADR-030 — Tokens de tema no Tailwind CSS v4 via @theme no CSS global

## Contexto e problema

O Tailwind CSS v4 muda a forma de configurar o tema: em vez de `tailwind.config.js/ts`, cores e
fontes são declaradas dentro do próprio CSS com a diretiva `@theme`. O projeto precisa de um lugar
único para as cores da marca e as fontes (Poppins, Bai Jamjuree) usadas em login e cadastro, com
os valores vindos do Design System (ADR-018).

## Critérios de decisão

- Um único ponto de verdade para cor e fonte
- Compatível com as classes utilitárias já usadas nos componentes (`bg-primary`,
  `text-text-secondary`)
- Alinhamento com a paleta e a tipografia publicadas no Design System (ADR-018)

## Alternativas consideradas

1. **Tokens declarados em `@theme` dentro de `src/styles/global.css`, sem
   `tailwind.config.ts`** — escolhida.
2. **`tailwind.config.ts` com `theme.extend.colors`, no padrão do Tailwind v3** — *perdeu porque:*
   o Tailwind v4 recomenda configuração em CSS e reduz a necessidade do arquivo de config; manter
   os dois criaria duas fontes de verdade para a mesma paleta.
3. **CSS Modules ou styled-components com variáveis próprias** — *perdeu porque:* todos os
   componentes existentes já usam classes utilitárias do Tailwind; introduzir outra abordagem de
   estilo no meio do desenvolvimento do cadastro duplicaria convenções.

## Decisão

Cores, fontes e demais tokens vivem no bloco `@theme` de `src/styles/global.css`
(`--color-primary`, `--color-text-primary`, `--font-display`, `--font-body` etc.), importado uma
única vez em `_app.tsx`. Os componentes consomem os tokens via classes utilitárias geradas
automaticamente pelo Tailwind (`bg-primary`, `font-display`).

## Consequências

### O que ganhamos

- Um arquivo único para ajustar toda a paleta e a tipografia
- Nomes de token (`primary`, `text-secondary`, `surface`) reaproveitáveis em qualquer componente
  sem prop de cor

### O que aceitamos em troca

- Os valores são copiados à mão do Design System (ADR-018): nada detecta se o `@theme` e o Figma
  divergirem, e a conferência depende de revisão
- Não há dark mode nem tema alternativo previsto na estrutura atual
- As fontes são carregadas via `@import` direto do Google Fonts no CSS, fora do mecanismo de
  otimização de fonte do Next.js (`next/font`)

## Validação

Nenhum componente declara cor literal (hex) fora de `src/styles/global.css`; toda cor nova de
marca entra primeiro no bloco `@theme`.

## Evidências

- `frontend/src/styles/global.css`
- `frontend/postcss.config.mjs` — `@tailwindcss/postcss`
- Ausência de `tailwind.config.ts`/`.js` no repositório
- ETI-170 Criar tela de Login e a base de componentes reutilizáveis
