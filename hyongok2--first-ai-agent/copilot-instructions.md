## first-ai-agent

> Claude Code를 위한 프로젝트 컨텍스트 및 개발 가이드입니다.

# CLAUDE.md - McpAgent 개발 가이드

Claude Code를 위한 프로젝트 컨텍스트 및 개발 가이드입니다.

## 📋 프로젝트 개요

**McpAgent**는 로컬 LLM(Ollama)과 MCP(Model Context Protocol) 서버를 통합한 지능형 AI 에이전트 시스템입니다.

### 🏗️ 아키텍처 특징
- **Clean Architecture**: 4계층 구조로 관심사 분리
- **Domain-Driven Design (DDD)**: 비즈니스 로직 중심 설계
- **5단계 처리 파이프라인**: 지능적 요청 처리
- **MCP 프로토콜**: 표준화된 도구 통합
- **의존성 주입**: 테스트 용이성과 유연성

### 🔄 핵심 처리 파이프라인

```
입력 정제 → 능력 선택 → 파라미터 생성 → 도구 실행 → 응답 생성
    ↓          ↓          ↓          ↓          ↓
InputRefine  Capability  Parameter  Tool       Response
Service      Selection   Generation Execution  Generation
            Service      Service              Service
```

## 🏛️ 아키텍처 구조

### Clean Architecture 4계층

```
src/McpAgent/
├── Presentation/           # 📱 프레젠테이션 계층
│   ├── Console/           # ConsoleUIService
│   └── Hosting/           # AgentHostService, InteractiveHostService
├── Application/           # 🎯 애플리케이션 계층
│   ├── Agent/            # AgentService (메인 서비스)
│   ├── Commands/         # CommandHandlerService
│   ├── Conversation/     # ConversationService
│   ├── Services/         # 5단계 파이프라인 서비스들
│   └── Interfaces/       # 애플리케이션 인터페이스
├── Domain/               # 🧠 도메인 계층
│   ├── Entities/         # 핵심 엔티티 (AgentRequest, AgentResponse 등)
│   ├── Services/         # AgentOrchestrator (도메인 서비스)
│   └── Interfaces/       # 도메인 인터페이스
└── Infrastructure/       # 🔧 인프라 계층
    ├── LLM/             # OllamaProvider
    ├── MCP/             # MCP 클라이언트 및 도구 실행기
    ├── Storage/         # InMemory 저장소
    ├── Logging/         # 파일 로깅 시스템
    └── Services/        # PromptService
```

### 핵심 컴포넌트

#### 1. AgentOrchestrator (Domain Service)
```csharp
// 위치: Domain/Services/AgentOrchestrator.cs
// 역할: 5단계 파이프라인 오케스트레이션
// 특징: 멀티사이클 처리, 누적 계획 관리
```

#### 2. 5단계 파이프라인 서비스들
```csharp
// Application/Services/
- InputRefinementService     // 사용자 입력 정제
- CapabilitySelectionService // 도구/능력 선택
- ParameterGenerationService // 도구 파라미터 생성
- ResponseGenerationService  // 최종 응답 생성

// Infrastructure/MCP/
- McpToolExecutor           // 도구 실행
```

#### 3. MCP 통합 시스템
```csharp
// Infrastructure/MCP/
- ProperMcpClientAdapter    // MCP 프로토콜 구현
- McpServerConnection       // MCP 서버 연결 관리
- McpToolExecutor          // 도구 실행 및 결과 처리
```

## ⚙️ 빌드 및 실행

### 기본 명령어
```bash
# 프로젝트 루트에서 빌드
dotnet build first-ai-agent.sln

# 에이전트 실행
cd src/McpAgent
dotnet run

# CLI 옵션 사용
dotnet run -- --model qwen3-32b --temperature 0.7

# 패키지 복원
dotnet restore

# 클린 빌드
dotnet clean && dotnet build
```

### 환경 설정
```bash
# 환경 변수 (MCPAGENT_ 프리픽스)
MCPAGENT_Agent__Llm__Model=qwen3-32b
MCPAGENT_Agent__Llm__Temperature=0.7
MCPAGENT_Agent__Mcp__Enabled=true
```

## 📁 주요 설정 파일

