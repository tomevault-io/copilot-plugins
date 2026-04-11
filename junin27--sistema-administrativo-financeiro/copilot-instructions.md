## sistema-administrativo-financeiro

> O plano gratuito do Google Gemini tem limites de requisições por minuto. Quando esses limites são atingidos, você recebe o erro 429.

# Limitações da API Google Gemini - Plano Gratuito

## Problema: Quota Excedida (Erro 429)

O plano gratuito do Google Gemini tem limites de requisições por minuto. Quando esses limites são atingidos, você recebe o erro 429.

## Soluções Implementadas

### 1. Mudança de Modelo
- **Antes:** `gemini-2.0-flash-exp` (experimental, limites mais restritivos)
- **Agora:** `gemini-1.5-flash` (estável, limites melhores)

### 2. Retry Automático
O sistema agora tenta novamente automaticamente quando recebe erro de quota:
- Até 3 tentativas
- Delay progressivo entre tentativas (backoff exponencial)
- Aguarda o tempo sugerido pela API quando disponível

### 3. Mensagens de Erro Melhoradas
Mensagens mais claras quando a quota é excedida, incluindo links para documentação.

## Limites do Plano Gratuito

Segundo a documentação do Google:
- **gemini-1.5-flash:** ~15 requisições por minuto
- **gemini-2.0-flash-exp:** Limites mais restritivos (experimental)

## Como Monitorar Uso

1. Acesse: https://aistudio.google.com/app/apikey
2. Clique na sua API key
3. Veja estatísticas de uso e limites

## Soluções Adicionais (se necessário)

### Opção 1: Aguardar
Aguarde alguns minutos e tente novamente. Os limites são resetados periodicamente.

### Opção 2: Upgrade do Plano
Se precisar de mais requisições, considere fazer upgrade do plano no Google AI Studio.

### Opção 3: Implementar Rate Limiting
Adicionar delays entre requisições no código para evitar atingir os limites.

## Configuração Atual

O modelo pode ser alterado no arquivo `backend/src/config/settings.py`:

```python
gemini_model: str = Field(default="gemini-1.5-flash", ...)
```

Ou via variável de ambiente:
```bash
GEMINI_MODEL=gemini-1.5-flash
```

## Modelos Disponíveis

- `gemini-2.5-flash` - **Recomendado** (mais recente, rápido, estável)
- `gemini-2.0-flash` - Alternativa estável
- `gemini-2.0-flash-exp` - Experimental
- `gemini-2.5-pro` - Mais poderoso, mas mais lento
- `gemini-pro-latest` - Versão "latest" do modelo pro

**Nota:** Os modelos disponíveis variam conforme a versão da API. Use o script `test_models.py` para verificar quais modelos estão disponíveis na sua API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/junin27)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/junin27)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
