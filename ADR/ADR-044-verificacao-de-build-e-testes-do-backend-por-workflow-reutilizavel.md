---
id: ADR-044
titulo: Verificar build e testes de todo PR do back-end com o workflow reutilizável java-quality
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Gabriel Rodrigues
area: Back-end
substitui: ADR-042
substituida-por: —
---

# ADR-044 — Verificar build e testes de todo PR do back-end com o workflow reutilizável java-quality

## Contexto e problema

A ADR-042 registrou que o CI do back-end valida apenas a governança do PR (título, nome da branch
e mensagens de commit) e que a suíte de testes existe, mas não roda no CI. Em novembro, o back-end
tinha 12 classes de teste — 11 unitárias com Mockito e o `ApiApplicationTests`, que sobe o contexto
completo do Spring — e nenhuma era executada antes do merge. O `Dockerfile` também empacota com
`-DskipTests`.

Com isso, um PR que quebra a compilação, um teste ou uma migration do Flyway chega à `dev` com
todos os checks verdes. O front-end já tinha uma verificação de qualidade por workflow reutilizável
(ADR-034), e o regulamento exige testes automatizados e integração contínua.

A pergunta é como compilar e testar o back-end em todo PR, sem segredos no repositório e no mesmo
modelo de workflow central que o front-end usa.

## Critérios de decisão

- Reaproveitar o modelo de workflow central do `vitryne/.github`, como no front-end
- Rodar os testes contra o mesmo banco usado em desenvolvimento, incluindo as migrations
- Não acrescentar dependência ao projeto só para o CI

## Alternativas consideradas

1. **Workflow reutilizável `java-quality.yml` no `vitryne/.github`, com PostgreSQL como serviço do
   job, par de chaves JWT descartável gerado no runner e `mvn -B verify`** — escolhida.
2. **Workflow escrito direto no repositório do back-end** — *perdeu porque:* quebraria o padrão de
   regra central da ADR-034, com dois lugares para manter a mesma lógica de CI.
3. **Testcontainers para subir o PostgreSQL de dentro dos testes** — *perdeu porque:* acrescenta
   dependência e código de teste para obter o mesmo banco que o serviço do job entrega sem mudar o
   projeto. Vale reabrir se os testes de integração precisarem de mais de um serviço.
4. **Banco em memória (H2) nos testes** — *perdeu porque:* as migrations são escritas para
   PostgreSQL e a imagem de desenvolvimento é `pgvector/pgvector:pg15`; testar em outro banco não
   garante que as migrations funcionam no banco real.

## Decisão

O back-end tem dois workflows:

- **Governança** (`.github/workflows/ci.yml`), que continua chamando `governance.yml@main`, agora com
  `permissions: contents: read` e `concurrency` por número de PR (grupo `ci-`).
- **Qualidade** (`.github/workflows/quality.yml`), que chama `java-quality.yml@main` em PR para `dev`
  e `main`, nos eventos `opened`, `synchronize` e `reopened`, com `permissions: contents: read` e
  `concurrency` por número de PR (grupo `qualidade-`).

O job `Qualidade` do `java-quality.yml`:

1. Sobe o PostgreSQL como serviço (`pgvector/pgvector:pg15`, a mesma imagem do `docker-compose.yml`)
   e espera o banco ficar pronto com `pg_isready`
2. Configura o Java 21 (Temurin) com cache do Maven
3. Gera um par de chaves RSA no runner, gravando o `app.key` e sobrescrevendo o `app.pub` apenas
   dentro da execução
4. Executa `mvn -B verify`: compila, roda todos os testes e empacota

As credenciais do banco no workflow existem só durante a execução e não pertencem a nenhum ambiente.

O check `qualidade / Qualidade` é obrigatório nos rulesets `protecao-dev` e `protecao-main` do
back-end, junto com `governanca / Governança`.

O workflow usa `mvn`, instalado no runner, e não o `./mvnw`: o wrapper existe no repositório, mas a
pasta `.mvn/wrapper` não é versionada.

## Consequências

### O que ganhamos

- Nenhum PR entra na `dev` ou na `main` sem compilar e sem passar nos testes
- Todas as migrations são aplicadas num banco vazio a cada PR, o que pega migration quebrada antes
  do merge
- Back-end e front-end seguem o mesmo modelo: governança e qualidade obrigatórias, regra central

### O que aceitamos em troca

- O `java-quality.yml` é específico para Spring Boot com PostgreSQL e chaves JWT em classpath; não é
  um workflow Java genérico
- O passo que gera as chaves precisa mudar quando o back-end passar a ler as chaves por variável de
  ambiente, no deploy
- A suíte cobre testes unitários e um teste de contexto; os testes de integração com o banco ainda
  não existem (fila do README)
- O workflow aponta para `@main` do repositório central, que exige PR e o check de governança, mas
  não exige aprovação (ADR-047)
- O `./mvnw` continua sem funcionar fora da máquina de quem gerou o projeto

## Validação

Todo PR para `dev` ou `main` do back-end mostra o check `qualidade / Qualidade`, e a consulta às
regras da branch (`GET /repos/Vitryne/backend/rules/branches/dev`) lista esse check como obrigatório.

## Evidências

- `.github/.github/workflows/java-quality.yml` — workflow reutilizável
- `backend/.github/workflows/quality.yml` — chamada ao `java-quality.yml@main`
- `backend/.github/workflows/ci.yml` — governança com `permissions` e `concurrency`
- Rulesets `protecao-dev` e `protecao-main` do back-end — checks obrigatórios
- ETI-382 — CI de qualidade do back-end e padronização dos workflows
- ADR-042 — decisão substituída
- ADR-034 — decisão equivalente no front-end
  