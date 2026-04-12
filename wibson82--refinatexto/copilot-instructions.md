## refinatexto

> Este arquivo documenta decisões arquiteturais, padrões e preferências específicas do projeto RefinaTexto para facilitar a continuidade do desenvolvimento.

# CLAUDE.md - Instruções para Claude no Projeto RefinaTexto

## Propósito

Este arquivo documenta decisões arquiteturais, padrões e preferências específicas do projeto RefinaTexto para facilitar a continuidade do desenvolvimento.

## Arquitetura Geral

### Visão Geral
RefinaTexto é um serviço macOS SwiftUI que se integra com o menu de contexto (Services) para refinamento/reescrita de texto via Ollama local.

**Camadas**:
1. **Service Layer** (AppDelegate + ServiceProvider): Interface com macOS Services
2. **UI Layer** (RefinaTextoDialogView): SwiftUI components
3. **ViewModel** (RefinaTextoViewModel): Lógica de estado e orquestração
4. **LLM Layer** (OllamaClient): Chamadas HTTP ao Ollama
5. **Config Layer** (AppConfig): Persistência de configurações

### Fluxo de Dados

```
TextEdit/Safari (seleção)
    ↓
ServiceProvider.refinaTextoService()
    ↓
ServiceUIRouter (abre janela NSWindow)
    ↓
RefinaTextoDialogView (SwiftUI)
    ↓
RefinaTextoViewModel (generate async)
    ↓
OllamaClient.chat()
    ↓
PromptTemplates.buildMessages()
    ↓
Ollama HTTP API (localhost:11434)
    ↓
Resultado → Pasteboard → App original
```

## Convenções de Código

### Nomenclatura
- **Enums**: PascalCase (ex: `RefinaTextoMode`, `OllamaError`)
- **Structs/Classes**: PascalCase (ex: `RefinaTextoViewModel`, `OllamaClient`)
- **Functions/Variables**: camelCase (ex: `buildMessages()`, `inputText`)
- **File Names**: Correspondem ao tipo principal (ex: `RefinaTextoViewModel.swift`)
- **Prefixos**: Evitar; usar namespacing via struct/class quando necessário

### Organização de Pastas
```
RefinaTexto/
├── Config/           # AppConfig.swift
├── LLM/             # OllamaClient.swift, PromptTemplates.swift
├── Models/          # RefinaTextoMode.swift, data models
├── Service/         # ServiceProvider.swift
├── UI/              # RefinaTextoDialogView.swift, Components
├── ViewModel/       # RefinaTextoViewModel.swift
├── AppDelegate.swift
├── RefinaTextoApp.swift
└── Info.plist
```

### Swift Patterns

1. **Async/Await**: Preferência para operações assíncronas (não closures ou combine)
2. **@MainActor**: ViewModels devem usar `@MainActor class` para garantir atualizações na main thread
3. **Actor**: OllamaClient usa `actor` para thread-safety
4. **@StateObject/@ObservedObject**: Usar corretamente em SwiftUI views
5. **Error Handling**: Use `enum` para erros especializados (ex: `OllamaError`)

### Exemplo de Pattern Async
```swift
@MainActor
func generate() async {
    isLoading = true
    do {
        let result = try await ollamaClient.chat(...)
        self.outputText = result
    } catch let error as OllamaError {
        self.errorMessage = error.errorDescription
    }
    isLoading = false
}
```

## Decisões Arquiteturais

### 1. AppConfig com UserDefaults
- **Decisão**: Usar `UserDefaults` para persistência simples
- **Razão**: Sem necessidade de dados complexos; UserDefaults é suficiente e nativo
- **Alternativas Rejeitadas**: CoreData (overkill), Keychain (desnecessário)

### 2. OllamaClient como Actor
- **Decisão**: Usar `actor` para thread-safety
- **Razão**: Múltiplas requisições podem ocorrer; actor garante acesso serializado
- **Timeout**: 300s (5 min) para acomodar modelos com reasoning complexo

