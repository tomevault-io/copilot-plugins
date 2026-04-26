## ai-photobooth

> > **Konsep:** Photobooth online dengan AI style transformation (Ghibli, Disney, Anime, dll).

@AGENTS.md

# 📸 AI Photobooth — Implementation Plan

> **Konsep:** Photobooth online dengan AI style transformation (Ghibli, Disney, Anime, dll).
> Kompetitor utama: Fremio.id. Unique value: foto langsung bisa di-transform ke berbagai art style via AI.

---

## 🗺️ Overview Arsitektur

```
Frontend (Next.js 14 App Router)
├── Halaman utama & landing
├── Photobooth Studio (capture + AI transform + frame)
└── Gallery & sharing

Backend (Next.js API Routes)
├── /api/transform   → AI image style transfer
├── /api/generate-frame  → AI frame generation
└── /api/upload      → Handling gambar

External Services
├── Google Gemini API (image-to-image style transfer)
├── Cloudinary atau Vercel Blob (image storage)
└── Vercel (deployment)
```

---

## 🛠️ Tech Stack

| Layer | Teknologi | Alasan |
|---|---|---|
| Framework | Next.js 14 (App Router) | SSR, API routes, file-based routing |
| Styling | Tailwind CSS + shadcn/ui | Cepat, konsisten, composable |
| AI Image | Google Gemini 2.0 Flash (`gemini-2.0-flash-preview-image-generation`) | Image-to-image support, $0.0195/output |
| Image Storage | Vercel Blob atau Cloudinary | Mudah integrasi dengan Next.js |
| Kamera | Browser MediaDevices API | Native, no library needed |
| Canvas | HTML5 Canvas API | Compositing foto + frame |
| State | Zustand | Ringan, cocok untuk wizard flow |
| Deploy | Vercel | Native Next.js support |

---

## 📁 Struktur Folder

```
ai-photobooth/
├── src/app/
│   ├── page.tsx                    # Landing page
│   ├── studio/
│   │   └── page.tsx                # Main photobooth studio
│   ├── gallery/
│   │   └── page.tsx                # Galeri hasil foto
│   └── api/
│       ├── transform/
│       │   └── route.ts            # AI style transformation endpoint
│       ├── generate-frame/
│       │   └── route.ts            # AI frame generation endpoint
│       └── upload/
│           └── route.ts            # Upload & storage
│
├── components/
│   ├── studio/
│   │   ├── CameraCapture.tsx       # Komponen kamera
│   │   ├── StyleSelector.tsx       # Pilih AI style
│   │   ├── FramePicker.tsx         # Pilih / generate frame
│   │   ├── PhotoCanvas.tsx         # Canvas compositing
│   │   ├── TransformProgress.tsx   # Loading state AI
│   │   └── DownloadShare.tsx       # Download & share
│   ├── landing/
│   │   ├── Hero.tsx
│   │   ├── FeatureShowcase.tsx
│   │   └── StylePreview.tsx
│   └── ui/                         # shadcn components
│
├── lib/
│   ├── gemini.ts                   # Gemini API client & helpers
│   ├── canvas.ts                   # Canvas compositing utilities
│   ├── styles.ts                   # Style definitions & prompts
│   └── storage.ts                  # Image upload/retrieve
│
├── store/
│   └── studio.ts                   # Zustand store untuk studio flow
│
├── public/
│   └── frames/                     # Default frame templates (PNG transparan)
│
└── types/
    └── index.ts                    # TypeScript types
```

---

## 🎯 Fase Development

### Fase 1 — Core Photobooth (Target: bisa foto & download)

**Goal:** User bisa buka kamera, foto, pilih frame, download. Belum ada AI.

#### 1.1 Setup Project

```bash
npx create-next-app@latest ai-photobooth --typescript --tailwind --app
cd ai-photobooth
npx shadcn@latest init
npx shadcn@latest add button card badge slider tabs
npm install zustand
npm install @google/generative-ai
```

#### 1.2 Landing Page (`app/page.tsx`)

