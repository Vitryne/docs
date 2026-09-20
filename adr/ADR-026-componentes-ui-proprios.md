---
id: ADR-017
titulo: Componentes de UI próprios em vez de biblioteca de componentes pronta
status: Proposta
data: 2026-09-20
autor: Equipe
decisores: « a confirmar »
area: Front-end
substitui: —
substituida-por: —
---

# ADR-017 — Componentes de UI próprios em vez de biblioteca de componentes pronta

## Contexto e problema

As telas de login e de cadastro do lojista precisam de inputs, botões, checkbox e um select
customizado, todos seguindo o mesmo tema visual definido em `src/styles/global.css`. O time é
pequeno (duas pessoas no front) e o número de telas ainda é reduzido.

## Critérios de decisão

- Consistência visual com os tokens de tema já definidos
- Controle total sobre marcação e classes aplicadas
- Sem overhead de aprender a API de uma biblioteca externa para um volume pequeno de telas
- Bundle enxuto

## Alternativas consideradas

1. **Componentes próprios em `src/components`** (`Button`, `Input`, `Checkbox`, `SelectUf`,
   `MaskedInput`, `ButtonGoogle`) — escolhida para registro, sujeita à validação.
2. **Biblioteca de primitivas com acessibilidade embutida (Radix UI, shadcn/ui)** — *perdeu
   porque:* exige compor o tema por cima de cada primitiva e aprender a API de cada componente;
   para o volume atual de telas (login e cadastro) o ganho de acessibilidade não compensou o tempo
   de integração.
3. **Biblioteca de componentes completa com tema próprio (Material UI, Chakra UI)** — *perdeu
   porque:* impõe um sistema de estilos próprio que concorre com o Tailwind já adotado, e a
   aparência teria que ser sobrescrita quase por completo para bater com a identidade visual da
   marca.

## Decisão

Componentes de interface construídos manualmente em `src/components`, cada um como função
`React.FC<Props>` estilizada com classes utilitárias do Tailwind que apontam para os tokens do
tema (`bg-primary`, `text-text-secondary`, `border-border` etc.). Nenhum componente de terceiros
para inputs, botões, checkbox ou select; ícones vêm do pacote `react-icons`.

## Consequências

### O que ganhamos

- Controle total do HTML e das classes aplicadas em cada componente
- Nenhuma dependência de tema externo para sobrescrever
- Fácil de ajustar quando os tokens do Design System oficial chegarem

### O que aceitamos em troca

- Sem acessibilidade tratada: `SelectUf` implementa um dropdown próprio sem `role`/`aria-*` nem
  navegação por teclado
- Cada componente reimplementa estado e estilo de foco/hover manualmente, e já diverge entre si —
  `Input` não recebe `value`/`onChange` (não controlado), enquanto `MaskedInput` é controlado via
  `value`/`onAccept`
- Sem testes de componente

## Validação

Novas telas devem reaproveitar os componentes existentes em `src/components` antes de escrever
HTML solto; qualquer componente de UI de terceiros para inputs, botões ou seleção entra apenas
mediante nova ADR.

## Evidências

- `frontend/src/components/button.tsx`, `input.tsx`, `checkbox.tsx`, `select-uf.tsx`,
  `masked-input.tsx`, `button-google.tsx`
- `frontend/package.json` — `react-icons`, sem `@radix-ui/*`, `@mui/material` ou `@chakra-ui/*`
- ETI-170 Criar tela de Login e a base de componentes reutilizáveis
- ETI-206 Aplicando máscaras e tipos de campo no cadastro do lojista
