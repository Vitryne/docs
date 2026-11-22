---
id: ADR-026
titulo: Turbopack como bundler de desenvolvimento e build
status: Aceita
data: 2026-09-20
autor: Henrique Pacheco
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: —
substituida-por: —
---

# ADR-026 — Turbopack como bundler de desenvolvimento e build

## Contexto e problema

O Next.js 15 oferece o Turbopack como alternativa ao Webpack para os comandos de desenvolvimento e
build. O projeto ainda tem poucas páginas, mas o cadastro do lojista e o restante do portal do
parceiro devem crescer nas próximas etapas.

## Critérios de decisão

- Tempo de reload no ambiente de desenvolvimento
- Compatibilidade com os pacotes já usados (Axios, react-imask, react-icons, Tailwind v4)

## Alternativas consideradas

1. **Turbopack em dev e build (`next dev --turbopack`, `next build --turbopack`)** — escolhida.
2. **Webpack, bundler padrão anterior do Next.js** — *perdeu porque:* o Turbopack já é estável
   para o Next 15 e reduz o tempo de refresh durante o desenvolvimento das telas de cadastro, que
   têm bastante Tailwind e SVG.

## Decisão

Os scripts `dev` e `build` do `package.json` usam a flag `--turbopack`. Nenhuma configuração
adicional de bundler foi criada.

## Consequências

### O que ganhamos

- Build e ambiente de desenvolvimento mais rápidos, sem configuração extra

### O que aceitamos em troca

- O Turbopack ainda tem paridade parcial de plugins com o ecossistema Webpack; se algum pacote
  futuro exigir loader ou plugin específico de Webpack, pode ser necessário reverter o build dessa
  página ou funcionalidade

## Validação

`npm run dev` e `npm run build` rodam com a flag `--turbopack` já configurada nos scripts, sem
necessidade de flags adicionais.

## Evidências

- `frontend/package.json` — scripts `dev` e `build` com `--turbopack`
- `next` 15.5.18
