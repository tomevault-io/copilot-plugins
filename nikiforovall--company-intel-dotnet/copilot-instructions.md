## company-intel-dotnet

> Modus operandi - for any type of research store it in _plans including subagents.

<modus_operandi>
Modus operandi - for any type of research store it in _plans including subagents.
</modus_operandi>

<solution>
Each meaningful step make sure solution builds.
</solution>

<build_and_restart>
To build and restart the API after code changes:
1. Stop the resource: `mcp aspire execute_resource_command(api, resource-stop)`
2. Build: `dotnet build src/CompanyIntel.Api -p:WarningLevel=0 /clp:ErrorsOnly`
3. Start the resource: `mcp aspire execute_resource_command(api, resource-start)`
Always stop before building to avoid file lock errors.
</build_and_restart>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NikiforovAll)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NikiforovAll)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
