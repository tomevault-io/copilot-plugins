## langchain-examples

> LangChain JS (v1, `createAgent`) için Türkçe eğitim reposu. Basitten zora 12 modül; her modül gerçek bir dünya use case'ine eşlenir. GitHub'da paylaşılacak.

# langchain-examples

LangChain JS (v1, `createAgent`) için Türkçe eğitim reposu. Basitten zora 12 modül; her modül gerçek bir dünya use case'ine eşlenir. GitHub'da paylaşılacak.

## Dil Kuralları (önemli)

- Tüm klasör, dosya, fonksiyon ve değişken adları **İngilizce**.
- Yalnızca README metinleri ve kod içi yorumlar **Türkçe**.
- İleride İngilizce çeviri eklenebilir yapıda tut.

## Teknik Kararlar

- LLM sağlayıcı: **Google Gemini** (`@langchain/google-genai`) — ücretsiz API katmanı, okuyucu kartsız çalıştırabilir.
- TypeScript + `tsx`; her demo CLI'dan çalışır: `npx tsx examples/NN-*/index.ts` (kısayol: `npm run NN`).
- Bağımlılıklar: `langchain@^1`, `@langchain/core`, `@langchain/google-genai`, `zod`, `dotenv`; dev: `typescript`, `tsx`.
- `package.json`: `"type": "module"`; `tsconfig.json`: strict, NodeNext, ES2022.
- API anahtarı `.env` içindeki `GOOGLE_API_KEY`'den okunur; anahtar yoksa anlaşılır Türkçe hata ver.
- RAG modüllerinde harici DB yok: `MemoryVectorStore` + repo içindeki örnek `.md` doküman.
- Kod kısa tutulur (~50–120 satır), tek dosya, bol Türkçe yorum.

## Repo Yapısı

```
├── README.md                 # Türkçe ana sayfa: tanıtım, kurulum, müfredat tablosu
├── package.json / tsconfig.json / .env.example / .gitignore / LICENSE (MIT)
└── examples/
    ├── 01-hello-langchain/
    │   ├── README.md         # konu anlatımı (TR) + "Gerçek Dünya Karşılığı"
    │   └── index.ts          # çalıştırılabilir demo
    └── ... (aynı şablon)
```

Her modül README şablonu: **Ne öğreneceksin** → **Kavram anlatımı** → **Gerçek Dünya Karşılığı** → **Çalıştır** → **Beklenen çıktı** → **Kendin dene**.

## Müfredat (basit → zor)

| # | Klasör | LangChain kavramı | Gerçek dünya use case |
|---|--------|-------------------|----------------------|
| 01 | 01-hello-langchain | model başlatma, `invoke`, mesajlar | Müşteri e-postasını özetleyen araç |
| 02 | 02-prompt-templates | system prompt, mesaj rolleri, şablonlar | E-ticaret ürün açıklaması üretici |
| 03 | 03-structured-output | Zod şeması ile yapılandırılmış çıktı | Faturadan JSON alan çıkarma |
| 04 | 04-tools | `tool()` tanımlama, tool calling | Döviz kuru asistanı (sahte API) |
| 05 | 05-first-agent | `createAgent`: model + tools + systemPrompt | Sipariş takip destek asistanı |
| 06 | 06-streaming | `stream()`, token/adım akışı | Canlı yazıyor-efektli CLI sohbet |
| 07 | 07-memory | checkpointer, çok turlu konuşma, thread'ler | Bağlamı hatırlayan destek sohbeti |
| 08 | 08-rag-basics | embeddings, `MemoryVectorStore`, retriever | Şirket SSS'inden soru-cevap |
| 09 | 09-rag-agent | retrieval'ı tool olarak veren agentic RAG | Bilgi bankası asistanı |
| 10 | 10-middleware | guardrail/middleware, girdi-çıktı denetimi | PII maskeleme + konu dışı red |
| 11 | 11-multi-agent | subagent / supervisor deseni | Araştırmacı + yazar rapor üretimi |
| 12 | 12-final-project | hepsinin birleşimi | E-ticaret destek botu: RAG + tools + hafıza + guardrail |

Zorluk eğrisi: 01–03 sadece model API'si (agent yok), 04–07 tek agent yetkinlikleri, 08–09 RAG, 10–12 üretim desenleri.

## Doğrulama

- `npm install && npx tsc --noEmit` temiz geçmeli.
- `.env`'de `GOOGLE_API_KEY` ile `npm run 01` … `npm run 12` sırayla, hatasız ve README'deki beklenen davranışla çalışmalı.

## Kapsam Dışı (şimdilik)

İngilizce çeviri, web UI, LangGraph düşük seviye API, MCP, deployment — README yol haritasında "gelecek" olarak notlanır.

## Süreç Notları

- Push/GitHub repo oluşturma yalnızca kullanıcı isteyince yapılır.
- Commit'ler anlamlı gruplar halinde (iskelet, sonra modül grupları).

---
> Source: [gulbaki/langchain-examples](https://github.com/gulbaki/langchain-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
