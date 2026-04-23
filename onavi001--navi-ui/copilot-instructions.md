## navi-ui

> Cómo crear componentes en Navi-UI


# Plantilla de Componente

Todo componente debe seguir exactamente este patrón:
```tsx
import * as React from 'react'
import { cn } from '@/utils/cn'

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost' | 'destructive'
  size?: 'sm' | 'md' | 'lg'
  loading?: boolean
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'primary', size = 'md', loading, children, disabled, ...props }, ref) => {
    return (
      <button
        ref={ref}
        disabled={disabled || loading}
        aria-busy={loading}
        className={cn(baseStyles, variants[variant], sizes[size], className)}
        {...props}
      >
        {children}
      </button>
    )
  }
)
Button.displayName = 'Button'

export { Button }
```

## Checklist obligatorio por componente
- [ ] forwardRef
- [ ] Props extendidas del elemento HTML base
- [ ] displayName asignado
- [ ] Variantes como objeto de clases
- [ ] Estados: default, hover, focus, disabled, loading, error
- [ ] Dark mode automático vía CSS Variables
- [ ] WCAG 2.2 AA (roles ARIA, contraste, keyboard nav)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onavi001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
