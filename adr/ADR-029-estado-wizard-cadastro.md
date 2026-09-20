---
id: ADR-020
titulo: Estado do cadastro em etapas controlado localmente por índice
status: Proposta
data: 2026-09-20
autor: Equipe
decisores: « a confirmar »
area: Front-end
substitui: —
substituida-por: —
---

# ADR-020 — Estado do cadastro em etapas controlado localmente por índice

## Contexto e problema

O cadastro do lojista (ETI-171, ETI-198, ETI-206) é dividido em 6 etapas — dados da loja,
identidade visual, categoria & entrega, horários, dados bancários e primeiro produto — navegáveis
pela sidebar. O projeto não tem biblioteca de gerenciamento de estado (Redux, Zustand, Context API
dedicada) nem de máquina de estados.

## Critérios de decisão

- Navegação simples entre etapas
- Permitir voltar para uma etapa já concluída pela sidebar
- Não adicionar dependência nova para um fluxo de 6 telas fixas

## Alternativas consideradas

1. **`useState(stepIndex)` no componente `Register`, com array `STEPS` mapeando `id`/`label`/
   `Component`, e cada etapa controlando seus próprios campos** — escolhida para registro, sujeita
   à validação.
2. **Máquina de estados com XState** — *perdeu porque:* o fluxo é linear, com poucas transições
   (avançar, voltar, pular para etapa concluída); o custo de aprender e manter XState não se paga
   para 6 passos fixos.
3. **Estado global do formulário inteiro em Context API, compartilhado entre as 6 etapas** —
   *perdeu porque:* nenhuma etapa envia dados para a API ainda; centralizar o estado antes de ter
   os contratos de request definidos geraria retrabalho.

## Decisão

`Register` mantém `stepIndex` em `useState` e renderiza o `Component` correspondente do array
`STEPS`, passando apenas `onNext`. Cada etapa (`StepStore`, `StepIdentity` etc.) mantém seus
próprios campos em `useState` local, sem repassar valores para o componente pai.

## Consequências

### O que ganhamos

- Implementação simples, sem dependência nova
- Cada etapa isolada da outra, fácil de desenvolver em paralelo por duas pessoas

### O que aceitamos em troca

- Dado preenchido em uma etapa (por exemplo, o CNPJ em `StepStore`) não sobrevive à troca de etapa
  nem fica disponível para compor o payload final do cadastro, porque não é elevado a `Register`
- Ao voltar para uma etapa já concluída pela sidebar (`selectStep`), os campos aparecem vazios de
  novo, já que o componente da etapa é remontado sem estado anterior
- Não há indicação de progresso salvo (rascunho) nem validação bloqueando avanço com campos
  obrigatórios vazios

## Validação

Nenhuma etapa deve chamar Axios ou o serviço da API diretamente. A composição do payload final do
cadastro depende de uma decisão futura — elevar o estado a `Register` ou introduzir Context — antes
de existir integração real com o backend.

## Evidências

- `frontend/src/pages/register.tsx`
- `frontend/src/components/steps-register/step-store.tsx`
- `frontend/src/components/sidebar-register.tsx`
- ETI-171 Criar etapa de dados da loja no cadastro
- ETI-198 Criar sidebar da tela de registro
- ETI-206 Aplicando máscaras e tipos de campo no cadastro do lojista
