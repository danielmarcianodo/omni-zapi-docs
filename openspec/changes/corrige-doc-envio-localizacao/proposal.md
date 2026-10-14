# Proposal

## Why

A página de envio de localização abre com um aviso que diz que o endpoint `POST /v1/channels/{channelId}/messages` não entrega localização em nenhum caso (`messages/send-location.mdx:9-15`, e o mesmo bloco em `en/` e `es/`). O aviso cita a resposta `json: cannot unmarshal string into Go struct field SendMessageInput.latitude of type float64` para coordenadas em texto, e não entrega para coordenadas em número.

Esse erro não é deste endpoint. Ele foi anotado testando a rota de compatibilidade Z-API, com o corpo copiado da Z-API (`PONTOS.md:28`: "Não funciona copiando body exato da Z-API"). O texto passou para `BACKEND-MENSAGENS.md:13-18` como se valesse para o envio nativo, e de lá para a página no commit `4fb9413`. A página original, do commit `a3ff138`, não tinha aviso.

Medido no código do endpoint nativo:

- a borda repassa o corpo sem conversão (`pennsylvania-frisco`, `SendMessageInteractor.java:50-51`, lido por quem pediu a mudança);
- o serviço de envio declara `latitude` e `longitude` como `String` (`arkansas-barling` `origin/main-phoenix` `64d7d9d` e `origin/develop-phoenix` `1391c97`, `src/main/java/com/arkansas/barling/core/boundary/input/message/MessageAttachmentInput.java:28-29`), então texto é aceito;
- número também é aceito: o projeto usa Spring Boot 4.1.1 (`build.gradle`), que traz Jackson 3.1.5 (`spring-boot-dependencies-4.1.1.pom:83`), sem `ObjectMapper` próprio para o HTTP e sem configuração de coerção (`application.yaml:27-29`; nenhuma ocorrência de `coercion` no `spring-boot-jackson-4.1.1.jar`). Um teste com `JsonMapper` do Jackson 3.1.5 sobre uma classe com os mesmos campos `String` desserializou `-23.0696347` e `-23` como `"-23.0696347"` e `"-23"`. É indício, não prova: nenhuma chamada de ponta a ponta com número foi feita, então a página diz só "em texto, como no exemplo".

Por canal, o que acontece com a localização depois disso (`arkansas-fouke` `origin/develop-phoenix` `ed4c9e1`):

- **Meta Cloud API**: mensagem de localização com coordenadas, `name` e `address` (`fouke-meta-whatsapp/.../MessageLocationProcessorInteractor.java:43-48`).
- **WhatsApp Web**: o envio monta o `send-location` com `title`, que hoje é sempre nulo, e não com `name` (`fouke-zapi-whatsapp/.../MessageLocationProcessorInteractor.java:33-42`). O nome do lugar não vai. O que a Z-API faz com `title` nulo, se aceita e se entrega as coordenadas e o endereço, não foi medido: é a tarefa 3.2 do `irrahgroup/arkansas-barling#200`, que propõe a correção. Até lá, a página não promete a entrega.
- **Instagram**: em produção (`arkansas-barling` `origin/main-phoenix`), não há recusa na borda (`git grep ValidateChannelContentTypeService origin/main-phoenix` sem resultado), então a chamada é aceita, e o `fouke-meta-instagram` não entrega o tipo (`MessageUnsupportedTypeProcessorInteractor.java:25-28`). Em `origin/develop-phoenix` há recusa com `422 UNSUPPORTED_CHANNEL_TYPE` (`ValidateChannelContentTypeService.java:41-45`). O que o código garante nos dois é que a mensagem não é entregue, e é isso que a página diz.
- **Messenger**: não existe mensagem de localização; vai um texto com o nome (ou "Localização"), o endereço e `https://www.google.com/maps?q=<lat>,<lng>` (`fouke-meta-messenger/.../MessageLocationProcessorInteractor.java:41-54`).
- **Telegram**: `sendLocation` só com as coordenadas; nome e endereço não vão (`fouke-telegram/.../SendLocationMessageTelegramInteractor.java:40-46`).

O dono do produto aprovou a correção em 14/10/2026.

## What Changes

