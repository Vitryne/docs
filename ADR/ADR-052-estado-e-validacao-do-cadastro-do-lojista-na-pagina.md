---
id: ADR-052
titulo: Centralizar o estado do cadastro do lojista na página, validar por etapa e converter para o DTO num mapper
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Henrique Pacheco
area: Front-end
substitui: ADR-027
substituida-por: —
---

# ADR-052 — Centralizar o estado do cadastro do lojista na página, validar por etapa e converter para o DTO num mapper

## Contexto e problema

A ADR-027 decidiu que o cadastro em etapas seria controlado pelo índice da etapa atual, com cada
etapa guardando o próprio estado. A consequência registrada foi que os dados de cada etapa não
chegavam à página de cadastro, e não havia como montar o envio final. A fila de decisões também
registrava que o formulário não tinha nenhuma validação.

Com a integração do cadastro à API (ETI-324), o portal passou a enviar todos os dados da loja numa
única chamada ao `POST /auth/register/merchant`. A pergunta é onde os dados das etapas ficam, como
são validados e como viram o corpo da requisição.

## Critérios de decisão

- Todos os dados disponíveis num único lugar no momento do envio
- Erros mostrados na etapa em que estão, sem deixar o lojista avançar com campo inválido
- O formato da API separado do formato dos campos da tela
- Nenhuma biblioteca nova, mantendo as máscaras da ADR-029

## Alternativas consideradas

1. **Estado único na página de cadastro, passado a cada etapa por props, com validação por etapa
   num módulo próprio e conversão para o DTO num mapper** — escolhida.
2. **Manter o estado local de cada etapa (ADR-027)** — *perdeu porque:* os dados não chegam à página
   e não há como montar o envio.
3. **Biblioteca de formulário com esquema de validação (por exemplo, React Hook Form com Zod)** —
   *perdeu porque:* acrescenta duas dependências e exige reescrever as cinco etapas, que já usam
   campos controlados com `react-imask` (ADR-029).
4. **Contexto do React ou store global para o cadastro** — *perdeu porque:* o estado só é usado nesta
   página; um contexto acrescentaria uma camada sem outro consumidor.

## Decisão

A página `src/pages/register.tsx` guarda o índice da etapa e um único objeto `formData`
(`RegisterFormData`, iniciado com `INITIAL_REGISTER_DATA`). Cada etapa recebe `data`, `onChange`,
que mescla os campos alterados no objeto, e `errors`. As cinco etapas são: dados da loja, dados
bancários, entrega e categorias, horários e identidade visual.

A validação fica em `src/lib/register-validation.ts`, com um validador por etapa
(`validateStep(etapa, dados)`), sem biblioteca. As regras incluem campos obrigatórios, formato de
e-mail, senha com no mínimo 6 caracteres, uma letra maiúscula e um número, CNPJ com 14 dígitos, CEP
com 8 dígitos, documento do titular da conta e aceite dos termos. Os erros de uma etapa só aparecem
depois que o lojista tenta avançar. Na última etapa, todas as etapas são validadas de novo, e a
página volta para a primeira que tiver erro.

No envio, `toRegisterDTO` (`src/lib/register-mapper.ts`) converte os campos da tela no
`RegisterStoreRequestDTO`: remove as máscaras, converte os dias da semana para os valores da API e
monta os períodos de funcionamento. O `RegisterService` envia a requisição, e o lojista é levado ao
`/dashboard`.

## Consequências

### O que ganhamos

- O envio do cadastro passa a existir, com todos os dados de uma vez
- O lojista vê o erro na etapa certa e não chega ao fim com campo inválido
- O formato da API fica isolado no mapper; as etapas não conhecem o DTO

### O que aceitamos em troca

- A página inteira é renderizada de novo a cada tecla digitada, o que é aceitável no tamanho atual
  do formulário
- A validação confere a quantidade de dígitos do CNPJ, do CPF e do CEP, mas não os dígitos
  verificadores
- As regras de validação existem no front-end e no back-end (ADR-037) e podem divergir
- O logo e o banner da loja são enviados com imagens de exemplo enquanto não há armazenamento de
  imagens

## Validação

Tentar avançar com um campo obrigatório vazio mostra o erro na etapa e não muda de etapa; um
cadastro completo chega ao `POST /auth/register/merchant` e leva ao `/dashboard`.

## Evidências

- `frontend/src/pages/register.tsx` — estado único e controle das etapas
- `frontend/src/lib/register-validation.ts` — validadores por etapa
- `frontend/src/lib/register-mapper.ts` — conversão para o DTO
- `frontend/src/interfaces/register-form.ts` e `register-interface.ts` — formato da tela e da API
- `frontend/src/service/register-service.ts` — envio do cadastro
- ETI-324 — integração do cadastro no front-end
- ADR-027 — decisão substituída
