## fluydo-ia

> Siga estes passos **na ordem**:

# Corrigir: "Chave da API Gemini inválida ou não configurada"

Siga estes passos **na ordem**:

---

## 1. Obter uma chave da API

1. Abra no navegador: **https://aistudio.google.com/apikey**
2. Faça login com sua conta Google.
3. Clique em **"Create API key"** (ou **"Get API key"**).
4. Copie a chave (algo como `AIzaSy...`). Deixe ela na área de transferência.

---

## 2. Criar ou editar o .env.local

1. Abra a pasta do projeto no Explorer:  
   **C:\Dropbox\Dados\Altair\Fluydo-IA**

2. Verifique se existe o arquivo **.env.local** nessa pasta (na mesma pasta do **package.json**).  
   - Se não existir, crie um arquivo de texto e salve com o nome: **.env.local**  
   - Se existir, abra com o Bloco de notas ou Cursor.

3. Deixe **só esta linha** no arquivo (troque pela sua chave):

   ```
   GEMINI_API_KEY=AIzaSy_sua_chave_aqui_sem_aspas
   ```

   **Importante:**
   - Não use aspas: nem `"` nem `'`
   - Não deixe espaço antes nem depois do `=`
   - O nome da variável é exatamente: **GEMINI_API_KEY**
   - Cole a chave que você copiou no lugar de `AIzaSy_sua_chave_aqui_sem_aspas`

4. Salve o arquivo e feche.

---

## 3. Reiniciar o servidor

O Next.js só lê o `.env.local` quando inicia. Por isso:

1. Feche o terminal onde está rodando **npm run dev** (ou a janela do **INICIAR-CHAT.bat**).
2. Abra de novo e rode de novo: **npm run dev** (ou duplo clique em **INICIAR-CHAT.bat**).
3. Espere aparecer **"Ready"**.

---

## 4. Testar se a chave foi carregada

No navegador, abra:

**http://localhost:3000/api/check-env**

- Se aparecer **"GEMINI_API_KEY_definida": true** → a chave está carregada. Teste o chat de novo.
- Se aparecer **false** → o `.env.local` está no lugar errado, com nome errado ou com variável errada. Volte ao passo 2.

---

## 5. Testar o chat

Abra **http://localhost:3000**, digite **oi** e envie. Deve responder sem a mensagem de erro da chave.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AltairDaxxel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AltairDaxxel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