### 3. Service Handler Síncrono
- **Decisão**: `refinaTextoService()` é síncrono; abre janela async
- **Razão**: NSServices usa callbacks síncronos; a janela é aberta em DispatchQueue.main.async
- **Alternativa**: Poderia ser feito via NSWindowController, mas SwiftUI + NSHostingController é mais simples

### 4. PromptTemplates como Struct com Static Methods
- **Decisão**: Não instanciar; usar `PromptTemplates.buildMessages()`
- **Razão**: Não há estado; é puramente uma coleção de funções utilitárias
- **Benefit**: Fácil testar, refatorar, estender

### 5. Info.plist Manual vs. Auto-generation
- **Decisão**: Desabilitar auto-generation; usar `Info.plist` manual
- **Razão**: NSServices requer configuração XML que auto-generation não suporta bem
- **Ficheiro**: `GENERATE_INFOPLIST_FILE = NO` em build settings

## Configurações e Defaults

### Endpoints
```
Padrão:  http://localhost:11434
Path:    /api/chat
Model:   llm2.5-thinking (customizável)
```

### UserDefaults Keys
```
RefinaTexto.OllamaEndpoint  → URL completa (sem path)
RefinaTexto.OllamaModel     → Nome do modelo
```

### Timeouts
```
URLRequest:     300 segundos (5 min)
URLSession:     600 segundos (10 min)
```

## Modo/Template Mapping

### Grammar
- **System Prompt**: Corrigir sem alterar sentido
- **Opções Adicionais**: Nenhuma
- **Esperado**: Resultado rápido

### Tone/Style
- **System Prompt**: Reescrever mantendo conteúdo com tom específico
- **Opções**: Picker com [Formal, Amigável, Direto, Técnico, Criativo]
- **Valor Default**: Formal

### Prompt
- **System Prompt**: Converter em prompt estruturado com seções (Identidade, Contexto, Objetivo, Restrições, Saída)
- **Opções Adicionais**: Nenhuma
- **Esperado**: Markdown formatado

### PRIMO
- **System Prompt**: Estrutura em 8 seções (Identidade, Contexto, Objetivo, Regras, Entradas, Passos, Saídas, Critérios)
- **Opções Adicionais**: Nenhuma
- **Esperado**: Markdown formatado

## Erros e Tratamento

### OllamaError Cases
```swift
case invalidURL              // URL do Ollama mal formatada
case connectionRefused       // localhost:11434 não respondendo
case timeout                 // Requisição expirou
case invalidResponse         // Resposta não é JSON válido
case modelNotFound           // Modelo não existe no Ollama
case decodingError(String)   // JSON parsing falhou
case networkError(String)    // Outro erro de rede
case unknown(String)         // Qualquer outro erro
```

Cada erro inclui:
- `errorDescription`: Mensagem amigável para UI
- `recoverySuggestion`: Instruções de troubleshooting

## UI/UX Guidelines

### Estados da View
- **Idle**: Texto carregado, botões ativos
- **Loading**: Botão Gerar desabilitado, ProgressView visível
- **Success**: Resultado preenchido, botão Copiar/Aplicar ativados
- **Error**: Banner vermelho com mensagem + sugestão

### Tamanho de Janela
```
Min Width:  700px
Min Height: 600px
Aspect Ratio: Livre (resizável)
```

### Comportamentos
- **Cancelar**: Fecha sem aplicar (Cmd+W)
- **Aplicar**: Substitui seleção e fecha (Enter/Cmd+Return)
- **Copiar**: Copia sem fechar
- **Tab Navigation**: Funciona entre campos

## Security & Privacy

### Dados do Usuário
- ✅ Não persistir texto automaticamente
- ✅ Não enviar dados fora de localhost
- ✅ Sem logging do conteúdo em Release
- ✅ Sem telemetria
- ✅ Sem analytics

