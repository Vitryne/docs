---
id: ADR-032
titulo: Cerimônias do time e artefatos que cada uma produz
status: Aceita
data: 2026-09-20
autor: Hugo Zuin
decisores: Hugo Zuin
area: Governança
substitui: —
substituida-por: —
---

# ADR-032 — Cerimônias do time e artefatos que cada uma produz

## Contexto e problema

Sete pessoas que estudam e trabalham, com agenda comum limitada às aulas e ao fim de semana. Não
existe janela diária em que todos estejam disponíveis, e a grade da faculdade já mudou duas vezes
no semestre.

O regulamento exige que o time conduza review e retrospectiva a cada duas semanas, publique o
resultado das duas no Jira ou no Confluence e aponte as horas nas tarefas. A avaliação individual
depende de evidência de participação — quem esteve, quem faltou, quem entregou.

Sem regra escrita, o que foi combinado em reunião vira memória de quem estava, e a evidência que a
banca vai pedir não existe no dia em que for cobrada.

## Critérios de decisão

- Atender à exigência de review, retrospectiva e planning a cada duas semanas, com resultado publicado
- Caber na agenda real de sete pessoas, sem depender de encontro diário
- Produzir evidência de participação individual, incluindo ausências
- Registro que alguém de fora consiga achar e ler sem ter estado na reunião

## Alternativas consideradas

1. **Três cerimônias por sprint — Planning, Review e Retrospectiva — no encontro obrigatório de domingo, com ata publicada por cerimônia e alinhamentos curtos em aula sem obrigatoriedade de ata** — escolhida.
2. **Scrum completo, com daily** — *perdeu porque:* não há horário em que as sete pessoas estejam
   livres todos os dias, e uma daily que metade do time não assiste deixa de ser cerimônia e vira
   ruído.
3. **Kanban puro, com revisão de backlog quando necessário** — *perdeu porque:* o regulamento
   exige review e retrospectiva a cada duas semanas com resultado publicado, e cadência informal
   não gera a evidência.

## Decisão

Sprints de duas semanas, fechando no domingo.

| Cerimônia | Quando | Produz |
|---|---|---|
| Planning | Primeiro encontro da sprint | Meta, escopo selecionado, capacidade e riscos |
| Review | Encontro de fechamento | Itens demonstrados, aceites, feedback e o que não fechou |
| Retrospectiva | Encontro de fechamento | O que funcionou, o que não funcionou, ações e desfecho das ações anteriores |
| Extraordinária | Sob demanda | Motivo e encaminhamentos |

O encontro de domingo é obrigatório para todos. O domingo que fecha a sprint concentra Review,
Retrospectiva e a Planning da sprint seguinte.

Os alinhamentos de terça e quarta acontecem na primeira aula, servem para encaminhamento e ajuste
de rota, e não são cerimônias: só geram registro quando produzirem decisão ou artefato que precise
de rastreio.

Cada cerimônia gera ata publicada no Confluence, com
participantes e ausentes registrados. Reunião fora da cadência entra como Extraordinária. Quando
houver gravação, a transcrição entra como página irmã da ata.

As horas são apontadas no Jira no dia em que o trabalho aconteceu, não consolidadas no fim da
sprint.

## Consequências

### O que ganhamos

- Evidência pronta a cada sprint, sem precisar reconstruir depois
- Ausência registrada por nome, que é o que sustenta a avaliação individual
- Decisão localizável pelo nome da página, sem abrir a reunião inteira
- Um único encontro obrigatório por semana, compatível com a agenda de todos

### O que aceitamos em troca

- Sem daily, um impedimento pode ficar até uma semana sem aparecer para o time
- Escrever ata custa tempo de quem facilita, e hoje é sempre a mesma pessoa nas três cerimônias —
  ponto único de falha e de carga
- O calendário depende da grade da faculdade, que já mudou uma vez e pode mudar de novo
- Ata publicada com atraso perde a função: a evidência vale pela data em que existiu
- Concentrar três cerimônias em um domingo faz desse o encontro mais longo do ciclo, e cansaço no
  fim compromete justamente a retrospectiva

## Validação

Toda sprint encerrada tem ata de Planning, Review e Retrospectiva publicada, com participantes e
ausentes nomeados, e as horas da sprint aparecem apontadas nas tarefas do Jira.

## Evidências

- POL-002 Cerimônias e Calendário, no Confluence
- Atas `ATA-S07-PLAN`, `ATA-S07-REVIEW`, `ATA-S07-RETRO`, `ATA-S08-PLAN`, `ATA-S08-REVIEW`,
  `ATA-S08-RETRO` e `ATA-S09-PLAN`, com transcrições anexas
- Política de apontamento de horas, no Confluence
- Regulamento Escola de TI 2026, seção "Sobre o processo" — review e retrospectiva a cada duas
  semanas, artefatos publicados e horas apontadas no Jira
