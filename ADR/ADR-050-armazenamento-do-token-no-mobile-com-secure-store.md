---
id: ADR-050
titulo: Guardar o token do app no armazenamento seguro do sistema com expo-secure-store, e no localStorage quando o app roda na web
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Leonardo Xavier
area: Mobile
substitui: —
substituida-por: —
---

# ADR-050 — Guardar o token do app no armazenamento seguro do sistema com expo-secure-store, e no localStorage quando o app roda na web

## Contexto e problema

A ADR-038 definiu que o back-end autentica com JWT, devolvido no corpo da resposta do login e do
cadastro, e que as demais rotas exigem o token no cabeçalho `Authorization`. A ADR-021 centralizou
as chamadas do app numa única instância do Axios. Com a integração do login e do cadastro do
cliente (ETI-321), o app passou a precisar guardar o token no aparelho.

O app é distribuído para Android (ADR-022) e, por usar Expo, também roda no navegador durante o
desenvolvimento. Um token guardado sem proteção no aparelho pode ser lido por outro processo com
acesso ao armazenamento do app ou por um backup.

A pergunta é onde o app guarda o token e como ele chega às requisições.

## Critérios de decisão

- Token protegido no aparelho, sem ficar em texto puro
- Disponível para todas as chamadas pela instância central do Axios
- Funcionar também na versão web usada no desenvolvimento
- Nenhuma mudança no back-end

## Alternativas consideradas

1. **`expo-secure-store` no Android e no iOS, que usa o armazenamento seguro do sistema, com
   `localStorage` na web e interceptores na instância central do Axios** — escolhida.
2. **`AsyncStorage`** — *perdeu porque:* grava em texto puro, sem criptografia, no armazenamento do
   app.
3. **Guardar o token só em memória (estado do React)** — *perdeu porque:* o token se perde quando o
   app é fechado.
4. **`expo-secure-store` em todas as plataformas** — *perdeu porque:* a biblioteca não funciona no
   navegador, e a versão web do app quebraria.

## Decisão

O módulo `src/Shared/Api/tokenStorage.ts` concentra o token, na chave `accessToken`, com três
funções assíncronas: `saveToken`, `getToken` e `clearToken`. No Android e no iOS, elas usam o
`expo-secure-store`; quando `Platform.OS` é `web`, usam o `localStorage`.

O login (`loginForm.ts`) e o cadastro do cliente (`useRegister.ts`) chamam `saveToken` depois da
resposta de sucesso. A instância central do Axios (`src/Shared/Api/apiConfig.ts`) tem dois
interceptores:

- **Requisição:** acrescenta `Authorization: Bearer <token>` em todas as chamadas, exceto nas rotas
  `/auth/`, para que um token vencido nunca impeça o próprio login
- **Resposta:** quando a API devolve 401 fora das rotas `/auth/`, apaga o token e chama o tratador
  registrado por `setUnauthorizedHandler`, previsto para levar o usuário de volta ao login

## Consequências

### O que ganhamos

- No aparelho, o token fica no armazenamento seguro do sistema, e não em texto puro
- Nenhuma tela ou hook monta o cabeçalho de autenticação
- Um token recusado pela API é apagado automaticamente

### O que aceitamos em troca

- Na versão web, o token fica no `localStorage`, que pode ser lido por qualquer script da página;
  essa versão é usada só no desenvolvimento
- Não há renovação do token (ver "Expiração e revogação de token" na fila)
- Por enquanto, o token salvo é usado apenas nas requisições: a sessão do cliente não é restaurada
  ao abrir o app, sair da conta não apaga o token, e nenhuma tela registra o tratador de 401

## Validação

Depois do login no Android, o token fica gravado no `expo-secure-store` com a chave `accessToken`,
as chamadas à API levam o cabeçalho `Authorization` e uma resposta 401 apaga o token.

## Evidências

- `mobile/src/Shared/Api/tokenStorage.ts` — gravação, leitura e remoção do token
- `mobile/src/Shared/Api/apiConfig.ts` — interceptores de requisição e de resposta
- `mobile/src/Customer/Hooks/loginForm.ts` e `useRegister.ts` — gravação após login e cadastro
- `mobile/package.json` — dependência `expo-secure-store`
- ETI-321 — integração do login e do cadastro no mobile
- ADR-021 — instância central do Axios
- ADR-038 — autenticação JWT no back-end