### appsettings.json 구조
```json
{
  "Agent": {
    "Llm": {
      "Provider": "ollama",
      "Model": "qwen3-32b",
      "Endpoint": "http://localhost:11434",
      "MaxTokens": 8192,
      "Temperature": 0.7,
      "ContextWindows": { "qwen3:32b": 32768 }
    },
    "Mcp": {
      "Enabled": true,
      "Servers": [
        {
          "Name": "mcp-server-framework",
          "Command": "uv",
          "Args": ["run", "mcp-agent"],
          "Env": { "PYTHONPATH": "C:/src/work/mcp/mcp-server-framework" }
        }
      ]
    },
    "Agent": {
      "MaxHistoryLength": 50,
      "MaxToolChainIterations": 5,
      "EnableToolChaining": true
    }
  }
}
```

### 프롬프트 템플릿 (Resources/Prompts/)
- `input-refinement.txt` - 입력 정제용
- `capability-selection.txt` - 능력 선택용
- `parameter-generation.txt` - 파라미터 생성용
- `response-generation.txt` - 응답 생성용
- `conversation-summary.txt` - 대화 요약용

## 🔍 로깅 시스템

### 로그 파일 구조
```
Logs/
├── system-{date}.log                    # 시스템 로그
└── RequestResponse/{date}/              # 요청/응답 로그
    ├── {timestamp}-{model}-{service}.log
    └── {timestamp}-mcp-server-{tool}.log
```

### 로그 레벨 설정
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "McpAgent": "Debug",
      "Microsoft": "Warning"
    }
  }
}
```

---

# 🧹 클린 코드 개발 가이드

## 📋 코딩 표준

### 1. 네이밍 규칙

#### 클래스 및 인터페이스
```csharp
// ✅ 올바른 예
public class AgentOrchestrator { }
public interface ILlmProvider { }
public class InputRefinementService { }

// ❌ 피해야 할 예
public class agent_orchestrator { }
public class ILLMProvider { }
public class inputRefSvc { }
```

#### 메서드 및 속성
```csharp
// ✅ 올바른 예
public async Task<AgentResponse> ProcessRequestAsync(AgentRequest request)
public string ConversationId { get; set; }
private readonly ILogger<AgentOrchestrator> _logger;

// ❌ 피해야 할 예
public async Task<AgentResponse> process(AgentRequest req)
public string conversationID { get; set; }
private readonly ILogger<AgentOrchestrator> logger;
```

### 2. 클래스 구조 원칙

#### 단일 책임 원칙 (SRP)
```csharp
// ✅ 올바른 예 - 각 서비스가 단일 책임
public class InputRefinementService 
{
    // 오직 입력 정제만 담당
    public async Task<RefinedInput> RefineInputAsync(...)
}

public class CapabilitySelectionService 
{
    // 오직 능력 선택만 담당
    public async Task<SystemCapability> SelectCapabilityAsync(...)
}
```

#### 의존성 역전 원칙 (DIP)
```csharp
// ✅ 올바른 예 - 추상화에 의존
public class AgentOrchestrator
{
    private readonly IInputRefinementService _inputRefinementService;
    private readonly ILlmProvider _llmProvider;
    
    public AgentOrchestrator(
        IInputRefinementService inputRefinementService,
        ILlmProvider llmProvider)
    {
        _inputRefinementService = inputRefinementService;
        _llmProvider = llmProvider;
    }
}
```

### 3. 메서드 설계 원칙

#### 메서드 길이 및 복잡성
```csharp
// ✅ 올바른 예 - 작고 집중된 메서드
public async Task<ToolExecution?> ExecuteToolAsync(
    SystemCapability selectedCapability,
    RefinedInput refinedInput,
    CancellationToken cancellationToken)
{
    var toolName = ExtractToolName(selectedCapability);
    var selectedTool = await GetToolByNameAsync(toolName, cancellationToken);
    
    if (selectedTool == null)
        return await HandleToolNotFoundAsync(toolName, cancellationToken);
    
    return await ExecuteValidToolAsync(selectedTool, refinedInput, cancellationToken);
}

// 각 단계를 별도 메서드로 분리
private string ExtractToolName(SystemCapability capability) { ... }
private async Task<ToolDefinition?> GetToolByNameAsync(string toolName, CancellationToken ct) { ... }
```

#### 매개변수 관리
```csharp
// ✅ 올바른 예 - 매개변수 객체 사용
public class ProcessingContext
{
    public RefinedInput RefinedInput { get; init; }
    public IReadOnlyList<ConversationMessage> ConversationHistory { get; init; }
    public string SystemContext { get; init; }
    public CancellationToken CancellationToken { get; init; }
}

