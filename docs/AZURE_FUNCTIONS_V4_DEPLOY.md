# Compatibilidade Azure Functions 4 — I Blue It V5

## Escopo
Patch mínimo de recuperação: manter a API Node.js com `index.js` e `function.json` (modelo de programação v3), rotas e dependências originais. A única alteração funcional é `host.json`: Extension Bundle `[4.0.0, 5.0.0)`. Não se afirma compatibilidade completa até testar API e banco.

## Antes de mesclar o PR
Na Azure Function App `iblueit-server-side`:
1. Confirme **Azure Functions Runtime 4.x** com App Setting `FUNCTIONS_EXTENSION_VERSION=~4`.
2. Configure `FUNCTIONS_WORKER_RUNTIME=node`.
3. Configure o **runtime Node.js 22 no serviço** (o Node 22 do GitHub Actions altera somente o build). Em Windows, configure a versão na stack e verifique `WEBSITE_NODE_DEFAULT_VERSION=~22`. Em Linux, configure a stack Node 22 / `linuxFxVersion` (conforme plano e OS; exemplo CLI `az functionapp config set -g <RESOURCE_GROUP> -n iblueit-server-side --linux-fx-version 'node|22'` para planos compatíveis).
4. Verifique `AzureWebJobsStorage` com armazenamento válido.
5. Configure `MongoDbAtlas` com a connection string real do Atlas em **App settings**, nunca em código, GitHub ou frontend. A conta de banco é diferente da conta Atlas.
6. Em Atlas **Network Access**, autorize os IPs de saída da Function App ou rede privada; evite abertura global permanente.
7. Ajuste CORS para a origem HTTPS do frontend, se necessário.
8. Se usadas, verifique as demais configurações originais como `URL_API_IA`.

## Deploy correto
O workflow existente só publica em push para `main` ou acionamento manual pelo `workflow_dispatch`.
**Não use Re-run jobs no run antigo:** ele repetirá o commit antigo com bundle 1.x.
Após revisar a configuração na Azure, mescle o PR em `main`. Isso inicia um **novo deploy**. Acompanhe Actions, Function App → Functions, e Log Stream.

## Verificação inicial
- Bundle 1.8.1 deixou de aparecer como erro de inicialização.
- As funções HTTP aparecem na Azure.
- `POST /api/login` com JSON vazio `{}` retorna 400 por validação do código antigo (teste de execução, **não** valida MongoDB).
- Testar com credenciais de aplicação válidas e, depois, leitura `GET /api/pacients?sort=asc` com `GameToken`, evitando operações que escrevam na base durante o diagnóstico.
- Conferir versão MongoDB do cluster: Mongoose 5 ainda é legado. O sucesso do build não comprova compatibilidade com Node 22 nem Atlas.

## Fora do escopo
Atualizar Mongoose, refatorar conexões, alterar endpoints ou migrar o modelo de programação Node.js v3→v4. Essas mudanças deverão ocorrer em outra branch com testes.
