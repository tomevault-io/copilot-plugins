## formulariodetreinamento

> Aplicação web leve, segura e otimizada para registro de treinamentos de funcionários. O sistema foi refatorado para uma arquitetura modular baseada em componentes simples (`index.php` atua como bootstrap).

# Projeto: Ferramenta de Cadastro de Treinamentos (DigitalSat)

Aplicação web leve, segura e otimizada para registro de treinamentos de funcionários. O sistema foi refatorado para uma arquitetura modular baseada em componentes simples (`index.php` atua como bootstrap).

## 📋 Visão Geral

O sistema coleta dados de treinamentos via formulário web e envia notificações detalhadas por e-mail (com anexos) via SMTP, além de gerar relatórios em CSV.

### Stack Tecnológica
*   **Backend:** PHP 8.2 (Apache).
*   **Frontend:** HTML5, Bootstrap 5.3 (via CDN).
*   **Infraestrutura:** Docker & Docker Compose (Multi-Stage Build) com fuso horário ajustado para o Brasil (`America/Sao_Paulo`).
*   **Libs:** `phpmailer/phpmailer`, `vlucas/phpdotenv`.

---

## 🏗️ Estrutura de Arquivos

A estrutura é modularizada para facilitar a manutenção, onde o `index.php` engloba e inicializa a aplicação:

```
/
├── .env.example      # Modelo de configuração
├── composer.json     # Dependências (PHPMailer, Dotenv)
├── docker-compose.yml
├── Dockerfile        # Build Multi-Stage (Builder -> Production)
└── public/
    ├── index.php     # Bootstrap da Aplicação
    ├── config.php    # Configurações globais e de filiais
    ├── functions.php # Funções auxiliares
    ├── controller.php# Processamento (POST/GET) e Envio
    └── view.php      # Estrutura HTML/Frontend
```

---

## ⚙️ Configuração e Execução

### Comandos Rápidos

| Ação | Comando | Descrição |
| :--- | :--- | :--- |
| **Deploy/Iniciar** | `docker compose up -d --build` | Compila a imagem otimizada e inicia a aplicação na porta `:8081`. |
| **Parar** | `docker compose down` | Encerra os containers. |
| **Ver Logs** | `docker compose logs -f` | Monitoramento em tempo real. |

### Configuração `.env`

Copie `.env.example` para `.env` e configure:
*   **`APP_ENV`**: Use `production` para envio real. Se `local`, gera arquivo `public/email_mock.html`.
*   **SMTP Credentials**: Dados do servidor de e-mail.
*   **Configurações de Fuso Horário**: O fuso horário do ambiente Docker (`TZ`) está configurado como `America/Sao_Paulo` (Brasília).

---

## 🛡️ Segurança (Hardened)

O projeto implementa camadas rigorosas de segurança para operar em produção:

1.  **CSRF Protection:** Token criptográfico único por sessão para prevenir falsificação de requisição.
2.  **Rate Limiting:** Bloqueio de envio em massa (trottle de 30 segundos por sessão).
3.  **Strict Whitelisting:** Validação de entradas (`filial`, `tipo_treinamento`) contra listas permitidas restritas em `config.php`.
4.  **Secure Upload:**
    *   Validação de **MIME Type Real** (conteúdo binário) do arquivo.
    *   Limite de tamanho (5MB).
    *   Permite apenas PDF e Imagens.
5.  **Session Hardening:** Cookies configurados com `HttpOnly`, `Secure` (se HTTPS) e `SameSite=Strict`.
6.  **Sanitização:** Todos os inputs passam por `htmlspecialchars` e `strip_tags`.

## 🐳 Otimização Docker
Utiliza **Multi-Stage Build**:
1.  **Stage 1 (Builder):** Instala dependências do sistema (Git, Zip) e roda `composer install`.
2.  **Stage 2 (Final):** Imagem limpa contendo apenas PHP+Apache, código fonte e pacote `tzdata` (configurado no fuso horário `America/Sao_Paulo`). Sem restos de cache ou ferramentas de build.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wppslander)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wppslander)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
