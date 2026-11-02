# Registro de Decisões Arquiteturais (ADRs) — Vitryne

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
| [ADR-001](ADR-001-registro-de-decisoes-em-adrs-versionadas.md) | Registrar decisões arquiteturais como ADRs versionadas no repositório | Aceita | Governança |
| [ADR-002](ADR-002-quatro-repositorios-por-aplicacao.md) | Separar o projeto em quatro repositórios por aplicação sob uma organização GitHub | Aceita | Governança |
| [ADR-003](ADR-003-processo-de-desenvolvimento-e-estrategia-de-branches.md) | Processo de desenvolvimento, estratégia de branches e definition of done | Aceita | Governança |
| [ADR-004](ADR-004-backend-como-monolito-java-spring-boot.md) | Backend como monólito único em Java 21 com Spring Boot 4 | Aceita | Back-end |
| [ADR-005](ADR-005-arquitetura-em-camadas-com-entidades-ricas.md) | Organizar o backend em camadas técnicas com regra de negócio nas entidades | Aceita | Back-end |
| [ADR-006](ADR-006-postgresql-com-flyway-e-schema-versionado.md) | PostgreSQL como banco único com schema versionado em Flyway | Aceita | Back-end |
| [ADR-007](ADR-007-dtos-em-records-separando-contrato-de-api.md) | Separar contrato da API das entidades JPA usando DTOs em records | Aceita | Back-end |
| [ADR-008](ADR-008-lombok-para-reduzir-boilerplate.md) | Adotar Lombok para reduzir código repetitivo no backend | Aceita | Back-end |
| [ADR-009](ADR-009-tratamento-de-erro-centralizado.md) | Centralizar tratamento de erro em RestControllerAdvice com exceções próprias | Aceita | Back-end |
| [ADR-010](ADR-010-nomenclatura-em-ingles-no-codigo-e-no-banco.md) | Nomenclatura em inglês no código e no banco, documentação em português | Aceita | Back-end |
| [ADR-011](ADR-011-autenticacao-stateless-com-jwt.md) | Autenticação stateless com JWT e filtro do Spring Security | Aceita | Back-end |
| [ADR-012](ADR-012-cors-liberado-no-desenvolvimento.md) | Liberar CORS para qualquer origem durante o desenvolvimento | Aceita | Back-end |
| [ADR-013](ADR-013-docker-compose-para-ambiente-local.md) | Padronizar o ambiente local com Docker e Docker Compose | Aceita | Infraestrutura |
| [ADR-014](ADR-014-frontend-nextjs-pages-router-axios.md) | Frontend web em Next.js com Pages Router e camada de serviço sobre Axios | Aceita | Front-end |
| [ADR-015](ADR-015-navegacao-react-navigation.md) | Navegação do app mobile com React Navigation em vez de Expo Router | Aceita | Mobile |
| [ADR-016](ADR-016-design-system-no-figma-como-base-do-prototipo.md) | Design System no Figma como base de componentes do protótipo | Aceita | Produto e UI |
| [ADR-017](ADR-017-prototipo-de-alta-fidelidade-no-figma.md) | Protótipo no Figma como fonte da verdade da interface para o desenvolvimento | Aceita | Produto e UI |
| [ADR-018](ADR-018-padrao-de-estilos-mobile.md) | Padrão de estilos do app mobile com StyleSheet, tokens compartilhados e nomes em snake_case | Aceita | Mobile |
| [ADR-019](ADR-019-organizacao-de-pastas-mobile.md) | Organização de pastas do app mobile por domínio, com telas e componentes em pasta própria | Aceita | Mobile |
| [ADR-020](ADR-020-padrao-de-nomes-mobile.md) | Nomes em inglês, com PascalCase em pastas e componentes e camelCase em arquivos comuns | Aceita | Mobile |
| [ADR-021](ADR-021-padrao-chamadas-api-mobile.md) | Padrão de chamadas de API do app mobile com Axios e conversão de DTO centralizada | Aceita | Mobile |
| [ADR-022](ADR-022-escolha-do-expo-mobile.md) | Base do app mobile com Expo em vez de React Native CLI | Aceita | Mobile |
| [ADR-023](ADR-023-icones-svg-mobile.md) | Ícones do app mobile com @expo/vector-icons e SVG próprio, sem ícones em PNG | Aceita | Mobile |
| [ADR-024](ADR-024-componentes-ui-proprios.md) | Componentes de UI próprios em vez de biblioteca de componentes pronta | Aceita | Front-end |
| [ADR-025](ADR-025-convencao-pr-ownership-ecossistema.md) | Convenção de PR e ownership compartilhados com o ecossistema Vitryne | Substituída por ADR-034 | Front-end |
| [ADR-026](ADR-026-turbopack-bundler.md) | Turbopack como bundler de desenvolvimento e build | Aceita | Front-end |
| [ADR-027](ADR-027-estado-wizard-cadastro.md) | Estado do cadastro em etapas controlado localmente por índice | Aceita | Front-end |
| [ADR-028](ADR-028-tokens-tema-tailwind-v4.md) | Tokens de tema no Tailwind CSS v4 via @theme no CSS global | Aceita | Front-end |
| [ADR-029](ADR-029-mascaras-react-imask.md) | Máscaras de campo com react-imask e estado local por campo | Aceita | Front-end |
| [ADR-030](ADR-030-cerimonias-e-artefatos-de-sprint.md) | Cerimônias do time e artefatos que cada uma produz | Aceita | Governança |
| [ADR-031](ADR-031-onde-cada-artefato-vive.md) | Um papel por ferramenta para decidir onde cada artefato do projeto vive | Aceita | Governança |
| [ADR-032](ADR-032-lideranca-por-frente-tecnica.md) | Liderança por frente técnica, com um integrante coringa e papéis de processo acumulados | Aceita | Governança |
| [ADR-033](ADR-033-camada-de-integracao-do-frontend-service-e-interfaces.md) | Separar a integração do front-end com a API em src/service e os contratos de dados em src/interfaces | Aceita | Front-end |
| [ADR-034](ADR-034-governanca-e-qualidade-de-pr-por-workflows-reutilizaveis-no-frontend.md) | Validar PRs do front-end com workflows reutilizáveis de governança e qualidade e dono de código por time | Aceita | Front-end |
| [ADR-035](ADR-035-deploy-do-frontend-na-vercel-por-github-actions.md) | Publicar o front-end na Vercel por GitHub Actions, com ambientes de preview, homologação e produção | Aceita | Front-end |

