---
id: ADR-057
titulo: Guardar as imagens no Cloudinary, enviadas pelo back-end e registradas no banco para limpeza das sem uso
status: Proposta
data: 2026-11-27
autor: João Ehlers
decisores: João Ehlers
area: Back-end
substitui: —
substituida-por: —
---

# ADR-057 — Guardar as imagens no Cloudinary, enviadas pelo back-end e registradas no banco para limpeza das sem uso

## Contexto e problema

O produto guarda as fotos como uma lista de URLs (`product.photo_urls`, `text[]`), e a primeira é a
capa no carrinho e no checkout. Não havia upload: o portal do lojista gerava uma URL `blob:`, válida
só no navegador, e o `product-mapper.ts` a trocava por uma imagem do placehold.co. Na prática, todo
produto ficava com uma única foto de exemplo.

O back-end vai para o Render no plano gratuito (ver "Plataforma de deploy do back-end" na fila), que
apaga o disco a cada redeploy ou hibernação. Gravar o arquivo no servidor perderia as imagens.

A pergunta é onde as imagens ficam guardadas, como chegam lá e o que acontece com as que são enviadas
e nunca usadas. Esse tema estava na fila como "Armazenamento de imagens".

## Critérios de decisão

- As imagens sobrevivem a redeploy e reinício do servidor
- Sem custo e sem cartão de crédito no plano gratuito
- Credenciais só em variável de ambiente
- A validação do arquivo e a permissão (só a loja dona) ficam no back-end
- Imagens sem uso não se acumulam indefinidamente
- Pouco código e pouca configuração para um time de TCC

## Alternativas consideradas

1. **Cloudinary, com o upload passando pelo back-end** — escolhida.
2. **Armazenamento compatível com S3 (Supabase Storage ou Cloudflare R2)** — *perdeu porque:* o
   Supabase oferece 1 GB no plano gratuito, e o R2 pede cartão de crédito. Nenhum dos dois entrega CDN
   com redimensionamento sem configuração extra. Continua sendo uma opção forte se o Cloudinary ficar
   pequeno ou se o time quiser fugir da dependência de um fornecedor, já que a API S3 é padrão.
3. **Guardar os bytes no PostgreSQL e servir por um endpoint da API** — *perdeu porque:* incha o banco
   (o Postgres gratuito do Render tem 1 GB e expira em 30 dias), toda imagem passaria pela API, que
   hiberna no plano gratuito, e não há CDN.
4. **Upload direto do portal para o Cloudinary, com assinatura gerada pelo back-end** — *perdeu
   porque:* tira do back-end a validação de tipo e tamanho e o registro de quem enviou, e exige mais
   código no portal. Vale reabrir se o tráfego de upload pela API virar gargalo.
5. **Disco do servidor** — *perdeu porque:* o Render apaga o disco no redeploy.

## Decisão

As imagens ficam no Cloudinary. O back-end lê a credencial da variável `CLOUDINARY_URL` e, se ela não
estiver definida, sobe normalmente, mas o upload responde 503. A variável `CLOUDINARY_FOLDER`, que é
opcional e vale `vitryne` por padrão, separa os ambientes.

- **Envio:** `POST /uploads/images` (perfil STORE, multipart, campo `files`) recebe de 1 a 8 imagens
  JPG, PNG ou WEBP de até 5 MB e devolve as URLs na ordem enviada. O tipo é conferido pela assinatura
  do arquivo, não pela extensão. O envio é tudo ou nada: um arquivo inválido barra todos, e uma falha
  no meio apaga o que já tinha subido. As imagens vão para `<pasta>/stores/<storeId>/products/`.
- **Registro:** cada imagem enviada vira uma linha em `uploaded_image`, com a loja, o produto (vazio
  enquanto não é usada) e `orphaned_since`, que marca desde quando a imagem está sem uso.
- **Uso no produto:** o cadastro e a edição aceitam no `photoUrls` só URLs enviadas pela própria loja
  e que não estejam em outro produto, até 8 fotos, sem repetição. A primeira continua sendo a capa.
  As URLs externas que um produto já tinha antes desta decisão podem ser mantidas na edição, mas não
  adicionadas.
- **Liberação:** a foto removida na edição e as fotos de um produto excluído ficam sem uso.
- **Limpeza:** uma rotina agendada (a cada 1 hora, ajustável por `IMAGE_ORPHAN_CLEANUP_INTERVAL`) apaga
  do Cloudinary e do banco as imagens sem uso há mais de 24 horas (`IMAGE_ORPHAN_RETENTION`), em lotes
  de 100. A imagem sai primeiro do Cloudinary e depois do banco: se o Cloudinary falhar, o registro
  fica e a imagem é tentada de novo na próxima execução.

A rota é genérica de propósito, para depois atender o logo e o banner da loja, que ainda usam
imagens de exemplo no cadastro (fora desta decisão).

## Consequências

### O que ganhamos

- As fotos reais aparecem no catálogo, no carrinho e no checkout, até 8 por produto
- As imagens sobrevivem ao redeploy e são entregues por CDN
- Um produto não pode usar imagem de outra loja nem URL externa arbitrária
- Imagem enviada e abandonada não fica para sempre no armazenamento
- A troca de fornecedor fica concentrada na interface `ImageStorage`

### O que aceitamos em troca

- Dependência de um fornecedor externo e dos limites do plano gratuito do Cloudinary
- O upload passa pela API: até 40 MB por requisição ocupam a memória e a banda do servidor
- Uma imagem sem uso existe por até cerca de 25 horas antes de ser apagada
- Produtos antigos com URL do placehold.co continuam com ela até alguém trocar as fotos
- Se a imagem sair do Cloudinary e o banco falhar logo em seguida, o registro fica apontando para uma
  imagem que não existe mais até a próxima limpeza, que tenta apagar de novo e remove o registro
- Sem `CLOUDINARY_URL` no ambiente publicado não há upload; a variável precisa entrar na configuração
  do Render

## Validação

O `ProductImageIntegrationTest` cobre o envio, a recusa de URL externa e de foto de outra loja, o
limite de 8 fotos, a liberação na edição e na exclusão e a limpeza. Um `grep -r "placehold.co"` em
`backend/src/main` não retorna nada.

## Evidências

- `backend/src/main/java/com/vitryne/api/image/controller/ImageUploadController.java` — rota de upload
- `backend/src/main/java/com/vitryne/api/image/service/ImageService.java` — envio e vínculo com o produto
- `backend/src/main/java/com/vitryne/api/image/service/OrphanImageCleanupJob.java` — limpeza
- `backend/src/main/java/com/vitryne/api/image/storage/CloudinaryImageStorage.java` — ponto de troca do fornecedor
- Migration `ETI-431-create_uploaded_image.sql`
- ETI-431 — upload de fotos de produto
