## aoa-mvp

> AOA (Analyze, Optimize, Automate) is an AI-powered lead recovery platform for home service SMBs in Brazil (HVAC, plumbing, solar, electrical, landscaping). These businesses lose ~40% of inbound leads because owners are physically on jobs when prospects call or message.

# AOA MVP — Master Build Prompt

## What is AOA?

AOA (Analyze, Optimize, Automate) is an AI-powered lead recovery platform for home service SMBs in Brazil (HVAC, plumbing, solar, electrical, landscaping). These businesses lose ~40% of inbound leads because owners are physically on jobs when prospects call or message.

AOA deploys AI agents on WhatsApp and voice calls that answer instantly, qualify prospects in natural Brazilian Portuguese, and capture every lead into a real-time dashboard. After qualification, the AI generates a Solana Pay USDC deposit link, moving the prospect from "interested" to "committed." Deposits are held on-chain and released on job completion.

## Architecture Overview

```
LEAD CAPTURE LAYER
├── WhatsApp → Evolution API (webhook) → Fastify backend → Claude API → qualify → Supabase
├── Voice Call → Vapi.ai (voice agent) → webhook → Supabase
└── Both channels feed the same AI brain and the same database

PAYMENT LAYER
├── After qualification → generate Solana Pay link (USDC deposit)
├── Deposit held in escrow (Solana program or simple transfer for MVP)
└── Provider confirms job done → funds release

DASHBOARD
├── React app (Vite + TailwindCSS)
├── Real-time lead cards via Supabase Realtime subscriptions
├── Lead status: new → contacted → deposit_paid → job_complete
└── Simple metrics: leads today, conversion rate, revenue recovered
```

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Backend | Node.js + Fastify | Fast, lightweight, great for webhooks |
| AI | Anthropic Claude API (Sonnet) | Best Portuguese quality, tool use for structured extraction |
| WhatsApp | Evolution API (self-hosted or cloud) | Open-source, free, Brazilian community, REST API |
| Voice | Vapi.ai | Pre-built voice agents, Portuguese TTS/STT, webhook output |
| Database | Supabase (PostgreSQL + Realtime) | Free tier, real-time subscriptions, auth built-in |
| Frontend | React + Vite + TailwindCSS | Fast dev, clean UI, easy deployment |
| Blockchain | Solana Pay SDK + @solana/web3.js | Generate USDC payment links |
| Deployment | Railway (backend) + Vercel (frontend) | Free/cheap tiers, easy deploy |

## Database Schema (Supabase)

### Table: `businesses`
```sql
CREATE TABLE businesses (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  phone TEXT,
  whatsapp_number TEXT,
  services TEXT[] DEFAULT '{}',
  service_area TEXT,
  business_hours JSONB DEFAULT '{"start": "08:00", "end": "18:00"}',
  ai_prompt_context TEXT, -- custom info for the AI agent
  solana_wallet_address TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### Table: `leads`
```sql
CREATE TABLE leads (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  business_id UUID REFERENCES businesses(id),
  channel TEXT NOT NULL CHECK (channel IN ('whatsapp', 'voice')),
  status TEXT DEFAULT 'new' CHECK (status IN ('new', 'qualifying', 'qualified', 'deposit_sent', 'deposit_paid', 'job_scheduled', 'job_complete', 'lost')),
  
  -- Contact info
  contact_name TEXT,
  contact_phone TEXT,
  
  -- Qualification data (extracted by AI)
  service_needed TEXT,
  urgency TEXT CHECK (urgency IN ('low', 'medium', 'high', 'emergency')),
  problem_description TEXT,
  preferred_schedule TEXT,
  location TEXT,
  
  -- AI conversation
  conversation_summary TEXT,
  raw_messages JSONB DEFAULT '[]',
  
  -- Payment
  deposit_amount_usdc DECIMAL(10,2),
  solana_pay_url TEXT,
  solana_tx_signature TEXT,
  deposit_confirmed_at TIMESTAMPTZ,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  qualified_at TIMESTAMPTZ
);
```

### Table: `messages`
```sql
CREATE TABLE messages (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  lead_id UUID REFERENCES leads(id),
  role TEXT NOT NULL CHECK (role IN ('prospect', 'assistant')),
  content TEXT NOT NULL,
  channel TEXT NOT NULL CHECK (channel IN ('whatsapp', 'voice')),
  created_at TIMESTAMPTZ DEFAULT now()
);
```

## AI Agent System Prompt

The AI agent plays the role of a virtual receptionist for the home service business. The prompt must:

1. Greet warmly in Brazilian Portuguese
2. Identify itself as the business's virtual assistant (e.g., "Olá! Sou a assistente virtual da ClimaTech Refrigeração")
3. Understand what service the prospect needs
4. Assess urgency (broken AC in 38°C = emergency, routine maintenance = low)
5. Capture: name, phone (if WhatsApp doesn't provide), location/neighborhood, preferred schedule
6. Summarize and confirm details
7. Close with: "Um técnico entrará em contato para confirmar o agendamento"
8. After qualification, generate the deposit link message

The AI should use Claude's tool_use to extract structured data:

```json
{
  "tool": "qualify_lead",
  "input": {
    "contact_name": "Maria Santos",
    "service_needed": "Instalação de ar-condicionado split",
    "urgency": "high",
    "problem_description": "Precisa instalar 2 splits no apartamento novo antes de se mudar semana que vem",
    "preferred_schedule": "Segunda ou terça de manhã",
    "location": "Vila Mariana, São Paulo"
  }
}
```

## Backend API Routes

### WhatsApp Webhook
```
POST /api/webhooks/evolution
- Receives incoming WhatsApp messages from Evolution API
- Looks up or creates lead by phone number
- Loads conversation history from messages table
- Calls Claude API with system prompt + history + new message
- Saves assistant response to messages table
- Sends reply back via Evolution API
- If AI returns qualify_lead tool call, updates lead status and data
```

### Vapi Webhook
```
POST /api/webhooks/vapi
- Receives call completion data from Vapi
- Extracts structured qualification data from call summary
- Creates lead with status 'qualified'
- Triggers deposit link generation if qualified
```

### Leads API
```
GET    /api/leads              - List all leads (with filters)
GET    /api/leads/:id          - Get single lead with messages
PATCH  /api/leads/:id          - Update lead status
POST   /api/leads/:id/deposit  - Generate Solana Pay deposit link
```

### Solana Pay
```
POST /api/payments/create-deposit
- Takes lead_id and amount
- Generates Solana Pay transfer request URL
- Stores URL in lead record
- Returns URL for AI to send via WhatsApp

