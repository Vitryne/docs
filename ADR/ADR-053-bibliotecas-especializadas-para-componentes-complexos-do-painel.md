---
id: ADR-053
titulo: Aceitar bibliotecas especializadas para componentes complexos do painel, mantendo próprios os componentes básicos
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: —
substituida-por: —
---

# ADR-053 — Aceitar bibliotecas especializadas para componentes complexos do painel, mantendo próprios os componentes básicos

## Contexto e problema

A ADR-024 decidiu construir os componentes de interface do front-end do zero, sem biblioteca de
componentes pronta, e descartou kits como Radix UI, MUI e Chakra. O motivo foi manter o controle
visual sobre o Design System e evitar dependências que trazem estilo e API próprios para toda a
interface.

Com a construção do painel do lojista, surgiram componentes de outra natureza:

- Gráficos no dashboard (ETI-342) e no financeiro (ETI-343): vendas por dia, pedidos por horário e
  recebimentos por dia
- Um quadro de pedidos com uma coluna por status, em que o lojista arrasta o pedido para mudar o
  status (ETI-225)
- Filtros de pedidos com seleção de um intervalo de datas e de uma faixa de valor (ETI-228)

Esses componentes exigem escalas, eixos e tooltips, arrastar e soltar com mouse, toque e teclado,
seleção de intervalo num calendário e um controle deslizante com dois marcadores. Construí-los do
zero com qualidade, principalmente em acessibilidade, tem um custo muito maior do que o de um botão
ou de um campo.

A pergunta é quando o front-end pode usar uma biblioteca em vez de um componente próprio. Esta ADR
complementa a ADR-024, que continua valendo para os componentes básicos.

## Critérios de decisão

- Os componentes básicos continuam próprios, como decidido na ADR-024
- Acessibilidade por teclado já resolvida pela biblioteca, quando o componente for interativo
- Estilo aplicado com as classes do Tailwind e os tokens do `@theme` (ADR-028)
- Uma biblioteca pequena e focada para cada problema, nunca um kit completo
- Compatibilidade com React 19 e com o Pages Router

## Alternativas consideradas

1. **Aceitar uma biblioteca focada por componente complexo, com critério explícito, mantendo
   próprios os componentes básicos** — escolhida.
2. **Construir todos os componentes do zero, aplicando a ADR-024 sem exceção** — *perdeu porque:*
   exigiria reimplementar escalas e eixos de gráfico, arrastar e soltar acessível, seleção de
   intervalo em calendário e um slider com dois marcadores; o motivo da ADR-024 vale para
   componentes simples, não para estes.
3. **Adotar um kit completo de componentes (MUI, ou shadcn/ui com todo o Radix)** — *perdeu
   porque:* traz estilo e API para toda a interface, o mesmo motivo pelo qual a ADR-024 o descartou.
4. **Usar os campos nativos do HTML (`<input type="date">` e `<input type="range">`) nos
   filtros** — *perdeu porque:* mudam de aparência entre navegadores, não selecionam um intervalo
   num único calendário e não aceitam os estilos do projeto.

## Decisão

Um componente pode usar biblioteca externa quando atende a todas estas condições:

- Não é um componente básico (botão, campo, select, checkbox, modal e similares continuam próprios)
- Resolve um problema complexo e isolado, que a biblioteca cobre melhor do que uma implementação
  própria
- A biblioteca é focada nesse problema e aceita os estilos do Tailwind e os tokens do projeto
- A adoção é registrada em ADR

Aplicação no painel:

| Componente | Biblioteca | Onde | Alternativa descartada |
| --- | --- | --- | --- |
| Gráficos | `recharts` | `dashboard/components/sales-chart.tsx`, `hourly-orders-chart.tsx` e `finance/components/daily-receipts-chart.tsx` | Chart.js, que desenha em canvas e é configurado por objeto imperativo; Nivo e ECharts, mais pesados e com sistema de tema próprio |
| Arrastar e soltar no quadro de pedidos | `@hello-pangea/dnd` | `orders/components/kanban/orders-board.tsx` e `orders-column.tsx` | `react-beautiful-dnd`, arquivado, do qual esta é o fork mantido; `dnd-kit`, de nível mais baixo, que exige montar colunas e acessibilidade à mão |
| Intervalo de datas | `react-day-picker`, no modo de intervalo e com o idioma `ptBR` | `orders/components/filters/period-picker.tsx` | Campo de data nativo |
| Faixa de valor | `@radix-ui/react-slider`, apenas este pacote do Radix | `orders/components/filters/orders-filters-modal.tsx` | Campo de faixa nativo |

Nos gráficos, cada componente recebe os dados por props, converte para o formato do Recharts e fica
dentro de um `ResponsiveContainer`; o tooltip é um componente próprio, para seguir o protótipo. No
quadro, soltar um pedido em outra coluna chama `onMove`, que atualiza o status pelo
`orders-service`.

## Consequências

### O que ganhamos

- Gráficos, arrastar e soltar, calendário e slider prontos e com acessibilidade por teclado onde há
  interação
- A ADR-024 continua valendo para a base da interface, agora com um critério claro para exceções
- Cada biblioteca resolve um único problema e pode ser trocada sem afetar o resto

### O que aceitamos em troca

- Quatro dependências a mais para manter atualizadas, usadas só no painel
- O Radix UI entra no projeto, apenas com o slider, apesar de a ADR-024 citá-lo como descartado
- As cores dos gráficos estão escritas em hexadecimal nos componentes, iguais às do `@theme`, porque
  o Recharts recebe a cor como valor da prop; uma mudança de paleta precisa ser repetida nos
  gráficos
- Os gráficos são SVG sem alternativa em texto para leitores de tela
- A faixa do filtro de valor é fixa, de R$ 0 a R$ 1.000

## Validação

O `package.json` do front-end lista `recharts`, `@hello-pangea/dnd`, `react-day-picker` e
`@radix-ui/react-slider`, sem outro pacote do Radix nem kit de componentes. Os gráficos aparecem no
`/dashboard` e no `/finance`, arrastar um pedido para outra coluna muda o status dele, e os filtros
de período e de valor funcionam com o teclado.

## Evidências

- `frontend/package.json` — as quatro dependências
- `frontend/src/components/features/panel/dashboard/components/` — gráficos de vendas e de pedidos por horário
- `frontend/src/components/features/panel/finance/components/daily-receipts-chart.tsx`
- `frontend/src/components/features/panel/orders/components/kanban/` — quadro de pedidos
