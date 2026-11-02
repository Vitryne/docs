---
id: ADR-034
titulo: Validar PRs do front-end com workflows reutilizáveis de governança e qualidade e dono de código por time
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: ADR-025
substituida-por: —
---

# ADR-034 — Validar PRs do front-end com workflows reutilizáveis de governança e qualidade e dono de código por time

## Contexto e problema

A ADR-025 registrou que o front-end validava o título do PR com o workflow reutilizável
`pr-title.yml`, do repositório `vitryne/.github`, e que o `CODEOWNERS` apontava `@vitryne/admin`
como dono de todo o código. Esse arranjo mudou em outubro, em três tickets:

- ETI-291: o `pr-title.yml` foi substituído pelo workflow reutilizável `governance.yml`
- ETI-292: entrou um workflow de qualidade, que chama o `node-quality.yml`
- ETI-295: o `CODEOWNERS` passou a apontar o time da área, além do grupo de administração

Hoje não existe `pr-title.yml` no repositório, e o texto da ADR-025 já não descreve nem o que
valida o PR nem quem o revisa. A pergunta é como o front-end valida e revisa cada mudança, agora
que há mais de uma verificação e mais de um dono.

Esta ADR substitui a ADR-025. O conteúdo do `governance.yml` e do `node-quality.yml` fica no
repositório `vitryne/.github` e não foi lido para esta ADR.

## Critérios de decisão

- Reaproveitar a regra mantida no repositório central, sem copiar a regra em cada repositório
- Verificação automática antes do merge, sem depender de disciplina manual
- Revisão distribuída entre o time da área e a administração

## Alternativas consideradas

1. **Workflows reutilizáveis de `vitryne/.github` (`governance.yml` e `node-quality.yml`),
   disparados em PR, e `CODEOWNERS` com o time da área e a administração** — escolhida.
2. **Regra própria do front-end (ex.: commitlint e job de lint escritos no repositório)** —
   *perdeu porque:* duplicaria a validação que o repositório central já mantém para os demais
   repositórios do ecossistema, com risco de divergir com o tempo. É o mesmo motivo que a
   ADR-025 registrou para descartar a regra local.
3. **Manter só a validação de título da ADR-025, com `@vitryne/admin` como único dono** —
   *perdeu porque:* já foi substituída no código; a ADR-025 apontava o grupo único de owners como
   possível gargalo de revisão.

## Decisão

O repositório consome dois workflows reutilizáveis de `vitryne/.github`, ambos na `main` do
repositório central:

- **Governança** (`.github/workflows/ci.yml`), que chama `governance.yml@main`, em PR para `dev`
  e `main`, nos eventos `opened`, `edited`, `synchronize` e `reopened`.
- **Qualidade** (`.github/workflows/quality.yml`), que chama `node-quality.yml@main`, em PR para
  `dev` e `main`, nos eventos `opened`, `synchronize` e `reopened`, com `concurrency` por número
  de PR cancelando a execução anterior.

O `CODEOWNERS` define `@Vitryne/frontend` e `@Vitryne/admin` como donos de todo o código (`*`).

Os scripts `lint`, `typecheck` e `format:check` existem no `package.json`. Esta ADR não registra
quais deles o `node-quality.yml` executa, porque isso é definido no repositório central.

## Consequências

### O que ganhamos

- Validação de PR mantida em um lugar só, sem regra duplicada no front-end
- Verificação de governança e de qualidade a cada push no PR
- Dois grupos de donos em vez de um, o que reduz o risco de gargalo apontado na ADR-025

### O que aceitamos em troca

- Mudar a regra de governança ou de qualidade depende do repositório `vitryne/.github`, fora do
  controle direto do time de front-end
- Os workflows apontam para `@main` do repositório central: uma mudança lá afeta os PRs do
  front-end sem passar por revisão neste repositório
- Não há teste automatizado no front-end, então a verificação de qualidade não cobre
  comportamento

## Validação

Todo PR para `dev` ou `main` mostra os checks dos workflows `CI` e `Qualidade`, e a aprovação de
um dos donos do `CODEOWNERS` é exigida antes do merge.

## Evidências

- `frontend/.github/workflows/ci.yml` — chamada a `governance.yml@main`
- `frontend/.github/workflows/quality.yml` — chamada a `node-quality.yml@main`
- `frontend/.github/CODEOWNERS` — `* @Vitryne/frontend @Vitryne/admin`
- `frontend/package.json` — scripts `lint`, `typecheck` e `format:check`
- ETI-291 — substitui a validação de título pelo workflow de governança
- ETI-292 — adiciona o workflow de qualidade ao front-end
- ETI-295 — define o time da área como dono do código
