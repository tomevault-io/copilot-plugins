## vibe-coder-kit

> Bu dosya opencode tarafından her oturumda otomatik okunur.

# OpenCode Agent Setup

Bu dosya opencode tarafından her oturumda otomatik okunur.

---

## Zorunlu İlk Adım

Her yeni oturumda **`.agent/skills/project-context-primer/SKILL.md`** oku ve uygula. Bu atlanmaz.

---

## Kritik Kurallar

### Güvenlik
- `DROP`, `DELETE`, `TRUNCATE`, `rm -rf`, `git push --force` (ana branch): **Onay al before running**
- Mevcut testler geçiyorsa ve görev o kodu doğrudan değiştirmiyorsa — **dokunma**
- Migration/production deploy öncesi: **rollback adımı tanımla**

### Kod Temizliği
- Yorumlar sadece fonksiyon/metot başlıklarında
- `console.log`, `console.warn`, `debugger`, `print()`: commit öncesi **kaldır**
- Hassas değerler (API key, token): **.env kullan**, kesinlikle koda yazma
- Yeni env variable eklendiğinde: **.env.example güncelle** (açıklama zorunlu)

### Kod Kalitesi
- Aynı kod 3+ yerde tekrar ediyorsa → **extract et**
- Yeni soyutlama öncesi: "3+ yerde kullanılıyor mu? 6 ay sonra anlaşılır mı?" → Hayır ise **bırak**
- Refactor: feature commit'inden ayrı, bağımsız commit

---

## İş Akışı

```
YENİ OTURUM
  └─ project-context-primer (zorunlu)

FİKİR     → brainstorming → SCOPE-<slug>.md
PLAN      → writing-plans → implementation_plan.md → architecture-review
GELİŞTİRME → test-driven-execution
KOMİT     → github skill (doğrudan git komutu ÇALISTIRMA)
           → code-review → conventional commit → PR
```

---

## Skill & Agent Yapısı

**.agent/** dizini:
- `skills/` — süreç tanımları (her biri SKILL.md ile)
- `agents/` — uzman persona tanımları
- `rules/` — zorunlu kurallar (code-hygiene, code-quality, safety)
- `knowledge/` — konuşmalar arası kalıcı bilgi

**Temel skill dosyaları:**

| Skill | Ne Zaman |
|-------|----------|
| `project-context-primer` | Her oturum başında — zorunlu |
| `github` | Tüm git işlemlerinde |
| `code-review` | Commit öncesi |
| `brainstorming` | Fikir/gereksinim netleştirilirken |
| `writing-plans` | Scope onaylandıktan sonra |
| `knowledge-base-update` | Öğrenilen her şeyde |

**Agent personaları:** Aria (UI/UX), Felix (Frontend), Bora (Backend), Deva (DevOps)

---

## Kritik Davranış

- **Belirsizlikte sor** — varsayım yapma
- **Kapsamı sessizce genişletme** — görev dışı sorun görürsen bildir
- **"Çalışıyor" ≠ "bitti"** — testler geçiyor + temiz kod + docs güncel
- **5+ dosya etkileniyorsa** — önce plan göster, onay al

---
> Source: [omergocmen/vibe-coder-kit](https://github.com/omergocmen/vibe-coder-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
