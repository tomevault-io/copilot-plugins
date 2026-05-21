## testing-events-go

> Convenções e práticas para testes automatizados em Go

# Ao trabalhar com testes automatizados em Go

## Estrutura e Organização
- Os arquivos de teste devem estar no mesmo pacote que o código testado, com o sufixo `_test.go`
- Para testes de integração ou testes que precisam de isolamento, use o sufixo `_test` no nome do pacote
- Espelhe a estrutura de diretórios do código fonte nos testes
- Não implemente testes para interfaces, apenas para implementações concretas

## Nomenclatura
- Arquivos de teste devem seguir o padrão `nome_do_arquivo_test.go` (ex: `user_service_test.go`)
- Funções de teste devem começar com `Test` seguido do nome da função ou método testado em PascalCase
- Use nomes descritivos que indiquem o que está sendo testado (ex: `TestUserService_Create_ValidUser`)
- Para subtestes, use `t.Run()` com nomes descritivos em formato de frase (ex: `t.Run("should return error when email is invalid", func(t *testing.T) {...})`)

## Execução
- Sempre execute os testes a partir do diretório raiz do projeto
- Sempre execute os testes a partir do container com o docker compose exec
- Use o comando `go test ./...` para executar todos os testes do projeto
- Para testes específicos, use `go test ./caminho/para/pacote`
- Para testes com cobertura, use `go test ./... -coverprofile=coverage.out`

## Práticas Recomendadas
- Use tabelas de testes (`table-driven tests`) para testar múltiplos cenários
- Utilize subtestes com `t.Run()` para organizar melhor os testes
- Use `testify` ou pacotes similares para asserções mais expressivas
- Implemente testes de benchmark quando relevante com o prefixo `Benchmark`
- Crie exemplos executáveis com o prefixo `Example` para documentação

## Execução com Docker
- Use o Docker para garantir ambiente consistente: `docker compose exec app go test ./...`
- Para testes com cobertura: `docker compose exec app go test ./... -coverprofile=coverage.out`
- Para visualizar a cobertura: `docker compose exec app go tool cover -html=coverage.out -o coverage.html`

## Mocks e Stubs
- Use interfaces para facilitar a criação de mocks
- Prefira ferramentas como `gomock` ou `testify/mock` para geração de mocks
- Mantenha os mocks em um subdiretório `mocks/` dentro do pacote testado

## Testes de Integração
- Mantenha testes de integração separados dos testes unitários
- Use tags de compilação para separar testes de integração: `// +build integration`
- Execute testes de integração explicitamente: `go test ./... -tags=integration`

---
> Source: [devfullcycle/golangtechweek](https://github.com/devfullcycle/golangtechweek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
