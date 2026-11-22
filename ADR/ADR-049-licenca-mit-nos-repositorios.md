---
id: ADR-049
titulo: Licenciar os repositórios da organização sob MIT, com documentos de terceiros fora da licença
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Gabriel Rodrigues
area: Governança
substitui: —
substituida-por: —
---

# ADR-049 — Licenciar os repositórios da organização sob MIT, com documentos de terceiros fora da licença

## Contexto e problema

O `backend` e o `frontend` tinham LICENSE MIT, enquanto o `mobile`, o `docs` e o `.github` não
tinham licença nenhuma. Sem arquivo de licença, quem não é autor não tem permissão de reaproveitar
o código, mesmo vendo-o no GitHub, e repositórios da mesma organização ficavam em situações
diferentes.

O `docs` também guarda um documento que não foi produzido pelo time: o regulamento da Escola de TI,
na pasta `Regulamentação`.

A pergunta é qual licença vale para os repositórios e o que fica fora dela.

## Critérios de decisão

- A mesma licença em todos os repositórios da organização
- Manter o que já existia no back-end e no front-end, se não houver motivo para mudar
- Licença simples, adequada a um projeto acadêmico
- Não licenciar material que não pertence ao time

## Alternativas consideradas

1. **MIT em todos os repositórios, com o mesmo texto do back-end e do front-end** — escolhida.
2. **Apache 2.0** — *perdeu porque:* a concessão de patentes e o arquivo `NOTICE` não fazem diferença
   num projeto acadêmico, e trocaria a licença que já existia em dois repositórios.
3. **GPL** — *perdeu porque:* obriga quem reaproveitar o código a abrir o trabalho derivado, uma
   restrição sem motivo para este projeto, e também divergiria do que já existia.
4. **Manter os repositórios sem licença** — *perdeu porque:* deixa o código sem permissão de uso
   para quem não é autor e mantém repositórios da mesma organização em situações diferentes.

## Decisão

Os repositórios `frontend`, `backend`, `mobile`, `docs` e `.github` têm um arquivo `LICENSE` na
raiz com a licença MIT e a linha `Copyright (c) 2026 Vitryne`. O README de cada repositório indica
a licença.

A licença cobre o código e a documentação produzidos pelo time. Documentos de terceiros guardados
no `docs`, como o regulamento da Escola de TI na pasta `Regulamentação`, não pertencem ao time e
não ficam sob a licença MIT.

## Consequências

### O que ganhamos

- Todos os repositórios da organização com a mesma licença
- Permissão clara para reaproveitar o código, inclusive no portfólio de cada integrante
- Material de terceiros separado do que o time pode licenciar

### O que aceitamos em troca

- A MIT permite que qualquer pessoa reaproveite o código, inclusive em uso comercial, sem
  obrigação de devolver melhorias
- A exceção dos documentos de terceiros está registrada nesta ADR, mas não no README do `docs`
- O titular dos direitos aparece como "Vitryne", e não como os integrantes; a autoria individual
  fica no histórico de commits

## Validação

O arquivo `LICENSE` com a licença MIT existe na raiz dos cinco repositórios da organização.

## Evidências

- `backend/LICENSE` e `frontend/LICENSE` — licença MIT que já existia
- `mobile/LICENSE` — adicionado na ETI-385
- `docs/LICENSE` e seção de licença do `docs/README.md` — adicionados na ETI-390
- `.github/LICENSE` — adicionado na ETI-391