public async Task<SystemCapability> SelectCapabilityAsync(ProcessingContext context)
{
    // 매개변수가 명확하고 관리하기 쉬움
}
```

### 4. 예외 처리 패턴

#### 구조화된 예외 처리
```csharp
// ✅ 올바른 예
public async Task<AgentResponse> ProcessRequestAsync(AgentRequest request, CancellationToken cancellationToken)
{
    try
    {
        _logger.LogInformation("Starting request processing for conversation {ConversationId}", request.ConversationId);
        
        var result = await ExecuteProcessingPipeline(request, cancellationToken);
        
        _logger.LogInformation("Request processing completed for conversation {ConversationId}", request.ConversationId);
        return AgentResponse.Success(result, request.ConversationId);
    }
    catch (OperationCanceledException)
    {
        _logger.LogWarning("Request processing was cancelled for conversation {ConversationId}", request.ConversationId);
        return AgentResponse.Failure("Processing was cancelled", request.ConversationId);
    }
    catch (AgentException ex)
    {
        _logger.LogError(ex, "Agent error occurred for conversation {ConversationId}: {Message}", 
            request.ConversationId, ex.Message);
        return AgentResponse.Failure(ex.Message, request.ConversationId);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Unexpected error occurred for conversation {ConversationId}", request.ConversationId);
        return AgentResponse.Failure("An unexpected error occurred", request.ConversationId);
    }
}
```

### 5. 비동기 패턴

#### ConfigureAwait 사용
```csharp
// ✅ 라이브러리 코드에서
public async Task<string> GenerateResponseAsync(string input)
{
    var result = await _llmProvider.GenerateAsync(input).ConfigureAwait(false);
    return result;
}

// ✅ 애플리케이션 코드에서 (ASP.NET Core가 아닌 콘솔 앱)
public async Task<AgentResponse> ProcessAsync(AgentRequest request)
{
    var response = await _orchestrator.ProcessRequestAsync(request);
    return response;
}
```

#### CancellationToken 전파
```csharp
// ✅ 모든 비동기 메서드에 CancellationToken 전달
public async Task<AgentResponse> ProcessRequestAsync(AgentRequest request, CancellationToken cancellationToken = default)
{
    var refinedInput = await _inputRefinementService.RefineInputAsync(request.Message, cancellationToken);
    var capability = await _capabilitySelectionService.SelectCapabilityAsync(refinedInput, cancellationToken);
    var response = await _responseGenerationService.GenerateResponseAsync(refinedInput, capability, cancellationToken);
    
    return AgentResponse.Success(response, request.ConversationId);
}
```

---

# 🏗️ 아키텍처 패턴 & 베스트 프랙티스

## 1. Clean Architecture 구현

### 계층별 책임

#### Presentation Layer
```csharp
// 책임: UI/UX, 사용자 입력 처리, 응답 표시
// 의존성: Application Layer만 참조

// ✅ 올바른 예
public class ConsoleUIService
{
    private readonly IAgentService _agentService; // Application Layer
    
    public async Task HandleUserInputAsync(string input)
    {
        var response = await _agentService.ProcessAsync(input);
        DisplayResponse(response);
    }
}
```

#### Application Layer
```csharp
// 책임: 유스케이스, 애플리케이션 로직, 오케스트레이션
// 의존성: Domain Layer, Infrastructure Interface

public class AgentService : IAgentService
{
    private readonly AgentOrchestrator _orchestrator; // Domain Service
    private readonly ILlmProvider _llmProvider; // Infrastructure Interface
    
    public async Task<AgentResponse> ProcessAsync(AgentRequest request)
    {
        // 애플리케이션 로직 구현
        return await _orchestrator.ProcessRequestAsync(request);
    }
}
```

#### Domain Layer
```csharp
// 책임: 핵심 비즈니스 로직, 도메인 규칙
// 의존성: 외부 의존성 없음 (순수 도메인)

