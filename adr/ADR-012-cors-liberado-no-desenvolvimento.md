---
id: ADR-012
titulo: Liberar CORS para qualquer origem durante o desenvolvimento
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: « a confirmar »
area: Back-end
substitui: —
substituida-por: —
---

# ADR-012 — Liberar CORS para qualquer origem durante o desenvolvimento

## Contexto e problema

Web e mobile rodam em portas e hosts que variam por máquina: Next.js em `localhost:3000`, Expo
em porta dinâmica, e o mesmo app aberto pelo navegador em modo web. Com CORS restrito, cada
pessoa precisaria cadastrar a própria origem para conseguir chamar a API local.

A configuração atual libera qualquer origem, qualquer cabeçalho e os métodos principais, em todos
os caminhos.

Esta ADR registra uma decisão de conveniência que tem prazo de validade, para que ela não chegue
ao deploy por esquecimento.

## Critérios de decisão

- Sete pessoas conseguem rodar web, mobile e API localmente sem configurar nada
- Nenhum bloqueio de navegador durante o desenvolvimento das telas
- A escolha precisa ser revisável antes de qualquer ambiente exposto

## Alternativas consideradas

1. **`allowedOrigins("*")` em todos os caminhos** — escolhida para o ambiente de desenvolvimento.
2. **Lista de origens permitidas por ambiente** — *perdeu por enquanto:* é a configuração correta,
   mas exige separar perfis de configuração, que o projeto ainda não tem. É o destino desta ADR.
3. **Proxy reverso servindo API e front na mesma origem** — *perdeu porque:* resolve o navegador e
   não resolve o mobile, e adiciona infraestrutura local que o time teria que manter.

## Decisão

CORS liberado para qualquer origem enquanto o projeto roda apenas em máquina local, sem ambiente
publicado e sem autenticação em vigor.

Gatilho de revisão, obrigatório: esta ADR precisa ser substituída antes do primeiro deploy ou
assim que o JWT da ADR-011 entrar, o que vier primeiro. Com credencial em jogo, origem liberada
deixa de ser conveniência e passa a ser falha.

## Consequências

### O que ganhamos

- Nenhum atrito de ambiente entre as três frentes
- Front e mobile evoluem sem depender de configuração no backend

### O que aceitamos em troca

- Qualquer página web consegue chamar a API em nome de quem estiver autenticado, assim que houver
  autenticação
- Combinação de origem liberada com credencial é bloqueada pelo próprio navegador, então a
  configuração vai quebrar sozinha quando o JWT entrar
- A configuração é global: não há como restringir só as rotas sensíveis depois

## Validação

Enquanto esta decisão valer, não existe ambiente da aplicação acessível fora da rede local.

## Evidências

- `backend/src/main/java/com/vitryne/api/config/CorsConfig.java` — `allowedOrigins("*")`,
  `allowedHeaders("*")`, mapeamento `/**`
- ADR-011 Autenticação stateless com JWT
