---
id: ADR-035
titulo: Publicar o front-end na Vercel por GitHub Actions, com ambientes de preview, homologação e produção
status: Aceita
data: 2026-11-01
autor: Henrique Pacheco
decisores: Gabriel Rodrigues e Henrique Pacheco
area: Front-end
substitui: —
substituida-por: —
---

# ADR-035 — Publicar o front-end na Vercel por GitHub Actions, com ambientes de preview, homologação e produção

## Contexto e problema

O regulamento exige publicação em ambiente, com a plataforma a critério do time. A fila do
README listava a plataforma de deploy como decisão não tomada. No front-end ela já foi
implementada pelo ETI-294, sem ADR: o workflow `deploy.yml` publica na Vercel.

A pergunta é como o front-end é publicado e quais ambientes existem. O deploy do back-end fica
fora desta ADR; a ADR-021 cita o Render como destino dele, sem decisão registrada.

## Critérios de decisão

- Publicação automática a cada mudança, sem passo manual
- Separação entre revisão de PR, homologação e produção
- Verificação mínima de que o site respondeu depois da publicação

## Alternativas consideradas

1. **Vercel acionada por GitHub Actions, com `vercel build` e `vercel deploy --prebuilt`, e o
   deploy automático do Git da Vercel desligado** — escolhida.
2. **Deploy automático pela integração Git da Vercel** — *perdeu porque:* o `vercel.json` a
   desliga (`git.deploymentEnabled: false`), e o workflow assume o controle de quando e onde
   publicar. O repositório não registra o motivo dessa escolha; os critérios acima são os que o
   fluxo atual atende.

## Decisão

O workflow `.github/workflows/deploy.yml` publica o front-end na Vercel em PR para `dev` e `main`
e em push para `dev` e `main`. O ambiente depende do evento:

| Evento | Ambiente | Endereço |
|---|---|---|
| Pull request | `preview` | URL gerada pela Vercel |
| Push em `dev` | `homologacao` | `https://homolog.vitrynes.com` (alias) |
| Push em `main` | `producao` | `https://www.vitrynes.com` |

O fluxo baixa a configuração com `vercel pull`, roda `vercel build` (com `--prod` em produção) e
publica com `vercel deploy --prebuilt`. Depois faz um smoke test em `/login`, com novas tentativas,
que falha se o site terminar em outro domínio. A execução usa `concurrency` por PR ou branch,
cancelando a anterior só em PR, e pula PRs do Dependabot. A versão do Node vem de `.nvmrc`.

Credenciais da Vercel ficam em `vars` e `secrets` do GitHub, nunca no repositório.

## Consequências

### O que ganhamos

- Cada PR ganha um preview, e `dev` e `main` publicam sem passo manual
- Homologação e produção separadas por branch, cada uma com endereço próprio
- O smoke test evita marcar como publicado um site que não responde

### O que aceitamos em troca

- O pipeline depende de segredos e variáveis do GitHub e de um token da Vercel
- O smoke test só verifica que `/login` responde; não cobre o resto das páginas nem a integração
  com a API
- O deploy do front-end não depende do resultado do workflow de qualidade (ADR-034): são
  workflows separados
- Sem teste automatizado, nada além do build e do smoke test barra um deploy quebrado
- O deploy do back-end continua sem decisão registrada

## Validação

Um push em `main` termina o job `Deploy` com ambiente `producao` e o smoke test verde, e o resumo
do job mostra o endereço publicado.

## Evidências

- `frontend/.github/workflows/deploy.yml` — ambientes, `vercel build`, `vercel deploy` e smoke test
- `frontend/vercel.json` — `git.deploymentEnabled: false`
- `frontend/.nvmrc` — versão do Node usada no workflow
- ETI-294 — adiciona workflow de deploy na Vercel