Buat landing page dengan:
- Hero section: tagline + CTA button "Mulai Foto"
- Preview beberapa hasil AI transform (before/after)
- Section fitur utama
- Navigasi ke `/studio`

#### 1.3 Studio Layout (`app/studio/page.tsx`)

Wizard flow dengan 4 step:
```
Step 1: Capture  →  Step 2: AI Style  →  Step 3: Frame  →  Step 4: Download
```

Gunakan Zustand store untuk state antar step:

```typescript
// store/studio.ts
interface StudioStore {
  currentStep: 1 | 2 | 3 | 4;
  capturedPhoto: string | null;       // base64 original
  transformedPhoto: string | null;    // base64 after AI
  selectedStyle: AIStyle | null;
  selectedFrame: Frame | null;
  finalPhoto: string | null;          // composited result
  
  setStep: (step: number) => void;
  setCapturedPhoto: (photo: string) => void;
  setTransformedPhoto: (photo: string) => void;
  setSelectedStyle: (style: AIStyle) => void;
  setSelectedFrame: (frame: Frame) => void;
  setFinalPhoto: (photo: string) => void;
  reset: () => void;
}
```

#### 1.4 Komponen Kamera (`components/studio/CameraCapture.tsx`)

```typescript
// Fitur yang perlu diimplementasi:
// - Akses kamera via navigator.mediaDevices.getUserMedia
// - Preview live kamera dalam <video> element
// - Flip kamera (front/back) untuk mobile
// - Tombol capture → gambar ke canvas → simpan sebagai base64
// - Atau: upload dari galeri sebagai alternatif
// - Countdown timer 3-2-1 sebelum capture (opsional, tapi keren)
```

Spesifikasi teknis:
- Resolusi target: 1080x1080 atau 3:4 (portrait photobooth style)
- Mirror mode untuk selfie (transform: scaleX(-1))
- Error handling jika kamera tidak diizinkan

#### 1.5 Canvas Compositing (`lib/canvas.ts`)

```typescript
// Fungsi yang perlu dibuat:
// - composePhotoWithFrame(photoBase64, frameBase64): Promise<string>
//   Overlay frame transparan (PNG) di atas foto
// - resizeImage(base64, maxWidth, maxHeight): string
//   Resize sebelum dikirim ke API (max 1MB untuk Gemini)
// - base64ToBlob(base64): Blob
// - canvasToBase64(canvas): string
```

---

### Fase 2 — AI Style Transformation (Killer Feature)

**Goal:** User pilih style, foto dikirim ke Gemini, hasil ditampilkan.

#### 2.1 Definisi AI Styles (`lib/styles.ts`)

