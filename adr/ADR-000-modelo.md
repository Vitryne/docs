---
id: ADR-NNN
titulo: « frase curta que diz a decisão, não o assunto — "Adotar X para Y", não "Sobre X" »
status: Proposta
data: AAAA-MM-DD
autor: « quem escreveu a ADR »
decisores: « quem decidiu — nomes, não "equipe" »
area: « Governança · Back-end · Front-end · Mobile · Infraestrutura · Produto e UI »
substitui: —
substituida-por: —
---

<!--
MODELO — não é uma decisão e não entra no índice do README.

Como usar:
1. Duplique este arquivo e renomeie para ADR-NNN-titulo-curto.md
   (minúsculas, sem acento, hífen como separador)
2. NNN é o próximo ID livre, indicado no final do índice do README
3. Substitua tudo que está entre « » e apague os comentários conforme escrever
4. Acrescente a linha da ADR no índice do README e atualize o próximo ID livre
5. Abra o pull request junto com a mudança que a decisão governa

O que não vai numa ADR: como o sistema funciona, tutorial de uso, passo a passo de
implementação. ADR registra a decisão, o que foi descartado e o que se aceitou em troca.

Preencha `substitui` quando esta ADR tomar o lugar de outra, e volte na antiga para mudar o
status para Substituída e preencher `substituida-por`. ADR publicada não se edita.
-->

# ADR-NNN — « mesmo título do front matter »

## Contexto e problema

« O que está acontecendo no projeto que obriga alguém a decidir. Fatos verificáveis — número de
telas, tamanho do time, exigência do regulamento, comportamento do código hoje — e não opinião.
Termine deixando claro qual é a pergunta em aberto.

Dois ou três parágrafos curtos bastam. Quem ler daqui a seis meses precisa entender o problema
sem conhecer a conversa que gerou a ADR. »

## Critérios de decisão

« O que fez uma alternativa ganhar da outra. São estes critérios que a seção seguinte usa para
justificar cada descarte — se um critério não aparece em nenhum "perdeu porque", ele provavelmente
não era critério. »

- « critério »
- « critério »
- « critério »

## Alternativas consideradas

« Liste a escolhida em primeiro e as descartadas na sequência, com o motivo de cada descarte. Não
invente alternativa de enfeite: se só havia duas opções reais, liste duas.

O motivo do descarte é a parte mais cobrada — escreva o que a alternativa custava ou não
entregava, não apenas que ela é pior. Se uma alternativa descartada continua sendo forte, diga
isso e registre em que condição ela deve ser reaberta. »

1. **« alternativa escolhida »** — escolhida.
2. **« alternativa descartada »** — *perdeu porque:* « o que custava ou não entregava ».
3. **« alternativa descartada »** — *perdeu porque:* « o que custava ou não entregava ».

## Decisão

« O que fica valendo, no presente e de forma afirmativa: "o backend usa X", não "vamos avaliar
X". Inclua as convenções que a decisão cria e que alguém precisa seguir amanhã.

Se a decisão ainda não foi implementada por inteiro, diga o que falta aqui mesmo, em vez de
deixar o leitor supor que está tudo feito. »

## Consequências

### O que ganhamos

- « efeito concreto, não adjetivo »
- « efeito concreto, não adjetivo »

### O que aceitamos em troca

« A seção que dá valor à ADR. Custo, risco, limitação e porta que se fecha. Uma ADR sem nada aqui
ou está incompleta ou está vendendo a decisão.

Quando a decisão tem prazo ou gatilho de revisão — "isso precisa mudar antes do deploy" — o lugar
é aqui. »

- « custo, risco ou limitação »
- « custo, risco ou limitação »

## Validação

« Como alguém verifica, olhando o projeto, que a decisão está sendo seguida. Uma ou duas frases
checáveis: um comando que roda, uma busca que não retorna nada, um arquivo que existe. Não repita
a decisão em outras palavras. »

## Evidências

« Onde a decisão aparece no projeto. Caminho de arquivo, chave do Jira, arquivo do Figma, item do
regulamento. Se não houver nenhuma evidência, a decisão talvez ainda não tenha sido tomada — e o
lugar dela é a fila do README, não uma ADR. »

- « `caminho/do/arquivo` — o que ele mostra »
- « ETI-NNN — título da issue »
