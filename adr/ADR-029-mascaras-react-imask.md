---
id: ADR-029
titulo: Máscaras de campo com react-imask e estado local por campo
status: Aceita
data: 2026-09-20
autor: Henrique Pacheco
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: —
substituida-por: —
---

# ADR-029 — Máscaras de campo com react-imask e estado local por campo

## Contexto e problema

O cadastro do lojista (ETI-206) exige CNPJ e CEP formatados enquanto o usuário digita. O projeto
não usa biblioteca de formulário — não há `react-hook-form`, `formik` nem `zod` no
`package.json` — então cada campo mantém seu próprio estado local.

## Critérios de decisão

- Máscara aplicada em tempo real sem corromper o valor digitado (cursor, borracha)
- Compatível com componente React controlado
- Sem acoplar a um framework de formulário que ainda não foi decidido

## Alternativas consideradas

1. **`react-imask` (`IMaskInput`) com `value`/`onAccept` ligados a `useState` na própria etapa** —
   escolhida.
2. **Máscara manual com regex no `onChange`** — *perdeu porque:* replicar o comportamento de
   cursor e edição no meio do texto do IMask à mão custa mais e quebra com mais facilidade.
3. **Adotar `react-hook-form` com resolver de máscara agora** — *perdeu porque:* nenhuma tela do
   projeto envia formulário para a API ainda (`LoginService` retorna objeto vazio); introduzir a
   biblioteca antes de existirem regras de validação de negócio definidas (formato de CNPJ, regra
   de senha etc.) seria decisão prematura.

## Decisão

Campos que exigem máscara usam o componente `MaskedInput`, que envolve `IMaskInput` do
`react-imask`; cada tela guarda o valor mascarado em `useState` próprio (ver `uf`, `cep`, `cnpj`
em `StepStore`). Campos sem máscara usam o componente `Input`, que hoje não é controlado (sem
`value`/`onChange`).

## Consequências

### O que ganhamos

- Máscara de CNPJ e CEP correta, mantida por uma biblioteca testada, sem lógica de formatação
  escrita à mão

### O que aceitamos em troca

- Campos ficam divididos entre controlados (com máscara) e não controlados (sem máscara) dentro
  da mesma etapa, e nenhum dos dois está de fato ligado a um objeto de formulário
- Ao trocar de etapa no wizard de cadastro, o valor digitado em `StepStore` se perde, porque não é
  repassado para `Register`
- Sem validação de formato (CNPJ ou CEP inválido passa) nem de obrigatoriedade

## Validação

Todo campo com máscara passa por `MaskedInput`/`IMaskInput`; nenhuma tela implementa regex de
máscara própria.

## Evidências

- `frontend/src/components/masked-input.tsx`
- `frontend/src/components/steps-register/step-store.tsx`
- `frontend/package.json` — `react-imask` ^7.6.1, ausência de `react-hook-form`, `formik` ou `zod`
- ETI-206 Aplicando máscaras e tipos de campo no cadastro do lojista
