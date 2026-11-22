---
id: ADR-051
titulo: Guardar o token do portal no localStorage quando o lojista pede para lembrar o dispositivo, e no sessionStorage nos demais casos
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Henrique Pacheco e Gabriel Rodrigues
area: Front-end
substitui: —
substituida-por: —
---

# ADR-051 — Guardar o token do portal no localStorage quando o lojista pede para lembrar o dispositivo, e no sessionStorage nos demais casos

## Contexto e problema

A ADR-038 definiu que o back-end autentica com JWT: o `POST /auth/login` devolve no corpo da
resposta o `accessToken`, o `tokenType` e o `expiresIn`, e as demais rotas exigem o token no
cabeçalho `Authorization`. Com a integração do login e do cadastro no portal (ETI-324), o front-end
passou a precisar guardar esse token entre uma requisição e outra, inclusive depois de recarregar a
página.

A tela de login tem a opção "Lembrar este dispositivo", marcada por padrão. A pergunta é onde o
portal guarda o token e como essa opção muda isso.

## Critérios de decisão

- O token precisa estar disponível em todas as chamadas, sem cada service tratar disso
- A escolha de "lembrar o dispositivo" precisa ter efeito real
- Nenhuma mudança no back-end, que entrega o token no corpo da resposta
- Um único lugar para gravar, ler e apagar o token

## Alternativas consideradas

1. **`localStorage` quando o lojista marca "Lembrar este dispositivo" e `sessionStorage` quando
   não marca, com um interceptor do Axios que envia o token** — escolhida.
2. **Cookie `HttpOnly` gravado pelo back-end** — *perdeu porque:* exige que o back-end passe a emitir
   o cookie, proteção contra CSRF e CORS com credenciais, o que muda o contrato da ADR-038. É a
   opção mais protegida contra scripts maliciosos e deve ser reaberta depois do MVP.
3. **Guardar o token só em memória (estado do React)** — *perdeu porque:* o token se perde a cada
   recarga de página, e o lojista teria de entrar de novo a cada atualização.
4. **Sempre `localStorage`** — *perdeu porque:* ignora a escolha do lojista e mantém o acesso aberto
   em computadores compartilhados.

## Decisão

O módulo `src/lib/auth-storage.ts` concentra o token, na chave `vitryne:access-token`:

- `saveToken(token, remember)` apaga qualquer token anterior e grava no `localStorage` quando
  `remember` é verdadeiro, ou no `sessionStorage` quando é falso
- `getToken()` procura no `localStorage` e depois no `sessionStorage`, e devolve `null` quando roda
  no servidor
- `clearToken()` apaga dos dois

A tela de login chama `saveToken` com o valor da opção "Lembrar este dispositivo" e leva ao
`/dashboard`. O `useHttp` (`src/components/hooks/use-http.ts`) tem um interceptor de requisição que
acrescenta `Authorization: Bearer <token>` em todas as chamadas, exceto nas rotas `/auth/`. O botão
de sair da barra lateral do painel chama `clearToken`.

## Consequências

### O que ganhamos

- O lojista continua conectado depois de recarregar a página, e o "Lembrar este dispositivo" decide
  se a sessão sobrevive ao fechamento do navegador
- Nenhum service ou tela lida com o token diretamente
- Gravar, ler e apagar ficam num único arquivo

### O que aceitamos em troca

- O `localStorage` e o `sessionStorage` podem ser lidos por qualquer script que rode na página; uma
  falha de XSS exporia o token
- O portal não acompanha a expiração do token: um token vencido continua sendo enviado até o
  back-end recusar, e não há renovação (ver "Expiração e revogação de token" na fila)
- Esta decisão trata só do armazenamento do token; o acesso às páginas do painel não depende dela

## Validação

Depois de entrar com "Lembrar este dispositivo" marcado, a chave `vitryne:access-token` aparece no
`localStorage`; com a opção desmarcada, aparece no `sessionStorage`. As chamadas à API levam o
cabeçalho `Authorization`, e ao sair a chave some dos dois.

## Evidências

- `frontend/src/lib/auth-storage.ts` — gravação, leitura e remoção do token
- `frontend/src/components/hooks/use-http.ts` — interceptor que envia o token
- `frontend/src/pages/login.tsx` — opção "Lembrar este dispositivo" e chamada a `saveToken`
- `frontend/src/components/features/panel/components/panel-sidebar.tsx` — `clearToken` ao sair (ETI-389)
- ETI-324 — integração do login e do cadastro no front-end
- ADR-038 — autenticação JWT no back-end
