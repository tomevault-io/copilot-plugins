## kompra-justa-nfce-api

> Esta é a **API backend serverless** do app **Kompra Justa** (projeto irmão em `../kompra-justa`).

# AGENTS.md - Kompra Justa NFC-e Parser API

## Contexto do projeto

Esta é a **API backend serverless** do app **Kompra Justa** (projeto irmão em `../kompra-justa`).

- **Função**: Receber URL de QR Code de NFC-e, consultar a SEFAZ e retornar os dados estruturados em JSON.
- **Runtime**: Node.js 20.x na AWS Lambda.
- **Região AWS**: `sa-east-1`.
- **Endpoint atual**: `POST https://scb4lft3c8.execute-api.sa-east-1.amazonaws.com/nfce/parse`
- **Handler**: `dist/handler.handler` (compilado de `src/handler.ts`).

## Arquitetura

```
App (Capacitor/Vue)  -->  API Gateway  -->  Lambda (this)  -->  SEFAZ
                                      (axios + cheerio)
```

- A Lambda faz scraping do HTML da página de consulta pública da NFC-e.
- Não usa nenhuma API oficial da SEFAZ — faz HTTP GET na mesma URL que o QR Code aponta.
- A resposta é parseada com Cheerio.

## Como fazer deploy na AWS

**NUNCA** faça commit/push/deploy sem confirmação explícita do usuário.

O comando único para gerar o pacote de deploy:

```bash
npm run package
```

Isso executa:
1. `npm run clean` — remove `dist/` e `lambda.zip`
2. `npm run build` — compila TypeScript (`tsc`)
3. `npm run zip` — cria `lambda.zip` contendo `dist/`, `node_modules/`, `package.json`

O deploy na AWS é **manual** via console:
- Console AWS > Lambda > Função `nfce-parser`
> Código > Upload from > .zip file
> Selecionar `lambda.zip`

**Nota**: O comando `zip` usa PowerShell (`Compress-Archive`). Funciona no Windows. Se o ambiente for Linux/Mac, pode ser necessário usar `zip -r lambda.zip dist node_modules package.json`.

## Convenções de código

- TypeScript com `strict: false` (tsconfig.json).
- `module: "CommonJS"` (necessário para Lambda).
- Estilo simples, sem frameworks pesados.
- Todos os parsers devem retornar um objeto com `{ marketName, purchaseDate, total, items }`.
- Tratar erros da SEFAZ com status codes HTTP semânticos (502, 503, 504), **nunca** 500 genérico.
- Sempre logar erros com `console.error` para CloudWatch.

## Estrutura de arquivos

```
src/
  handler.ts          # Único arquivo de código-fonte (handler + parsers + helpers)
dist/
  handler.js          # Compilado pelo tsc
  handler.d.ts        # Declarações de tipo
```

**Não** crie múltiplos arquivos `.ts` a menos que o usuário peça explicitamente. O projeto é pequeno e intencionalmente monolítico.

## Dependências

- `axios` — requisições HTTP (timeout 15s, headers de browser realista).
- `cheerio` — parsing HTML (jQuery-like, server-side).
- `@types/aws-lambda` — tipos da Lambda (dev).
- `typescript` — compilação (dev).

## Variáveis de ambiente

Não há variáveis de ambiente obrigatórias na Lambda. A URL da API está hardcoded no front-end (`VITE_NFCE_API_BASE_URL`).

## Possíveis problemas conhecidos

1. **SEFAZ bloqueia AWS**: A SEFAZ-SP (e outras) frequentemente bloqueiam ou limitam requisições de IPs de datacenter. Erros 503/429 são comuns.
2. **HTML muda**: Se a SEFAZ alterar o layout da página, o parser `parseSP` pode quebrar. Erros 502 com "layout pode ter mudado" indicam isso.
3. **Nota não disponível**: Notas emitidas há poucos minutos podem retornar 404.
4. **QR Code com pipe literal (`|`)**: O handler normaliza automaticamente para `%7C`.

## Relação com o front-end

- O front está em `../kompra-justa`.
- Service que chama esta API: `src/services/nfce.service.ts`.
- Composable que gerencia o scan: `src/composables/usePurchaseScanner.ts`.
- Sempre que alterar status codes ou formato de resposta da API, verifique se o front trata corretamente.

## Regras

- Nunca altere a estrutura de resposta de sucesso (`{ marketName, purchaseDate, total, items }`) sem verificar o front.
- Nunca remova headers CORS (`Access-Control-Allow-Origin: *`).
- Nunca commite/push na branch principal sem confirmação do usuário.

---
> Source: [Mateuss18/kompra-justa-nfce-api](https://github.com/Mateuss18/kompra-justa-nfce-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
