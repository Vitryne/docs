# Decisões Arquiteturais (ADR) — Vitryne

Registro das decisões arquiteturais do projeto, versionado junto com o código. Esta pasta é a
fonte da verdade; a página de decisões arquiteturais no Confluence aponta para cá e não duplica
o conteúdo.


## Como funciona

- Um arquivo Markdown por decisão, nomeado `ADR-NNN-titulo-curto.md`
- ID sequencial, **nunca reaproveitado**, mesmo se a ADR for descartada
- Status: **Proposta** · **Aceita** · **Substituída** · **Descartada**
- **ADR publicada não se edita.** Decisão revista entra como ADR nova; a anterior passa a
  Substituída e ganha o campo `substituida-por` no front matter
- Autoria identificada no front matter de cada arquivo
- Formato: MADR adaptado para português — contexto e problema, critérios, alternativas
  consideradas com o motivo de cada uma ter perdido, decisão, consequências, validação e
  evidências no projeto

ADR registra **a decisão, o que foi descartado e o que se aceitou em troca**. Como o sistema
funciona é outro documento.

Para abrir uma ADR nova: duplique o [`ADR-000-modelo.md`](ADR-000-modelo.md), use o próximo número
livre indicado abaixo da tabela e abra o pull request junto com a mudança que a decisão governa. O
modelo traz, em cada seção, o que se espera nela — ele é um arquivo de apoio, não uma decisão, e
por isso não aparece no índice.

## Índice

| ID | Decisão | Status | Área |
|---|---|---|---|
| [ADR-001](ADR-001-registro-de-decisoes-em-adrs-versionadas.md) | Registrar decisões arquiteturais como ADRs versionadas no repositório | Proposta | Governança |
| [ADR-002](ADR-002-quatro-repositorios-por-aplicacao.md) | Separar o projeto em quatro repositórios por aplicação sob uma organização GitHub | Proposta | Governança |
| [ADR-003](ADR-003-processo-de-desenvolvimento-e-estrategia-de-branches.md) | Processo de desenvolvimento, estratégia de branches e definition of done | Proposta | Governança |
| [ADR-004](ADR-004-backend-como-monolito-java-spring-boot.md) | Backend como monólito único em Java 21 com Spring Boot 4 | Proposta | Back-end |
| [ADR-005](ADR-005-arquitetura-em-camadas-com-entidades-ricas.md) | Organizar o backend em camadas técnicas com regra de negócio nas entidades | Proposta | Back-end |
| [ADR-006](ADR-006-postgresql-com-flyway-e-schema-versionado.md) | PostgreSQL como banco único com schema versionado em Flyway | Proposta | Back-end |
| [ADR-007](ADR-007-dtos-em-records-separando-contrato-de-api.md) | Separar contrato da API das entidades JPA usando DTOs em records | Proposta | Back-end |
| [ADR-008](ADR-008-lombok-para-reduzir-boilerplate.md) | Adotar Lombok para reduzir código repetitivo no backend | Proposta | Back-end |
| [ADR-009](ADR-009-tratamento-de-erro-centralizado.md) | Centralizar tratamento de erro em RestControllerAdvice com exceções próprias | Proposta | Back-end |
| [ADR-010](ADR-010-nomenclatura-em-ingles-no-codigo-e-no-banco.md) | Nomenclatura em inglês no código e no banco, documentação em português | Proposta | Back-end |
| [ADR-011](ADR-011-autenticacao-stateless-com-jwt.md) | Autenticação stateless com JWT e filtro do Spring Security | Proposta | Back-end |
| [ADR-012](ADR-012-cors-liberado-no-desenvolvimento.md) | Liberar CORS para qualquer origem durante o desenvolvimento | Proposta | Back-end |
| [ADR-013](ADR-013-referencia-entre-agregados-por-identificador.md) | Referenciar agregados por identificador em vez de relação JPA | Proposta | Back-end |
| [ADR-014](ADR-014-tipo-para-valores-monetarios.md) | Migrar valores monetários de Double para BigDecimal | Proposta | Back-end |
| [ADR-015](ADR-015-docker-compose-para-ambiente-local.md) | Padronizar o ambiente local com Docker e Docker Compose | Proposta | Infraestrutura |
| [ADR-016](ADR-016-frontend-web-em-nextjs-com-pages-router.md) | Frontend web em Next.js com Pages Router e camada de serviço sobre Axios | Proposta | Front-end |
| [ADR-017](ADR-017-mobile-com-react-navigation.md) | Navegação do app mobile com React Navigation em vez de Expo Router | Proposta | Mobile |
| [ADR-018](ADR-018-design-system-no-figma-como-base-do-prototipo.md) | Design System no Figma como base de componentes do protótipo | Proposta | Produto e UI |
| [ADR-019](ADR-019-prototipo-de-alta-fidelidade-no-figma.md) | Protótipo no Figma como fonte da verdade da interface para o desenvolvimento | Proposta | Produto e UI |


## Fila de decisões ainda não registradas

### Decisões que o projeto ainda não tomou

