---
id: ADR-027
titulo: Convenção de PR e ownership compartilhados com o ecossistema Vitryne
status: Aceita
data: 2026-09-20
autor: Henrique Pacheco
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: —
substituida-por: —
---

# ADR-027 — Convenção de PR e ownership compartilhados com o ecossistema Vitryne

## Contexto e problema

O frontend faz parte de um ecossistema com repositórios de backend, mobile e documentação
(Vitryne). Para manter rastreabilidade das tarefas do Jira, os PRs precisam seguir um padrão de
título, e o repositório precisa de um dono definido para aprovação.

Esta ADR é um recorte da ADR-003, que define o processo de desenvolvimento de toda a organização:
aqui fica registrado apenas como o repositório do frontend aplica essa regra.

## Critérios de decisão

- Reaproveitar convenções já usadas nos outros repositórios do ecossistema
- Não exigir manutenção duplicada da mesma regra de commit/PR em cada repositório

## Alternativas consideradas

1. **Workflow reutilizável `vitryne/.github/.github/workflows/pr-title.yml@main`, disparado em
   `pull_request`, mais `CODEOWNERS` apontando `@vitryne/admin` para todo o repositório** —
   escolhida.
2. **Regra de commit/PR própria do frontend (ex.: commitlint local)** — *perdeu porque:*
   duplicaria a mesma validação já mantida centralmente para os outros repositórios do
   ecossistema, com risco de divergir com o tempo.
3. **Sem validação automatizada de título de PR, revisão só manual** — *perdeu porque:* o time já
   usa o padrão `ETI-XXX` nos nomes de branch e nos commits, e sem checagem automática esse padrão
   passa a depender só de disciplina manual.

## Decisão

O repositório consome o workflow reutilizável de validação de título de PR do repositório
`vitryne/.github`, disparado em `opened`, `edited`, `synchronize` e `reopened`. O `CODEOWNERS`
define `@vitryne/admin` como dono de todo o código (`*`), exigindo essa revisão em qualquer PR.

## Consequências

### O que ganhamos

- Título de PR padronizado e rastreável ao card do Jira (`ETI-XXX`) sem manter regra local
- Ponto único de aprovação de mudanças no repositório

### O que aceitamos em troca

- Qualquer ajuste na regra de título de PR depende do repositório central `vitryne/.github`, fora
  do controle direto do time de frontend
- Com um único grupo de owners (`@vitryne/admin`), a revisão pode virar gargalo se esse grupo não
  escalar junto com o time

## Validação

PRs abertos no repositório mostram o check do workflow `pr-title.yml` e exigem aprovação de
`@vitryne/admin` antes do merge.

## Evidências

- `frontend/.github/workflows/pr-title.yml`
- `frontend/.github/CODEOWNERS`
- Histórico de commits e branches no padrão `ETI-XXX` (ex.: ETI-206, ETI-171, ETI-170, ETI-198)
