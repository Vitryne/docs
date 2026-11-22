---
id: ADR-047
titulo: Proteger main e dev por rulesets em todos os repositórios, com dev como branch padrão e bypass restrito aos administradores
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Gabriel Rodrigues
area: Governança
substitui: —
substituida-por: —
---

# ADR-047 — Proteger main e dev por rulesets em todos os repositórios, com dev como branch padrão e bypass restrito aos administradores

## Contexto e problema

A ADR-003 definiu o fluxo `main` para releases, `dev` para integração e uma branch por tarefa, com
merge commit, uma aprovação e `push --force` bloqueado. Uma auditoria dos repositórios em novembro
mostrou que a configuração não acompanhava o texto em quatro pontos:

- A branch padrão era a `main` nos três repositórios de código, embora a documentação dissesse
  `dev`. Houve PRs de tarefa abertos contra a `main` por engano no mobile
- A `main` do `.github`, de onde todos os repositórios consomem os workflows reutilizáveis, não
  exigia PR: uma alteração direta ali mudava o CI de todos os repositórios sem verificação
- A regra de uma aprovação existia nos rulesets, mas o bypass dos administradores era usado sem
  critério escrito: 20 de 57 PRs mergeados entre 20/10 e 17/11 não tinham review
- Ao trocar a branch padrão para `dev`, o ruleset da `main` do back-end, que tinha como alvo a opção
  "Default branch", passou a proteger a `dev` e deixou a `main` sem regra até ser corrigido

A ADR-003 também proíbe reescrever qualquer histórico já enviado, o que impede corrigir uma
mensagem de commit ou incluir um arquivo esquecido na própria branch antes do review.

A pergunta é qual configuração de proteção vale em cada repositório, quem pode contorná-la e em
que situação. Esta ADR complementa a ADR-003, que continua valendo no restante.

## Critérios de decisão

- A configuração efetiva precisa ser igual ao que está documentado
- Nenhuma mudança chega à `main` ou à `dev` sem PR e sem os checks obrigatórios
- Exceções existem, mas têm dono, situação e registro
- Corrigir a própria branch antes do review não pode colocar em risco o trabalho de outra pessoa

## Alternativas consideradas

1. **Rulesets por repositório com alvo explícito pelo nome da branch, `dev` como branch padrão,
   bypass só para administradores em PRs de CI e governança, e `--force-with-lease` permitido na
   branch de tarefa antes do review** — escolhida.
2. **Remover o bypass de todos** — *perdeu porque:* mudanças no próprio CI precisam ser validadas
   em PR antes de existir quem as aprove com contexto; sem bypass, o time ficaria travado justamente
   ao corrigir a esteira.
3. **Manter o bypass livre para administradores** — *perdeu porque:* a regra de aprovação deixa de
   valer na prática, e o histórico de PRs passa a contradizer a documentação.
4. **Manter a proibição total de reescrever histórico enviado (ADR-003)** — *perdeu porque:* a
   única forma de corrigir uma mensagem de commit ou um arquivo esquecido seria acumular commits de
   correção, sujando o histórico que o merge commit preserva.

## Decisão

**Branch padrão.** `dev` é a branch padrão do `frontend`, do `backend` e do `mobile`. O `docs` e o
`.github` não têm `dev` e usam a `main`.

**Rulesets.** Cada repositório de código tem os rulesets `protecao-dev` e `protecao-main`, com o
alvo definido pelo nome da branch (`refs/heads/dev` e `refs/heads/main`), nunca pela opção
"Default branch". As regras são:

- Exclusão da branch e force push bloqueados
- PR obrigatório, com 1 aprovação de code owner, aprovação descartada a cada novo push, aprovação
  exigida do último push e conversas resolvidas
- Apenas merge commit como método de merge
- Checks obrigatórios: `governanca / Governança` em todos; `qualidade / Qualidade` no front-end, no
  back-end e no mobile; `qualidade / Formatação` no front-end e no mobile

O `docs` tem as mesmas regras na `main`, com `governanca / Governança` como check. O `.github` exige
PR, o check `governanca / Governança` e apenas merge commit, com 0 aprovações, porque é mantido por
uma única pessoa.

**Bypass.** Apenas os administradores da organização (Gabriel Rodrigues e Hugo Zuin) podem ignorar
as regras, e somente em PRs de CI e governança. Quem usa o bypass comenta no PR o motivo.

**Force push.** Continua bloqueado em `main` e `dev`. Na própria branch de tarefa, é permitido com
`git push --force-with-lease`, e só enquanto o PR não recebeu review. Depois do review, correções
entram como commit novo.

## Consequências

### O que ganhamos

- A configuração de cada repositório corresponde ao que o CONTRIBUTING descreve
- PRs novos sugerem a `dev` como destino, e clones novos começam nela
- Nenhuma alteração no CI central entra sem PR e sem o check de governança
- Cada uso de bypass fica justificado no próprio PR

### O que aceitamos em troca

- O bypass continua tecnicamente disponível aos administradores; a restrição depende de disciplina
  e do registro no PR
- O `.github` não exige aprovação: uma mudança do mantenedor nos workflows centrais afeta todos os
  repositórios sem revisão de outra pessoa
- O `--force-with-lease` depende de a pessoa respeitar o limite "antes do review"; o GitHub não
  impede o force push em branches de tarefa
- A restrição de nome de branch por ruleset exige plano Enterprise; o padrão de nome segue validado
  apenas pelo check de governança

## Validação

A consulta `GET /repos/Vitryne/<repositório>/rules/branches/main` (e `/dev`) retorna as regras e os
checks descritos acima para cada repositório, e a branch padrão do `frontend`, do `backend` e do
`mobile` é `dev`.

## Evidências

- Rulesets `protecao-dev` e `protecao-main` do `frontend`, do `backend` e do `mobile`
- Ruleset da `main` do `.github` — PR obrigatório, 0 aprovações, check de governança
- `.github/CONTRIBUTING.md` — seções de bypass, force push e comandos com cuidado
- ETI-381 — configurações da organização no GitHub
- ADR-003 — processo de desenvolvimento, que esta ADR complementa
