---
id: ADR-042
titulo: Validar PRs do back-end com o workflow de governança reutilizável e dono de código por time
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Henrique Pacheco e João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-042 — Validar PRs do back-end com o workflow de governança reutilizável e dono de código por time

## Contexto e problema

A ADR-003 registrou que a integração contínua valida apenas o formato do título do PR, pelo
workflow `pr-title.yml`, e que o `CODEOWNERS` aponta todos os arquivos para `@vitryne/admin`. No
back-end, isso mudou em outubro: o `pr-title.yml` foi substituído pelo workflow de governança
(ETI-291), e o `CODEOWNERS` passou a apontar o time da área (ETI-295). O front-end passou pela
mesma mudança, registrada na ADR-034.

A pergunta é o que valida e quem revisa um PR do back-end hoje. Esta ADR complementa a ADR-003,
que continua valendo no restante: fluxo de branches, merge commit e definição de pronto.

O conteúdo do `governance.yml` fica no repositório `vitryne/.github` e não foi lido para esta
ADR.

## Critérios de decisão

- Reaproveitar a regra mantida no repositório central, sem copiá-la
- Verificação automática antes do merge
- Revisão distribuída entre o time da área e a administração

## Alternativas consideradas

1. **Workflow reutilizável `governance.yml` de `vitryne/.github`, mais `CODEOWNERS` com o time da
   área e a administração** — escolhida.
2. As demais alternativas não estão registradas para o back-end — confirmar com o time. Para o
   front-end, a ADR-034 descartou a regra própria de cada repositório porque duplicaria a
   validação central.

## Decisão

O `.github/workflows/ci.yml` do back-end chama `vitryne/.github/.github/workflows/governance.yml@main`
em PR para `dev` e `main`, nos eventos `opened`, `edited`, `synchronize` e `reopened`. O
`.github/CODEOWNERS` define `@Vitryne/backend` e `@Vitryne/admin` como donos de todo o código
(`*`).

O repositório não tem workflow de build, de teste automatizado nem de deploy. A suíte de testes
existe (testes unitários e de contexto Spring) e não roda no CI.

## Consequências

### O que ganhamos

- Validação de governança mantida em um lugar só
- Dois grupos de donos em vez de um

### O que aceitamos em troca

- O CI não compila o projeto nem roda testes: um PR que quebra o build ou um teste passa pela
  governança, e a proteção contra código quebrado é a revisão humana
- Mudar a regra depende do repositório `vitryne/.github`, e o workflow aponta para `@main`, então
  uma mudança lá afeta os PRs do back-end sem passar por revisão aqui
- Não há deploy automatizado no repositório; a plataforma de publicação do back-end segue sem
  decisão registrada (fila do README)

## Validação

Todo PR para `dev` ou `main` mostra o check do workflow `CI`, e a aprovação de um dos donos do
`CODEOWNERS` é exigida antes do merge.

## Evidências

- `backend/.github/workflows/ci.yml` — chamada a `governance.yml@main`
- `backend/.github/CODEOWNERS` — `* @Vitryne/backend @Vitryne/admin`
- ETI-291 — substitui a validação de título pelo workflow de governança
- ETI-295 — define o time da área como dono do código
- ADR-003 — processo de desenvolvimento, que esta ADR complementa
- ADR-034 — decisão equivalente no front-end