Próximo ID livre: **ADR-036**.


## Fila de decisões ainda não registradas

### Decisões que o projeto ainda não tomou

| Tema | Por que precisa de decisão | Referência |
|---|---|---|
| Escala de 1M escritas para 2M leituras | Exigência do regulamento; define estratégia de cache, réplica de leitura e índices | Regulamento, requisito (d) |
| Cache | Redis aparece na stack divulgada no `docs/README.md` e não existe no `pom.xml`; decidir se entra, onde e com que política de invalidação | Regulamento, requisito (d) |
| Websocket | Exigência do regulamento: pelo menos um caso de uso; falta escolher qual — acompanhamento de pedido é o candidato natural | Regulamento, requisito (e) |
| Integração de pagamento | Exigência do regulamento em ambiente de homologação; gateway a escolher, com PIX previsto no produto | Regulamento, requisito (f) |
| Plataforma de deploy do back-end | O front-end publica na Vercel (ADR-035). Falta o back-end: a ADR-021 já cita o Render como destino, sem decisão registrada | Regulamento, requisito (g) |
| Estratégia de testes automatizados | Exigência do regulamento: unitários e de integração. Hoje há apenas o teste de contexto do Spring, e nem o front nem o mobile têm teste | ETI-123, ETI-159, ETI-160 |
| Integração contínua | No front-end há governança e qualidade por workflows reutilizáveis (ADR-034). Falta confirmar o que o `node-quality.yml` executa, incluir teste automatizado e cobrir back-end e mobile | ADR-003, ADR-034 |
| Versionamento da API | Prefixo de versão e política de mudança incompatível; hoje os endpoints não têm versão | ADR-007 |
| Paginação e ordenação | `listProducts` devolve a lista inteira; catálogo não escala assim | `ProductController.java` |
| Formato de erro padronizado | Avaliar `ProblemDetail` (RFC 7807) no lugar do `Map` montado à mão | ADR-009 |
| Geolocalização e busca por proximidade | Função central do produto, sem decisão de tecnologia nem de modelagem | Casos de uso |
| Observabilidade | Hoje há log em controller; falta decidir formato, nível e destino | — |
| Valores do Design System no código | Como manter cor, espaçamento e raio iguais entre o Figma, o `@theme` do `global.css` e o `commonStyles.ts` do mobile, sem detecção automática de divergência | ADR-016, ADR-018, ADR-028 |
| Ligação entre componente do Figma e componente do código | Hoje a correspondência é manual e não é verificável | ADR-016 |
| Composição do payload do cadastro do lojista | O estado de cada etapa não sobe para `Register`, então não há como montar o envio final | ADR-027, ADR-029 |
| Validação de formulário no front | Sem biblioteca e sem regra de obrigatoriedade ou formato; CNPJ e CEP inválidos passam | ADR-029 |
| Acessibilidade | Contraste, alvo de toque e leitor de tela sem critério; `SelectUf` é um dropdown próprio sem `role`, `aria-*` nem navegação por teclado | ADR-024 |
| Expiração e revogação de token | Tempo de vida, refresh token e o que fazer para invalidar antes do vencimento | ADR-011 |
| Publicação do app iOS | A ADR-022 entrega só Android por APK; o iOS fica como melhoria futura, sem decisão de quando | ADR-022 |
| Licenciamento do código | `backend` e `frontend` têm LICENSE MIT; `mobile`, `docs` e `.github` não têm licença nenhuma | repositórios |
| Política de uso de IA no desenvolvimento | `AGENTS.md`, `CLAUDE.md` e `.claude/settings.json` existem só no `mobile`; o regulamento restringe uso abusivo de IA e de código de terceiros | `mobile/AGENTS.md` |