### Sandbox
- Status: Habilitado com exceções para `localhost`
- Network: Client apenas (sem server)
- Files: Padrão do usuário (sem necessidade especial)

## Testing

### Unit Tests (Planejado v1.1)
```
RefinaTextoTests/
├── PromptTemplatesTests.swift
├── OllamaClientTests.swift (com mocks)
└── AppConfigTests.swift
```

### Manual Tests
- [x] TextEdit: Texto selecionado → Refinar → Resultado
- [x] Modo Gramática
- [x] Modo Tone (com picker)
- [x] Modo Prompt
- [x] Modo PRIMO
- [x] Ollama offline → Erro claro
- [x] Modelo inexistente → Erro claro
- [ ] Safari (nota: nem todos apps suportam Services)
- [ ] VS Code (nota: nem todos apps suportam Services)

## Build & Deployment

### Build Settings Importantes
```
GENERATE_INFOPLIST_FILE = NO        # Usa Info.plist manual
INFOPLIST_FILE = RefinaTexto/Info.plist
MACOSX_DEPLOYMENT_TARGET = 12.0
SWIFT_VERSION = 5.0
CODE_SIGN_STYLE = Automatic
```

### Versioning
- Formato: MAJOR.MINOR (ex: 1.0, 1.1, 2.0)
- CFBundleVersion: Number only (ex: 1, 2, 3)
- CFBundleShortVersionString: MAJOR.MINOR (ex: 1.0)

### App Store
- Status: ❌ Não publicado (planejado futuro)
- Sandbox: Requerido para App Store (já habilitado)
- Notarização: Necessária para distribuição outside App Store

## Dependências

### Externas
- Nenhuma! Zero dependências externas (Swift + AppKit padrão)

### Internas
```
RefinaTextoApp
    ↓ AppDelegate
        ↓ ServiceProvider
            ↓ RefinaTextoDialogView
                ↓ RefinaTextoViewModel
                    ↓ OllamaClient
                    ↓ PromptTemplates
                ↓ AppConfig
```

## Próximas Features (v1.1+)

### Curto Prazo (v1.1)
- [ ] Settings UI para endpoint/modelo
- [ ] Streaming de tokens (arquitetura pronta)
- [ ] Histórico local (seguro, sem cloud)
- [ ] Atalhos customizáveis

### Médio Prazo (v1.2+)
- [ ] Suporte para múltiplos modelos em UI
- [ ] Templates customizáveis por usuário
- [ ] Integração com Spotlight
- [ ] Preferências persistidas por modo

### Longo Prazo (v2.0+)
- [ ] Publicação App Store
- [ ] Suporte para outras LLM APIs (OpenAI, Anthropic)
- [ ] Sync entre máquinas via iCloud
- [ ] Estatísticas e telemetria (opt-in)

## Troubleshooting de Desenvolvimento

### Build Falha com "Info.plist not found"
→ Confirmar `INFOPLIST_FILE = RefinaTexto/Info.plist` em build settings

### Service não aparece no menu
→ Rodar `/System/Library/CoreServices/pbs -update` no terminal

### Ollama timeout/connection refused
→ Verificar se `ollama serve` está rodando: `curl http://localhost:11434/api/tags`

### UI congelada durante geração
→ Confirmar que `generate()` é `async` e usa `@MainActor`

### JSON decode error
→ Verificar resposta do Ollama com curl:
```bash
curl http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{"model":"llm2.5-thinking","messages":[{"role":"user","content":"test"}],"stream":false}'
```

## Recursos Úteis

- [Apple macOS Services Documentation](https://developer.apple.com/documentation/foundation/nsserviceprovider)
- [Swift Concurrency](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency)
- [Ollama API Docs](https://github.com/jmorganca/ollama/blob/main/docs/api.md)
- [SwiftUI Best Practices](https://developer.apple.com/documentation/swiftui)

---

**Última Atualização**: 13/02/2026
**Mantido por**: Wibson Pires Teixeira + Claude

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Wibson82)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Wibson82)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