- `{,en/,es/}messages/send-location.mdx`: o aviso falso sai. No lugar, um aviso só da limitação que existe, a do nome do lugar no WhatsApp Web, ainda com o rótulo **Em implementação.** (ver Decisão abaixo). O aviso não diz que as coordenadas e o endereço chegam, por decisão do dono em 14/10/2026, até a tarefa 3.2 do barling #200 medir. A conceituação ganha o formato das coordenadas: em texto, como no exemplo.
- `snippets/channel-support.mdx`, notas de `LOCATION`: a nota única de Instagram e Messenger vira uma nota por canal, para WhatsApp Web, Instagram, Messenger e Telegram, dizendo o que chega em cada um. No Instagram, a nota diz que a mensagem não é entregue, mesmo quando a chamada é aceita, sem prometer recusa nem código HTTP. A matriz de marcas não muda.
- `{pt,en,es}/messages/openapi-location.json`: a descrição de `latitude` e `longitude` diz que o valor vai em texto, como no exemplo. O `type` continua `string`, que é a forma do exemplo.

### Decisão: o rótulo de estado continua

O linter (`scripts/check-docs.py:277-280` e `:338-367`) amarra o rótulo **Em implementação.** ao estado `broken` do tipo, e esse estado vem do catálogo do painel (`pennsylvania-lancaster`, `src/config/playground-methods.ts:181`, `status: 'broken'`). `broken` quer dizer "existe e falha ou entrega menos" (`scripts/check-docs.py:282-289`), e o WhatsApp Web entrega menos: sem o nome do lugar. O estado continua `broken` aqui e no painel até a correção do `arkansas-barling#200` subir, e o rótulo continua, agora restrito ao que de fato falta.

### Fora de escopo

- **A matriz de canais** (marcas ✅/❌). O Messenger continua ❌: não existe mensagem de localização, e o painel (`playground-methods.ts:182`, `NOT_META_SOCIAL`) não oferece Messenger para esse tipo. Mudar a marca é mudança coordenada com o painel. A nota do Messenger diz o que a API manda no lugar.
- **O código `422` do Instagram**. A recusa na borda existe em `origin/develop-phoenix` do `arkansas-barling` e não em `origin/main-phoenix` (`git grep ValidateChannelContentTypeService origin/main-phoenix` sem resultado). A página diz que o envio é recusado, sem prometer o status HTTP.
- **A rota de compatibilidade Z-API `send-location`**. A seção de compatibilidade não tem página dela (`docs.json:340-357` e `ls z-api/messages`). A correção da rota é a change `envio-de-localizacao` do `irrahgroup/pennsylvania-reading#22`, e a tarefa 3.4 dela publica aqui a página `z-api/messages/send-location` (ou uma nota em `z-api/limitations.mdx`) com o formato aceito e as respostas 400, depois do deploy em produção.
- **A retirada do rótulo e do aviso do WhatsApp Web**. É a tarefa 4.2 do `irrahgroup/arkansas-barling#200`, junto com o lancaster.
- **`PONTOS.md` e `BACKEND-MENSAGENS.md`**. São notas de backlog, não páginas do menu.
- **A descrição do endpoint no OpenAPI** ("Funciona apenas quando a janela de conversa de 24 horas estiver aberta"), que só vale para a Meta Cloud API. É assunto de todas as páginas de envio, não só desta.

## Capabilities

### New Capabilities

- `messages/send-location`: o que a página de envio de localização diz sobre o funcionamento do endpoint, o formato das coordenadas e o que chega em cada canal.

### Modified Capabilities

<!-- Nenhuma. -->

## Impact

| Arquivo | Mudança |
|---|---|
| `{,en/,es/}messages/send-location.mdx` | aviso trocado; parágrafo de coordenadas |
| `snippets/channel-support.mdx` | notas de `LOCATION`, nos três idiomas |
| `{pt,en,es}/messages/openapi-location.json` | descrição de `latitude` e `longitude` |

**Sem impacto**: `docs.json`, a matriz de marcas, as outras páginas de envio, a seção de compatibilidade Z-API.

## Rastreabilidade

Sem task no ClickUp. Pedido direto, com aprovação do dono do produto em 14/10/2026. Relacionado: `irrahgroup/arkansas-barling#200` (nome do lugar no WhatsApp Web; a tarefa 3.2 dele roda antes do merge deste PR) e `irrahgroup/pennsylvania-reading#22` (rota compatível, com a tarefa 3.4 de documentação).

```
branch   corrige-doc-envio-localizacao
título   fix: corrige a página de envio de localização
```
