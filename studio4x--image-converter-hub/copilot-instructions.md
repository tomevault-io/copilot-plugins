## image-converter-hub

> Este arquivo orienta agentes de IA dentro do repositório Image Converter Hub.

# AGENTS.md — Image Converter Hub

## Função deste arquivo

Este arquivo orienta agentes de IA dentro do repositório Image Converter Hub.

Use também:

- `AI_RULES.md` — stack, bibliotecas e padrões gerais do projeto.

Em caso de conflito:

1. regra explícita do usuário;
2. `AGENTS.md`;
3. `AI_RULES.md`;
4. padrão já existente no código.

---

## Projeto

Image Converter Hub é uma ferramenta 100% client-side para otimização, conversão e download de imagens.

O produto deve priorizar:

- privacidade;
- processamento local;
- alta performance;
- experiência mobile-first;
- visual premium;
- simplicidade de uso.

Não introduzir backend para processamento de imagens.

---

## Stack principal

Seguir a stack definida em `AI_RULES.md`.

Complementos específicos deste projeto:

- Canvas API para processamento local;
- JSZip para downloads em lote;
- Vercel para deploy;
- controle de versão em `src/lib/version.ts`.

---

## Regra principal

Todo processamento de imagem deve acontecer no navegador do usuário.

Não usar:

- backend;
- webhooks;
- n8n;
- APIs externas;
- upload obrigatório para servidor;
- processamento remoto.

Exceção apenas se o usuário pedir explicitamente e aceitar a mudança de arquitetura.

---

## Processamento de imagens

Preservar comportamento esperado:

- PNG e WEBP devem manter transparência quando possível;
- JPG/JPEG não suporta alpha, então deve usar fundo branco;
- evitar perda desnecessária de qualidade;
- otimizar performance para múltiplos arquivos;
- evitar travar a UI em operações pesadas;
- exibir feedback claro durante processamento;
- tratar erros por arquivo.

Preferir APIs nativas do navegador antes de adicionar bibliotecas pesadas.

---

## UI/UX

O app deve parecer uma ferramenta moderna, simples e premium.

Prioridades:

- mobile-first;
- elementos grandes e fáceis de tocar;
- fontes legíveis;
- feedback visual claro;
- loading/progress durante conversão;
- estados de erro e sucesso;
- layout confortável em desktop até aproximadamente `1400px`;
- responsividade com `sm:`, `md:`, `lg:`.

Manter estética:

- glassmorphism quando fizer sentido;
- bordas arredondadas grandes;
- sombras suaves;
- gradientes discretos;
- ícones Lucide.

---

## Estrutura e código

Respeitar o padrão atual do projeto.

Não concentrar código novo em arquivos gigantes se a funcionalidade crescer.

Quando uma função ficar grande, considerar separar em:

- `src/lib/`;
- `src/hooks/`;
- `src/components/`.

Não modificar arquivos base do `shadcn/ui` diretamente.

---

## Performance

Antes de adicionar funcionalidade, considerar:

- peso da biblioteca;
- impacto no bundle;
- processamento em lote;
- consumo de memória;
- responsividade durante conversão;
- compatibilidade mobile.

Evitar bibliotecas pesadas quando Canvas API ou APIs nativas resolverem.

---

## Versionamento

Ao realizar melhoria significativa de layout ou funcionalidade, incrementar:

- `src/lib/version.ts`

Não alterar versão para mudanças triviais, comentários ou ajustes internos sem impacto no produto.

---

## Validação

Antes de concluir tarefa relevante, executar quando aplicável:

```bash
npm run build
```

Se o build falhar:

- corrigir o erro quando estiver dentro do escopo;
- informar claramente se o erro for anterior à tarefa ou depender de decisão externa.

---

## Git e publicação

Repositório oficial:

- `https://github.com/studio4x/image-converter-hub`

Branch:

- `main`

Quando o usuário solicitar publicação:

1. verificar alterações locais:

```bash
git status
```

2. validar build:

```bash
npm run build
```

3. publicar:

```bash
git add .
git commit -m "mensagem clara e descritiva"
git push origin main
```

Regras:

- nunca subir código quebrado;
- nunca commitar `.env`, tokens, senhas ou credenciais;
- manter mensagens de commit claras e objetivas;
- não fazer commit/push sem solicitação explícita do usuário.

---

## Resposta esperada do agente

Para tarefa relevante, informar:

- o que foi alterado;
- arquivos modificados;
- validação executada;
- versão atual, se alterada;
- pendências reais.

Evitar explicações longas em ajustes simples.

---
> Source: [studio4x/image-converter-hub](https://github.com/studio4x/image-converter-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
