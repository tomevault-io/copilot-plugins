## hackgurupi

> ❌ Erro ao solicitar análise de IA: Falha ao chamar Gemini após múltiplas tentativas

# 🔍 Diagnóstico de Problemas com Google Gemini API

## ❌ Erro Comum
```
❌ Erro ao solicitar análise de IA: Falha ao chamar Gemini após múltiplas tentativas
```

## 🛠️ Melhorias Implementadas

### 1. **Logs Detalhados**
Agora o sistema mostra exatamente o que está acontecendo em cada tentativa:
- ✅ Número da tentativa (1/3, 2/3, 3/3)
- ✅ Status HTTP recebido (200, 400, 401, 403, 404, 429, 500, etc)
- ✅ Mensagens de erro específicas
- ✅ Tempo de espera entre tentativas

### 2. **Mais Tentativas**
- ❌ **ANTES:** 2 tentativas apenas
- ✅ **AGORA:** 3 tentativas com tempo de espera progressivo

### 3. **Timeouts Configurados**
- ✅ **Conexão:** 30 segundos
- ✅ **Leitura:** 60 segundos

### 4. **Mensagens de Erro Específicas**
- **HTTP 400:** Requisição inválida (problema no formato dos dados)
- **HTTP 401/403:** Chave da API inválida ou sem permissão
- **HTTP 404:** Modelo não encontrado
- **HTTP 429:** Rate limiting (muitas requisições)
- **HTTP 500+:** Erro no servidor do Google

## 🧪 Como Testar

### 1. **Teste Simples da API**
Acesse o endpoint de teste que criei:

```bash
# Via navegador ou curl
curl http://localhost:8080/test/gemini
```

Este endpoint:
- ✅ Verifica se a chave está configurada
- ✅ Faz uma requisição simples ao Gemini
- ✅ Mostra logs detalhados
- ✅ Retorna sucesso ou erro específico

### 2. **Verifique os Logs**
Após fazer uma requisição, verifique o console. Você verá algo assim:

**Se funcionar:**
```
🧪 === TESTE DE CONEXÃO COM GEMINI API === 🧪
🔑 Chave da API (primeiros 10 caracteres): AIzaSyAeCh...
🔄 Iniciando comunicação com Google Gemini API...
📡 Tentativa 1/3 de conectar ao Gemini...
📊 Status HTTP recebido: 200
✅ Resposta recebida do Google Gemini com sucesso!
✅ Teste concluído com sucesso!
```

**Se der erro:**
```
🧪 === TESTE DE CONEXÃO COM GEMINI API === 🧪
🔑 Chave da API (primeiros 10 caracteres): AIzaSyAeCh...
🔄 Iniciando comunicação com Google Gemini API...
📡 Tentativa 1/3 de conectar ao Gemini...
📊 Status HTTP recebido: 403
❌ Gemini API retornou HTTP 403: {...detalhes do erro...}
⚠️ Erro na tentativa 1: Erro HTTP 403 na API do Gemini (Chave da API inválida ou sem permissão)
⏱️  Aguardando 1000 ms antes da próxima tentativa...
...
❌ Falha após 3 tentativas. Último erro: ...
```

## 🔑 Possíveis Problemas e Soluções

### 1. **Chave da API Inválida ou Expirada**

**Sintomas:**
- HTTP 401 ou 403
- Mensagem: "Chave da API inválida ou sem permissão"

**Solução:**
1. Acesse: https://makersuite.google.com/app/apikey
2. Gere uma nova chave
3. Atualize no `application.properties`:
```properties
gemini.api.key=SUA_NOVA_CHAVE_AQUI
```

### 2. **Modelo Não Disponível**

**Sintomas:**
- HTTP 404
- Mensagem: "Modelo não encontrado"

**Solução:**
O modelo `gemini-2.0-flash-exp` pode ter sido descontinuado. Tente usar outro modelo:

```java
// Em GeminiClient.java, linha 37, altere para:
String urlStr = "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key="
```

Modelos disponíveis:
- `gemini-1.5-flash` (recomendado, mais rápido)
- `gemini-1.5-pro` (mais capaz, mais lento)
- `gemini-pro` (versão anterior)

### 3. **Rate Limiting (Muitas Requisições)**

**Sintomas:**
- HTTP 429
- Mensagem: "Rate limited"

**Solução:**
- O tier gratuito permite **15 requisições por minuto**
- Aguarde 1 minuto e tente novamente
- O sistema agora espera automaticamente e tenta novamente

### 4. **Problemas de Rede/Firewall**

**Sintomas:**
- Timeout
- "Connection refused"
- Erro antes de receber status HTTP

**Solução:**
1. Verifique sua conexão com a internet
2. Verifique se o firewall não está bloqueando `generativelanguage.googleapis.com`
3. Se estiver atrás de proxy, configure:
```java
// Adicione antes da conexão:
System.setProperty("http.proxyHost", "seu-proxy");
System.setProperty("http.proxyPort", "porta");
```

### 5. **Quota Excedida**

**Sintomas:**
- HTTP 429 persistente
- Mensagem sobre quota

**Solução:**
- Tier gratuito: **1 milhão de tokens por mês**
- Aguarde o próximo ciclo de billing (próximo mês)
- Ou faça upgrade para um plano pago

## 📊 Verificando Uso da API

Acesse seu dashboard do Google AI Studio:
- URL: https://makersuite.google.com/app/apikey
- Veja quantas requisições você fez
- Verifique se está perto do limite

## 🚀 Teste Agora!

1. **Reinicie a aplicação** (para carregar as melhorias)
2. **Acesse:** http://localhost:8080/test/gemini
3. **Veja os logs** no console
4. **Identifique o erro específico**
5. **Aplique a solução correspondente**

## 💡 Dica: Teste com Requisição Direta

Se quiser testar a chave diretamente (fora da aplicação):

```bash
curl -X POST \
  'https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=SUA_CHAVE_AQUI' \
  -H 'Content-Type: application/json' \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Diga olá"
      }]
    }]
  }'
```

Se isso funcionar, o problema está no código Java.
Se não funcionar, o problema está na chave ou na rede.

## 📞 Precisa de Ajuda?

Se nenhuma dessas soluções funcionar:
1. Copie os logs completos do console
2. Verifique o status HTTP retornado
3. Verifique a mensagem de erro detalhada
4. Me informe o que encontrou!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giseleveloso)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/giseleveloso)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
