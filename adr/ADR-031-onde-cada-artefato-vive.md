---
id: ADR-031
titulo: Um papel por ferramenta para decidir onde cada artefato do projeto vive
status: Aceita
data: 2026-09-20
autor: Hugo Zuin
decisores: Hugo Zuin
area: Governança
substitui: —
substituida-por: —
---

# ADR-031 — Um papel por ferramenta para decidir onde cada artefato do projeto vive

## Contexto e problema

O projeto usa cinco ferramentas: Jira, Confluence, GitHub, Google Drive e Figma. Cada uma entrou
resolvendo um problema pontual, sem que ninguém decidisse o que pertence a qual.

## Critérios de decisão

- Uma fonte por tipo de artefato, sem cópia paralela que alguém tenha que manter
- Cada artefato na ferramenta que faz ele funcionar melhor — versionamento, comentário ou
  colaboração
- Endereço em um lugar só, sem link repetido por várias páginas
- Caminho curto para apresentar qualquer artefato

## Alternativas consideradas

1. **Um papel definido por ferramenta, com uma tabela única de links no Confluence** — escolhida.
2. **Tudo no Confluence** — *perdeu porque:* não versiona código e não comporta edição
   colaborativa de documento longo com o conforto do Google Docs.
3. **Tudo no GitHub, em Markdown** — *perdeu porque:* parte do time não edita Markdown com
   conforto, e ata, painel de sprint e matriz de evidências pedem tabela e macro do Confluence.
4. **Tudo no Google Drive** — *perdeu porque:* não versiona junto do código nem se liga a tarefa,
   e o regulamento exige que código e artefatos versionados fiquem no GitHub.

## Decisão

| Artefato | Onde vive |
|---|---|
| Tarefas, sprints e apontamento de horas | Jira |
| Processo, políticas, atas, painéis e conformidade | Confluence |
| Código-fonte | GitHub — `backend`, `frontend`, `mobile` |
| Decisões arquiteturais | GitHub — `docs/adr` |
| PDFs oficiais dos artefatos (diagramas, requisitos, personas) | GitHub — `docs` |
| Originais editáveis desses documentos | Google Drive |
| Design System e protótipo | Figma |
| Regras de contribuição e templates da organização | GitHub — `.github` |

Regra de derivação entre Drive e `docs`: o documento nasce e é editado no Drive; o PDF publicado
no `docs` é a versão congelada daquele documento. Mudou o conteúdo, edita-se no Drive e reexporta.

Os endereços de tudo isso ficam em uma tabela única de links no Confluence. Qualquer página que
precise citar um endereço aponta para essa tabela em vez de repetir a URL.

## Consequências

### O que ganhamos

- A pergunta "onde está X" tem uma resposta só, e ela não depende de perguntar para alguém
- Endereço mantido em um lugar: mudou o link, muda em um ponto
- Cada artefato na ferramenta que o trata bem — código versionado, ata comentável, documento
  editável a várias mãos
- Caminho curto para apresentar qualquer artefato em avaliação

### O que aceitamos em troca

- Drive e `docs` guardam o mesmo documento em formatos diferentes: é duplicação assumida, e o PDF
  envelhece em silêncio se alguém editar o original e não reexportar
- Nada automatiza essa reexportação; ela depende de disciplina e de revisão

- A tabela de links só funciona enquanto for a única: link solto copiado em página antiga continua
  apontando para onde apontava
- Artefato que não se encaixa claramente em nenhuma linha da tabela vai precisar de decisão nova

## Validação

Nenhum artefato editável é mantido em duas ferramentas ao mesmo tempo, todo PDF no `docs` tem
original correspondente no Drive, e nenhuma página do Confluence repete um endereço que já está na
tabela de links.

## Evidências

- Tabela de links e ferramentas, no Confluence, com a regra de não repetir endereço
- Estrutura do espaço do Confluence: processo, atas, sprints, conformidade e documentação técnica
- `Vitryne/docs` — PDFs dos diagramas, requisitos, personas e regulamento
- Pasta do Google Drive com os documentos editáveis
- ETI-200 Consolidar ADRs no repositório de documentação