POST /api/webhooks/solana
- Monitors for incoming USDC transfers (polling or webhook)
- Confirms deposit and updates lead status
```

## Frontend Dashboard Pages

### `/dashboard` — Main View
- Real-time lead cards using Supabase Realtime
- Each card shows: name, service, urgency badge, channel icon (WhatsApp/phone), status, time since creation
- Click card to expand conversation history
- Filter by: status, urgency, channel, date

### `/dashboard/lead/:id` — Lead Detail
- Full conversation thread
- Qualification summary
- Payment status
- Action buttons: mark as contacted, mark job complete, generate deposit link

### `/dashboard/metrics` — Simple Analytics
- Leads captured today / this week / this month
- Conversion rate (qualified / total)
- Revenue recovered estimate (qualified leads × average ticket)
- Response time average
- Channel breakdown (WhatsApp vs voice)

## Demo Configuration

For the hackathon demo, hardcode one fictional business:

```json
{
  "name": "ClimaTech Refrigeração",
  "slug": "climatech",
  "services": ["Instalação de ar-condicionado", "Manutenção preventiva", "Limpeza de filtros", "Conserto de ar-condicionado", "Instalação de câmara fria"],
  "service_area": "São Paulo - Zona Sul e Centro",
  "business_hours": {"start": "08:00", "end": "18:00"},
  "ai_prompt_context": "ClimaTech Refrigeração é uma empresa familiar com 8 anos de experiência em ar-condicionado residencial e comercial na zona sul de São Paulo. Trabalhamos com todas as marcas. Visita técnica custa R$150, descontada do serviço. Instalação de split a partir de R$800. Manutenção preventiva R$250. Atendemos emergências 24h com taxa adicional de R$200."
}
```

## Environment Variables

```env
# Backend
PORT=3000
NODE_ENV=development

# Anthropic
ANTHROPIC_API_KEY=

# Supabase
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Evolution API
EVOLUTION_API_URL=
EVOLUTION_API_KEY=
EVOLUTION_INSTANCE_NAME=

# Vapi
VAPI_API_KEY=
VAPI_WEBHOOK_SECRET=

# Solana
SOLANA_RPC_URL=https://api.devnet.solana.com
SOLANA_MERCHANT_WALLET=
SOLANA_NETWORK=devnet
USDC_MINT_ADDRESS=4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU  # devnet USDC

# Frontend
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
VITE_API_URL=http://localhost:3000
```

## Build Order (Priority)

### Phase 1 — AI Brain + WhatsApp (Days 1-5)
1. Set up Supabase project and run schema migrations
2. Build Fastify backend with health check route
3. Implement Claude API service with system prompt and tool_use
4. Build WhatsApp webhook handler (receive → AI → reply)
5. Connect to Evolution API (send replies back)
6. Test full WhatsApp conversation loop

### Phase 2 — Dashboard + Real-time (Days 6-9)
1. Scaffold React app with Vite + Tailwind
2. Connect Supabase client with Realtime subscriptions
3. Build lead card components
4. Build lead detail page with conversation thread
5. Build metrics panel
6. Deploy frontend to Vercel

### Phase 3 — Voice + Solana Pay (Days 10-14)
1. Configure Vapi voice agent with Portuguese voice
2. Set up Vapi webhook to capture call data
3. Implement Solana Pay link generation
4. Add deposit link delivery via WhatsApp after qualification
5. Build payment status tracking on dashboard
6. Deploy backend to Railway

### Phase 4 — Polish + Demo Prep (Days 15-18)
1. Add ClimaTech branding to dashboard
2. Handle edge cases (voice messages, off-topic questions, spam)
3. Write demo script
4. Rehearse demo flow 10+ times
5. Record backup video in case live demo fails

## Key Implementation Notes

- All AI conversations must be in Brazilian Portuguese
- Use Claude's tool_use feature for structured data extraction, not regex parsing
- Supabase Realtime is critical for the live demo — leads must appear on dashboard within 2-3 seconds of qualification
- Solana Pay URLs follow the format: `solana:<recipient>?amount=<amount>&spl-token=<mint>&reference=<reference>`
- For the demo, use Solana devnet and devnet USDC
- The Vapi agent should use the same business context as the WhatsApp agent for consistency
- All timestamps in UTC, display in São Paulo timezone (UTC-3) on dashboard

---
> Source: [mubcar/aoa-mvp](https://github.com/mubcar/aoa-mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
