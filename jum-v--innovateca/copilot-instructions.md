## innovateca

> 📄 **PRD – InnovaTeca (MVP Hackathon)**


📄 **PRD – InnovaTeca (MVP Hackathon)**

## 1. Visão Geral

O **InnovaTeca** é uma plataforma que conecta empresas em busca de soluções tecnológicas com universidades/NITs que possuem inovações licenciáveis. O MVP foca em:

* **Entrada da demanda empresarial** (descrição do problema ou dor).
* **Busca semântica por tecnologias** cadastradas por universidades.
* **Conexão simples** via formulário de contato.

---

## 2. Objetivos do MVP

* Validar a proposta de **matchmaking IA + licenciamento** em até 2 dias de hackathon.
* Garantir que **empresas consigam encontrar** tecnologias compatíveis a partir de sua “dor”.
* Garantir que **universidades consigam cadastrar** suas tecnologias de forma rápida e clara.
* Gerar uma experiência fluida e demonstrável no pitch.

---

## 3. Personas

* **Empresa/Executivo de Inovação:** busca soluções práticas para problemas de negócio.
* **Universidade/NIT:** deseja divulgar e licenciar suas tecnologias para o setor produtivo.

---

## 4. Jornadas do Usuário

### **Jornada 1: Empresa (Novo Usuário - Em Busca de uma Solução)**

Este fluxo é projetado para capturar o interesse e converter um visitante em um usuário ativo de forma fluida.

1. **Aterrissagem na Página Inicial:**
    - O usuário chega e encontra dois pontos de interação principais:
        - **(A) O Campo de Prompt:** Um convite claro para descrever uma necessidade ou desafio tecnológico.
        - **(B) O Feed de Tecnologias:** Uma vitrine com as últimas inovações cadastradas (com mock data para MVP), permitindo a descoberta passiva.
2. **Primeira Interação (Gatilho de Engajamento):**
    - O usuário digita sua "dor" ou desafio no campo de prompt (ex: *"preciso de um bioplástico mais resistente e barato"*).
    - Ao clicar para buscar, a plataforma não mostra os resultados imediatamente. Em vez disso, aciona o **Modal de Login/Cadastro**.
3. **Conversão (Criação de Conta):**
    - O modal informa que, para ver as tecnologias compatíveis, é preciso fazer parte da plataforma.
    - O usuário clica em **"Cadastre-se"**.
    - Ele é redirecionado para o **Formulário de Cadastro de Empresa**, onde preenche informações básicas (para MVP pode ser só **nome, email e senha**, com CNPJ e dados avançados opcionais).
4. **Acesso e Busca (O "Matchmaking"):**
    - Após o cadastro e login, o usuário é levado para o ambiente interno da plataforma.
    - Ao tentar fazer a busca de novo o usuário é redirecionado para uma página de **     Chatbot,** listando as tecnologias com o maior **"Grau de Compatibilidade"**.
    - A página de Chatbot também contém um prompt abaixo, caso o usuário queira iterar mais sobre a sua pesquisa.
5. **Análise e Conexão:**
    - O usuário clica em uma tecnologia de interesse para ver a **Página de Detalhes da Tecnologia**.
    - Nesta página, ele encontra as informações-chave: resumo, TRL, descrição completa e a universidade responsável.
    - Se estiver convencido, ele clica em **"Entrar em contato com a universidade"**, que abre um **formulário de interesse**.
    - Esse formulário envia uma notificação/email diretamente para o NIT responsável pela tecnologia, iniciando a conexão.

---

### **Jornada 2: Universidade/NIT (Novo Usuário - Ofertando uma Tecnologia)**

Este fluxo é mais direto, focado em popular a plataforma com inovações.

1. **Aterrissagem e Acesso:**
    - Um representante do NIT chega à página inicial.
    - Ele clica em **"Login"** ou **"Entrar"**, e em seguida na opção para **Cadastrar Universidade**.
2. **Cadastro Institucional:**
    - Ele é direcionado ao **Formulário de Cadastro de Universidade**, onde preenche os dados institucionais básicos (Nome da universidade, email institucional, etc.).
3. **Cadastro de Tecnologia:**
    - Após o login, o NIT pode **cadastrar uma nova tecnologia**.
    - Ele preenche o **Formulário de Cadastro de Tecnologia** (título, resumo, descrição detalhada, TRL).
    - Para apoiar, o sistema pode oferecer uma função de **IA que gera automaticamente um resumo em linguagem “business-friendly”**.
4. **Publicação:**
    - A tecnologia cadastrada fica disponível na plataforma, visível no feed da página inicial e pronta para ser encontrada pela busca semântica das empresas.

---

👉 Assim, o **MVP** fica mais enxuto e viável para 2 dias:

- **Empresa**: descobre tecnologias via busca → contato direto com universidade.
- **Universidade**: cadastra tecnologias → fica disponível no feed e na busca.

Quer que eu formate isso em um **PRD enxuto do MVP revisado** (com user stories + critérios de aceitação) para já estar pronto para o hackathon?

---

## 5. Funcionalidades do MVP

### **Must Have (obrigatório para 2 dias)**

* [ ] Landing page com **prompt de busca** e **feed de tecnologias (mock inicial)**.
* [ ] Cadastro/login básico (email + senha).
* [ ] Cadastro de tecnologias (formulário simples).
* [ ] Armazenamento em banco de dados (tecnologias + usuários).
* [ ] Busca semântica via embeddings (ex: OpenAI ou Hugging Face).
* [ ] Página de detalhes da tecnologia.
* [ ] Botão “Entrar em contato” → formulário → envio de email/notificação ao NIT.

### **Nice to Have (se sobrar tempo no hackathon)**

* [ ] Geração automática de resumo “business-friendly” da tecnologia via IA.
* [ ] Grau de compatibilidade mostrado como porcentagem.
* [ ] Analytics simples para NIT (ex: nº de visualizações da tecnologia).

---

## 6. Critérios de Aceitação

### Para Empresa

* Quando eu digito uma dor no prompt e faço cadastro, **vejo resultados relevantes** listados por compatibilidade.
* Posso clicar em uma tecnologia e **acessar detalhes claros**.
* Posso clicar em “Entrar em contato” e **enviar interesse ao NIT** com sucesso.

### Para Universidade/NIT

* Posso criar uma conta institucional.
* Posso cadastrar uma tecnologia com **título, resumo, descrição e TRL**.
* Ao salvar, a tecnologia aparece no feed e pode ser encontrada por empresas.

---

## 7. Regras de Negócio

* Somente usuários cadastrados podem ver resultados completos.
* Cada tecnologia deve estar vinculada a uma universidade/NIT.
* Contato entre empresa e universidade é iniciado **via formulário interno**, não diretamente por email pessoal (para evitar spam).

---

## 8. Stack Técnica (sugestão para hackathon)

* **Frontend:** Next.js + Tailwind + Shadcn.
* **Ícones:** Lucide.
* **Backend/API:** Next.js API routes.
* **Banco:** PostgreSQL (Supabase ou Neon).
* **IA/Busca:** embeddings via OpenAI (`text-embedding-3-small`) + banco vetorial (Supabase pgvector).
* **Framework de agente de IA:** Mastra AI.
* **Provedor de LLm:** Mastra AI.
* **Auth:** NextAuth.js (email/senha simples).
* **Email:** Resend (para envio de interesse ao NIT).

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jum-v)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jum-v)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
