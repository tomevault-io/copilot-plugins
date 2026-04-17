## suprema

> Você está trabalhando no SUPREMA BELEZA 5.0 — plataforma premium de beleza com IA.

# SUPREMA BELEZA — Matriz Gênesis (Regras Invioláveis para Cursor)

Você está trabalhando no SUPREMA BELEZA 5.0 — plataforma premium de beleza com IA.

## REGRAS ABSOLUTAS (NUNCA VIOLE)

### 1. ZERO CORES HARDCODED
NEVER use:
- `#D4AF37`, `#1A1A1A`, `#fff`, `#000`
- `bg-white`, `bg-gray-*`, `text-gray-*`, `text-white`

ALWAYS use CSS variables:
- `bg-[var(--color-background)]`
- `bg-[var(--color-surface)]`
- `text-[var(--color-foreground)]`
- `text-[var(--color-muted)]`
- `border-[var(--color-border)]`

### 2. USE SHADCN/UI COMPONENTS
Always import from '@/components/ui/':
import { Button } from '@/components/ui/button'
import { Card, CardContent } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Input } from '@/components/ui/input'

### 3. REAL DATA ONLY
- NO mock data, fake data, placeholder arrays
- Todos os dados vêm do Supabase via hooks (useProducts, useSalons, useAppointments, etc.)
- Queries sempre usam o cliente de '@/lib/supabaseClient'

### 4. TEMAS DINÂMICOS
- Tudo via src/styles/theme.css (CSS variables)
- Toggle de tema funcional com persistência

### 5. ESTADOS OBRIGATÓRIOS
- Toda página com dados do Supabase deve ter loading, error e empty states

### 6. ESTRUTURA CANÔNICA
src/
├── components/ui/     # shadcn/ui (NÃO MODIFICAR)
├── components/layout/ # SupremeHeader, SupremeFooter
├── hooks/             # useAuth, useProducts, useSalons, etc.
├── lib/               # supabaseClient.ts (ÚNICO)
├── styles/theme.css   # SSOT de variáveis

## VALIDAÇÃO ANTES DE COMMIT
grep -r "#[0-9a-fA-F]\{3,6\}" src/ → deve retornar vazio
grep -r "bg-white\|bg-gray-\|text-gray-" src/ → vazio
grep -r "mock\|fake\|placeholder" src/ → vazio
npm run build → deve passar

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AbnadabyBonaparte) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
