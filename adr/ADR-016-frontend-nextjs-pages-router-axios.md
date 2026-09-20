---
id: ADR-016
titulo: Frontend web em Next.js com Pages Router e camada de serviço sobre Axios
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Front-end
substitui: —
substituida-por: —
---

# ADR-016 — Frontend web em Next.js com Pages Router e camada de serviço sobre Axios

## Contexto e problema

O regulamento exige aplicação web em HTML5 e JavaScript, com framework a critério do time. A
aplicação web atende dois públicos: consumidores, que navegam e compram, e lojistas, que
gerenciam a loja pelo portal do parceiro.

O projeto começou em React com Vite e migrou para Next.js no setup da ETI-63. Dentro do Next.js
existem dois roteadores possíveis, e o projeto adotou o Pages Router — decisão que nunca foi
registrada e pode ter sido escolha de familiaridade, não de arquitetura.

## Critérios de decisão

- Roteamento pronto, sem montar configuração à mão
- TypeScript desde o início
- Estrutura que sustente telas públicas de catálogo e telas internas do portal
- Curva compatível com quem já conhece React

## Alternativas consideradas

1. **Next.js 15 com Pages Router, TypeScript, Tailwind CSS v4 e Axios encapsulado em hook** —
   escolhida para registro, sujeita à validação.
2. **Next.js com App Router** — *perdeu porque:* é o padrão atual do framework e o caminho para
   Server Components, mas traz um modelo mental diferente — layouts aninhados, componentes de
   servidor, limites de cliente — que o time não tinha quando fez o setup. Vale reabrir agora: a
   aplicação tem poucas telas, e migrar depois do portal pronto custa muito mais.
3. **React com Vite e React Router** — *perdeu porque:* foi a escolha inicial e foi substituída; o
   Next.js entrega roteamento por arquivo e renderização no servidor sem configuração.
4. **Aplicação separada para consumidor e para lojista** — *perdeu porque:* dobraria o trabalho de
   um time de dois no front, com base de componentes duplicada.

## Decisão

Next.js 15 com Pages Router. TypeScript com caminho `@/*` apontando para `src`. Tailwind CSS v4
via PostCSS para estilo, com componentes base próprios em `src/components`. ESLint com a
configuração do Next.

Acesso à API isolado em duas camadas: o hook `useHttp` cria a instância do Axios com `baseURL`
vindo de `NEXT_PUBLIC_API_URL` e concentra o interceptador de erro; os arquivos de
`src/service` expõem as operações de cada domínio. Nenhuma tela chama Axios direto.

## Consequências

### O que ganhamos

- Roteamento por arquivo, sem configuração
- Um único ponto para tratar erro, cabeçalho e URL base da API
- Componentes base reutilizados entre as telas do portal

### O que aceitamos em troca

- Pages Router recebe menos atenção da comunidade a cada versão, e material novo assume App Router
- Sem biblioteca de estado nem de cache de dados, cada tela vai resolver carregamento e erro do
  seu jeito, e isso diverge rápido entre duas pessoas
- `src/styles/global.css` declara `--color-primary: #9530D9` no bloco `@theme` do Tailwind v4 (não
  há `tailwind.config.ts` — a configuração de tema é feita em CSS), e não há confirmação de que essa
  é a cor da marca — os tokens do Design System ainda não chegaram ao código
- Sem teste automatizado no front

## Validação

Nenhum componente importa `axios` diretamente, e toda chamada à API passa por um arquivo de
`src/service`.

## Evidências

- `frontend/package.json` — `next` 15.5.18, `react` 19.1.0, `axios` ^1.17.0, `tailwindcss` ^4
- `frontend/src/pages/` — `_app.tsx`, `_document.tsx`, `login.tsx`, `register.tsx`
- `frontend/src/hooks/use-http.ts` e `frontend/src/service/login-service.ts`
- `frontend/src/styles/global.css` — bloco `@theme` com os tokens de cor e fonte
- ETI-63 Setup Inicial do Projeto Frontend Web
- ETI-170 Criar tela de Login e a base de componentes reutilizáveis
