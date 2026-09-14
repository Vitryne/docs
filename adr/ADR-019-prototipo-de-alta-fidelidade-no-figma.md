---
id: ADR-019
titulo: Protótipo no Figma como fonte da verdade da interface para o desenvolvimento
status: Proposta
data: 2026-09-13
autor: Hugo Zuin
decisores: Hugo Zuin
area: Produto e UI
substitui: —
substituida-por: —
---

# ADR-019 — Protótipo no Figma como fonte da verdade da interface para o desenvolvimento

## Contexto e problema

O protótipo de alta fidelidade acumula dois papéis. É artefato exigido, com a disciplina cobrando
correspondência entre a tela apresentada e a tela entregue. E é o que o desenvolvimento lê para
implementar: quem escreve a tela em Next.js ou em React Native abre o protótipo, não o Design
System.

A primeira versão do protótipo foi um arquivo HTML navegável, publicado no Drive e ainda
referenciado pelo `README.md` do repositório `docs`.

Esse formato tem um problema estrutural: o protótipo em HTML e o Design System no Figma evoluem
separados. Componente alterado no Figma não chega ao HTML, e tela ajustada no HTML não volta para
o sistema. Com duas fontes, a pergunta "qual é a tela certa?" não tem resposta.

## Critérios de decisão

- Uma referência só para o desenvolvimento, sem ambiguidade sobre qual tela vale
- Alteração de componente refletida em todas as telas que o usam
- Protótipo navegável, para validar fluxo antes de codificar
- Ferramenta em que quem desenha já trabalha

## Alternativas consideradas

1. **Protótipo no Figma, construído sobre os componentes do Design System, com o HTML anterior
   preservado apenas como registro histórico** — escolhida.
2. **Manter o protótipo em HTML como fonte** — *perdeu porque:* ele não consome o Design System,
   e quem mantém tela em HTML está escrevendo código que será jogado fora. Em troca, abre-se mão
   de um protótipo que roda no navegador sem conta no Figma.
3. **Manter os dois em sincronia** — *perdeu porque:* dobra o trabalho a cada ajuste e a
   divergência aparece na primeira semana corrida.
4. **Pular o protótipo e ir direto para o código** — *perdeu porque:* o artefato é exigido, e
   validar fluxo em código custa muito mais caro do que validar em tela desenhada.

## Decisão

O protótipo de alta fidelidade vive no Figma, no arquivo `Vitryne — Design System & Portal do
Parceiro`, montado a partir dos componentes da biblioteca da ADR-018. Tela nova parte de componente
existente; quando o componente não existe, ele nasce no Design System antes da tela.

O protótipo é a fonte da verdade da interface para o desenvolvimento. Tela a implementar se lê
nele, não no Design System — o Design System é o insumo que o monta.

O arquivo `Arquivo original do projeto` e o HTML no Drive ficam como registro do que veio antes,
sem manutenção. O `README.md` do repositório `docs` precisa deixar de apontar para o HTML como
protótipo corrente.

A migração está em andamento: protótipo web na ETI-202, protótipo mobile na ETI-167. Enquanto ela
não fecha, parte das telas — mobile, portal do parceiro e entregador — ainda não existe no Figma.

## Consequências

### O que ganhamos

- Uma referência só para quem implementa, e ajuste no Design System propaga para as telas
- Protótipo navegável para validar fluxo antes de escrever código
- Índice de telas ligado a caso de uso, que é a evidência pedida na avaliação

### O que aceitamos em troca

- Perde-se o protótipo que abria no navegador sem conta: quem avalia precisa acessar o Figma
- A migração consome tempo em sprints que também têm entrega de tela
- Como o protótipo é a referência do desenvolvimento, tela que ainda não existe ou está
  desatualizada no Figma vira implementação parada ou divergente
- Enquanto as duas fontes coexistem, é preciso deixar explícito qual está valendo — e hoje o
  `README.md` do `docs` aponta para a antiga
- Não existe ainda um índice que relacione cada mockup ao seu caso de uso

## Validação

O `README.md` do `docs` aponta para o Figma como protótipo corrente, e existe um índice
relacionando cada mockup ao seu caso de uso.

## Evidências

- [Arquivo Figma](https://www.figma.com/design/36njOKWxqVbjBS5M8ZAZ3f/Vitryne?node-id=0-1&t=4ydouk8ZtFEd5bX7-1)
- ETI-202 Migrar e refinar o protótipo web no Figma — em andamento
- ETI-167 Refinar protótipo mobile e migrar para o Figma — em andamento
- Página `Mapa Protótipo → Componente` no arquivo do Design System, que registra a correspondência
  entre o protótipo anterior e os componentes novos
- `docs/README.md` — link para `Prototipo Interativo e Design System - Vitryne.html` no Drive
- Regulamento Escola de TI 2026 — protótipo de alta fidelidade e mockups das telas
