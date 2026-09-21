---
id: ADR-015
titulo: Padronizar o ambiente local com Docker e Docker Compose
status: Aceita
data: 2026-09-13
autor: Equipe
decisores: Henrique Pacheco e João Ehlers
area: Infraestrutura
substitui: —
substituida-por: —
---

# ADR-015 — Padronizar o ambiente local com Docker e Docker Compose

## Contexto e problema

Sete pessoas em máquinas diferentes, com sistemas operacionais diferentes, precisando da mesma
versão de Java e do mesmo PostgreSQL para rodar a API. Instalação manual gera divergência de
versão e o clássico "na minha máquina funciona", além de consumir a primeira hora de quem entra
no projeto.

O regulamento também exige deploy da aplicação, com plataforma a critério do time.

## Critérios de decisão

- Setup em um comando, sem instalar Java nem PostgreSQL na máquina
- Mesma versão de runtime e de banco para todo mundo
- Imagem pronta para publicar quando houver ambiente
- Nenhum segredo dentro da imagem

## Alternativas consideradas

1. **Dockerfile multi-stage para a API e `docker-compose.yml` com API e PostgreSQL** — escolhida.
2. **Instalação manual de JDK e PostgreSQL** — *perdeu porque:* divergência de versão entre
   máquinas e nenhum caminho de deploy aproveitável.
3. **Banco gerenciado compartilhado em nuvem** — *perdeu porque:* obriga a estar online, deixa
   todo mundo mexendo no mesmo dado e traz custo.
4. **Banco em memória para desenvolvimento** — *perdeu porque:* o projeto usa recurso específico
   do PostgreSQL, como coluna de array, e o banco de desenvolvimento precisa se comportar como o
   de verdade.

## Decisão

`Dockerfile` multi-stage: build com Maven sobre Temurin 21, runtime com `eclipse-temurin:21-jre-alpine`
carregando apenas o JAR. `docker-compose.yml` sobe API e PostgreSQL com volume nomeado para os
dados.

Configuração por variável de ambiente, lida de um `.env` que nunca entra no repositório. Toda
variável nova é declarada no `.env.example`, sem valor.

## Consequências

### O que ganhamos

- `docker compose up --build` e o ambiente está de pé
- Mesma versão de Java e de PostgreSQL para todos
- Imagem que já serve para o deploy exigido, quando a plataforma for escolhida

### O que aceitamos em troca

- Docker vira pré-requisito, com o custo de memória que isso tem em máquina modesta
- `docker compose down -v` apaga o banco local, e a recuperação é recriar do zero pelas migrations
- Sem `.dockerignore`, o contexto de build carrega mais do que precisa
- A imagem de banco declarada é `pgvector/pgvector:pg15`, uma variante com extensão de vetores que
  o projeto não usa — ver fila de decisões

## Validação

Clonar o repositório, copiar `.env.example` para `.env`, rodar `docker compose up --build` e obter
a API respondendo, com as migrations aplicadas.

## Evidências

- `backend/Dockerfile` — build multi-stage
- `backend/docker-compose.yml` — serviços `postgres` e `api`, volume `vitryne_postgres_data`
- `Vitryne/.github/CONTRIBUTING.md` — seção de segurança, `.env` fora do Git
- Regulamento Escola de TI 2026, requisito arquitetural (g): deploy da aplicação
