## gaia

> Orientações para o Claude Code (e outros agentes) ao trabalhar neste repositório.

# CLAUDE.md

Orientações para o Claude Code (e outros agentes) ao trabalhar neste repositório.

## Sobre o projeto

Site institucional **estático** da **Gaia Soluções Ambientais**, empresa de consultoria, regularização e gestão ambiental para propriedades rurais e empresas (CAR, licenciamento, laudos, reflorestamento, consultoria).

Objetivo do site: apresentar a empresa, demonstrar autoridade técnica e **converter visitantes em solicitações de orçamento** (contato facilitado via WhatsApp e formulário).

## Stack

- **HTML5 semântico** — página única (`index.html`)
- **CSS3** — variáveis CSS, Flexbox, Grid, `clamp()`, media queries
- **JavaScript Vanilla** (sem build/bundler)
- Bibliotecas via CDN: **AOS** (animações on scroll), **Font Awesome**, **Google Fonts**
- Formulário: **Web3Forms**
- Hospedagem: **GitHub Pages** (repo `joaomagalha/Gaia`)
- Domínio: `gaiasolucoesambientais.com.br` (Registro.br, configurado em `CNAME`)

> Não há `package.json`, build, testes ou dependências de Node. É HTML/CSS/JS puro servido estaticamente.

## Estrutura de arquivos

```
index.html              # Página única com todas as seções
CNAME                   # Domínio personalizado do GitHub Pages
Css/
  style.css             # Estilos principais (desktop-first base + componentes)
  responsivo.css        # Media queries / ajustes responsivos
Js/
  script.js             # Inicialização geral (AOS, etc.)
  menu.js               # Menu mobile / navegação responsiva
  efeitos.js            # Efeitos visuais e microinterações
  formulario.js         # Lógica de envio do formulário (Web3Forms)
Imagens/                # Banners, logos, fotos dos serviços
```

Seções da página (em `index.html`, na ordem): Header → Hero → Sobre Nós → Diferenciais → Serviços → Como Trabalhamos → Projetos → Depoimentos → CTA → Formulário → Footer.

## Como rodar / visualizar

Por ser estático, basta abrir `index.html` no navegador, ou servir localmente:

```bash
python3 -m http.server 8000
# acesse http://localhost:8000
```

O deploy é automático: `git push` para a branch `main` publica no GitHub Pages.

## Design System

**Cores** (definir/usar como variáveis CSS):

| Token | Hex |
|-------|-----|
| Verde Principal | `#5D6640` |
| Verde Escuro | `#2E3224` |
| Marrom | `#77604B` |
| Branco | `#FFFFFF` |
| Cinza Claro | `#F5F5F5` |

**Tipografia:** Títulos em **Montserrat**, textos em **Poppins**. Hierarquia h1–h5 com tipografia fluida (`clamp()`).

**Estética:** clean, moderna, elegante, institucional, inspirada na natureza, premium.

## Responsividade — Mobile First

Breakpoints de referência: Mobile `320px+` · Tablet `768px+` · Desktop `1024px+` · Ultra Wide `1700px+`.
Prioridades: legibilidade, escalabilidade, performance e boa distribuição dos cards.

## Convenções de código

- HTML semântico e acessível (alt em imagens, labels em formulários, hierarquia de headings).
- Nomes de classes claros e descritivos.
- Preferir variáveis CSS para cores/espaçamentos em vez de valores fixos repetidos.
- Manter a separação atual: estilos base em `style.css`, ajustes responsivos em `responsivo.css`.
- JS organizado por responsabilidade (um arquivo por área: menu, efeitos, formulário).
- Evitar código desnecessário e priorizar performance.
- **Preservar sempre a identidade visual da Gaia** (paleta e tipografia acima).

## Diretrizes de UX/UI

Ao sugerir ou implementar mudanças: priorizar clareza visual, espaçamentos consistentes, boa hierarquia e **foco em conversão**. Facilitar o acesso ao WhatsApp. Considerar mobile antes do desktop.

---
> Source: [joaomagalha/Gaia](https://github.com/joaomagalha/Gaia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