```typescript
export interface AIStyle {
  id: string;
  name: string;
  description: string;
  emoji: string;
  prompt: string;           // Prompt untuk Gemini
  previewImage: string;     // Contoh hasil (untuk UI)
  isPremium: boolean;
}

export const AI_STYLES: AIStyle[] = [
  {
    id: "ghibli",
    name: "Studio Ghibli",
    emoji: "🌸",
    description: "Nuansa magis ala Spirited Away & Totoro",
    prompt: "Transform this photo into Studio Ghibli animation style art. Use soft watercolor-like colors, hand-drawn aesthetic with gentle outlines, dreamy atmosphere, and the characteristic Miyazaki art style. Keep the person's facial features and identity recognizable.",
    isPremium: false,
  },
  {
    id: "disney-classic",
    name: "Disney Klasik",
    emoji: "🏰",
    description: "Gaya animasi Disney era 90an",
    prompt: "Transform this photo into classic Disney animation style from the 1990s golden era. Use bold clean outlines, vibrant saturated colors, expressive eyes, and fairy-tale aesthetic. Keep facial proportions recognizable while applying the Disney character style.",
    isPremium: false,
  },
  {
    id: "pixar",
    name: "Pixar 3D",
    emoji: "🤖",
    description: "Render 3D ala Toy Story & Up",
    prompt: "Transform this photo into Pixar 3D animation style. Apply smooth 3D rendered look with characteristic Pixar lighting (soft ambient + rim light), slightly exaggerated but lovable proportions, and the polished CGI aesthetic from movies like Up, Coco, or Soul.",
    isPremium: false,
  },
  {
    id: "anime",
    name: "Anime",
    emoji: "⛩️",
    description: "Gaya anime Jepang modern",
    prompt: "Transform this photo into modern Japanese anime style illustration. Use sharp clean outlines, large expressive eyes, vibrant colors, cel-shading technique, and dynamic anime aesthetic. Keep the person identifiable while applying full anime character style.",
    isPremium: false,
  },
  {
    id: "watercolor",
    name: "Watercolor",
    emoji: "🎨",
    description: "Lukisan cat air yang dreamy",
    prompt: "Transform this photo into a soft watercolor painting. Use delicate brush strokes, bleeding colors, soft edges, pastel tones, and the characteristic transparent layering of watercolor medium. Create an artistic, romantic, and painterly atmosphere.",
    isPremium: false,
  },
  {
    id: "cyberpunk",
    name: "Cyberpunk",
    emoji: "🌆",
    description: "Neon futuristik ala Blade Runner",
    prompt: "Transform this photo into cyberpunk digital art style. Apply neon lighting (purple, cyan, pink), futuristic urban atmosphere, high contrast shadows, holographic elements, and the gritty yet vibrant aesthetic of cyberpunk genre.",
    isPremium: true,
  },
  {
    id: "sketch",
    name: "Pencil Sketch",
    emoji: "🖍️",
    description: "Sketsa pensil artistik",
    prompt: "Transform this photo into a detailed pencil sketch artwork. Use realistic pencil stroke textures, hatching and cross-hatching for shading, varying line weights, and the authentic feel of hand-drawn graphite sketch on white paper.",
    isPremium: false,
  },
  {
    id: "pixel-art",
    name: "Pixel Art",
    emoji: "🎮",
    description: "Retro pixel art style",
    prompt: "Transform this photo into pixel art style with visible pixel grid, limited color palette (16-32 colors), retro 32-bit aesthetic reminiscent of SNES/Sega era games. Keep the subject recognizable in pixelated form.",
    isPremium: true,
  },
  {
    id: "kpop",
    name: "K-Pop Idol",
    emoji: "🌟",
    description: "Glowy aesthetic ala idol Korea",
    prompt: "Transform this photo into K-pop idol digital portrait style. Apply glass skin effect, soft glowing highlights, clean beauty aesthetic, pastel or jewel-tone color grading, and the polished high-fashion look typical of K-pop promotional photos.",
    isPremium: true,
  },
  {
    id: "oil-painting",
    name: "Oil Painting",
    emoji: "🖼️",
    description: "Lukisan minyak klasik",
    prompt: "Transform this photo into a classic oil painting. Use rich deep colors with visible brush strokes, impasto technique for highlights, old master portrait style with dramatic chiaroscuro lighting, and the warm golden tones characteristic of Renaissance or Baroque era portraits.",
    isPremium: true,
  },
];
```

#### 2.2 Gemini API Client (`lib/gemini.ts`)

```typescript
import { GoogleGenerativeAI } from "@google/generative-ai";

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);

export async function transformImageStyle(
  imageBase64: string,
  style: AIStyle
): Promise<string> {
  const model = genAI.getGenerativeModel({
    model: "gemini-2.0-flash-preview-image-generation",
  });

  // Resize image sebelum kirim (max ~1MB base64)
  const resizedImage = await resizeImageForAPI(imageBase64);

  const result = await model.generateContent([
    {
      inlineData: {
        mimeType: "image/jpeg",
        data: resizedImage,
      },
    },
    style.prompt,
  ]);

  // Extract image dari response
  const response = await result.response;
  for (const part of response.candidates?.[0]?.content?.parts ?? []) {
    if (part.inlineData) {
      return `data:${part.inlineData.mimeType};base64,${part.inlineData.data}`;
    }
  }

  throw new Error("No image generated from Gemini");
}

// Helper: resize image ke max 800px, quality 85%
async function resizeImageForAPI(base64: string): Promise<string> {
  // Implementasi via canvas di server atau sharp library
  // Gunakan sharp: npm install sharp
}
```

