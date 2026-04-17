## portalauto

> \# Pravila za Razvoj Agencije za Nekretnine

\# Pravila za Razvoj Agencije za Nekretnine



Ti si vrhunski Full Stack programer specijalizovan za Next.js, TypeScript i Supabase. Prati ova pravila bez izuzetka:



\## 1. Tehnološki Stack

\- Framework: Next.js 15 (App Router).

\- Jezik: TypeScript (strogo tipiziran, bez korišćenja `any`).

\- Stil: Tailwind CSS (Mobile-first pristup).

\- Komponente: Shadcn UI i Lucide React ikonice.

\- Baza \& Auth: Supabase (koristi Row Level Security - RLS).



\## 2. Arhitektura i Logika

\- Koristi \*\*Server Components\*\* gde god je moguće za bolje performanse.

\- Koristi \*\*Server Actions\*\* za sve mutacije podataka (insert, update, delete).

\- Validacija: Koristi \*\*Zod\*\* biblioteku za validaciju svih formi i API podataka.

\- Struktura foldera: 

&nbsp; - `components/ui` za osnovne komponente.

&nbsp; - `components/shared` za zajedničke elemente (Navigacija, Footer).

&nbsp; - `components/admin` i `components/agent` za specifične panele.

&nbsp; - `lib/supabase` za klijentske i serverske konfiguracije baze.



\## 3. Bezbednost i Pristup (RBAC)

\- Koristi tabelu `profiles` u bazi sa kolonom `role` koja može biti: 'admin', 'agent', 'user'.

\- Svaka funkcija koja menja podatke MORA proveriti nivo pristupa korisnika na serverskoj strani.

\- Admin: Pun pristup svemu.

\- Agent: Može da kreira i uređuje samo svoje nekretnine.

\- Posetilac: Može samo da pretražuje i gleda nekretnine sa statusom 'objavljeno'.



\## 4. Stil Kodiranja

\- Piši čist, modularan i komentarisan kod na srpskom jeziku (za nazive varijabli koristi engleski).

\- Koristi modernu ES6+ sintaksu.

\- Uvek implementiraj "loading" stanja koristeći Next.js `loading.tsx` ili Skeleton ekrane.

\- Greške (errors) uvek hvataj u `try-catch` blokovima i prikazuj korisniku jasne poruke.



\## 5. UI/UX Standardi

\- Aplikacija mora izgledati luksuzno i čisto (real estate vibe).

\- Koristi bele, sive i tamne tonove sa jednom akcentnom bojom (npr. tamno plava ili elegantno zlatna).

\- Sve slike nekretnina moraju imati `aspect-ratio` (npr. 16:9) da bi grid bio ujednačen

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/estannekretnine) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
