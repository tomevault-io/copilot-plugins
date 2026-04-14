## servidor-npl-laura

> Servidor Node.js/Express que opera a Laura, assistente virtual do escritório NPLADVS (direito trabalhista, Belém/PA). Atende leads via WhatsApp (Z-API), faz triagem, agenda consultas (Google Calendar) e gerencia o funil de vendas.

# Servidor NPL - Laura (Assistente Virtual)

## Visão Geral
Servidor Node.js/Express que opera a Laura, assistente virtual do escritório NPLADVS (direito trabalhista, Belém/PA). Atende leads via WhatsApp (Z-API), faz triagem, agenda consultas (Google Calendar) e gerencia o funil de vendas.

## Integração com CRM
O CRM frontend (hospedado no GitHub Pages, repositório separado) chama diretamente os endpoints deste servidor. **Este é o único backend** — não existe outro servidor para o CRM. Todas as chamadas do CRM vão para `https://servidor-npl.onrender.com`.

### Endpoints da API (todos POST requerem header `x-api-key`)

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/health` | Status do servidor |
| GET | `/api/conversas` | Lista conversas |
| GET | `/api/conversas/:id/mensagens` | Mensagens de uma conversa |
| GET | `/api/metricas` | Leads por etapa, conversas ativas |
| GET | `/api/analytics?dias=30` | Funil de conversão, A/B testing, scoring |
| GET | `/api/relatorio-semanal` | Dados do relatório semanal |
| GET | `/api/documentos/auditoria/:phone` | Mídias recebidas de um telefone |
| POST | `/api/enviar` | Enviar texto via WhatsApp `{phone, message, conversaId, usuario_nome}` |
| POST | `/api/enviar-audio` | Enviar áudio `{phone, audioBase64, conversaId, usuario_nome}` |
| POST | `/api/enviar-arquivo` | Enviar arquivo `{phone, fileUrl, fileName, mediaType, conversaId, usuario_nome}` |
| POST | `/api/pausar` | Pausar IA `{phone, minutes}` |
| POST | `/api/retomar` | Retomar IA `{phone}` |
| POST | `/api/chat` | Proxy Claude para CRM `{system, messages}` |
| POST | `/api/documentos/organizar` | Organizar docs `{phone, nome, tese}` |
| POST | `/api/documentos/cobrar` | Cobrar docs `{phone, nome, auditoria}` |
| POST | `/api/relatorio-semanal` | Disparar relatório |
| GET | `/api/pausar/status?phone=X` | Verificar se IA está pausada para um telefone |

### Resposta do /api/analytics
```json
{
  "periodo": "30 dias",
  "funil": {
    "leads_novos": 100,
    "fizeram_triagem": 60,
    "receberam_oferta": 40,
    "agendaram": 15,
    "perdidos": 20
  },
  "taxas": {
    "triagem": "60%",
    "oferta": "66.7%",
    "agendamento": "37.5%",
    "perda": "20%"
  },
  "leads_por_etapa": { "novo": 20, "contato": 30, "proposta": 30, "convertido": 10, "perdido": 10 },
  "score_medio_por_etapa": { "novo": 5, "contato": 25, "proposta": 55, "convertido": 75 },
  "ab_testing": {
    "A": { "total": 50, "convertido": 8, "taxa": "16%", "nome_variante": "consulta_gratuita" },
    "B": { "total": 50, "convertido": 12, "taxa": "24%", "nome_variante": "sem_risco" }
  },
  "eventos": { "primeiro_contato": 100, "lead_quente": 15, "consulta_agendada": 15, "followup_2h": 40 }
}
```

## Banco de Dados (Supabase — compartilhado com CRM)

### Tabelas principais

**leads** — dados dos leads
- `id`, `nome`, `telefone`, `email`, `escritorio` ('npl'), `instancia`
- `etapa_funil`: novo → contato → proposta → convertido / perdido
- `tese_interesse`, `notas`, `origem`
- `score` (0-100): scoring automático do lead
- `score_detalhes`: critérios (ex: "engajado,resposta_rapida,quer_agendar")
- `ab_variante`: "A" ou "B" (A/B testing)
- `criado_em`, `atualizado_em`, `data_primeiro_contato`

**conversas** — conversas ativas
- `id`, `telefone`, `titulo`, `status` (ativa/finalizada), `escritorio`, `instancia`
- `lead_id`, `etapa_conversa`, `criado_em`

**mensagens** — histórico de mensagens
- `id`, `conversa_id`, `role` (user/assistant), `content`
- `media_url`, `media_type` (audio/image/document)
- `manual` (bool), `usuario_nome`, `criado_em`

**metricas** — eventos rastreados
- `id`, `conversa_id`, `lead_id`, `evento`, `detalhes`, `escritorio`, `criado_em`
- Eventos: primeiro_contato, lead_quente, consulta_agendada, followup_2h, followup_4h, followup_24h, followup_72h, etapa_avancou

**tarefas** — tarefas do CRM
- `id`, `descricao`, `data_limite`, `prioridade`, `status`, `responsavel`

**aprendizados** — lições aprendidas pela IA
- `tipo`, `categoria`, `licao`, `contexto`, `resultado`, `efetividade`, `vezes_usado`

**npl_clientes_processos** — base de clientes antigos (importada)
- `nome_cliente`, `nome_normalizado`, `parte_contraria`, `materia`, `numero_processo`, `status_fase`

## Lead Scoring (automático)
Calculado a cada mensagem do lead. Critérios:
- Engajamento: 3+ msgs (+10), 6+ msgs (+10)
- Velocidade: resposta <2min (+15), <10min (+5)
- Mídia: áudio (+10), documento (+15)
- Urgência: palavras urgentes (+15), "quero agendar" (+20)
- Negativo: hesitante (-10), sem interesse (-30)

## A/B Testing
- Variante A ("consulta_gratuita"): "A consulta inicial é gratuita e sem compromisso"
- Variante B ("sem_risco"): "Você não paga nada pela primeira consulta"
- Atribuída ao criar lead, determinística pelo ID
- Resultados visíveis em /api/analytics

## Modelo de IA
- **Claude Sonnet 4** (claude-sonnet-4-20250514) — modelo principal para conversas
- MAX_TOKENS: 800 (respostas objetivas)
- Janela de contexto: 150 mensagens enviadas ao Claude
- Ficha do lead: 40 mensagens anteriores resumidas
- trimResponse: máx 6 frases / 800 chars

## Agendamento (Google Calendar)
- Detecção: só cria evento quando Laura diz "Agendado!" + data/hora
- Slot extraído da RESPOSTA da Laura (não do texto do lead)
- Anti-duplo: verifica metricas no banco (48h)
- Bloqueios: prefeitura/governo verificados no servidor antes de criar
- Referência: "já está agendada" não dispara novo evento
- Remarcação: detecta pedidos de mudança, cancela antigo + cria novo
- Reserva: 20 minutos por slot
- Escassez: comunica "últimos horários" quando ≤3 slots
- Janela: busca 10 dias úteis

## Bloqueios automáticos
- Prefeitura/governo municipal → não agenda
- Servidor público → pede confirmação
- Vínculo < 3 meses → não agenda
- Prescrição > 2 anos → não agenda
- Lead sem interesse → encerra
- **Trabalhador rural = CLT = ATENDE** (não confundir com governo)

## Follow-ups (automáticos, 8h-20h Belém)
- 2h: texto
- 4h: texto
- 24h: texto
- 72h: texto + marca lead como perdido
- Validação: descarta mensagens inválidas antes de enviar
- Contexto: referencia o caso específico do lead

## Áudio
- ElevenLabs: desativa automaticamente quando sem crédito
- Fallback: OpenAI TTS (formato Opus/OGG nativo WhatsApp)
- Geração: só quando lead envia áudio (não em follow-ups/confirmações)
- CRM: /api/enviar-audio salva media_url para player funcionar

## Mensagens outbound
- Quando atendente envia primeiro (isFromMe), cria conversa + lead
- Mensagem salva no histórico como manual

## Identificação de clientes existentes
- Busca por nome na tabela npl_clientes_processos
- Usa parte_contrária (empresa) para desambiguar homônimos
- 2 tentativas de confirmação, depois segue como lead novo

## Configuração
Variáveis de ambiente no Render:
- `API_KEY` — autenticação dos endpoints POST
- `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `ELEVENLABS_API_KEY`
- `SUPABASE_URL`, `SUPABASE_KEY`
- `ZAPI_INSTANCE_ID`, `ZAPI_TOKEN`, `ZAPI_CLIENT_TOKEN`
- `GOOGLE_CALENDAR_ID`, `GOOGLE_CALENDAR_CREDENTIALS`
- `OSMAR_PHONE`, `ALLOWED_ORIGINS`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/osmarnevesadvogado)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/osmarnevesadvogado)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
