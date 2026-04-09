## ceudata-mcp

> Servidor MCP em .NET 10 para interação com Microsoft SQL Server.

# Sharp MSSQL MCP Server - Guia de Desenvolvimento

Servidor MCP em .NET 10 para interação com Microsoft SQL Server.

## Comandos Rápidos

### Build e Execução
- Build: `dotnet build`
- Executar: `dotnet run --project src/SharpMssqlMcp`
- Testes: `dotnet test`
- Publish: `dotnet publish -c Release -o ./publish`

### Infraestrutura de Teste (Docker)
- Subir SQL Server: `docker-compose up -d`
- Setup do Banco: `Get-Content src/SharpMssqlMcp.Tests/setup-test-db.sql | docker exec -i mssql-mcp-test /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "SharpMssqlMcp_123!" -C`

## Estrutura do Projeto

- `src/SharpMssqlMcp`: Projeto principal (Console App)
  - `Models/`: DTOs para JSON-RPC e Resultados SQL.
  - `Services/`: Lógica central (Executor de query/procedure, Retry Policy, Substitutor de ENV).
  - `Tools/`: Handlers das ferramentas MCP (`execute_query`, `execute_procedure`, `get_datasources`).
  - `Program.cs`: Loop principal stdio e roteamento de mensagens.
- `src/SharpMssqlMcp.Tests`: Testes de Unidade e Integração.

## Convenções de Código

- **Estilo**: Use C# 13 (Latest). Nomes de classes em PascalCase, métodos em PascalCase, campos privados com `_prefix`.
- **SQL**: Sempre use queries parametrizadas. Use `SqlCommandBuilder.DeriveParameters` para stored procedures.
- **MCP**: Logs devem obrigatoriamente ir para `Console.Error` para não corromper o stream de saída (stdout).
- **JSON**: Use `System.Text.Json` para serialização.

## Adicionando Novas Ferramentas

1. Crie uma classe em `Tools/` implementando `IToolHandler`.
2. Registre a ferramenta no método `RegisterTools` em `Program.cs`.
3. Adicione testes de integração correspondentes em `src/SharpMssqlMcp.Tests`.

## Configuração de Segurança

- Variáveis de ambiente são suportadas no formato `${VAR_NAME}` dentro das connection strings no `appsettings.json`.
- O servidor valida limites de query (1MB), parâmetros (1000) e timeout (3600s).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/macintoxic)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/macintoxic)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
