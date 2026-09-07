## claude-vps-studio

> Você está em um VPS Ubuntu rodando o stack **claude-vps-studio**.

# VPS __DOMAIN__ — guia operacional

Você está em um VPS Ubuntu rodando o stack **claude-vps-studio**.
Este arquivo é o seu briefing — leia antes de tocar em qualquer coisa.

> **Nota:** os marcadores `__DOMAIN__` neste arquivo são substituídos pelo
> `setup.sh` durante a instalação inicial. Se você está vendo `__DOMAIN__`
> literal abaixo, o `setup.sh` ainda não foi executado.

---

## O que está no ar

| Subdomínio | Para que serve | Pasta |
|---|---|---|
| `learn.__DOMAIN__` | App de estudos curados gerados em conversa com o Claude. | `~/sites/learn/` |

Quando você criar projetos novos, adicione-os à tabela acima.

---

## Infraestrutura

- **Reverse proxy:** Traefik v3.7 (`~/infra/traefik/`) com Let's Encrypt automático (HTTP-01).
- **Rede docker externa:** `web` — todo container que vai ser exposto entra nela.
- **Docker daemon:** roda como root; o usuário do sistema está no grupo `docker`.
- **Portas públicas esperadas:** 80, 443 (HTTP/HTTPS). Adapte UFW/firewall conforme seu setup base.

### Subir um projeto novo (resumo)

1. `mkdir -p ~/sites/<slug> && cp ~/sites/_template/docker-compose.yml ~/sites/<slug>/`
2. Trocar placeholders no `docker-compose.yml`:
   - `SUA_IMAGEM:tag`
   - `NOME_DO_PROJETO` (3 ocorrências)
   - `SUBDOMINIO` (vira `<sub>.__DOMAIN__`)
   - `PORTA_INTERNA` (porta interna do container)
3. `cd ~/sites/<slug> && docker compose up -d`
4. Aguarde ~15–30s pra emissão do cert. Teste com `curl -sS -o /dev/null -w "%{http_code}\n" https://<sub>.__DOMAIN__`.

Detalhes em `~/infra/README.md`.

---

## learn.__DOMAIN__ — o app de estudos

**Conceito:** o humano te pede pra explicar/estudar um tema. Você gera uma página HTML rica em `~/sites/learn/public/<slug>/index.html` seguindo o design system definido, e o nginx do container `learn` serve direto (sem rebuild, sem deploy — basta salvar o arquivo).

### Fluxo obrigatório ao criar/editar um estudo

1. **Leia `~/sites/learn/STYLE.md` primeiro.** Esse arquivo define o design system (paleta, tipografia, componentes prontos no `style.css`). Não invente CSS local.
2. Pergunte ao humano o suficiente pra calibrar profundidade e foco — não saia gerando 2000 linhas sem alinhar.
3. Crie em `~/sites/learn/public/<slug>/index.html`. Slug em `kebab-case`.
4. Sempre carregue os CSS globais — nunca inline:
   ```html
   <link rel="stylesheet" href="/assets/fonts.css" />
   <link rel="stylesheet" href="/assets/style.css" />
   ```
5. **Atualize `~/sites/learn/public/index.html`** adicionando o card do novo estudo no topo da `<ol class="studies">`.
6. Entregue o link final: `https://learn.__DOMAIN__/<slug>/`.

### Estética — o que NÃO fazer no learn

- Tema escuro
- Monospace dominante (só pra código/labels/diagramas)
- Glow neon, gradientes vibrantes, animações decorativas
- Emojis decorativos
- Bibliotecas externas (Tailwind, etc) — só `style.css` global
- Imagens raster pra coisa que SVG resolve

Vibe alvo: *The New Yorker*, *Stripe Press*, blog da Anthropic.

### Iterar uma página existente

Edite o arquivo direto. Nginx serve estático, refresh do browser basta. Sem rebuild, sem `docker restart`.

---

## Sessões Claude paralelas (manager + threads)

Este VPS suporta uma arquitetura de **múltiplas sessões Claude rodando em paralelo**, cada uma acessível via web pelo Remote Control.

- **`manager`** — sessão fixa em tmux. Orquestra (cria, lista, mata) threads filhas.
- **`thread-<slug>`** — sessões temporárias, uma por estudo.
- **Script helper:** `~/bin/claude-thread {new,list,url,kill,peek}`.

Detalhes em `~/PROMPT.md` (prompt inicial pra você como "Claude principal") e `~/manager/CLAUDE.md` (briefing do manager).

---

## Convenções gerais

- **Não derrube containers existentes** sem confirmar com o humano.
- **Não mexa em `~/infra/traefik/`** a menos que ele peça explicitamente.
- **Português brasileiro** nas respostas e nos textos das páginas do learn (ajuste se preferir outro idioma).
- **Sem emoji** nas páginas geradas. Em conversa com o humano, à vontade.

## Diagnóstico rápido

```bash
docker ps                                      # o que tá rodando
docker logs --tail 50 -f traefik              # logs do proxy
docker logs --tail 50 -f <container>          # logs de um app
claude-thread list                            # sessões Claude ativas
curl -sS -o /dev/null -w "%{http_code}\n" https://<dominio>   # health check
```

## Modelo Claude

Quando precisar de Claude via API, o modelo padrão atual é `claude-opus-4-7` (Opus 4.7, contexto 1M).

---

*Este arquivo é vivo. Se você mudar a infra de forma significativa, atualize aqui.*

---
> Source: [luizomf/claude-vps-studio](https://github.com/luizomf/claude-vps-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
