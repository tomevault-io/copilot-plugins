## granaevo

> - NUNCA expor service_role key em qualquer arquivo JS enviado ao browser ou variável pública

# CLAUDE.md — Security Rules
# Stack: Supabase · Vercel · Cloudflare · Vite · Vanilla JS

## REGRAS ABSOLUTAS
- NUNCA expor service_role key em qualquer arquivo JS enviado ao browser ou variável pública
- NUNCA validar permissões, autenticação ou dados sensíveis no frontend
- NUNCA criar tabela pública sem RLS habilitado + políticas explícitas
- NUNCA criar view sem: WITH (security_invoker = true)
- NUNCA commitar .env, secrets ou tokens no GitHub
- NUNCA usar supabase.auth.admin no client-side
- NUNCA confiar em dados vindos do cliente — sempre revalidar no servidor
- NUNCA usar SQL dinâmico sem parametrização ($1, $2...)
- NUNCA criar função SECURITY DEFINER sem revisar escalada de privilégios
- NUNCA desabilitar rate limit em endpoints de auth ou dados sensíveis
- NUNCA aceitar uploads sem validar tipo MIME + tamanho no servidor

## AO INICIAR QUALQUER TAREFA
1. Verificar se a tarefa toca em autenticação, autorização ou dados do usuário
2. Identificar quais tabelas do Supabase são afetadas
3. Confirmar que RLS está habilitado + políticas corretas nessas tabelas
4. Confirmar que toda validação crítica está no servidor (api/)

## AO CRIAR QUALQUER API ROUTE OU EDGE FUNCTION (api/)
- Autenticar antes de qualquer lógica (getUser server-side, nunca client-side)
- Autorizar: verificar se o usuário autenticado tem acesso ao recurso específico
- Validar input antes de qualquer uso (nunca confiar em dados do body/query)
- Rate limit aplicado
- Erros não expõem stack trace ou detalhes internos
- Queries usam parâmetros ($1, $2), nunca string interpolation

## AO CRIAR MIGRATION OU ALTERAR BANCO
- Habilitar RLS na tabela nova
- Criar políticas para SELECT / INSERT / UPDATE (com WITH CHECK) / DELETE
- Views com security_invoker = true
- Rodar /god-eyes após qualquer migration

## PADRÃO DE POLÍTICA RLS OBRIGATÓRIO
-- SELECT
CREATE POLICY "select_own" ON tabela
  FOR SELECT USING (auth.uid() = user_id);
-- INSERT
CREATE POLICY "insert_own" ON tabela
  FOR INSERT WITH CHECK (auth.uid() = user_id);
-- UPDATE (WITH CHECK obrigatório — sem ele o usuário pode alterar user_id)
CREATE POLICY "update_own" ON tabela
  FOR UPDATE USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);
-- DELETE
CREATE POLICY "delete_own" ON tabela
  FOR DELETE USING (auth.uid() = user_id);

## ARMADILHAS CRÍTICAS DO SUPABASE
- Deletar usuário NÃO invalida JWT → revogar sessions explicitamente
- Views ignoram RLS por padrão → usar security_invoker = true em toda view
- UPDATE sem WITH CHECK permite alterar user_id → sempre incluir WITH CHECK
- Realtime pode bypassar RLS se não configurado → verificar publicação por tabela
- anon role tem acesso a tabelas sem política → bloquear explicitamente
- Funções SECURITY DEFINER ignoram RLS do chamador → auditar todas
- Variáveis de ambiente incluídas no bundle JS do Vite são públicas → nunca colocar service_role

## COMANDO /god-eyes
Ver: .claude/commands/god-eyes.md

---
> Source: [GranaDev/granaevo](https://github.com/GranaDev/granaevo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
