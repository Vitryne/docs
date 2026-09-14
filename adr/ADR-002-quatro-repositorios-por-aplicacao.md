---
id: ADR-002
titulo: Separar o projeto em quatro repositórios por aplicação sob uma organização GitHub
status: Proposta
data: 2026-09-13
autor: Equipe
decisores: Equipe
area: Governança
substitui: —
substituida-por: —
---

# ADR-002 — Separar o projeto em quatro repositórios por aplicação sob uma organização GitHub

## Contexto e problema

O produto tem três aplicações com ciclos e linguagens diferentes — API em Java, web em
Next.js, app em React Native — mais os artefatos formais da disciplina. Sete pessoas
trabalham em paralelo, divididas por frente.

Era preciso decidir se o código ficaria junto em um repositório só ou separado por aplicação,
e onde ficariam as regras comuns ao time.

## Critérios de decisão

- Cada frente trabalha sem esbarrar nas outras
- Histórico de commits legível por aplicação
- Regras de contribuição idênticas nos quatro repositórios, sem cópia manual
- Configuração de CI simples, compatível com o que a equipe sabe operar

## Alternativas consideradas

1. **Quatro repositórios — `backend`, `frontend`, `mobile`, `docs` — mais o repositório
   `.github` da organização** — escolhida.
2. **Monorepo único** — *perdeu porque:* exigiria ferramenta de build e versionamento
   coordenado que o time não domina, e todo PR passaria a disputar o mesmo histórico.
3. **Dois repositórios (API e clientes)** — *perdeu porque:* web e mobile têm stacks, donos e
   ritmos distintos; juntá-los só adiaria a separação.

## Decisão

Cinco repositórios sob a organização `Vitryne`: `backend`, `frontend`, `mobile`, `docs` e
`.github`. O `.github` hospeda o `CONTRIBUTING.md`, o template de pull request e os workflows
reutilizáveis, que o GitHub propaga automaticamente para os demais repositórios.

Trabalho que atravessa repositórios usa a mesma chave Jira nos dois, e o backend mergeia
primeiro — cliente não vai para a `dev` chamando endpoint que ainda não existe.

## Consequências

### O que ganhamos

- Frentes independentes, com histórico e revisão próprios
- Regra escrita uma vez no `.github` e válida para toda a organização
- CI declarado uma vez como workflow reutilizável

### O que aceitamos em troca

- Mudança de contrato de API exige coordenação manual entre dois PRs
- Não há build único que prove que os quatro repositórios funcionam juntos
- Clonar o projeto inteiro significa clonar quatro repositórios

## Validação

`CONTRIBUTING.md` e template de PR aparecem nos quatro repositórios sem duplicação de arquivo,
e o workflow de validação de título é chamado por referência, não copiado.

## Evidências

- `Vitryne/.github` — `CONTRIBUTING.md`, `.github/pull_request_template.md`,
  `.github/workflows/pr-title.yml`
- `.github/workflows/pr-title.yml` em `backend`, `frontend` e `mobile` usando
  `uses: vitryne/.github/.github/workflows/pr-title.yml@main`
- `docs/README.md` — tabela do ecossistema de repositórios
