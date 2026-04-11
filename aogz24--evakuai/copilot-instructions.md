## evakuai

> 1. Kunjungi [Google AI Studio](https://makersuite.google.com/app/apikey)

# Setup Google Gemini API

## Langkah-langkah Setup:

### 1. Dapatkan API Key Gemini

1. Kunjungi [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Login dengan akun Google Anda
3. Klik **"Get API Key"** atau **"Create API Key"**
4. Copy API key yang dihasilkan

### 2. Konfigurasi Environment Variable

Edit file `.env.local` dan ganti placeholder dengan API key Anda:

```env
NEXT_PUBLIC_GEMINI_API_KEY=YOUR_ACTUAL_API_KEY_HERE
```

Contoh:

```env
NEXT_PUBLIC_GEMINI_API_KEY=AIzaSyBxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 3. Install Dependencies

```bash
npm install
```

Package `@google/generative-ai` sudah ditambahkan ke dependencies.

### 4. Restart Development Server

```bash
npm run dev
```

## Fitur AI yang Aktif:

### ✅ Rekomendasi Bencana (Gemini Pro)

- Analisis kontekstual berdasarkan lokasi, bencana sekitar, dan cuaca
- Rekomendasi action: stay_home, prepare, evacuate_now, seek_shelter
- Risk level: low, medium, high, critical
- Penjelasan dalam Bahasa Indonesia yang natural
- Confidence score berdasarkan kualitas data

### 🔮 Fitur Mendatang (Gemini Pro Vision)

- Analisis foto bencana yang diupload warga
- Klasifikasi jenis dan tingkat keparahan bencana
- Auto-tagging laporan

## Cara Kerja:

1. **Primary**: Jika API key valid, menggunakan **Gemini AI**
2. **Fallback**: Jika Gemini gagal/tidak ada API key, menggunakan **Rule-based Algorithm**

## Keuntungan Gemini AI:

✅ **GRATIS** - Quota harian yang generous  
✅ **Cerdas** - Analisis kontekstual yang lebih baik  
✅ **Natural** - Penjelasan dalam Bahasa Indonesia yang lebih manusiawi  
✅ **Adaptif** - Bisa mempertimbangkan faktor kompleks  
✅ **Terupdate** - Model terus diperbarui Google

## Catatan Penting:

- API key GRATIS dengan quota 60 requests/menit
- Jangan commit API key ke Git (sudah ada di .gitignore)
- Untuk production, gunakan environment variable yang aman
- Monitor usage di [Google AI Studio](https://makersuite.google.com/)

## Troubleshooting:

### Error: "API key not valid"

- Pastikan API key sudah benar
- Cek di Google AI Studio apakah API key masih aktif

### Error: "Quota exceeded"

- Tunggu beberapa menit (quota reset per menit)
- Atau upgrade ke paid plan jika perlu

### Fallback ke Rule-based

- Cek console log untuk error message
- Pastikan .env.local sudah loaded
- Restart development server

## Testing:

Buka halaman `/recommendation` dan sistem akan:

1. Mencoba menggunakan Gemini AI
2. Jika gagal, otomatis fallback ke rule-based
3. Badge "Powered by Gemini" akan muncul jika berhasil

---

**Support**: [Google AI Studio Documentation](https://ai.google.dev/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aogz24)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aogz24)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