public class AgentOrchestrator
{
    // 핵심 비즈니스 로직 - 5단계 파이프라인 처리
    public async Task<AgentResponse> ProcessRequestAsync(AgentRequest request)
    {
        // 도메인 로직 구현
        // 1. 입력 정제
        // 2. 능력 선택
        // 3. 파라미터 생성
        // 4. 도구 실행
        // 5. 응답 생성
    }
}
```

#### Infrastructure Layer
```csharp
// 책임: 외부 시스템 통합, 데이터 접근, 기술적 구현
// 의존성: Domain Interface 구현

public class OllamaProvider : ILlmProvider
{
    public async Task<string> GenerateAsync(string prompt, CancellationToken cancellationToken)
    {
        // Ollama API 호출 구현
    }
}
```

## 2. 의존성 주입 패턴

### 서비스 등록 패턴
```csharp
// Program.cs에서 계층별 서비스 등록
services.ConfigureServices((context, services) =>
{
    // Domain Services (핵심 비즈니스 로직)
    services.AddSingleton<AgentOrchestrator>();
    
    // Application Services (유스케이스)
    services.AddSingleton<IAgentService, AgentService>();
    services.AddSingleton<IInputRefinementService, InputRefinementService>();
    services.AddSingleton<ICapabilitySelectionService, CapabilitySelectionService>();
    services.AddSingleton<IParameterGenerationService, ParameterGenerationService>();
    services.AddSingleton<IResponseGenerationService, ResponseGenerationService>();
    
    // Infrastructure Services (외부 시스템)
    services.AddSingleton<ILlmProvider, OllamaProvider>();
    services.AddSingleton<IMcpClientAdapter, ProperMcpClientAdapter>();
    services.AddSingleton<IConversationRepository, InMemoryConversationRepository>();
    
    // Presentation Services (UI)
    services.AddSingleton<ConsoleUIService>();
});
```

### 생성자 주입 패턴
```csharp
// ✅ 올바른 생성자 주입
public class AgentOrchestrator
{
    private readonly ILogger<AgentOrchestrator> _logger;
    private readonly IInputRefinementService _inputRefinementService;
    private readonly ICapabilitySelectionService _capabilitySelectionService;

    public AgentOrchestrator(
        ILogger<AgentOrchestrator> logger,
        IInputRefinementService inputRefinementService,
        ICapabilitySelectionService capabilitySelectionService)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _inputRefinementService = inputRefinementService ?? throw new ArgumentNullException(nameof(inputRefinementService));
        _capabilitySelectionService = capabilitySelectionService ?? throw new ArgumentNullException(nameof(capabilitySelectionService));
    }
}
```

## 3. 설정 관리 패턴

### 강타입 설정
```csharp
// ✅ 올바른 설정 클래스 구조
public class AgentConfiguration
{
    public LlmConfiguration Llm { get; set; } = new();
    public McpConfiguration Mcp { get; set; } = new();
    public AgentSettings Agent { get; set; } = new();
}

public class LlmConfiguration
{
    public string Provider { get; set; } = "ollama";
    public string Model { get; set; } = "llama3";
    public int MaxTokens { get; set; } = 8192;
    public double Temperature { get; set; } = 0.7;
    
    // 검증 메서드
    public void Validate()
    {
        if (string.IsNullOrEmpty(Provider))
            throw new ArgumentException("Provider is required");
        
        if (MaxTokens <= 0)
            throw new ArgumentException("MaxTokens must be positive");
            
        if (Temperature < 0.0 || Temperature > 2.0)
            throw new ArgumentException("Temperature must be between 0.0 and 2.0");
    }
}
```

### 설정 주입 패턴
```csharp
// Program.cs에서 설정 등록
services.Configure<AgentConfiguration>(context.Configuration.GetSection("Agent"));
services.Configure<LlmConfiguration>(context.Configuration.GetSection("Agent:Llm"));

// 서비스에서 설정 사용
public class OllamaProvider : ILlmProvider
{
    private readonly IOptions<LlmConfiguration> _options;
    
    public OllamaProvider(IOptions<LlmConfiguration> options)
    {
        _options = options;
        _options.Value.Validate(); // 설정 검증
    }
}
```

## 4. 로깅 패턴

### 구조화된 로깅
```csharp
// ✅ 구조화된 로깅 사용
public class AgentOrchestrator
{
    private readonly ILogger<AgentOrchestrator> _logger;
    
