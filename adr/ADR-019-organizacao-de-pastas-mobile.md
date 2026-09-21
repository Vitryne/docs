---
id: ADR-019
titulo: Organização de pastas do app mobile por domínio, com telas e componentes em pasta própria
status: Aceita
data: 2026-09-19
autor: Leonardo Xavier
decisores: Leonardo Xavier e Bruno Abrahim
area: Mobile
substitui: —
substituida-por: —
---

# ADR-019 — Organização de pastas do app mobile por domínio, com telas e componentes em pasta própria

## Contexto e problema

O app mobile atende consumidores e entregadores, com telas, chamadas de API e regras próprias para
cada um, além de partes usadas pelos dois. Sem uma regra de organização, arquivos dos dois fluxos
se misturam e fica difícil saber onde criar ou encontrar cada coisa.

Esta ADR define como as pastas do projeto são organizadas.

## Critérios de decisão

- Separação clara entre os fluxos de consumidor e de entregador
- Saber onde cada tipo de arquivo fica sem precisar procurar
- Tela, componente e estilo de uma funcionalidade próximos uns dos outros
- Estrutura que o time de dois consiga manter

## Alternativas consideradas

1. **Organização por domínio (`Customer`, `Delivery`, `Shared`), com tela e componente em pasta
   própria contendo `index.tsx` e `styles.tsx`** — escolhida.
2. **Organização por tipo de arquivo (`src/Screens`, `src/Components`, `src/Hooks` no nível
   raiz)** — *perdeu porque:* mistura consumidor e entregador nas mesmas pastas, e cada pasta
   cresce sem limite conforme o app aumenta.
3. **Tela e estilo no mesmo arquivo** — *perdeu porque:* deixa o arquivo da tela longo e mistura
   JSX com estilo, dificultando a leitura.

## Decisão

O código fica em `src/`, dividido em três domínios:

- `Customer`: fluxo do consumidor
- `Delivery`: fluxo do entregador
- `Shared`: o que os dois usam, como navegação geral e `commonStyles`

Cada domínio tem as mesmas pastas: `Api` (chamadas HTTP), `Components` (componentes usados em mais
de uma funcionalidade do domínio), `Hooks` (estado e regra de negócio, um hook por tela),
`Navigation` (rotas do domínio), `Screens` (telas), `Styles` (estilos do domínio) e `Types`
(tipos, como a lista de parâmetros de rota).

Dentro de `Screens`:

- Tela simples é uma pasta com `index.tsx` e `styles.tsx` — por exemplo, `Login`
- Funcionalidade com várias telas é uma pasta com `Components`, `Screens` e um `index.tsx` que
  exporta as telas — por exemplo, `Register`, com `RegisterPersonalData`, `RegisterDocuments`,
  `RegisterVehicle` e `RegisterBankAccount` em `Screens`

Toda tela e todo componente é uma pasta com `index.tsx` (JSX) e `styles.tsx` (estilos). A pasta
da tela não guarda lógica: ela fica no hook do domínio.

Imagens, ícones e fontes ficam em `assets/`, na raiz do projeto, fora de `src/`.

Componentes ficam no nível mais próximo de quem usa: primeiro nos `Components` da funcionalidade,
depois nos `Components` do domínio, e só vão para `Shared` quando os dois domínios precisarem.

## Consequências

### O que ganhamos

- Consumidor e entregador evoluem separados, cada um na sua pasta
- Toda tela e todo componente têm a mesma estrutura, o que facilita achar qualquer coisa
- Estilo sempre ao lado do componente que usa
- O `index.tsx` da funcionalidade permite importar as telas por um caminho só

### O que aceitamos em troca

- Muitas pastas pequenas, mesmo para componentes simples
- Vários arquivos com o mesmo nome (`index.tsx` e `styles.tsx`), o que exige olhar o caminho no
  editor para saber qual está aberto
- Decidir se um componente é da funcionalidade, do domínio ou do `Shared` depende de revisão
- Imports entre domínios ficam longos (`../../../`) sem alias configurado

## Validação

Toda tela e todo componente é uma pasta com `index.tsx` e `styles.tsx`, nenhum arquivo de tela
contém lógica de negócio, e nenhum domínio importa telas ou hooks do outro — só do `Shared`.

## Evidências

- `mobile/src/Customer`, `mobile/src/Delivery` e `mobile/src/Shared`
- `mobile/src/Delivery/` — `Api`, `Components`, `Hooks`, `Navigation`, `Screens`, `Styles` e
  `Types`
- `mobile/src/Delivery/Screens/Register/` — `Components`, `Screens` e `index.tsx`
- `mobile/src/Delivery/Screens/Register/Components/PrimaryButton/` — `index.tsx` e `styles.tsx`
