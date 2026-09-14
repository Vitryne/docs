---
id: ADR-001
titulo: Registrar decisões arquiteturais como ADRs versionadas no repositório
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: Equipe
area: Governança
substitui: —
substituida-por: —
---

# ADR-001 — Registrar decisões arquiteturais como ADRs versionadas no repositório

## Contexto e problema

O projeto acumulou decisões técnicas sem registro formal: escolhas de stack, padrões de
código, estratégia de branches e convenções de banco. Quem entra depois não encontra o
porquê, e quem decidiu já não lembra das alternativas descartadas.

A disciplina exige a consolidação de ADRs como artefato da fase Pré-código, e a Avaliação 05
pergunta diretamente sobre decisões registradas — qual é a mais cara de reverter, por que a
alternativa rejeitada perdeu. Sem registro, não há como responder apontando artefato.

O time já mantém documentações de governança no Confluence e documentações técnicas no Github e suas versões editáveis no Drive. A pergunta é onde o registro de decisão deve viver e em que formato.

## Critérios de decisão

- Uma única fonte da verdade, sem cópia que divirja
- Registro versionado junto com o que ele descreve
- Formato simples o bastante para ninguém deixar de escrever por preguiça
- Rastreável: dá para ver quando a decisão mudou e quem mudou

## Alternativas consideradas

1. **Arquivos Markdown em `docs/adr/` no repositório `Vitryne/docs`** — escolhida.
2. **Páginas do Confluence, uma por decisão** — *perdeu porque:* o registro ficaria separado do
   código que ele governa, sem diff nem histórico legível, e o Confluence já concentra
   processo e artefatos formais. Duas fontes da verdade divergem com o tempo.
3. **Seção de arquitetura no README de cada repositório** — *perdeu porque:* mistura decisão
   com instrução de uso, não comporta histórico de decisões substituídas e duplicaria o
   conteúdo em quatro repositórios.
4. **Não registrar e confiar no histórico do Git** — *perdeu porque:* commit registra o que
   mudou, não o que foi descartado nem em troca de quê.

## Decisão

As ADRs vivem em `docs/adr/` no repositório `Vitryne/docs`, um arquivo Markdown por decisão,
no formato MADR adaptado para português. A página de decisões arquiteturais no Confluence é
apenas o ponteiro para a pasta.

Convenções:

- ID sequencial `ADR-NNN`, nunca reaproveitado
- Status: Proposta · Aceita · Substituída · Descartada
- ADR publicada não se edita. Decisão revista entra como ADR nova e a anterior vira Substituída
- Autoria identificada no front matter
- O `README.md` da pasta mantém o índice e a fila de decisões ainda não registradas
- `ADR-000-modelo.md` é o ponto de partida de toda ADR nova: não é decisão e não entra no índice

## Consequências

### O que ganhamos

- Resposta com artefato na mão quando o professor perguntar por uma decisão
- Contexto preservado para quem chegar depois, inclusive alternativas descartadas
- Decisão e código mudam no mesmo pull request, sob a mesma revisão

### O que aceitamos em troca

- Custo de escrever: toda decisão relevante passa a ter um arquivo a mais no PR
- Risco de a pasta envelhecer se ninguém cobrar o registro na revisão
- Registro imutável significa conviver com ADRs Substituídas no diretório, e não apagá-las

## Validação

A pasta `docs/adr/` existe com índice atualizado, o link está preenchido na página que aponta
para ela, e qualquer decisão citada em apresentação tem ADR correspondente.

## Evidências

- ETI-200 — Consolidar ADRs no repositório de documentação, com as convenções de ID, status e
  seções
- Material do professor: `note_bibliography.md` (MADR, Nygard, Joel Parker Henderson) e
  `evaluation_01_artifacts.md` (Pré-código — Consolidação de ADRs)
