## smartfood

> 1. Kunjungi [Google AI Studio](https://aistudio.google.com/)

# Setup Gemini API Key untuk SmartFood

## Langkah-langkah untuk mendapatkan Gemini API Key:

### 1. Dapatkan API Key dari Google AI Studio
1. Kunjungi [Google AI Studio](https://aistudio.google.com/)
2. Login dengan akun Google kamu
3. Klik "Get API key" di pojok kanan atas
4. Pilih "Create API key" atau gunakan yang sudah ada
5. Copy API key yang diberikan

### 2. Konfigurasi di Project
1. Buka file `config.env`
2. Ganti `YOUR_GEMINI_API_KEY_HERE` dengan API key yang kamu dapatkan
3. Simpan file

Contoh isi `config.env`:
```
PORT=3000
MONGO_URI=mongodb+srv://s22210609:Sealtiel12345Project@cluster0.vxqcb9r.mongodb.net/smartfood?retryWrites=true&w=majority
NODE_ENV=development
GEMINI_API_KEY=AIzaSyC...your_actual_api_key_here
```

### 3. Restart Server
Setelah mengkonfigurasi API key, restart server:
```bash
npm run dev
# atau
node server.js
```

### 4. Verifikasi
Server akan menampilkan pesan:
- "Gemini API Key configured: Yes" jika berhasil
- "Gemini API Key configured: No" jika masih ada masalah

### 5. Test API
Upload gambar makanan melalui frontend untuk menguji apakah AI analysis berfungsi.

## Troubleshooting

### Error 400 Bad Request
- Pastikan gambar yang diupload valid (JPEG, PNG)
- Pastikan ukuran gambar tidak terlalu besar (max 4MB)

### Error 403 Forbidden
- API key tidak valid atau quota habis
- Periksa apakah API key sudah benar

### Error 500 Internal Server Error
- Periksa console server untuk detail error
- Pastikan semua environment variables sudah dikonfigurasi

## Catatan Penting
- Jangan share API key kamu ke publik
- API key memiliki quota harian, monitor penggunaan
- Backup API key di tempat yang aman 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TaebenuRelin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TaebenuRelin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
