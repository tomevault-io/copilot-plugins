## semsa-suporte

> Script PowerShell de suporte técnico corporativo para a Semsa/Manaus. Padroniza configuração de PCs, instala programas, habilita RDP/VNC/assistência remota e faz ingresso em domínio.

# semsa-suporte

## Objetivo
Script PowerShell de suporte técnico corporativo para a Semsa/Manaus. Padroniza configuração de PCs, instala programas, habilita RDP/VNC/assistência remota e faz ingresso em domínio.

## Stack
- PowerShell 5.1+ (script principal com ~1209 linhas)
- .NET Framework: System.Windows.Forms, System.Drawing, System.Security.Cryptography
- Batch (.bat) como wrapper para elevação UAC
- Encoding: UTF-8 sem BOM (crítico para PS 5.1 e `irm | iex`)

## Comandos
```powershell
# Execução remota (produção)
irm https://raw.githubusercontent.com/techbakimports/semsa-suporte/refs/heads/main/suporte.ps1 | iex

# Execução local
powershell -ExecutionPolicy Bypass -File .\suporte.ps1

# Via batch (eleva UAC automaticamente)
.\SEMSA_SUPORTE.bat
```

## Funções (suporte.ps1) — GUI-only, 3 funções
| Função | Linha | Descrição |
|--------|-------|-----------|
| `Get-PCInfo` | 5 | Hardware: SO, CPU, RAM, discos, impressoras (usa Get-CimInstance) |
| `Test-RemoteAssistance` | 185 | Diagnóstico de Assistência Remota com output colorido |
| `Show-PadronizacaoGUI` | 251 | GUI WinForms dark theme — entry point único do script |

### GUI: etapas de padronização (dentro de Show-PadronizacaoGUI)
1. Fuso horário  2. Assistência Remota  3. RDP  4. Renomear PC (opcional)
5. Ingressar domínio (opcional)  6. Conta admin  7. Instalar programas (Chocolatey ou Standalone)

## Programas corporativos pré-configurados
Java 8, WinRAR, VLC, Foxit Reader, PDF24, LibreOffice, Chrome, Firefox, Kaspersky AV, UltraVNC, Fusion

## Requisitos
- Windows 7 SP1+ / Server 2008 R2+, PowerShell 5.1+
- Execução como Administrador (obrigatório para todas as funções)
- Rede local `\\balbina\f$\INSTALL_SEMSA-2023\` disponível para instalação corporativa

## Regras
- Sempre testar localmente antes de atualizar o script no GitHub (é consumido via `irm | iex` em produção)
- `autorun.inf` é para distribuição via USB/CD — não remover
- O caminho de rede `\\balbina\f$\` é específico da infraestrutura Semsa — não generalizar

---
> Source: [techbakimports/semsa-suporte](https://github.com/techbakimports/semsa-suporte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
