## workorderblender

> - **Project Name**: WorkOrderBlender


# WorkOrderBlender Core Project Rules

## Project Overview
- **Project Name**: WorkOrderBlender
- **Framework**: .NET Framework 4.8 (Windows Forms application)
- **Platform**: Windows x64 only
- **Purpose**: Work Order SDF Consolidator tool for consolidating work order SDF files with pending changes preview

## Core Architecture
- Main entry point: [Program.cs](mdc:Program.cs)
- Primary UI: [MainForm.cs](mdc:MainForm.cs) and [MainForm.Designer.cs](mdc:MainForm.Designer.cs)
- Configuration: [UserConfig.cs](mdc:UserConfig.cs) and [settings.xml](mdc:settings.xml)
- Database utilities: [SqlCeUtils.cs](mdc:SqlCeUtils.cs)
- Edit tracking: [InMemoryEditStore.cs](mdc:InMemoryEditStore.cs)

## Development Standards
- **Always use native .NET Framework 4.8 components and libraries** when possible
- **Never modify source databases** - only use databases for reading
- **Enable DEBUG logging** throughout the application
- **Use single-line comments** frequently for code understanding
- **Keep code DRY and robust** - no redundant code
- **Reuse and improve existing functionality** over implementing new features
- **Check for unused functions** and remove them

## File Organization
- Utility methods with shared functionality should be in utility libraries
- Dialog forms should inherit from appropriate base classes
- Configuration data should use [UserConfig.cs](mdc:UserConfig.cs) serialization
- Database schema is defined in [resources/wo_schema.xml](mdc:resources/wo_schema.xml)

## Error Handling
- All database operations should be wrapped in try-catch blocks
- Use [Program.Log()](mdc:Program.cs) for logging errors and debug information
- Show user-friendly error messages via MessageBox when appropriate
- Never crash the application - always handle exceptions gracefully

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HomesteadCabinet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
