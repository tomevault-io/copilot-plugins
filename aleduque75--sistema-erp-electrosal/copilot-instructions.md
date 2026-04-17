## sistema-erp-electrosal

> 1 # Visão Geral do Projeto e Base de Conhecimento

1 # Visão Geral do Projeto e Base de Conhecimento
2
3 Bem-vindo ao guia do sistema Electrosal! Este documento serve como um ponto de partida rápido para entender o projeto e navegar pela sua documentação mais detalhada.
4
5 ---
6
7 ## 🚀 Status Atual do Projeto (via Gemini CLI)
8
9 Para uma visão atualizada do que está sendo trabalhado, o que já foi feito e o que está por vir, consulte as seções específicas na [Base de
Conhecimento](#base-de-conhecimento-do-projeto).
10
11 ---
12
13 ## 🛠️ Contexto de Desenvolvimento e Operação (via Gemini CLI)
14
15 Para informações sobre como configurar o ambiente, acessar serviços ou resolver problemas comuns, consulte os documentos detalhados na [Base de
Conhecimento](#base-de-conhecimento-do-projeto).
16
17 ---
18
19 # Base de Conhecimento do Projeto
20
21 Esta seção contém links para a documentação detalhada do projeto, organizada por temas. Você pode acessar e, com a devida permissão e funcionalidades do frontend, editar
esses documentos diretamente.
22
23 ---
24
25 ## 📚 Temas Gerais e Decisões de Design
26
27 _ [**Temas Criados**](docs/01_temas_criados.md): Visão geral de decisões de arquitetura, padrões e convenções de projeto.
28 _ [**Funcionalidades Finalizadas**](docs/02_funcionalidades_finalizadas.md): Descrição das funcionalidades já implementadas e em produção.
29 _ [**Funcionalidades em Andamento**](docs/03_funcionalidades_em_andamento.md): Detalhes sobre o que está ativamente em desenvolvimento.
30 _ [**Funcionalidades a Fazer**](docs/04_funcionalidades_a_fazer.md): O roadmap futuro e o backlog de funcionalidades planejadas.
31
32 ---
33
34 ## 💡 Tutoriais e Dicas de Desenvolvimento
35
36 _ [**Acesso ao Banco de Dados (psql)**](docs/tutoriais/acesso_banco_de_dados.md): Guia rápido para acessar e interagir com o banco de dados de desenvolvimento via `psql
37
38 ---
39
40 ## 🔗 Guias de Integração
41
42 _ [**Guia de Integração e Depuração da Evolution API**](docs/integracoes/evolution_api_guide.md): Documentação completa sobre a configuração, uso e depuração da
integração com a Evolution API.
43
44 ---
45
46 **Observação para o Gemini CLI:**
47
48 Quando precisar de informações sobre o projeto, utilize os documentos na pasta `docs/`. Você pode listar os arquivos em `docs/` e seus subdiretórios com:
49 `curl http://localhost:3002/api/knowledge-base`
50 E ler o conteúdo de um arquivo específico com:
51 `curl http://localhost:3002/api/knowledge-base/nome_do_arquivo.md`
52
53 Para atualizar um arquivo:
54 `curl -X PUT -H "Content-Type: application/json" -d '{"content": "Novo conteúdo aqui"}' http://localhost:3002/api/knowledge-base/nome_do_arquivo.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aleduque75) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
