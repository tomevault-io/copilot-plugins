## curva-mestra-system

> **Sistema SaaS Multi-Tenant para Clínicas de Harmonização Facial e Corporal**

# Regras do Projeto [Curva Mestra]
**Sistema SaaS Multi-Tenant para Clínicas de Harmonização Facial e Corporal**  
Gestão inteligente de estoque Rennova via DANFE (NF-e) + controle de lotes, validades, licenças e consumo por paciente.

## Stack
**100% Firebase (Google Cloud) – Zero DevOps, MVP em 5 semanas**

```yaml
Frontend:     Next.js 15 (App Router) + TypeScript + Tailwind CSS + Shadcn/ui + Lucide Icons
PWA/Mobile:   Next.js PWA + Capacitor (iOS/Android via web)
Backend:      Firebase Functions 2nd gen (TypeScript + Python 3.11)
Banco:        Firestore in Native Mode (multi-tenant com tenant_id + RLS)
Auth:         Firebase Authentication + Custom Claims + Magic Link + 2FA
Storage:      Firebase Storage (/danfe/{tenant_id}/{nf_id}.pdf)
OCR + IA:     Python (pytesseract + pdf2image + OpenCV) + Vertex AI Gemini 1.5 Flash (fallback)
Real-time:    Firestore Realtime Listeners + TanStack Query
Fila/Cron:    Firebase Scheduled Functions + Extensions (Trigger Email, TTL)
Deploy:       Firebase Hosting + Functions (firebase deploy)
Monitoramento: Firebase Crashlytics + Performance + Google Cloud Logging
Testes:       Firebase Emulator Suite + Playwright E2E + Jest
CI/CD:        GitHub Actions + firebase preview channels
```

## Convencoes
**1. Nomeação e Estrutura**

```text
src/
├── app/                  # Next.js 15 App Router
│   ├── (auth)/           # rotas públicas
│   ├── (admin)/          # system_admin
│   ├── (clinic)/         # clinic_admin + clinic_user
│   └── layout.tsx
├── components/           # UI reutilizáveis (shadcn)
├── lib/                  # firebase.ts, utils.ts, parser-rennova.ts
├── hooks/                # useTenant, useInventory, useRealtime
├── types/                # Tenant, UserRole, NFImport, ProdutoRennova
└── public/
```

**2. Multi-Tenant (OBRIGATÓRIO em TODAS as regras Firestore/Functions)**

```ts
// Custom Claims (setados no admin)
{
  tenant_id: "clinic_abc123",
  role: "clinic_admin" | "clinic_user" | "system_admin",
  is_system_admin: boolean,
  active: boolean
}
```

**3. Regras de Segurança Firestore (RLS)**
```js
match /tenants/{tenantId}/{document=**} {
  allow read, write: if request.auth != null 
    && request.auth.token.tenant_id == tenantId
    && request.auth.token.active == true;
}
match /tenants/{tenantId}/{document=**} {
  allow read, write: if request.auth.token.is_system_admin == true;
}
```

**4. Convenções de Código**
```ts
// Arquivos
components/ui/button.tsx
lib/firebase.ts
functions/src/ocr-rennova.py
// Variáveis
tenantId, clinicAdmin, nfImportId
// Funções
parseRennovaDanfe(pdfBuffer): Promise<ParsedNF>
// Commits (Conventional Commits)
feat: adiciona parser automático DANFE Rennova
fix: corrige extração de lote com hífen
chore: atualiza dependências firebase
```

**5. Parser DANFE Rennova (RegEx Oficial v4.0 – NUNCA altere sem teste)**

```python
# functions/src/ocr-rennova.py
LOT_REGEX = r"Lt:\s*([A-Z0-9\-]+)"
QTD_REGEX = r"Q:\s*([\d,]+)"
VAL_REGEX = r"Dt\. Val\.:\s*(\d{2}/\d{2}/\d{4})"
COD_REGEX = r"^(\d{7,8})\s"  # início da linha
NOME_PRODUTO = tudo até "Lt:"
```

**6. Status Padronizados**
```ts
// NF Import
"pending" | "processing" | "success" | "error" | "novo_produto_pendente"

// Solicitação
"criada" | "agendada" | "aprovada" | "reprovada" | "cancelada"

// Licença
"ativa" | "pendente" | "expirada" | "suspensa"
```

**7. Roles e Permissões**
```json
// NF-e 026229 - 25/03/2025 (já extraída corretamente)
{
  "numero": "026229",
  "produtos": [
    {
      "codigo": 3029055,
      "nome_produto": "TORNEIRA DESCARTAVEL 3VIAS LL",
      "lote": "SCTPAB002B",
      "quantidade": 5,
      "dt_validade": "01/06/2029",
      "valor_unitario": 1.55
    },
    // ... demais 12 produtos
  ]
}
```

**9. Comandos Úteis (salve no terminal)**
```bash
# Local com emuladores
firebase emulators:start

# Testar parser Python localmente
python functions/src/ocr-rennova.py --file "samples/026229.pdf"

# Deploy apenas functions OCR
firebase deploy --only functions:ocrRennova

# Preview URL automática
github.com/usuario/curva-mestra/pull/42 → https://curva-mestra--pr-42.web.app
```

**10. Próximos Passos (ordem obrigatória para Claude)**
Criar projeto Firebase + habilitar Auth, Firestore, Storage, Functions
Configurar Custom Claims + RLS multi-tenant
Implementar parser Python com Gemini fallback (100% acurácia)
Criar Portal Admin (system_admin) → licenças + produtos master
Upload DANFE + trigger automático → inventory realtime
Dashboard com alertas de vencimento/estoque
Sistema de solicitações com status agendada/aprovada

*NUNCA quebre o multi-tenant. NUNCA ignore o parser Rennova. NUNCA deploy sem testar com a NF-e 026229.*

Projeto iniciado em: 07/11/2025
Stack travada até v1.0 (Firebase only)
Claude AI é o arquiteto oficial – siga este arquivo à risca.

## Funcionalidades Desabilitadas

### Importação de PDF (DANFE Rennova)
**Status**: DESABILITADO permanentemente (por ora)
**Motivo**: Decisão de simplificar o MVP e focar apenas em cadastro manual
**Arquivos Afetados**:
- `src/app/api/parse-nf/route.ts` (API route não utilizada - manter para referência futura)

**Decisão de Produto**:
1. MVP irá operar APENAS com cadastro manual de produtos no Portal Admin
2. Funcionalidade de importação via DANFE pode ser implementada no futuro
3. API route `/api/parse-nf` permanece no código como referência, mas não está conectada à interface

**Sistema Atual**:
- System Admin cadastra produtos manualmente em `/admin/products/new`
- Campos: código (7-8 dígitos) e nome do produto (uppercase automático)
- Produtos ficam disponíveis no catálogo master para todas as clínicas


---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GScandelari)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GScandelari)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
