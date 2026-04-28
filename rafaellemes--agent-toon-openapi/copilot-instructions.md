## agent-toon-openapi

> - **ingest-api**: URL ou ficheiro OpenAPI/Swagger.

# TooN_OpenApi — Contexto Global do Agente

## Skills Disponíveis
- **ingest-api**: URL ou ficheiro OpenAPI/Swagger.
- **consult-api**: Perguntas sobre APIs ingeridas, geração de código.
- **diff-api**: "diff", "comparar versões", "o que mudou".
- **validate-api**: "validar payload", "checar JSON". Também usada
  internamente pelo consult-api após gerar código.
- **testgen-api**: "gerar testes", "criar testes de integração".
- **exportgen-api**: "exportar", "gerar bloco TooN", "injetar contexto",
  "copiar para outra thread". Gera bloco compacto pronto para colar em
  qualquer system prompt ou thread de outro agente.

## Arquitetura de Storage
- A pipeline isola os dados processados das rotinas operacionais no diretório raiz do banco, sob namespace específico.
- `toon.txt`: descoberta rápida (arquivo que é lido pelo LLM no início para entender o que a API faz, agora rico com nós de Requisições e Respostas e varredura de Array em payloads `body:a!`).
- `mapping.json`: dados técnicos exatos (usado de forma granular, lido via `jq`).

### Como Gerar e Ler o Toon
1. **Para Criar (Ingerir API):**
   Rode os scripts em cascata (substituindo pela URL/Arquivo da API):
   `python .claude/skills/ingest-api/scripts/parse_spec.py <API_URL> > /tmp/spec.json`
   `python .claude/skills/ingest-api/scripts/transform_toon.py /tmp/spec.json`
   *(Isso criará a estrutura isolada sob `storage/apis/<namespace>/`)*
2. **Para Ler/Consultar:**
   Leia primeiramente o arquivo semântico pelo root: `cat storage/apis/<namespace>/toon.txt` para adquirir entendimento da topologia e dos payloads gerados em `Req:` e `Res:`. Em seguida pesquise a operação final usando `jq` no `mapping.json`.

### Referência Rápida: Gramática TooN
Ao analisar o arquivo `toon.txt`, utilize este glossário de decodificação de contrato absoluto:
- **Métodos**: `GET`, `POST`, `PUT`, `DEL`, `PATCH`, `HEAD`, `OPT` (nomes explícitos)
- **Tipos de Dado**: `s`=string, `i`=integer, `b`=boolean, `a`=array, `o`=object
- **Marcadores**: `!`=obrigatório, `?`=opcional

## Estratégia de Contexto para Geração de Código

### 1º — Pedido actual (prioridade máxima)
"gera em Go" → usa Go, ignora histórico.

### 2º — Histórico da thread (fallback passivo)
Linguagem, cliente HTTP, padrão, nomenclatura, auth mencionados
anteriormente → aplicar sem precisar repetir.

### 3º — Sem contexto
Perguntar: "Em qual linguagem e com quais convenções?"

### Regra de ouro
Contexto afecta APENAS a forma do código.
Dados técnicos (url, método, params) vêm SEMPRE do mapping.json.

## Validação Automática Pós-Geração de Código
Após gerar código de integração, SEMPRE:
1. Extrair payload do snippet
2. Chamar validate_payload.py contra o mapping
3. Válido → entregar | Inválido → corrigir silenciosamente (máx. 2x)

## Regras Globais
1. NUNCA inventar campos, params ou endpoints não presentes no mapping.
2. NUNCA cat do mapping.json completo — sempre jq cirúrgico.
3. SEMPRE resolver contexto antes de gerar código.
4. SEMPRE validar payload gerado antes de entregar.
5. SEMPRE informar custo estimado em tokens.
6. Se endpoint não existir no toon.txt — declarar e listar disponíveis.
7. Se um namespace faltar em storage/apis/ — instruir a usar ingest-api.
8. Em multi-namespace — NUNCA misturar params ou endpoints de APIs distintas.

## Auth da Spec
O campo `AUTH:` no toon.txt e o campo `security` no mapping.json são
populados automaticamente pelo transform_toon.py a partir dos
securitySchemes da spec. O agente deve usar esses dados para gerar
código com autenticação correcta sem que o dev precise declarar na thread.
Prioridade: auth declarada na thread > auth da spec > placeholder comentado.

## Geração de Cliente HTTP (clientgen-api)

### Lógica de estrutura
1. Pedido explícito ("standalone", "nova classe") → seguir
2. Ficheiro/classe aberta na thread → standalone
3. Sem contexto, rota única → classe com um método
4. Sem contexto, API completa/tag → classe completa

### Responsabilidade
O script extrai o contrato. O LLM gera o código.
Sem restrição de linguagem ou framework.

## Caminhos de Storage
- toon.txt:    storage/apis/<namespace>/toon.txt
- mapping:     storage/apis/<namespace>/mapping.json
- metrics:     storage/apis/<namespace>/metrics.json
- diffs:       storage/diffs/diff_<base>_vs_<target>_<ts>.txt
- validations: storage/validations/validation_<op>_<ts>.txt
- tests:       storage/tests/<ns>_<op>_<lang>_<ts>.<ext>
- clients:     storage/clients/<ns>_<scope>_<lang>_<ts>.<ext>
- exports:     storage/exports/<ns>_<scope>_<ts>.txt  (apenas com --save)


## Linguagem Cognitiva (Cognitive Language)
Você deve pensar e responder ao usuário exatamente no mesmo idioma em que ele fez o prompt. Exemplo: Se o prompt for em Inglês, pense e responda em Inglês. Se for em Português do Brasil, pense e responda em Português do Brasil (PT-BR). Se for em Espanhol, etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rafaellemes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
