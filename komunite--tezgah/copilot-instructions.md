## tezgah

> Tezgah, Claude Code için yazılmış bir SaaS başlatma skill setidir. Türkiye'deki solopreneur'lerin production-ready SaaS uygulaması kurmasını sağlar.

# Tezgah

Tezgah, Claude Code için yazılmış bir SaaS başlatma skill setidir. Türkiye'deki solopreneur'lerin production-ready SaaS uygulaması kurmasını sağlar.

## Proje Yapısı

- `skills/` — Claude Code skill dosyaları (SKILL.md formatında)
- `bin/` — CLI aracı giriş noktası
- `lib/` — CLI kaynak kodu
- `test/` — CLI testleri
- `.github/` — GitHub şablonları, CI/CD ve release workflow

## Skill'ler

Orkestratör: `saas-launcher` — Tüm süreci yönetir, kullanıcıyla keşif görüşmesi yapar, diğer skill'leri sırayla çağırır.

Uzman skill'ler (her biri bağımsız da kullanılabilir):
- `saas-database` — Supabase veritabanı (şema, RLS, migration, pooling)
- `saas-auth` — Kimlik doğrulama (OAuth, Magic Link, JWT)
- `saas-payments` — Ödeme sistemi (Stripe, Lemon Squeezy)
- `saas-email` — E-posta altyapısı (Resend, DNS)
- `saas-storage` — Dosya depolama (Supabase Storage)
- `saas-landing-seo` — Landing page ve SEO
- `saas-legal` — Yasal uyumluluk (KVKK, GDPR)
- `saas-api-security` — API güvenliği ve rate limiting
- `saas-testing` — Test stratejisi (Vitest, Playwright)
- `saas-analytics` — Ürün analizi (PostHog)
- `saas-deployment` — Production deployment ve izleme

## Kurallar

- Tüm skill içerikleri ve CLI çıktıları Türkçedir.
- CLI sıfır runtime bağımlılıkla çalışır (sadece Node.js built-in modülleri).
- Skill dosyaları SKILL.md formatında, YAML frontmatter ile yapılandırılmıştır.
- Her skill'in `name` ve `description` frontmatter alanları vardır.
- ESLint kod kalitesi için kullanılır.
- Testler `test/cli.test.js` dosyasında, saf Node.js ile yazılmıştır (test framework yok).

## Varsayılan Tech Stack

Next.js (App Router) + TypeScript + Tailwind CSS + shadcn/ui + Supabase + Stripe + Resend + PostHog + Vercel

---
> Source: [komunite/tezgah](https://github.com/komunite/tezgah) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