| Tema | Por que precisa de decisão | Referência |
|---|---|---|
| Escala de 1M escritas para 2M leituras | Exigência do regulamento; define estratégia de cache, réplica de leitura e índices | Regulamento, requisito (d) |
| Cache | Redis aparece na stack divulgada no `docs/README.md` e não existe no `pom.xml`; decidir se entra, onde e com que política de invalidação | Regulamento, requisito (d) |
| Websocket | Exigência do regulamento: pelo menos um caso de uso; falta escolher qual — acompanhamento de pedido é o candidato natural | Regulamento, requisito (e) |
| Integração de pagamento | Exigência do regulamento em ambiente de homologação; gateway a escolher, com PIX previsto no produto | Regulamento, requisito (f) |
| Plataforma de deploy | Exigência do regulamento, plataforma a critério do time | Regulamento, requisito (g) |
| Estratégia de testes automatizados | Exigência do regulamento: unitários e de integração. Hoje há apenas o teste de contexto do Spring | ETI-123, ETI-159, ETI-160 |
| Integração contínua | O CI valida só o título do PR. Decidir o que roda a cada mudança: build, teste, lint | ADR-003 |
| Versionamento da API | Prefixo de versão e política de mudança incompatível; hoje os endpoints não têm versão | ADR-007 |
| Paginação e ordenação | `listProducts` devolve a lista inteira; catálogo não escala assim | `ProductController.java` |
| Formato de erro padronizado | Avaliar `ProblemDetail` (RFC 7807) no lugar do `Map` montado à mão | ADR-009 |
| Geolocalização e busca por proximidade | Função central do produto, sem decisão de tecnologia nem de modelagem | Casos de uso |
| Observabilidade | Hoje há log em controller; falta decidir formato, nível e destino | — |
| Valores do Design System no código | Como manter cor, espaçamento e raio iguais entre Figma, `tailwind.config.ts` e `commonStyles`, sem detecção automática de divergência | ADR-018 |
| Ligação entre componente do Figma e componente do código | Hoje a correspondência é manual e não é verificável | ADR-018 |
| Estado e cache de dados no front | Nenhuma biblioteca adotada; cada tela resolve carregamento e erro do seu jeito | ADR-016 |
| Camada de acesso à API no mobile | `src/Api` existe vazio e os dados são mockados nos hooks | ETI-165 |
| Expiração e revogação de token | Tempo de vida, refresh token e o que fazer para invalidar antes do vencimento | ADR-011 |
| Acessibilidade | Contraste, tamanho de alvo de toque e leitor de tela não têm critério definido no Design System | ADR-018 |

### Decisões já implementadas que ainda não têm ADR

| Tema | Situação |
|---|---|
| Coluna `text[]` para URLs de fotos do produto | Denormalização implementada; decidir se fica ou vira tabela própria, e alinhar com o DER |
| Imagem `pgvector/pgvector:pg15` no Docker Compose | Variante com extensão de vetores que o projeto não usa; confirmar se é intencional |
| Versão de migration por timestamp com chave Jira | Convenção em uso; registrada dentro da ADR-006, pode virar ADR própria se mudar |
| React Compiler habilitado no mobile | `experiments.reactCompiler` ligado no `app.json`, sem decisão registrada |
| Instruções de agente versionadas | `AGENTS.md`, `CLAUDE.md` e `.claude/settings.json` no repositório mobile; o regulamento restringe uso abusivo de IA, então convém registrar a política do time |
| Tipografia por plataforma | Poppins no mobile e Inter no web, sem decisão registrada sobre a divergência |

## Inconsistências encontradas nos repositórios

Estas não são decisões pendentes, e sim correções a fazer. Estão aqui porque apareceram durante a
consolidação das ADRs.

| Divergência | Onde corrigir |
|---|---|
| `docs/README.md` divulga Redis na stack, e o projeto não usa Redis | `docs/README.md` |
| `docs/README.md` aponta o protótipo em HTML como corrente e lista arquivos Markdown que não existem na pasta | `docs/README.md` — ver ADR-019 |
| Template de PR pede título no formato `tipo(ETI-XXX): descrição`; o CI exige `tipo/ETI-XXX-descricao` | `Vitryne/.github` |
| Cor primária `#B07A1F` no `tailwind.config.ts`, divergente da marca `#9530D9` | `frontend/tailwind.config.ts` — ver ADR-018 |
| `target/`, `.idea/` e `.DS_Store` versionados no `backend` | `backend/.gitignore` |

## Referências

- [MADR — Markdown Any Decision Records](https://adr.github.io/madr/)
- [Michael Nygard, *Documenting Architecture Decisions*](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Joel Parker Henderson — coleção de templates de ADR](https://github.com/joelparkerhenderson/architecture-decision-record)
- Regulamento Escola de TI 2026 — requisitos arquiteturais e artefatos exigidos
- Material da disciplina: [`endersonmenezes/talks`](https://github.com/endersonmenezes/talks/tree/main/courses/escola-de-ti), pasta `courses/escola-de-ti`
- ETI-200 Consolidar ADRs no repositório de documentação
