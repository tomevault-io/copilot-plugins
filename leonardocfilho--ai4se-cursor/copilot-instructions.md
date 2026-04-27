## ai4se-cursor

> Stack tecnológico e estrutura do projeto


# Stack Tecnológico

## Core

| Tecnologia | Versão | Uso |
|------------|--------|-----|
| React | 18+ | UI Library |
| TypeScript | 5+ | Linguagem (strict mode) |
| Vite | 5+ | Build tool |

## UI & Estilização

| Tecnologia | Uso |
|------------|-----|
| Material-UI (MUI) | Componentes UI |
| @mui/icons-material | Ícones |
| @emotion/react | CSS-in-JS (MUI) |

```tsx
// ✅ Uso correto do MUI
import { Button, TextField, Box } from '@mui/material';
import { useTheme } from '@mui/material/styles';

function LoginForm() {
  const theme = useTheme();
  return (
    <Box sx={{ p: 2, bgcolor: 'background.paper' }}>
      <TextField label="Email" fullWidth margin="normal" />
      <Button variant="contained" color="primary">Entrar</Button>
    </Box>
  );
}
```

## Data Fetching

| Tecnologia | Uso |
|------------|-----|
| @tanstack/react-query | Server state |
| axios | HTTP client |

```tsx
// ✅ Uso correto do React Query
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => api.get('/users').then(res => res.data),
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: CreateUserInput) => api.post('/users', data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

## Formulários & Validação

| Tecnologia | Uso |
|------------|-----|
| react-hook-form | Gerenciamento de forms |
| zod | Schema validation |
| @hookform/resolvers | Integração RHF + Zod |

```tsx
// ✅ Uso correto de React Hook Form + Zod
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const userSchema = z.object({
  name: z.string().min(2, 'Nome deve ter pelo menos 2 caracteres'),
  email: z.string().email('Email inválido'),
  age: z.number().min(18, 'Deve ser maior de idade'),
});

type UserFormData = z.infer<typeof userSchema>;

function UserForm({ onSubmit }: { onSubmit: (data: UserFormData) => void }) {
  const { register, handleSubmit, formState: { errors } } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <TextField {...register('name')} error={!!errors.name} helperText={errors.name?.message} />
      <TextField {...register('email')} error={!!errors.email} helperText={errors.email?.message} />
      <Button type="submit">Salvar</Button>
    </form>
  );
}
```

## Testes

| Tecnologia | Uso |
|------------|-----|
| Vitest | Test runner |
| @testing-library/react | Component testing |
| @testing-library/user-event | User interactions |
| MSW | API mocking |
| Playwright | E2E testing |

```typescript
// ✅ Configuração vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
    coverage: {
      provider: 'v8',
      thresholds: {
        statements: 80,
        branches: 75,
        functions: 80,
        lines: 80,
      },
    },
  },
});
```

---

# Estrutura de Pastas

```
src/
├── modules/                 # Features do domínio
│   ├── auth/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── types/
│   │   ├── schemas/
│   │   └── index.ts
│   ├── users/
│   ├── rooms/
│   └── reservations/
│
├── shared/                  # Código compartilhado
│   ├── components/          # Componentes reutilizáveis
│   │   ├── Button/
│   │   ├── Modal/
│   │   ├── DataTable/
│   │   └── index.ts
│   ├── hooks/               # Hooks genéricos
│   │   ├── useDebounce.ts
│   │   ├── useLocalStorage.ts
│   │   └── index.ts
│   ├── utils/               # Funções utilitárias
│   │   ├── format.ts
│   │   ├── validation.ts
│   │   └── index.ts
│   ├── lib/                 # Configurações de libs
│   │   ├── axios.ts
│   │   ├── queryClient.ts
│   │   └── index.ts
│   └── constants/           # Constantes globais
│       └── index.ts
│
├── types/                   # Tipos globais
│   ├── api.types.ts
│   ├── common.types.ts
│   └── index.ts
│
├── styles/                  # Tema e estilos globais
│   ├── theme.ts
│   └── global.css
│
├── test/                    # Setup de testes
│   ├── setup.ts
│   ├── mocks/
│   └── fixtures/
│
├── App.tsx
├── main.tsx
└── vite-env.d.ts
```

---

# Convenções de Import

```typescript
// ✅ Ordem de imports
// 1. React e bibliotecas externas
import { useState, useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Box, Button } from '@mui/material';

// 2. Módulos internos (via alias @/)
import { useAuth } from '@/modules/auth';
import { UserCard } from '@/modules/users';

// 3. Shared (via alias @/shared)
import { Modal, DataTable } from '@/shared/components';
import { useDebounce } from '@/shared/hooks';
import { formatDate } from '@/shared/utils';

// 4. Tipos
import type { User } from '@/types';

// 5. Imports relativos (mesmo módulo)
import { UserAvatar } from './UserAvatar';
import { useUserActions } from '../hooks';
```

---

# Aliases (tsconfig.json)

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/modules/*": ["src/modules/*"],
      "@/shared/*": ["src/shared/*"],
      "@/types": ["src/types"]
    }
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LeonardoCFilho) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
