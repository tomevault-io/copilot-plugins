## tokenfold

> $rawInput = [Console]::In.ReadToEnd()

# TaskResume Hook
# PowerShell template for Windows hook execution.

try {
    $rawInput = [Console]::In.ReadToEnd()
    if ($rawInput) {
        $null = $rawInput | ConvertFrom-Json
    }
} catch {
    Write-Error "[TaskResume] Invalid JSON input: $($_.Exception.Message)"
}

@{
    cancel = $false
    contextModification = ""
    errorMessage = ""
} | ConvertTo-Json -Compress

---
> Source: [snchimata/tokenfold](https://github.com/snchimata/tokenfold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