### Decisões já implementadas que ainda não têm ADR

« nenhuma registrada no momento »

## Inconsistências encontradas nos repositórios

Correções a fazer nos repositórios, encontradas durante a consolidação das ADRs.

- A Evidência da ADR-014 cita `frontend/src/hooks/use-http.ts`, mas o arquivo está em `frontend/src/components/hooks/use-http.ts`. A ADR-014 não se edita; o caminho correto consta na ADR-033.
- ADR-014: o caminho do `use-http.ts` na Evidência está errado, e a versão do `next` citada (15.5.18) está velha (15.5.28 no `package.json`). A lista de páginas cita 4 e o repositório tem 17.
- ADR-024: os componentes citados em `frontend/src/components/` estão em `frontend/src/components/ui/`, e o `button.tsx` não existe (há `button-primary.tsx`, `button-secondary.tsx` e `button-google.tsx`).
- ADR-026: a versão do `next` citada na Evidência (15.5.18) está velha.
- ADR-027: o cadastro tem 5 etapas, não 6 (sem o primeiro produto, e em outra ordem), e o `Component` é renderizado sem props, com o `onNext` no `StepFooter`. Os caminhos de evidência (`steps-register/`, `sidebar-register.tsx`) não existem; os arquivos estão em `components/features/store-register/`.
- ADR-028: a validação "nenhuma cor hex fora do `global.css`" é violada em `frontend/src/pages/login.tsx:26` (`to-[#3B0F63]/90`). O `--font-mono` (Geist Mono) existe no `@theme` e a ADR não o cita.
- ADR-029: o `MaskedInput` está em `frontend/src/components/ui/masked-input.tsx`, e o `StepStore` em `frontend/src/components/features/store-register/components/steps/step-store.tsx`, não nos caminhos da Evidência.

## Referências

- [MADR — Markdown Any Decision Records](https://adr.github.io/madr/)
- [Michael Nygard, *Documenting Architecture Decisions*](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Joel Parker Henderson — coleção de templates de ADR](https://github.com/joelparkerhenderson/architecture-decision-record)
- Regulamento Escola de TI 2026 — requisitos arquiteturais e artefatos exigidos
- Material da disciplina: [`endersonmenezes/talks`](https://github.com/endersonmenezes/talks/tree/main/courses/escola-de-ti), pasta `courses/escola-de-ti`
- ETI-200 Consolidar ADRs no repositório de documentação