#### 2.3 API Route Transform (`app/api/transform/route.ts`)

```typescript
import { NextRequest, NextResponse } from "next/server";
import { transformImageStyle } from "@/lib/gemini";
import { AI_STYLES } from "@/lib/styles";

export async function POST(req: NextRequest) {
  try {
    const { imageBase64, styleId } = await req.json();

    if (!imageBase64 || !styleId) {
      return NextResponse.json({ error: "Missing required fields" }, { status: 400 });
    }

    const style = AI_STYLES.find((s) => s.id === styleId);
    if (!style) {
      return NextResponse.json({ error: "Style not found" }, { status: 404 });
    }

    // Rate limiting sederhana: cek header IP
    // TODO: implementasi proper rate limiting dengan upstash/redis

    const transformedImage = await transformImageStyle(imageBase64, style);

    return NextResponse.json({ 
      success: true,
      image: transformedImage,
      styleId,
    });

  } catch (error) {
    console.error("Transform error:", error);
    return NextResponse.json(
      { error: "Failed to transform image" },
      { status: 500 }
    );
  }
}

// Set timeout lebih panjang karena AI processing butuh waktu
export const maxDuration = 60; // seconds
```

#### 2.4 Komponen Style Selector (`components/studio/StyleSelector.tsx`)

```typescript
// UI yang perlu diimplementasi:
// - Grid 2x5 atau 3x3 dari style cards
// - Setiap card: emoji + nama + preview thumbnail (before/after)
// - Badge "Premium" untuk style berbayar
// - Tombol "Transform" setelah pilih style
// - Loading state dengan progress message yang menarik
//   ("Menggambar dengan pensil Miyazaki...", "Memanggil seniman Ghibli...")
// - Handling error dengan retry option
```

#### 2.5 Transform Progress Component (`components/studio/TransformProgress.tsx`)

```typescript
// Tampilkan loading state yang engaging selama AI processing (bisa 5-15 detik):
const LOADING_MESSAGES: Record<string, string[]> = {
  ghibli: [
    "Miyazaki-san sedang melukis...",
    "Menambahkan sentuhan magis...",
    "Hampir jadi! Menyelesaikan detail...",
  ],
  disney: [
    "Bibbidi-bobbidi-boo! ✨",
    "Menghidupkan karakter Disney...",
    "Sentuhan akhir dari Castle...",
  ],
  // dst untuk tiap style
};
// Rotate messages setiap 3 detik selama loading
```

---

### Fase 3 — Frame System

**Goal:** User bisa pilih frame preset atau generate frame dengan AI.

#### 3.1 Frame Templates

Siapkan frame PNG transparan di `public/frames/`:
- `frame-classic-white.png` — Border putih clean
- `frame-film-strip.png` — Gaya film analog
- `frame-polaroid.png` — Gaya foto polaroid
- `frame-korean-booth.png` — Gaya photo booth Korea
- `frame-aesthetic-pink.png` — Border dekoratif pink
- `frame-minimal-black.png` — Border hitam minimal
- `frame-vintage.png` — Gaya vintage/retro
- `frame-none.png` — Tanpa frame (transparan)

Format: PNG 1080x1080, transparan di tengah (area foto), resolusi tinggi.

#### 3.2 Frame Picker Component (`components/studio/FramePicker.tsx`)

```typescript
// UI:
// - Tab: "Pilih Template" | "Generate dengan AI"
// - Tab Template: scrollable grid frame thumbnails
// - Tab Generate: input text prompt + tombol Generate
// - Preview real-time foto + frame yang dipilih
```

#### 3.3 AI Frame Generation (`app/api/generate-frame/route.ts`)

