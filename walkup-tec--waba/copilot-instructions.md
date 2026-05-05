## atualize-tudo

> Comando padrão "atualize tudo" - backup, build, doc, commits e push


# Comando "Atualize tudo"

Quando o usuário disser **"atualize tudo"**, execute nesta ordem:

1. **Backup (espelho fiel seletivo)** — garantir rotina para espelhar apenas as pastas definidas para `E:\` (agendada **diariamente 12:00**) e executar o backup agora
2. **Build** — `npm run build` (atualiza a pasta `dist/`)
3. **Arquivos locais** — Verificar se há alterações pendentes em código fonte
4. **Pasta doc** — Garantir que documentação esteja coerente com o estado atual do projeto
5. **Git** — Fazer commit das alterações (se houver) e `git push`

## Sequência de execução

```bash
# 0) Backup (PowerShell)
# - Script: C:\Scripts\backup-d-para-e.ps1
# - Destino: E:\
# - Log: D:\Backup-Logs
# - Agendado: tarefa "Backup D para E (12h)" (diário 12:00)

npm run build
git status
git add .
git commit -m "mensagem descritiva"   # se houver alterações
git push
```

## Rotina de Backup (origens específicas → E:\) — espelho fiel seletivo

### Script (criar/atualizar)

Salvar em `C:\Scripts\backup-d-para-e.ps1`:

```powershell
$destRoot = "E:\"
$logDir = "D:\Backup-Logs"
$mappings = @(
  @{ Source = "H:\Meu Drive\Drive Profissional"; Destination = "Meu Drive\Drive Profissional" },
  @{ Source = "D:\Projeto Bruno LV"; Destination = "Projeto Bruno LV" },
  @{ Source = "D:\Site Credilix"; Destination = "Site Credilix" },
  @{ Source = "D:\SOMA Promotora"; Destination = "SOMA Promotora" },
  @{ Source = "D:\Waba"; Destination = "Waba" }
)

New-Item -ItemType Directory -Force -Path $logDir | Out-Null

foreach ($map in $mappings) {
  robocopy $map.Source (Join-Path $destRoot $map.Destination) /MIR /R:2 /W:5 /XJ /FFT /Z /NP /TEE
}
```

### Agendar diariamente às 12:00 (criar/atualizar)

Rodar em PowerShell **como Administrador**:

```powershell
schtasks /Create /TN "Backup D para E (12h)" /SC DAILY /ST 12:00 /RL HIGHEST /F `
  /TR "powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:\Scripts\backup-d-para-e.ps1"
```

### Executar backup imediatamente (quando pedir “Atualize tudo”)

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:\Scripts\backup-d-para-e.ps1
```

## Notas

- Se não houver alterações para commit, pular o commit e fazer apenas o push (caso existam commits locais não enviados)
- A pasta `dist/` normalmente está no `.gitignore` — não comitar `dist/` a menos que o projeto exija
- Ajustar a mensagem do commit conforme o tipo de alteração (feat, fix, chore, docs)
- O destino `E:\` deve conter apenas as raízes permitidas: `Meu Drive`, `Projeto Bruno LV`, `Site Credilix`, `SOMA Promotora`, `Waba`

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