    public async Task<AgentResponse> ProcessRequestAsync(AgentRequest request, CancellationToken cancellationToken)
    {
        using var scope = _logger.BeginScope(new Dictionary<string, object>
        {
            ["ConversationId"] = request.ConversationId,
            ["RequestId"] = Guid.NewGuid().ToString()
        });
        
        _logger.LogInformation("Starting multi-cycle pipeline for conversation {ConversationId}", 
            request.ConversationId);
        
        try
        {
            var result = await ProcessPipeline(request, cancellationToken);
            
            _logger.LogInformation("Pipeline completed successfully in {CycleCount} cycles for conversation {ConversationId}", 
                result.CycleCount, request.ConversationId);
                
            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Pipeline failed for conversation {ConversationId}", 
                request.ConversationId);
            throw;
        }
    }
}
```

### 성능 로깅
```csharp
// ✅ 성능 모니터링
public async Task<ToolExecution> ExecuteToolAsync(string toolName, Dictionary<string, object> parameters)
{
    using var activity = ActivitySource.StartActivity($"Tool.Execute.{toolName}");
    var stopwatch = Stopwatch.StartNew();
    
    try
    {
        var result = await _toolExecutor.ExecuteAsync(toolName, parameters);
        
        _logger.LogInformation("Tool {ToolName} executed successfully in {ElapsedMs}ms", 
            toolName, stopwatch.ElapsedMilliseconds);
            
        return new ToolExecution
        {
            ToolName = toolName,
            Parameters = parameters,
            Result = result.Result,
            IsSuccess = result.IsSuccess,
            ElapsedTime = stopwatch.Elapsed
        };
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Tool {ToolName} execution failed after {ElapsedMs}ms", 
            toolName, stopwatch.ElapsedMilliseconds);
        throw;
    }
}
```

## 5. 테스트 가능한 설계

### 인터페이스 기반 설계
```csharp
// ✅ 테스트 가능한 인터페이스
public interface IInputRefinementService
{
    Task<RefinedInput> RefineInputAsync(
        string originalInput,
        IReadOnlyList<ConversationMessage> conversationHistory,
        string systemContext,
        CancellationToken cancellationToken = default);
}

// 테스트에서 Mock 사용 가능
[Test]
public async Task ProcessRequestAsync_ShouldRefineInput()
{
    // Arrange
    var mockInputRefinement = new Mock<IInputRefinementService>();
    mockInputRefinement
        .Setup(x => x.RefineInputAsync(It.IsAny<string>(), It.IsAny<IReadOnlyList<ConversationMessage>>(), It.IsAny<string>(), It.IsAny<CancellationToken>()))
        .ReturnsAsync(new RefinedInput("test", "test intent", "test query", ConfidenceLevel.High));
    
    var orchestrator = new AgentOrchestrator(mockLogger, mockInputRefinement.Object, ...);
    
    // Act
    var result = await orchestrator.ProcessRequestAsync(new AgentRequest("test", "conv1"));
    
    // Assert
    Assert.That(result.IsSuccess, Is.True);
    mockInputRefinement.Verify(x => x.RefineInputAsync(It.IsAny<string>(), It.IsAny<IReadOnlyList<ConversationMessage>>(), It.IsAny<string>(), It.IsAny<CancellationToken>()), Times.Once);
}
```

## 6. 확장성 패턴

### 전략 패턴으로 LLM 프로바이더 확장
```csharp
// ✅ 새로운 프로바이더 추가 용이
public interface ILlmProvider
{
    Task<string> GenerateAsync(string prompt, CancellationToken cancellationToken);
}

public class OllamaProvider : ILlmProvider { ... }
public class OpenAIProvider : ILlmProvider { ... }
public class AnthropicProvider : ILlmProvider { ... }

// 팩토리 패턴으로 프로바이더 선택
public class LlmProviderFactory
{
    public static ILlmProvider Create(string providerType, IConfiguration config)
    {
        return providerType.ToLower() switch
        {
            "ollama" => new OllamaProvider(config),
            "openai" => new OpenAIProvider(config),
            "anthropic" => new AnthropicProvider(config),
            _ => throw new NotSupportedException($"Provider {providerType} is not supported")
        };
    }
}
```

이 가이드를 통해 McpAgent 프로젝트의 코드 품질과 아키텍처 일관성을 유지할 수 있습니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hyongok2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hyongok2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
