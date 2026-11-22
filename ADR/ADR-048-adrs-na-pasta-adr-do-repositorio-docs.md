---
id: ADR-048
titulo: Manter as ADRs na pasta ADR do repositório docs, com as convenções da ADR-001
status: Aceita
data: 2026-11-22
autor: Gabriel Rodrigues
decisores: Gabriel Rodrigues e Hugo Zuin
area: Governança
substitui: ADR-001
substituida-por: —
---

# ADR-048 — Manter as ADRs na pasta ADR do repositório docs, com as convenções da ADR-001

## Contexto e problema

A ADR-001 decidiu registrar as decisões arquiteturais como arquivos Markdown versionados em
`docs/adr/`, no repositório `Vitryne/docs`, e definiu as convenções de ID, status, imutabilidade,
autoria e índice.

Na revisão do repositório `docs` (ETI-390), as pastas da raiz seguiam nomes com inicial maiúscula
(`Diagramas`, `Mapas`, `Regras de Negócio`, `Regulamentação`), e `adr` era a única fora desse padrão.
A pasta foi renomeada para `ADR/`.

Como a ADR-001 cita o caminho antigo e uma ADR publicada não se edita, a mudança precisa de uma ADR
nova. A decisão de fundo da ADR-001 continua a mesma; muda só o nome da pasta.

## Critérios de decisão

- Nomes de pasta consistentes na raiz do `docs`
- Manter todas as convenções da ADR-001
- Não quebrar os links entre as ADRs e o índice

## Alternativas consideradas

1. **Pasta `ADR/` na raiz do `docs`, com as convenções da ADR-001** — escolhida.
2. **Manter `adr/`** — *perdeu porque:* seria a única pasta da raiz fora do padrão de nomes do
   repositório.
3. **Outro nome, como `Decisões/`** — *perdeu porque:* o time chama esses registros de ADR em todo
   lugar, nos IDs dos arquivos, no índice e no Confluence; outro nome criaria dois termos para a
   mesma coisa.
4. **Mover as ADRs para o Confluence** — *perdeu porque:* os motivos da ADR-001 continuam valendo,
   já que o registro ficaria separado do repositório, sem diff nem histórico legível.

## Decisão

As ADRs ficam em `ADR/`, no repositório `Vitryne/docs`, um arquivo Markdown por decisão, no formato
MADR adaptado para português. A página de decisões arquiteturais no Confluence é apenas o ponteiro
para a pasta.

As convenções da ADR-001 continuam valendo:

- Arquivo nomeado `ADR-NNN-titulo-curto.md`, com ID sequencial nunca reaproveitado
- Status: Proposta · Aceita · Substituída · Descartada
- ADR publicada não se edita. Decisão revista entra como ADR nova; a anterior muda apenas o status
  para Substituída e ganha o campo `substituida-por` no front matter
- Autoria identificada no front matter
- O `README.md` da pasta mantém o índice, o próximo ID livre e a fila de decisões ainda não
  registradas
- `ADR-000-modelo.md` é o ponto de partida de toda ADR nova: não é decisão e não entra no índice

As ADRs existentes mantêm número e conteúdo. Os links do índice e entre as ADRs são relativos e
continuam funcionando depois da renomeação.

## Consequências

### O que ganhamos

- Pastas da raiz do `docs` com o mesmo padrão de nome
- As convenções de registro reunidas numa ADR em vigor, com o caminho correto
- A ADR-001 preservada como histórico, como manda a própria convenção

### O que aceitamos em troca

- Links externos que apontam para `adr/`, como os do Confluence, deixam de funcionar e precisam ser
  atualizados
- ADRs antigas continuam citando `docs/adr/` no texto e não podem ser corrigidas
- Em sistemas que não diferenciam maiúsculas de minúsculas (Windows e macOS), um clone antigo pode
  continuar mostrando a pasta como `adr/`; nesse caso, basta clonar o repositório de novo

## Validação

A pasta `ADR/` existe na raiz do `docs` com o índice atualizado, não existe pasta `adr/`, e a página
de ADRs do Confluence aponta para `ADR/`.

## Evidências

- `docs/ADR/README.md` — índice e convenções
- ETI-390 — renomeação de `adr/` para `ADR/`
- ADR-001 — decisão substituída
