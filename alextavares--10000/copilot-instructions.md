## 10000

> Boas notícias! A autenticação com a conta de serviço foi bem-sucedida. No entanto, ainda temos um erro ao tentar acessar a API do Gemini: `Insufficient Permission`.

# Habilitando a API do Gemini para a Conta de Serviço

Boas notícias! A autenticação com a conta de serviço foi bem-sucedida. No entanto, ainda temos um erro ao tentar acessar a API do Gemini: `Insufficient Permission`.

Este erro indica que a conta de serviço `android-habitai@android-habitai.iam.gserviceaccount.com` não tem permissão para acessar a API do Gemini ou que a API do Gemini não está habilitada para o projeto `android-habitai`.

## Passo 1: Habilitar a API do Gemini

1. Acesse o [Console do Google Cloud](https://console.cloud.google.com/)
2. Certifique-se de que o projeto `android-habitai` está selecionado
3. No menu lateral, vá para **"APIs e Serviços"** > **"Biblioteca"**
4. Na barra de pesquisa, digite **"Generative Language API"** ou **"Gemini API"**
5. Clique na API quando aparecer nos resultados
6. Clique no botão **"Habilitar"**

## Passo 2: Conceder Permissões à Conta de Serviço

Depois de habilitar a API, você precisa conceder as permissões corretas à conta de serviço:

1. No menu lateral do Console do Google Cloud, vá para **"IAM e Administrador"** > **"IAM"**
2. Localize a conta de serviço `android-habitai@android-habitai.iam.gserviceaccount.com`
3. Clique no ícone de edição (lápis) ao lado da conta
4. Clique em **"Adicionar outro papel"**
5. Pesquise e adicione os seguintes papéis:
   - **"AI Platform User"** ou **"Vertex AI User"**
   - **"Cloud AI Generative Language API User"**
6. Clique em **"Salvar"**

## Passo 3: Verificar Novamente

Depois de habilitar a API e conceder as permissões, execute novamente o script de teste:

```bash
node test-service-account.js
```

Se tudo estiver configurado corretamente, você deverá ver uma mensagem de sucesso e uma lista dos modelos disponíveis.

## Passo 4: Iniciar o Servidor Genkit

Depois de confirmar que a conta de serviço tem acesso à API do Gemini, você pode iniciar o servidor Genkit:

```bash
node genkit-server.js
```

E testar com:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"name":"Alexandre"}' http://localhost:3000/api/hello
```

## Observações Importantes

- As alterações de permissão podem levar alguns minutos para se propagar
- Certifique-se de que o faturamento está habilitado para o projeto, pois a API do Gemini pode exigir isso
- Se você continuar enfrentando problemas, verifique as cotas e limites da API no Console do Google Cloud

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alextavares)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alextavares)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