```typescript
// Gunakan Gemini untuk generate frame PNG transparan
// Prompt engineering yang penting:
const FRAME_GENERATION_PROMPT = (userPrompt: string) => `
Create a decorative photo frame/border for a photobooth photo.
The frame should: ${userPrompt}
Requirements:
- Transparent center (leave empty space for the photo)
- Decorative border design only around the edges
- High quality, detailed artwork
- Suitable for overlaying on a portrait photo
Output: PNG with transparent background showing only the frame border design.
`;

// Catatan: Image generation dari teks murni lebih baik pakai Imagen 3
// atau Stable Diffusion untuk frame generation
// Gemini Flash lebih optimal untuk image-to-image (transform style)
```

#### 3.4 Canvas Compositing (`lib/canvas.ts`)

```typescript
export async function composePhotoWithFrame(
  photoBase64: string,    // Foto yang sudah di-transform AI (atau original)
  frameBase64: string,    // Frame PNG transparan
  outputSize = 1080       // Output dalam pixels
): Promise<string> {
  // 1. Buat canvas outputSize x outputSize
  // 2. Draw foto sebagai background (cover/fill)
  // 3. Draw frame PNG di atas (overlay)
  // 4. Return sebagai base64 JPEG/PNG
  
  // Jalankan di browser menggunakan OffscreenCanvas atau Canvas API
}
```

---

### Fase 4 — Download, Share & Polish

#### 4.1 Download Component (`components/studio/DownloadShare.tsx`)

```typescript
// Fitur:
// - Download sebagai PNG (full quality 1080x1080)
// - Download sebagai JPEG (compressed, untuk sharing)
// - Copy to clipboard
// - Share via Web Share API (mobile)
// - Tombol "Foto Lagi" (reset ke step 1)

// Watermark opsional (untuk free tier):
// - Teks kecil "Made with [AppName]" di pojok
```

#### 4.2 Metadata & OG Tags

```typescript
// app/studio/page.tsx - metadata untuk sharing
export const metadata: Metadata = {
  title: "AI Photobooth Studio",
  description: "Ubah fotomu jadi karya seni AI — Ghibli, Disney, Anime & lebih!",
  openGraph: {
    images: ["/og-preview.jpg"],
  },
};
```

---

### Fase 5 — Monetization & Auth (Opsional untuk MVP)

#### 5.1 Sistem Kredit Sederhana

Strategi monetization:
- **Free:** 3 AI transforms per hari, 5 style tersedia, watermark
- **Premium (Rp15.000/bulan):** Unlimited transforms, semua style, no watermark, HD quality

```typescript
// Implementasi tanpa auth (pakai localStorage untuk MVP):
interface UsageTracker {
  date: string;           // "2025-03-22"
  transformCount: number; // Reset setiap hari
}

// Atau gunakan Vercel KV / Upstash Redis untuk server-side tracking by IP
```

#### 5.2 Auth (Jika Perlu)

Gunakan NextAuth.js dengan provider:
- Google OAuth (paling mudah untuk user Indonesia)
- Opsi: Magic Link via email

```bash
npm install next-auth
```

---

## 🔐 Environment Variables

Buat file `.env.local`:

```bash
# Wajib
GEMINI_API_KEY=your_gemini_api_key_here

# Untuk storage (pilih salah satu)
BLOB_READ_WRITE_TOKEN=your_vercel_blob_token   # Jika pakai Vercel Blob
CLOUDINARY_URL=cloudinary://...               # Jika pakai Cloudinary

# Opsional (untuk auth)
NEXTAUTH_SECRET=random_secret_string
NEXTAUTH_URL=http://localhost:3000
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...

# Opsional (untuk rate limiting)
UPSTASH_REDIS_REST_URL=...
UPSTASH_REDIS_REST_TOKEN=...
```

---

## 📋 Urutan Task untuk Claude Code

Jalankan task berikut **secara berurutan**. Setiap fase harus berjalan sebelum lanjut ke fase berikutnya.

### Minggu 1 — Fondasi

```
Task 1: Setup project Next.js 14 + Tailwind + shadcn/ui + Zustand
Task 2: Buat Zustand store untuk studio flow (store/studio.ts)
Task 3: Buat landing page (app/page.tsx) — simpel dulu, hero + CTA
Task 4: Buat studio layout dengan wizard steps (app/studio/page.tsx)
Task 5: Implementasi CameraCapture component — akses kamera, preview, capture
Task 6: Implementasi canvas compositing (lib/canvas.ts)
```

