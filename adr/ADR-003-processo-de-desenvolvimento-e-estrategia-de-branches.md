---
id: ADR-003
titulo: Processo de desenvolvimento, estratégia de branches e definition of done
status: Aceita
data: 2026-09-13
autor: Equipe
decisores: Bruno Abrahim, Gabriel Rodrigues, Henrique Pacheco, Hugo Zuin, Igor Costa, João Ehlers e Leonardo Xavier
area: Governança
substitui: —
substituida-por: —
---

# ADR-003 — Processo de desenvolvimento, estratégia de branches e definition of done

## Contexto e problema

Sete pessoas, três aplicações, sprints de duas semanas e uma exigência explícita da disciplina:
o time define e justifica seu próprio ciclo de vida de desenvolvimento — fluxo de trabalho,
estratégia de branches, integração contínua e definition of done. A cobrança é sobre governar
o próprio processo, não sobre adotar um método específico.

O risco concreto de um time desse tamanho é trabalho parado na máquina de alguém, histórico
ilegível, e merge de véspera sem revisão.

## Critérios de decisão

- Rastreabilidade entre tarefa do Jira, branch, commit e pull request
- Ninguém perde trabalho, em nenhuma circunstância
- Revisão real, não carimbo
- Regra simples o bastante para ser seguida sem consultar documento

## Alternativas consideradas

1. **`main` para releases, `dev` para integração e uma branch por tarefa, com merge commit** —
   escolhida.
2. **Trunk-based development** — *perdeu porque:* exige feature flags e uma suíte de testes
   confiável para proteger a branch principal; o time não tem nem uma coisa nem outra, e a
   disciplina exige branch separada por tarefa.
3. **Git Flow completo, com `release/` e `hotfix/` permanentes** — *perdeu porque:* cerimônia
   demais para um projeto sem versionamento público; o custo de manter quatro tipos de branch
   não se paga em duas semanas de sprint.
4. **Squash merge** — *perdeu porque:* apaga os commits individuais, e o time optou por manter
   cada commit no histórico como evidência de participação. Squash e rebase foram desabilitados
   no GitHub para que a única opção disponível já seja a correta.

## Decisão

Fluxo: card no Jira → branch a partir de `dev` no formato `tipo/ETI-XXX-descricao` → commits
diários no formato `ETI-XXX tipo: descrição` → pull request com título igual ao nome da branch
→ uma aprovação de quem não é o autor → merge commit em `dev`. A `main` recebe apenas releases,
ao fim de cada sprint.

Regras que sustentam o fluxo:

- `push --force` bloqueado; reversão se faz com `git revert`
- Branch dura de dois a três dias e nunca atravessa a sprint
- Pelo menos um commit e um push por dia trabalhado
- Hotfix sai da `main` e volta para a `dev` em um segundo PR
- CODEOWNERS aponta todos os arquivos para `@vitryne/admin`
- Nenhum `.env`, credencial ou dado pessoal real entra no diff

Definition of done, conforme o template de PR: título no padrão, diff sem segredo, testado
localmente, documentação atualizada se o setup mudou, e card do Jira com status e horas em dia.

A integração contínua hoje valida apenas o formato do título do pull request. Build, teste e
lint automatizados estão na fila de decisões — ver `README.md`.

## Consequências

### O que ganhamos

- Rastreabilidade completa: da chave Jira até o commit, com vínculo automático
- Histórico permanente e atribuível a cada pessoa, que é a evidência pedida nas avaliações
- Nenhum caminho fácil para perder trabalho

### O que aceitamos em troca

- Histórico da `dev` mais barulhento do que seria com squash
- Mensagem de commit ruim aprovada em revisão fica permanente
- Revisão por uma pessoa só: se o revisor não conhece a área, o PR passa mesmo assim
- CI que não roda testes não protege a `dev` contra código quebrado — a proteção é humana

## Validação

Toda branch na origem segue o padrão `tipo/ETI-NNN-descricao`, todo PR tem card correspondente
no Jira, e o workflow de título falha em pull request fora do formato.

## Evidências

- `Vitryne/.github/CONTRIBUTING.md`
- `Vitryne/.github/.github/workflows/pr-title.yml`
- `Vitryne/.github/.github/pull_request_template.md`
- Confluence 2.4 Padrão de Git, Branches e Commits (POL-004) e 2.3 Padrão de Issues no Jira
- Material do professor: `note_sdlc_spec_driven.md` — gestão do SDLC obrigatória como decisão
