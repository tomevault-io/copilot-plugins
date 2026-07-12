## industrialdaq

> This is **IndustrialDAQ**, a professional-grade industrial data acquisition and monitoring software for factory floors.

# IndustrialDAQ Project Instructions for Claude

## 1. Project Overview

This is **IndustrialDAQ**, a professional-grade industrial data acquisition and monitoring software for factory floors.

- **Target Platform**: Windows Industrial PCs
- **Framework**: .NET 8.0
- **Architecture**: Plugin-based, Multi-layered (UI, Application, Domain, Infrastructure, Cross-cutting).

## 2. Technology Stack & Library Preferences

When writing or suggesting code, STRICTLY use the following libraries:

- **UI Framework**: WPF + Prism 9 (MVVM pattern, Region-based navigation).
- **Protocols**: 
  - Modbus: `NSModbus4` v4.0.2 (namespace `Modbus.Device`, factory: `ModbusIpMaster.CreateIp(TcpClient)`) — NOT NModbus4, which is .NET Framework only.
  - OPC UA: `OPCFoundation.NetStandard.Opc.Ua` v1.5.374 (namespace `Opc.Ua`, `Opc.Ua.Client`).
  - MQTT: `MQTTnet 4.x`
  - Siemens S7: `S7netplus` v0.20.0 (namespace `S7.Net` — CAUTION: avoid RootNamespace collision, see §7).
- **Database (Historical)**: Entity Framework Core (SQLite for standalone, PostgreSQL for enterprise).
- **Database (Real-time)**: In-memory `ConcurrentDictionary<string, TagValue>`.
- **Message Bus**: `System.Threading.Channels` (Channel<T>) for high-performance backpressure queues. Use `MediatR` for decoupled module communication.
- **Logging**: `Serilog` (Structured logging).
- **DI Container**: `Microsoft.Extensions.DependencyInjection`.
- **Testing**: `xUnit`, `Moq`, `NBomber`.

## 3. Directory Structure

Adhere to this strict layer separation. Dependencies must only flow downwards:

```text
IndustrialDAQ.slnx
src/
  IndustrialDAQ.Core/           # Domain models, Interfaces, DTOs (NO external dependencies)
  IndustrialDAQ.Infrastructure/ # EF Core Context, Serilog config, Cryptography
  IndustrialDAQ.Acquisition/    # DeviceCollector, PeriodicTimer scheduling
  IndustrialDAQ.Processing/     # DynamicExpresso rules, Data aggregation
  IndustrialDAQ.Storage/        # HistoryWriter, Cache fallback (cache.db)
  IndustrialDAQ.Alarm/          # Alarm logic, Webhook/Email notifications
  IndustrialDAQ.UI/             # WPF App, Prism Shell, Views/ViewModels
sandbox/
  IndustrialDAQ.ConsoleTest/    # Console test harness
Plugins/
  Drivers.Modbus/               # Modbus TCP driver (NSModbus4)
  Drivers.OpcUA/                # OPC UA client driver
  Drivers.S7/                   # Siemens S7 PLC driver (RootNamespace: IndustrialDAQ.Drivers.S7)
```

## 4. Architectural Rules & Constraints

- **Message Bus & Queues**: ALWAYS use `Channel<T>` for producer-consumer scenarios (e.g., Acquisition -> Processing -> Storage). NEVER use `BlockingCollection`.
- **Acquisition Scheduling**: Use `PeriodicTimer` running in dedicated `Task` for device polling loops.
- **Hot Reloading**: Configuration changes (adding devices, changing cycle times) must be applied dynamically using `CancellationToken` to gracefully cancel existing tasks and spin up new ones without application restart.
- **Resilience**: Implement exponential backoff with ±20% jitter for network reconnections.
- **Quality Codes**: Use standard OPC quality codes (GOOD: 0xC0, UNCERTAIN: 0x40, BAD: 0x00, SUBSTITUTE: 0x80).

## 5. Coding Style & Conventions

- **Naming**: Classes/Structs/Interfaces/Methods/Properties: `PascalCase`. Private fields: `_camelCase`. Async methods MUST end with `Async`.
- **Async Programming**: ALL I/O operations MUST use `async/await`. NEVER `.Result` or `.Wait()`. `CancellationToken ct` MUST be passed down the entire async call chain.
- **Exception Handling**: Protocol drivers catch low-level exceptions and mark Quality.Bad for reads, throw `InvalidOperationException` for writes. NEVER empty `catch {}`.
- **Resource Management**: Always use `using` / `await using` for unmanaged resources.
- **Comments**: All comments in Chinese. File header, class summary, and public method `<summary>` required.

