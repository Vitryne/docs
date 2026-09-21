---
id: ADR-032
titulo: Liderança por frente técnica, com um integrante coringa e papéis de processo acumulados
status: Aceita
data: 2026-09-20
autor: Hugo Zuin
decisores: Bruno Abrahim, Gabriel Rodrigues, Henrique Pacheco, Hugo Zuin, Igor Costa, João Ehlers e Leonardo Xavier
area: Governança
substitui: —
substituida-por: —
---

# ADR-032 — Liderança por frente técnica, com um integrante coringa e papéis de processo acumulados

## Contexto e problema

Sete pessoas, um semestre letivo e seis frentes de trabalho: back-end, front-end web, mobile,
design e protótipo, documentação e o próprio processo. Todo mundo estuda, parte trabalha, e a
disponibilidade é desigual ao longo da semana.

Duas coisas dependem de como o time se divide. A primeira é operacional: frente sem responsável
acumula pendência, porque ninguém se sente encarregado de destravá-la, e decisão técnica pequena
fica esperando reunião. A segunda é individual: o regulamento exige participação ativa de todos
com evidência objetiva, e a avaliação 360º pontua o membro melhor avaliado como 100% e reduz os
demais proporcionalmente. Contribuição difusa é contribuição que não aparece.

## Critérios de decisão

- Cada frente técnica com alguém que responde por ela e decide sem precisar de reunião
- Contribuição individual visível e atribuível — commit, PR, ata, card, ADR
- Capacidade de reforçar a frente que travar, sem desmontar outra
- Carga compatível com quem estuda e trabalha

## Alternativas consideradas

1. **Um líder por frente técnica, um integrante coringa para reforçar onde travar, e os papéis de
processo acumulados em uma pessoa** — escolhida.
2. **Sem liderança, com decisão sempre coletiva dentro da frente** — *perdeu porque:* transforma
   escolha pequena em reunião, e ninguém responde pelo atraso quando a frente empaca.
3. **Rotação total, com todos passando por todas as frentes** — *perdeu porque:* aprender quatro
   stacks em um semestre consumiria o tempo de entrega, e responsabilidade de todos é
   responsabilidade de ninguém.
4. **Alocação fixa para todos, sem coringa** — *perdeu porque:* frente que trava só se destrava
   tirando alguém de outra frente, o que empurra o atraso de lugar em vez de resolver.
5. **Scrum por livro, com Scrum Master e Product Owner em pessoas diferentes** — *perdeu porque:*
   separar os dois papéis tiraria mais uma pessoa da construção, e o time não tem folga.
6. **Um líder técnico único decidindo pelas três frentes** — *perdeu porque:* cria gargalo de
   decisão e apaga a evidência individual dos demais, que é o que a 360º mede.

## Decisão

Cada frente técnica tem um líder. A alocação atual:

| Frente | Líder | Demais integrantes | Apoio |
|---|---|---|---|
| Back-end | João Ehlers | Henrique Pacheco | Igor Costa |
| Front-end web | Henrique Pacheco | Gabriel Rodrigues | Igor Costa |
| Mobile | Leonardo Xavier | Bruno Abrahim | Hugo Zuin |
| Design System e protótipo | Hugo Zuin | — | Leonardo Xavier |
| Documentação e artefatos formais | Hugo Zuin | Igor Costa | — |
| Processo, Jira, Confluence e facilitação das cerimônias | Hugo Zuin | — | — |
| Configuração da organização e CI | Gabriel Rodrigues | — | — |
| Pesquisa de mercado | Igor Costa | — | Hugo Zuin |

Igor Costa atua mais como coringa: reforça onde houver gargalo.

Regras que acompanham a alocação:

- O líder responde pela frente: destrava, revisa o que entra e decide as escolhas técnicas dela
- Liderança não é exclusividade: qualquer pessoa pode abrir PR em qualquer repositório, mas a
  revisão passa pelo líder da frente
- A aprovação final continua sendo do grupo `@vitryne/admin`, conforme a ADR-003
- Mudança de alocação é registrada na página de papéis do Confluence, nunca combinada só de viva voz

## Consequências

### O que ganhamos

- Decisão técnica de frente sai sem reunião: o líder decide e registra
- Tarefa com responsável desde o início, e frente travada tem para quem recorrer
- Contribuição atribuível por repositório e por ADR, que é evidência direta na avaliação individual
- Reforço disponível sem desmontar outra frente

### O que aceitamos em troca

- O líder vira gargalo de revisão da própria frente, e ausência dele para o fluxo
- Acumular processo, design, protótipo e documentação em uma pessoa concentra risco: se ela parar,
  param quatro frentes ao mesmo tempo
- Design e protótipo com operador único é gargalo e ponto único de falha
- O coringa contribui em pedaços espalhados por várias frentes, o que dilui a evidência individual
  dele — sem registro cuidadoso no card, o trabalho não aparece
- Especialização reduz a capacidade de substituição: quem nunca tocou em uma frente não assume no
  meio da sprint
- Volume de commit não mede contribuição: quem trabalha em documentação, processo, design ou como
  coringa aparece menos no histórico do Git e pode ser subestimado na 360º — vale olhar ata, card e
  ADR junto

## Validação

Toda frente tem líder nomeado na página de papéis, nenhuma tarefa entra na sprint sem responsável
no Jira, toda ADR tem autor identificado no front matter, e o trabalho do coringa aparece em card
com responsável, não apenas no commit.

## Evidências

- Autoria dos commits por repositório: `backend` com João Ehlers e Henrique Pacheco Alves;
  `frontend` com Henrique Pacheco Alves e Gabriel Rodrigues; `mobile` com Bruno Abrahim e Leonardo
  Xavier; `docs` com Hugo Zuin; `.github` com Gabriel Rodrigues
- Autoria das ADRs: 017 e 020 a 025 por Leonardo Xavier, 005 por João Ehlers, 018 e 019 por Hugo
  Zuin
- `.github/CODEOWNERS` — `@vitryne/admin` como aprovador de todo o código
- Página de time, papéis e responsabilidades, no Confluence
- Regulamento Escola de TI 2026 — participação ativa de todos com evidência objetiva, e avaliação
  360º com pontuação proporcional ao membro melhor avaliado
