# Tasks

## 1. Páginas

- [x] 1.1 Trocar o aviso de `{,en/,es/}messages/send-location.mdx` pelo aviso da limitação do WhatsApp Web, mantendo o rótulo de estado de cada idioma
- [x] 1.2 Acrescentar à conceituação das três páginas o formato das coordenadas: texto, como no exemplo, e número também aceito
- [x] 1.3 Verificar com `grep -rn "unmarshal\|não há caminho\|no working path\|no hay camino" messages en/messages es/messages` que nada sobrou

## 2. Notas por canal

- [x] 2.1 Em `snippets/channel-support.mdx`, trocar a nota única de `LOCATION` por notas de WhatsApp Web, Instagram, Messenger e Telegram, nos três idiomas
- [x] 2.2 Verificar com `git diff snippets/channel-support.mdx` que a linha `LOCATION` da `MATRIX` não mudou

## 3. OpenAPI

- [x] 3.1 Acrescentar à descrição de `latitude` e `longitude` dos três `openapi-location.json` que o valor vai como texto e que número também é aceito, sem mudar `type` nem `example`
- [x] 3.2 Validar os três JSON com `python3 -m json.tool`

## 4. Verificação

- [x] 4.1 Buscar `reading`, `frisco`, `barling`, `fouke`, `Go struct` e `Jackson` nos arquivos alterados fora de `openspec/`: vazio
- [x] 4.2 Rodar `python3 scripts/check-docs.py` e confirmar `check-docs: tudo certo`. Executado à mão por quem implementa: o repositório não tem `.github/workflows/`, então não há execução automática em PR
- [x] 4.3 Rodar `mint broken-links` e confirmar que nenhum link novo quebrou. Executado à mão por quem implementa
- [x] 4.4 Subir `mint dev` e conferir no HTML servido das três páginas o aviso novo, o parágrafo das coordenadas e as quatro notas; encerrar o servidor