## 6. Commands (Windows Environment)

- **Build Solution**: `dotnet build IndustrialDAQ.slnx`
- **Build Individual Driver**: `dotnet build Plugins/Drivers.XXX/Drivers.XXX.csproj`
- **Run Application**: `dotnet run --project src/IndustrialDAQ.UI/IndustrialDAQ.UI.csproj`
- **Run Tests**: `dotnet test`
- **Publish Single File**: `dotnet publish src/IndustrialDAQ.UI/IndustrialDAQ.UI.csproj -r win-x64 --self-contained -p:PublishSingleFile=true`

## 7. Known Pitfalls & Lessons Learned

### S7 Namespace Collision
The `S7netplus` library uses namespace `S7.Net`. If the project's RootNamespace is `Drivers.S7`, C# resolves `S7.Net` as `Drivers.S7.Net` and fails. **Fix**: Set RootNamespace to `IndustrialDAQ.Drivers.S7` in the .csproj.

### Modbus Library Compatibility
- `NModbus4` v2.1.0 targets .NET Framework only — incompatible with net8.0.
- `NModbus` v4.0.0-alpha010 has a completely different API (no `IModbusMaster`, `ModbusFactory`).
- `NSModbus4` v4.0.2 is the CORRECT choice: .NET Standard 2.0 fork of NModbus4. Namespace is `Modbus` (not `NModbus4`). Factory is `ModbusIpMaster.CreateIp(TcpClient)`. Methods on `IModbusMaster` do NOT accept `CancellationToken`.

### OPC UA DataValue.Value Type
In the OPC Foundation SDK version used, `DataValue.Value` is typed as `object` (not `Variant`). The `ConvertOpcValue` helper must accept `object`, not `Variant`.

### Prism Window AutoWireViewModel
`AutoWireViewModel` does NOT work for `Window` classes. Shell ViewModel must be registered as singleton and manually resolved in code-behind constructor: `DataContext = app.Container.Resolve<MainWindowViewModel>()`.

## 8. Current Progress (2026-05-05)

### Completed
- **Core layer**: Domain models (TagValue, TagPoint with TagAccess, DeviceConfig, Quality enum, AlarmRecord, AlarmRule, CalculationRule, AlarmSeverity/Status enums), IProtocolDriver interface, RealTimeStore with Channel<T>.
- **Infrastructure layer**: EF Core DbContext with SQLite, Serilog configuration, DeviceConfigurationLoader (multi-protocol JSON config with opcUaAddress/modbusAddress/s7Address).
- **Acquisition layer**: AcquisitionHost (IHostedService managing DeviceCollector lifecycle with ConcurrentDictionary), DeviceCollector (PeriodicTimer polling with exponential backoff + jitter), DriverFactory, AcquisitionChannel (bounded 50000), GetDriver() public API for write operations.
- **Processing layer**: DataProcessor (IHostedService consuming ChangeStream, DataTable.Compute expression evaluation, reverse-indexed tag-to-rule lookup).
- **Storage layer**: HistoryWriter (IHostedService, batch write to SQLite via IDbContextFactory<DaqDbContext>).
- **Alarm layer**: AlarmEngine (IHostedService, threshold evaluation with hysteresis anti-flap + cooldown anti-storm), AlarmChannel (unbounded Channel<AlarmRecord>).
- **UI layer**: (same as 2026-05-04)
- **Protocol Drivers (all compile clean, TagAccess-aware)**:
  - `Drivers.Modbus/ModbusTcpDriver.cs` — ReadTagsAsync filters out Write-only tags; WriteTagAsync rejects Read-only tags.
  - `Drivers.OpcUA/OpcUaDriver.cs` — same access control.
  - `Drivers.S7/S7Driver.cs` — same access control.
  - `Mocks/MockProtocolDriver.cs` — same access control.
- **ConsoleTest**: Full interactive test harness — loads JSON config, registers all 4 drivers, runs full pipeline, interactive write console (`write`, `status`, `help`, `exit`), real-time data observer, alarm observer.
- **Config**: `config/production-line.json` — S7-1500 at 192.168.3.10 with 12 tags (6 write commands, 6 read statuses), multi-protocol addresses per tag.

### Next Steps (suggested order)
1. Add MQTT driver (MQTTnet).
2. Write integration tests.
3. Real device testing with S7-1500 OPC UA Server.
4. Wire up UI ViewModels to real data pipeline (currently using mock data).

---
> Source: [lsiswin/IndustrialDAQ](https://github.com/lsiswin/IndustrialDAQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
