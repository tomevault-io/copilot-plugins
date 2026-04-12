## here-re-go

> - **`frontend/`** - Applicazione React 18+ con TypeScript

# Struttura Progetto SPOrTS

## **Organizzazione File e Cartelle**

### **Root Directory**
- **`frontend/`** - Applicazione React 18+ con TypeScript
- **`backend/`** - API Node.js + Express
- **`docs/`** - Documentazione API e sviluppo
- **`.cursor/`** - Regole Cursor IDE
- **`scripts/`** - Script di setup e database
- **`docker-compose.dev.yml`** - Configurazione Docker development

### **Frontend Structure** (`frontend/src/`)
```
frontend/src/
├── components/          # Componenti UI riutilizzabili
│   ├── ui/             # Componenti base (Button, Input, etc.)
│   ├── auth/           # Componenti autenticazione
│   ├── layout/         # Layout components
│   └── forms/          # Form components
├── pages/              # Pagine dell'applicazione
│   ├── admin/          # Pagine admin
│   └── [altre pagine]  # Pagine pubbliche
├── services/           # API services e data fetching
├── hooks/              # Custom React hooks
├── contexts/           # React contexts (Auth, Modal, etc.)
├── utils/              # Utility functions
├── types/              # TypeScript definitions
├── styles/             # CSS e styling
└── config/             # Configurazioni frontend
```

### **Backend Structure** (`backend/src/`)
```
backend/src/
├── controllers/        # Route controllers
├── models/            # Mongoose models
├── routes/            # API routes
├── middlewares/       # Express middlewares
├── services/          # Business logic
├── config/            # Configuration files
└── utils/             # Utility functions
```

## **Convenzioni Naming**

### **File e Cartelle**
- **PascalCase**: Componenti React, Classi
- **camelCase**: Funzioni, variabili, file utility
- **kebab-case**: Route paths, CSS classes
- **UPPER_SNAKE_CASE**: Costanti, environment variables

### **Componenti React**
```typescript
// ✅ DO: PascalCase per componenti
const VenueCard = () => { ... }
const BookingForm = () => { ... }

// ❌ DON'T: camelCase per componenti
const venueCard = () => { ... }
```

### **API Routes**
```javascript
// ✅ DO: kebab-case per routes
app.use('/api/match-announcements', require('./routes/matchAnnouncements'))
app.use('/api/venue-owners', require('./routes/venueOwners'))

// ❌ DON'T: camelCase per routes
app.use('/api/matchAnnouncements', require('./routes/matchAnnouncements'))
```

## **Organizzazione Codice**

### **Import Order**
1. **React imports**
2. **Third-party libraries**
3. **Internal components**
4. **Utilities and types**
5. **Styles**

```typescript
// ✅ DO: Import order corretto
import React, { useState, useEffect } from 'react'
import { useQuery } from '@tanstack/react-query'
import { Button } from '@/components/ui/button'
import { cn } from '@/utils/cn'
import type { Venue } from '@/types'
import './VenueCard.css'
```

### **File Organization**
- **Un componente per file**
- **Nomi file corrispondono al componente**
- **Index files per export multipli**
- **Separazione logica per feature**

### **Component Structure**
```typescript
// ✅ DO: Struttura componente standard
import React from 'react'
import type { ComponentProps } from './types'

interface Props extends ComponentProps {
  // props specifiche
}

export const ComponentName: React.FC<Props> = ({ prop1, prop2 }) => {
  // hooks
  // handlers
  // render
  return <div>...</div>
}
```

## **Multi-Tenant Architecture**

### **Tenant Identification**
- **Header-based**: `X-Tenant-ID: <tenant-slug>`
- **Subdomain-based**: `tenant-slug.sports.app`
- **JWT-based**: Token include tenant context

### **Data Isolation**
- **Ogni tenant ha dati completamente isolati**
- **Middleware tenant applicato a tutte le route**
- **Validazione automatica ownership**
- **Audit logging per compliance**

### **Tenant Middleware Usage**
```javascript
// ✅ DO: Applicare middleware tenant
app.use(TenantMiddleware.extractTenant)
app.use(TenantMiddleware.requireTenant)

// ✅ DO: Validare ownership
app.use(SecurityMiddleware.validateTenantOwnership)
```

## **Database Models**

### **Schema Organization**
- **Indices per performance**
- **Validation automatica**
- **Virtuals per computed fields**
- **Middleware pre/post save**

### **Model Structure**
```javascript
// ✅ DO: Struttura modello standard
const schema = new mongoose.Schema({
  // fields
}, {
  timestamps: true,
  toJSON: { virtuals: true }
})

// Indices
schema.index({ field: 1 })

// Virtuals
schema.virtual('computedField').get(function() { ... })

// Methods
schema.methods.customMethod = function() { ... }

// Middleware
schema.pre('save', function(next) { ... })
```

## **API Design**

### **Response Format**
```javascript
// ✅ DO: Formato response standard
{
  success: true,
  data: { ... },
  message: "Operation successful",
  pagination: { ... } // se applicabile
}

// ❌ DON'T: Formato inconsistente
{
  result: { ... },
  status: "ok"
}
```

### **Error Handling**
```javascript
// ✅ DO: Error handling standard
app.use((err, req, res, next) => {
  console.error(err.stack)
  res.status(500).json({
    success: false,
    error: 'Internal server error',
    message: process.env.NODE_ENV === 'development' ? err.message : 'Something went wrong'
  })
})
```

## **Security Best Practices**

### **Authentication**
- **JWT tokens per autenticazione**
- **bcryptjs per hashing password**
- **Rate limiting per API**
- **CORS configurato correttamente**

### **Input Validation**
```javascript
// ✅ DO: Validazione input
const { body, validationResult } = require('express-validator')

const validateVenue = [
  body('name').trim().isLength({ min: 2, max: 100 }),
  body('email').isEmail().normalizeEmail(),
  (req, res, next) => {
    const errors = validationResult(req)
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() })
    }
    next()
  }
]
```

## **Performance Optimization**

### **Frontend**
- **Lazy loading per routes**
- **Code splitting automatico**
- **Image optimization**
- **Bundle size monitoring**

### **Backend**
- **Database indexing**
- **Query optimization**
- **Caching strategies**
- **Connection pooling**

## **Testing Strategy**

### **Test Organization**
- **Unit tests per utilities**
- **Integration tests per API**
- **E2E tests per workflow**
- **Multi-tenant isolation tests**

### **Test Structure**
```
tests/
├── unit/              # Unit tests
├── integration/       # Integration tests
├── e2e/              # End-to-end tests
└── fixtures/         # Test data
```

## **Documentation Standards**

### **Code Comments**
- **JSDoc per funzioni pubbliche**
- **Inline comments per logica complessa**
- **README per ogni directory principale**
- **API documentation aggiornata**

### **Documentation Files**
- **`README.md`** - Overview progetto
- **`docs/API.md`** - Documentazione API
- **`docs/DEVELOPMENT.md`** - Guide sviluppo
- **`.cursor/rules/`** - Regole IDE

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tronkss27)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Tronkss27)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