### Minggu 2 — AI Integration

```
Task 7: Setup Gemini client (lib/gemini.ts) + definisi AI styles (lib/styles.ts)
Task 8: Buat API route /api/transform (app/api/transform/route.ts)
Task 9: Buat StyleSelector component dengan loading states
Task 10: Buat TransformProgress component dengan rotating messages per style
Task 11: Connect frontend → API → Gemini → tampilkan hasil
Task 12: Error handling & retry logic
```

### Minggu 3 — Frame & Polish

```
Task 13: Siapkan frame template assets (PNG transparan)
Task 14: Buat FramePicker component dengan tab template/generate
Task 15: Implementasi canvas compositing foto + frame
Task 16: Buat DownloadShare component
Task 17: Polish UI — animasi, transitions, responsive mobile
Task 18: Testing end-to-end flow
```

### Minggu 4 — Optimasi & Launch

```
Task 19: Implementasi rate limiting (pakai IP-based atau localStorage)
Task 20: SEO & OG tags
Task 21: Performance optimization (lazy load, image optimization)
Task 22: Deploy ke Vercel
Task 23: Mobile testing & bug fixes
```

---

## ⚠️ Catatan Penting untuk Implementasi

### Tentang Gemini Image Generation

- Model yang digunakan: `gemini-2.0-flash-preview-image-generation`
- Input gambar harus dikompresi dulu (idealnya < 1MB base64)
- Waktu response bisa 5-20 detik — wajib ada loading state yang engaging
- Gemini mungkin menolak gambar dengan konten tertentu — handle error dengan pesan yang friendly
- Cost estimasi: ~$0.02 per transform. Untuk 100 user/hari dengan 3 transform each = ~$6/hari

### Tentang Kamera di Browser

- HTTPS **wajib** untuk akses kamera (localhost tetap bisa)
- Minta permission secara eksplisit dengan UI yang jelas sebelum `getUserMedia()`
- Handle kasus: tidak ada kamera, permission ditolak, kamera sedang dipakai app lain
- Di iOS Safari, ada limitasi tersendiri untuk kamera — test di device nyata

### Tentang Frame PNG

- Frame harus PNG dengan alpha channel (transparan)
- Resolusi minimal 1080x1080 untuk hasil yang tajam
- Optimalkan ukuran file frame (< 500KB per frame)
- Bisa buat frame awal di Figma/Canva lalu export sebagai PNG

### Tentang Performance

- Jangan simpan base64 besar di Zustand — gunakan blob URL untuk preview
- Kompresi gambar sebelum kirim ke API (`canvas.toBlob()` dengan quality 0.8)
- Gunakan `next/image` untuk semua gambar statis
- AI transform result simpan sementara, jangan auto-upload ke storage

---

## 🎨 Design Direction

Referensi estetika: **Playful + Premium**. Bukan terlalu childish, bukan terlalu corporate.

- **Warna primer:** Deep purple (`#4F46E5`) + Pink accent (`#EC4899`)
- **Background:** Soft cream/off-white, bukan putih polos
- **Font display:** Instrument Serif atau Playfair Display (untuk judul)
- **Font body:** Plus Jakarta Sans (Indonesia-friendly, ada di Google Fonts)
- **Mood:** Seperti aplikasi foto Korea yang cantik — bersih, aesthetic, tapi fun

---

## 🚀 Quick Start Command

Setelah semua setup, jalankan:

```bash
npm run dev
# Buka http://localhost:3000
```

Untuk test AI transform tanpa frontend:

```bash
curl -X POST http://localhost:3000/api/transform \
  -H "Content-Type: application/json" \
  -d '{"imageBase64": "BASE64_STRING_HERE", "styleId": "ghibli"}'
```

---

*Plan ini bisa dijalankan langsung di Claude Code. Mulai dari Task 1 dan eksekusi secara berurutan. Tiap task sebaiknya ditest dulu sebelum lanjut ke task berikutnya.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/asrafmi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
