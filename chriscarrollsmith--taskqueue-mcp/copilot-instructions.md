## errors

> FS[FileSystemService: e.g., FileReadError] --> TM[TaskManager: Throws App Errors, e.g., ProjectNotFound, TaskNotDone]

# Error Flow

```mermaid
graph TD
    subgraph Core_Logic
        FS[FileSystemService: e.g., FileReadError] --> TM[TaskManager: Throws App Errors, e.g., ProjectNotFound, TaskNotDone]
        TM -->|App Error with code ERR_xxxx| CLI_Handler["cli.ts Command Handler"]
        TM -->|App Error with code ERR_xxxx| ToolExec["toolExecutors.ts: execute"]
    end

    subgraph CLI_Path
        CLI_Handler -->|App Error| CLI_Catch["cli.ts catch block"]
        CLI_Catch -->|Error Object| FormatCLI["client errors.ts formatCliError"]
        FormatCLI -->|"Error [ERR_xxxx]: message"| ConsoleOut["console.error Output"]
    end

    subgraph MCP_Server_Path
        subgraph Validation_Layer
            ToolExecVal["toolExecutors.ts Validation"] -->|App Error, e.g., MissingParameter| ExecToolErrHandler
        end
        
        subgraph App_Execution
            ToolExec -->|App Error with code ERR_xxxx| ExecToolErrHandler["tools.ts executeToolAndHandleErrors catch block"]
            ExecToolErrHandler -->|Map AppError to Protocol Error or Tool Result| ErrorMapping
            ErrorMapping -->|"If validation error (ERR_1xxx)"| McpError["Create McpError with appropriate ErrorCode"]
            ErrorMapping -->|"If business logic error (ERR_2xxx+)"| FormatResult["Format as isError true result"]

            McpError -->|Throw| SDKHandler["server index.ts SDK Handler"]
            FormatResult -->|"{ content: [{ text: Error [ERR_xxxx]: message }], isError: true }"| SDKHandler
        end

        SDKHandler -- Protocol Error --> SDKFormatError["SDK Formats as JSON-RPC Error Response"]
        SDKHandler -- Tool Result --> SDKFormatResult["SDK Formats as JSON-RPC Success Response"]

        SDKFormatError -->|"{ error: { code: -326xx, message: ... } }"| MCPClient["MCP Client"]
        SDKFormatResult -->|"{ result: { content: [...], isError: true } }"| MCPClient
    end
```

**Explanation of Updated Error Flow and Transformations:**

Errors are consistently through a unified `AppError` system:

1. **Validation Errors** (`ERR_1xxx` series)
   - Used for validation issues (e.g., MissingParameter, InvalidArgument)
   - Thrown by tool executors during parameter validation
   - Mapped to protocol-level McpErrors in `executeToolAndHandleErrors`

2. **Business Logic Errors** (`ERR_2xxx` and higher)
   - Used for all business logic and application-specific errors
   - Include specific error codes
   - Returned as serialized CallToolResults with `isError: true`

---
> Source: [chriscarrollsmith/taskqueue-mcp](https://github.com/chriscarrollsmith/taskqueue-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
