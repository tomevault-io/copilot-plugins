## sttotem

> App Android nativo Kotlin (AGP 8.7.3 / Gradle 8.10.2, minSdk 26 / targetSdk 35). Kiosk WebView que carrega `https://totemst.lovable.app/totem` com bridge JS (`AndroidBridge` + CustomEvents). Pagamento CliSiTef JNI; impressão térmica Gertec EasyLayer; backend Supabase Edge Functions (OkHttp, header `x-activation-token`, projeto `buviakhfibcsamucnjwu`). Ver `../CLAUDE.md` para princípios. Push é **manual** (não sincroniza com Lovable).

# CLAUDE.md — STTotem (app Android do totem, Gertec SK-210)

App Android nativo Kotlin (AGP 8.7.3 / Gradle 8.10.2, minSdk 26 / targetSdk 35). Kiosk WebView que carrega `https://totemst.lovable.app/totem` com bridge JS (`AndroidBridge` + CustomEvents). Pagamento CliSiTef JNI; impressão térmica Gertec EasyLayer; backend Supabase Edge Functions (OkHttp, header `x-activation-token`, projeto `buviakhfibcsamucnjwu`). Ver `../CLAUDE.md` para princípios. Push é **manual** (não sincroniza com Lovable).

## Build (NÃO há Java de sistema — use o JBR do Android Studio)
```
JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" ./gradlew :app:compileDebugKotlin   # verificação rápida
JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" ./gradlew :app:assembleDebug
```
Sem `JAVA_HOME` → "Unable to locate a Java Runtime". Daemon usa toolchain Java 21 (foojay). Testes são só templates — a verificação real é `compileDebugKotlin` + teste no totem.

## AndroidBridge (nativo → WebView)
- Exponha nativo ao WebView SÓ como `@JavascriptInterface` na inner class `AndroidBridge` (injetada como `window.AndroidBridge`). Métodos: getActivationToken, getDeviceSerial, clearActivation, isSitefAvailable, isFacePagAvailable, startFacePagLiveness/cancel, startSitefPayment, abortSitefPayment, enable/disable/isKioskMode.
- Resultado nativo → página SÓ via `evaluateJavascript` no UI thread disparando `window.dispatchEvent(new CustomEvent(...))` (payload escapado com `JSONObject.quote` + `JSON.parse` em IIFE try/catch). Eventos: `sitef-payment-result`, `sitef-message`, `sitef-qrcode`, `sitef-hide-qrcode`, `facepag-liveness-result`, `android-camera-error`, `totem-print-status`. Nunca chame função JS da página direto.
- (Lado web) NUNCA destaque método do bridge — invoque no próprio objeto.

## CliSiTef (pagamento) — padrões críticos de produção
- **Instância ÚNICA por processo (`by lazy`), NUNCA destruída/recriada** — a `libclisitef.so` mantém estado; recriar o objeto Java não reseta a lib e causa `configure() -12`.
- **Abort gracioso**: `abortTransaction(-1)` só SINALIZA; drene o laço respondendo `continueTransaction("")` a cada `onData` enquanto `isAborting`; deixe `onTransactionResult` fechar. NUNCA anule a instância / sleep cego / mate o processo durante a drenagem.
- **Reset de processo** (`restartForNativeReset`) é ÚLTIMO recurso: só se `configure()` deu -9/-12, ou o abort não produziu `onTransactionResult` em 10s (`WATCHDOG_ABORT_GRACE_MS`).
- **Guarda de prova**: sucesso do SDK com `nsuSitef`+`nsuHost`+`codAutorizacao` todos vazios → tratar como NÃO aprovado.
- Resultado único via `finalResultDispatched.compareAndSet`; libere `isTransactionActive` ANTES de despachar. Watchdog: 90s cartão / 320s PIX, rearmado a cada `onData`, cancela via abort gracioso.

## Impressão (EasyLayer)
- Após CADA operação (printHtml/printImage/scrollPaper/cutPaper) chame `waitUntilPrinterReady(stage)` (poll getStatus() 120ms, timeout 12s) e respeite os `Thread.sleep` calibrados. Rode jobs no `printExecutor` (single thread), nunca no main.
- Roteamento por `job.type`: `receipt`→printSummaryReceipt, `ticket`→printStyledTicket, `ingresso`→printIngresso.
- **QR do ingresso = BITMAP zxing** (`br.com.gertec.easylayer.zxing...QRCodeWriter`), px fixos `ingressoQrSizePx=348` iguais ao PrintConfig (escala 1:1), via `printImage` centralizado. NUNCA renderize o QR no HTML nem use `printBarcode`.
- **`qr_payload` CRU**: leia com `optRawString` (sem trim/normalize/uppercase) e imprima verbatim — validado na portaria. Descarte ingresso sem qr_payload.
- **`event_date`/`event_location`** já chegam formatados do totem-web: imprima verbatim com `PrinterUtils.sanitizeText`, NUNCA `formatDate`; omita linha se ausente.
- Todo texto dinâmico passa por `sanitizeText` + `truncate` + `escapeHtml`. Logo → data URI JPEG (html2bitmap da Gertec não renderiza WebP).

## Fila de impressão
- GET/POST em `{functionsBaseUrl}/totem-print-queue` com header `x-activation-token`; ordene jobs por `created_at` asc, unit_tickets por `(item_name, unit_number)`; UM job por vez (`isPrintingNow`); reporte `printing`→`printed`/`failed`. Parse defensivo: `nullIfBlank`/`optNullable*` (tratam ausente, vazio e `"null"` literal).

## Libs nativas / segredos / estilo
- Gertec/SiTef entram como `.aar`/`.jar` em `app/libs` (flatDir); `.so` em `jniLibs/{abi}`. **NUNCA remova `jniLibs.useLegacyPackaging=true` nem os abiFilters arm64-v8a/armeabi-v7a** (sem isso o SK-210 não faz dlopen das .so).
- OTP SiTef + `sitef_terminal_id` vêm do bootstrap → SharedPreferences `sttotem_prefs` (texto plano — dívida); leia via `LocalStorageManager`; recuse `startPayment` sem OTP.
- Nunca commite keystore (`*.jks` no .gitignore; release assinado por fora). Ao subir versão, **bump manual de `versionCode`** (o cache-bust `&v=` da WebView depende dele).
- Logs p/ Supabase são fire-and-forget (OkHttp `enqueue`, engula exceção, NUNCA dado sensível). Comentários/logs em pt-BR, tags MAIÚSCULAS (CLISITEF, PRINT_DEBUG, REPO_DEBUG, USB); comentários longos explicam incidentes — preserve.

## Armadilhas
- `.gradle/` e `.idea/` estão RASTREADOS no git — cuidado pra não commitar lixo de build (só os arquivos da tarefa).
- Código morto que confunde: `TotemLogger.kt` (não usado; MainActivity tem `sendLog` próprio). `SitefPaymentManager` (fluxo por Intent) é legado atrás de `USE_CLISITEF=true` e usa IP diferente do CliSitefManager — não confundir.
- Data class `SitefPaymentResult` está no arquivo de nome estranho `br.com.st.totem.payment.sitef.kt` — busque pelo símbolo.
- URL base do Supabase duplicada em 5 arquivos.

---
> Source: [GabinoxCreator/STTotem](https://github.com/GabinoxCreator/STTotem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
