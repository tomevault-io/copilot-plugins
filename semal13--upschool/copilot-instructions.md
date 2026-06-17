## talya-mvp-frontend

> Talya (PCOS AI) MVP — React/Vite, Tailwind, localStorage, mobil öncelik, Claude API güvenliği


# Talya MVP — Frontend ve UI/UX

Bu proje **Talya** PCOS yapay zeka uygulamasının MVP aşamasıdır. Aşağıdaki kurallara uy.

## React (Vite)

- Yalnızca **functional component** ve modern hook’lar (`useState`, `useEffect`, vb.); **class component kullanma**.
- **Veritabanı kullanma.** Kullanıcı verisi ve kalıcı state **yalnızca `localStorage`** (ve gerekirse `sessionStorage`) ile yönetilsin.

## Tailwind ve stil

- Stil için **yalnızca Tailwind utility class’ları**; geniş özel CSS dosyalarından ve global stillerden kaçın (gerekmedikçe yeni `.css` modülü ekleme).
- Düzen **mobil öncelikli**: önce küçük ekran, sonra `sm:` / `md:` / `lg:` ile genişlet.

## Marka renkleri

- Palet: **şefkatli lila, mor ve sıcak krem** tonları. Tailwind `theme` genişletmesi veya `bg-`, `text-`, `border-` ile tutarlı kullan; agresif veya soğuk gri-ağırlıklı UI’dan kaçın.

## Claude / API güvenliği

- Yapay zeka için **Anthropic Claude API** kullan.
- API anahtarlarını **asla koda yazma**; **`.env`** (ve `.env.local`) kullan, repoda **commit etme** (`.gitignore`’da tut).
- Vite’te istemci tarafında erişilecek değişkenler `VITE_` öneki ile tanımlanır: `import.meta.env.VITE_...`. Üretimde anahtarı tarayıcıya sızdırmamak için mümkünse **backend veya proxy** üzerinden çağrı tercih et.

## Örnekler

```tsx
// ❌ Kötü — class component
class Panel extends React.Component { ... }

// ✅ İyi — functional + hooks
function Panel() {
  const [x, setX] = useState(false);
  useEffect(() => { ... }, []);
  return <div className="min-h-screen bg-violet-50 px-4">...</div>;
}
```

```ts
// ❌ Kötü
const key = "sk-ant-...";

// ✅ İyi
const key = import.meta.env.VITE_ANTHROPIC_API_KEY;
```

---
> Source: [semal13/Upschool](https://github.com/semal13/Upschool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
