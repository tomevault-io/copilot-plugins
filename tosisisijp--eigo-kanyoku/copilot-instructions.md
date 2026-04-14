## eigo-kanyoku

> - Obsidian_to_Anki独自形式の品質基準を満たすための専用PowerShell操作

# Obsidian_to_Anki専用PowerShell操作

## 目的
- Obsidian_to_Anki独自形式の品質基準を満たすための専用PowerShell操作
- START/ENDブロック検証の自動化
- ENDマーカー後のLF必須判定
- プラグイン認識可能な形式への修正支援

## 前提条件
- [[202-Obsidian_to_Anki独自形式管理.mdc]] の品質基準に完全準拠
- [[110-PowerShell操作管理.mdc]] の基本操作を活用（旧103統合）
- Windows環境でのLF改行コード統一

## Obsidian_to_Anki形式検証

### 単一ファイル完全検証
```powershell
# Obsidian_to_Anki形式の完全検証関数
function Test-ObsidianToAnkiFile {
    param([string]$FilePath)
    
    Write-Host "=== $FilePath の検証 ===" -ForegroundColor Yellow
    
    if (-not (Test-Path $FilePath)) {
        Write-Host "❌ ファイルが存在しません" -ForegroundColor Red
        return $false
    }
    
    $content = Get-Content $FilePath -Raw
    $isValid = $true
    
    # 1. Obsidian_to_Ankiセクション存在確認
    if ($content -match "# Obsidian_to_Anki") {
        Write-Host "✅ Obsidian_to_Ankiセクション: 存在" -ForegroundColor Green
        
        # 2. START/ENDマーカー確認
        $hasStart = $content -match "START"
        $hasEnd = $content -match "END"
        
        if ($hasStart) { 
            Write-Host "✅ STARTマーカー: 存在" -ForegroundColor Green 
        } else { 
            Write-Host "❌ STARTマーカー: 不存在" -ForegroundColor Red
            $isValid = $false
        }
        
        if ($hasEnd) { 
            Write-Host "✅ ENDマーカー: 存在" -ForegroundColor Green
            
            # 3. ENDマーカー後の文字コード確認（Obsidian_to_Anki専用要件）
            $endIndex = $content.LastIndexOf("END")
            if ($endIndex -ge 0) {
                Write-Host "📍 ENDマーカー位置: $endIndex" -ForegroundColor Cyan
                $afterEndStart = $endIndex + 3
                $remainingLength = $content.Length - $afterEndStart
                
                # ⭐ Obsidian_to_Anki専用判定: ENDの後にLF必須
                if ($remainingLength -eq 0) {
                    Write-Host "❌ ENDがファイル末尾（プラグイン認識不可）" -ForegroundColor Red
                    Write-Host "   → ENDの後にLF改行コードが必要" -ForegroundColor Yellow
                    $isValid = $false
                } else {
                    $afterEnd = $content.Substring($afterEndStart, [Math]::Min(5, $remainingLength))
                    $bytes = [System.Text.Encoding]::UTF8.GetBytes($afterEnd)
                    
                    Write-Host "📋 ENDの後の文字:" -ForegroundColor Cyan
                    for ($i = 0; $i -lt $bytes.Length; $i++) {
                        $char = $bytes[$i]
                        $hex = "{0:X2}" -f $char
                        $ascii = if ($char -ge 32 -and $char -le 126) { [char]$char } else { "." }
                        
                        $status = switch ($hex) {
                            "20" { "❌ スペース（除去必要）"; $isValid = $false }
                            "0D" { "✅ CR（正常）" }
                            "0A" { "✅ LF（正常）" }
                            default { "⚠️ その他の文字"; $isValid = $false }
                        }
                        
                        Write-Host "  END+$($i+1): $hex ($ascii) $status"
                    }
                    
                    # 4. LF形式確認（Obsidian_to_Anki必須要件）
                    if ($bytes.Length -ge 1 -and $bytes[0] -eq 0x0A) {
                        Write-Host "✅ LF改行コード（プラグイン認識可能）" -ForegroundColor Green
                    } elseif ($bytes.Length -ge 2 -and $bytes[0] -eq 0x0D -and $bytes[1] -eq 0x0A) {
                        Write-Host "❌ CRLFあり（LF統一が必要）" -ForegroundColor Red
                        $isValid = $false
                    } elseif ($bytes.Length -ge 1 -and $bytes[0] -eq 0x20) {
                        Write-Host "❌ ENDの直後にスペース（除去が必要）" -ForegroundColor Red
                        $isValid = $false
                    }
                }
            }
        } else { 
            Write-Host "❌ ENDマーカー: 不存在" -ForegroundColor Red
            $isValid = $false
        }
        
        # 5. WithSpeech確認
        if ($content -match "WithSpeech") {
            Write-Host "✅ WithSpeech: 存在" -ForegroundColor Green
        } else {
            Write-Host "⚠️ WithSpeech: 不存在（推奨）" -ForegroundColor Yellow
        }
        
        # 6. 重複検出（202番ルール準拠）
        $obsidianToAnkiCount = ($content | Select-String -Pattern "# Obsidian_to_Anki" -AllMatches).Matches.Count
        $startCount = ($content | Select-String -Pattern "^START" -AllMatches).Matches.Count
        $endCount = ($content | Select-String -Pattern "^END" -AllMatches).Matches.Count
        
        Write-Host "📊 重複検出結果:" -ForegroundColor Cyan
        Write-Host "  # Obsidian_to_Anki セクション数: $obsidianToAnkiCount"
        Write-Host "  START マーカー数: $startCount"
        Write-Host "  END マーカー数: $endCount"
        
        if ($obsidianToAnkiCount -gt 1) {
            Write-Host "❌ Obsidian_to_Ankiセクション重複 ($obsidianToAnkiCount 個)" -ForegroundColor Red
            $isValid = $false
        } elseif ($startCount -ne $endCount) {
            Write-Host "❌ START/ENDマーカー数不一致 (START:$startCount, END:$endCount)" -ForegroundColor Red
            $isValid = $false
        } elseif ($startCount -gt 1) {
            Write-Host "❌ START/ENDペア重複 ($startCount 組)" -ForegroundColor Red
            $isValid = $false
        } else {
            Write-Host "✅ 重複なし（セクション:$obsidianToAnkiCount, START/END:$startCount組）" -ForegroundColor Green
        }
        
        # 7. ID重複検出
        $idMatches = $content | Select-String -Pattern "<!--ID: (\d+)-->" -AllMatches
        if ($idMatches.Matches.Count -gt 1) {
            Write-Host "❌ ID重複検出 ($($idMatches.Matches.Count) 個)" -ForegroundColor Red
            $idMatches.Matches | ForEach-Object { Write-Host "  ID: $($_.Groups[1].Value)" }
            $isValid = $false
        } elseif ($idMatches.Matches.Count -eq 1) {
            Write-Host "✅ ID: $($idMatches.Matches[0].Groups[1].Value)" -ForegroundColor Green
        } else {
            Write-Host "⚠️ ID未生成（初回同期待ち）" -ForegroundColor Yellow
        }
        
    } else {
        Write-Host "⚠️ Obsidian_to_Ankiセクション: 存在しない" -ForegroundColor Yellow
        return $false  # セクション自体がない場合は検証対象外
    }
    
    Write-Host ""
    return $isValid
}

# 使用例
# Test-ObsidianToAnkiFile "example.md"
```

### 複数ファイル一括検証
```powershell
# Obsidian_to_Anki形式の一括検証
function Test-AllObsidianToAnkiFiles {
    # 110番ルールの基本操作を活用（旧103統合）
    $mdFiles = Get-ChildItem -Path "." -Filter "*.md" | Where-Object { $_.Name -notlike "README.md" }
    
    Write-Host "=== Obsidian_to_Anki形式 一括検証開始 ===" -ForegroundColor Magenta
    Write-Host "対象ファイル数: $($mdFiles.Count)" -ForegroundColor Magenta
    Write-Host ""
    
    # Obsidian_to_Anki専用の結果分類
    $results = @{
        "正常" = @()
        "ENDスペース問題" = @()
        "改行コード問題" = @()
        "ENDファイル末尾問題" = @()  # ⭐ Obsidian_to_Anki専用
        "セクション不存在" = @()
        "マーカー不存在" = @()
        "その他問題" = @()
    }
    
    foreach ($file in $mdFiles) {
        $content = Get-Content $file.FullName -Raw
        
        if ($content -match "# Obsidian_to_Anki") {
            if ($content -match "END") {
                $endIndex = $content.LastIndexOf("END")
                $afterEndStart = $endIndex + 3
                $remainingLength = $content.Length - $afterEndStart
                
                # ⭐ Obsidian_to_Anki専用判定ロジック
                if ($remainingLength -eq 0) {
                    $results["ENDファイル末尾問題"] += $file.Name
                } else {
                    $afterEnd = $content.Substring($afterEndStart, [Math]::Min(5, $remainingLength))
                    $bytes = [System.Text.Encoding]::UTF8.GetBytes($afterEnd)
                    
                    if ($bytes.Length -ge 1 -and $bytes[0] -eq 0x20) {
                        $results["ENDスペース問題"] += $file.Name
                    } elseif ($bytes.Length -eq 1 -and $bytes[0] -eq 0x0A) {
                        $results["改行コード問題"] += $file.Name
                    } elseif ($bytes.Length -ge 2 -and $bytes[0] -eq 0x0D -and $bytes[1] -eq 0x0A) {
                        $results["正常"] += $file.Name
                    } else {
                        $results["その他問題"] += $file.Name
                    }
                }
            } else {
                $results["マーカー不存在"] += $file.Name
            }
        } else {
            $results["セクション不存在"] += $file.Name
        }
    }
    
    # Obsidian_to_Anki専用の結果表示
    Write-Host "=== Obsidian_to_Anki検証結果サマリー ===" -ForegroundColor Magenta
    foreach ($category in $results.Keys) {
        $count = $results[$category].Count
        if ($count -gt 0) {
            $color = switch ($category) {
                "正常" { "Green" }
                "セクション不存在" { "Yellow" }
                default { "Red" }
            }
            Write-Host "$category ($count 件):" -ForegroundColor $color
            $results[$category] | ForEach-Object { Write-Host "  - $_" }
            Write-Host ""
        }
    }
    
    return $results
}

# 使用例
# $results = Test-AllObsidianToAnkiFiles
```

## Obsidian_to_Anki形式修正

### ENDマーカー修正（LF追加）
```powershell
# ENDマーカー後にLF改行コードを追加
function Add-LFAfterEND {
    param([string]$FilePath)
    
    Write-Host "ENDマーカー後LF追加: $FilePath" -ForegroundColor Yellow
    
    $content = Get-Content $FilePath -Raw
    $endIndex = $content.LastIndexOf("END")
    
    if ($endIndex -eq -1) {
        Write-Host "❌ ENDマーカーが見つかりません" -ForegroundColor Red
        return $false
    }
    
    $afterEndStart = $endIndex + 3
    $remainingLength = $content.Length - $afterEndStart
    
    if ($remainingLength -eq 0) {
        # ENDがファイル末尾の場合、LF追加
        $newContent = $content + "`n"
        Set-Content $FilePath -Value $newContent -NoNewline -Encoding UTF8
        Write-Host "✅ ENDの後にLF追加完了" -ForegroundColor Green
        return $true
    } else {
        Write-Host "⚠️ ENDの後に既に文字が存在します" -ForegroundColor Yellow
        return $false
    }
}

# 使用例
# Add-LFAfterEND "example.md"
```

### ENDスペース除去
```powershell
# ENDマーカー後のスペースを除去
function Remove-SpaceAfterEND {
    param([string]$FilePath)
    
    Write-Host "ENDマーカー後スペース除去: $FilePath" -ForegroundColor Yellow
    
    $content = Get-Content $FilePath -Raw
    
    # "END " を "END" に置換（スペース除去）
    if ($content -match "END ") {
        $newContent = $content -replace "END ", "END"
        Set-Content $FilePath -Value $newContent -NoNewline -Encoding UTF8
        Write-Host "✅ ENDの後のスペース除去完了" -ForegroundColor Green
        return $true
    } else {
        Write-Host "⚠️ ENDの後にスペースは見つかりませんでした" -ForegroundColor Yellow
        return $false
    }
}

# 使用例
# Remove-SpaceAfterEND "example.md"
```

### 一括修正処理
```powershell
# Obsidian_to_Anki形式の一括修正
function Repair-AllObsidianToAnkiFiles {
    param([switch]$WhatIf)  # 実際の修正前の確認用
    
    Write-Host "=== Obsidian_to_Anki形式 一括修正開始 ===" -ForegroundColor Magenta
    
    # まず検証を実行
    $results = Test-AllObsidianToAnkiFiles
    
    $repairResults = @{
        "修正済み" = @()
        "修正不要" = @()
        "修正失敗" = @()
    }
    
    # ENDスペース問題の修正
    foreach ($fileName in $results["ENDスペース問題"]) {
        if ($WhatIf) {
            Write-Host "修正予定: $fileName (ENDスペース除去)" -ForegroundColor Cyan
        } else {
            if (Remove-SpaceAfterEND $fileName) {
                $repairResults["修正済み"] += "$fileName (スペース除去)"
            } else {
                $repairResults["修正失敗"] += "$fileName (スペース除去失敗)"
            }
        }
    }
    
    # ENDファイル末尾問題の修正
    foreach ($fileName in $results["ENDファイル末尾問題"]) {
        if ($WhatIf) {
            Write-Host "修正予定: $fileName (LF追加)" -ForegroundColor Cyan
        } else {
            if (Add-LFAfterEND $fileName) {
                $repairResults["修正済み"] += "$fileName (LF追加)"
            } else {
                $repairResults["修正失敗"] += "$fileName (LF追加失敗)"
            }
        }
    }
    
    # 改行コード問題の修正（110番ルール活用・旧103統合）
    foreach ($fileName in $results["改行コード問題"]) {
        if ($WhatIf) {
            Write-Host "修正予定: $fileName (LF統一)" -ForegroundColor Cyan
        } else {
            try {
                # 110番ルールの改行コード統一処理を活用（旧103統合）
                $content = Get-Content $fileName -Raw
                $content = $content -replace "`r`n", "`n"
                $content = $content -replace "`n", "`r`n"
                Set-Content $fileName -Value $content -NoNewline -Encoding UTF8
                
                $repairResults["修正済み"] += "$fileName (LF統一)"
            } catch {
                $repairResults["修正失敗"] += "$fileName (LF統一失敗)"
            }
        }
    }
    
    # 正常ファイルは修正不要
    foreach ($fileName in $results["正常"]) {
        $repairResults["修正不要"] += $fileName
    }
    
    # 修正結果表示
    Write-Host "=== 修正結果サマリー ===" -ForegroundColor Magenta
    foreach ($category in $repairResults.Keys) {
        $count = $repairResults[$category].Count
        if ($count -gt 0) {
            $color = switch ($category) {
                "修正不要" { "Green" }
                "修正済み" { "Cyan" }
                default { "Red" }
            }
            Write-Host "$category ($count 件):" -ForegroundColor $color
            $repairResults[$category] | ForEach-Object { Write-Host "  - $_" }
            Write-Host ""
        }
    }
    
    return $repairResults
}

# 使用例
# Repair-AllObsidianToAnkiFiles -WhatIf  # 修正内容の確認
# Repair-AllObsidianToAnkiFiles          # 実際の修正実行
```

## プラグイン連携確認

### 正規表現マッチング確認
```powershell
# Obsidian_to_Ankiプラグインの正規表現との互換性確認
function Test-PluginCompatibility {
    param([string]$FilePath)
    
    Write-Host "=== プラグイン互換性確認: $FilePath ===" -ForegroundColor Cyan
    
    $content = Get-Content $FilePath -Raw
    
    # プラグインが使用する正規表現: /^START\n([\s\S]*?\n)END/gm
    # PowerShellでの近似パターン
    $pattern = "(?m)^START`n([\s\S]*?`n)END"
    
    if ($content -match $pattern) {
        Write-Host "✅ プラグイン正規表現: マッチ成功" -ForegroundColor Green
        $matchContent = $Matches[1]
        Write-Host "📋 マッチした内容の長さ: $($matchContent.Length) 文字" -ForegroundColor Cyan
        
        # マッチした内容の先頭・末尾確認
        $preview = if ($matchContent.Length -gt 100) { 
            $matchContent.Substring(0, 50) + "..." + $matchContent.Substring($matchContent.Length - 50)
        } else { 
            $matchContent 
        }
        Write-Host "📋 マッチ内容プレビュー: $preview" -ForegroundColor Cyan
        
        return $true
    } else {
        Write-Host "❌ プラグイン正規表現: マッチ失敗" -ForegroundColor Red
        Write-Host "   → START/ENDブロックがプラグインに認識されません" -ForegroundColor Yellow
        return $false
    }
}

# 使用例
# Test-PluginCompatibility "example.md"
```

### ID生成確認
```powershell
# Obsidian_to_AnkiプラグインによるID自動生成の確認
function Test-IDGeneration {
    param([string]$FilePath)
    
    Write-Host "=== ID生成状況確認: $FilePath ===" -ForegroundColor Cyan
    
    $content = Get-Content $FilePath -Raw
    
    # ID行の存在確認
    if ($content -match "<!--ID: (\d+)-->") {
        $id = $Matches[1]
        Write-Host "✅ ID既存: $id" -ForegroundColor Green
        
        # IDの形式確認（13桁のタイムスタンプ）
        if ($id.Length -eq 13 -and $id -match "^\d+$") {
            Write-Host "✅ ID形式: 正常（13桁タイムスタンプ）" -ForegroundColor Green
        } else {
            Write-Host "⚠️ ID形式: 非標準" -ForegroundColor Yellow
        }
        
        return $true
    } else {
        Write-Host "⚠️ ID未生成: プラグイン同期待ち" -ForegroundColor Yellow
        Write-Host "   → 初回同期時にObsidian_to_Ankiプラグインが自動生成" -ForegroundColor Cyan
        return $false
    }
}

# 使用例
# Test-IDGeneration "example.md"
```

## 統合検証・修正ワークフロー

### 完全ワークフロー
```powershell
# Obsidian_to_Anki形式の完全検証・修正ワークフロー
function Invoke-ObsidianToAnkiWorkflow {
    param(
        [switch]$SkipBackup,
        [switch]$WhatIf
    )
    
    Write-Host "=== Obsidian_to_Anki 完全ワークフロー開始 ===" -ForegroundColor Magenta
    Write-Host "WhatIf モード: $WhatIf" -ForegroundColor Cyan
    Write-Host ""
    
    # Step 1: 事前バックアップ
    if (-not $SkipBackup -and -not $WhatIf) {
        Write-Host "Step 1: バックアップ作成中..." -ForegroundColor Yellow
        $timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
        $backupDir = "backup_$timestamp"
        New-Item -ItemType Directory -Path $backupDir -Force | Out-Null
        
        $mdFiles = Get-ChildItem -Path "." -Filter "*.md" | Where-Object { $_.Name -notlike "README.md" }
        $mdFiles | ForEach-Object { Copy-Item $_.FullName "$backupDir\" }
        Write-Host "✅ バックアップ完了: $backupDir" -ForegroundColor Green
        Write-Host ""
    }
    
    # Step 2: 現状確認
    Write-Host "Step 2: 現状確認..." -ForegroundColor Yellow
    $initialResults = Test-AllObsidianToAnkiFiles
    Write-Host ""
    
    # Step 3: 修正実行
    Write-Host "Step 3: 修正処理..." -ForegroundColor Yellow
    $repairResults = Repair-AllObsidianToAnkiFiles -WhatIf:$WhatIf
    Write-Host ""
    
    # Step 4: 修正後確認
    if (-not $WhatIf) {
        Write-Host "Step 4: 修正後確認..." -ForegroundColor Yellow
        $finalResults = Test-AllObsidianToAnkiFiles
        Write-Host ""
        
        # Step 5: プラグイン互換性確認
        Write-Host "Step 5: プラグイン互換性確認..." -ForegroundColor Yellow
        $compatibleFiles = @()
        $mdFiles = Get-ChildItem -Path "." -Filter "*.md" | Where-Object { $_.Name -notlike "README.md" }
        
        foreach ($file in $mdFiles) {
            $content = Get-Content $file.FullName -Raw
            if ($content -match "# Obsidian_to_Anki") {
                if (Test-PluginCompatibility $file.FullName) {
                    $compatibleFiles += $file.Name
                }
            }
        }
        
        Write-Host "✅ プラグイン互換: $($compatibleFiles.Count) ファイル" -ForegroundColor Green
        Write-Host ""
    }
    
    # 最終サマリー
    Write-Host "=== ワークフロー完了サマリー ===" -ForegroundColor Magenta
    Write-Host "初期状態:"
    Write-Host "  正常: $($initialResults['正常'].Count) ファイル"
    Write-Host "  問題: $(($initialResults.Values | Measure-Object -Sum Count).Sum - $initialResults['正常'].Count) ファイル"
    
    if (-not $WhatIf) {
        Write-Host "修正後:"
        Write-Host "  正常: $($finalResults['正常'].Count) ファイル"
        Write-Host "  問題: $(($finalResults.Values | Measure-Object -Sum Count).Sum - $finalResults['正常'].Count) ファイル"
        Write-Host "  プラグイン互換: $($compatibleFiles.Count) ファイル"
    }
}

# 使用例
# Invoke-ObsidianToAnkiWorkflow -WhatIf          # 確認のみ
# Invoke-ObsidianToAnkiWorkflow                  # 実際の修正実行
# Invoke-ObsidianToAnkiWorkflow -SkipBackup      # バックアップなしで実行
```

## 関連ファイル
- [[202-Obsidian_to_Anki独自形式管理.mdc]] - 形式定義・品質基準（必須参照）
- [[110-PowerShell操作管理.mdc]] - 基本操作・共通処理（旧103統合）
- [[004-PowerShell操作ガイド.mdc]] - 汎用PowerShell技術
- [[001-AI実装ガイドライン.mdc]] - AI実装時の制約

## 重要な注意事項

### Obsidian_to_Anki専用要件
- **ENDの後にLF必須**: プラグインの正規表現 `/^START\n([\s\S]*?\n)END/gm` に対応
- **ID管理**: プラグインが自動生成するため、手動作成・編集は禁止
- **WithSpeech**: 音声機能有効化のため推奨

### 他のプロジェクトでの使用禁止
- このルールはeigo-kanyoku + Obsidian_to_Anki専用です
- 汎用的なPowerShell操作は [[004-PowerShell操作ガイド.mdc]] を参照
- プロジェクト共通操作は [[110-PowerShell操作管理.mdc]] を参照（旧103統合）

### 安全性の確保
- 重要な修正前は必ずバックアップを作成
- WhatIfモードでの事前確認を推奨
- 修正後のプラグイン互換性確認を必須とする

### 重複セクション修正
```powershell
# 重複したObsidian_to_Ankiセクションを修正
function Fix-DuplicateObsidianToAnkiSections {
    param([string]$FilePath)
    
    Write-Host "重複セクション修正: $FilePath" -ForegroundColor Yellow
    
    if (-not (Test-Path $FilePath)) {
        Write-Host "❌ ファイルが見つかりません: $FilePath" -ForegroundColor Red
        return $false
    }
    
    try {
        $content = Get-Content $FilePath -Raw
        
        # 重複検出
        $obsidianToAnkiCount = ($content | Select-String -Pattern "# Obsidian_to_Anki" -AllMatches).Matches.Count
        
        if ($obsidianToAnkiCount -le 1) {
            Write-Host "✅ 重複なし（セクション数: $obsidianToAnkiCount）" -ForegroundColor Green
            return $true
        }
        
        Write-Host "⚠️ 重複検出（セクション数: $obsidianToAnkiCount）" -ForegroundColor Yellow
        
        # 最初のObsidian_to_Ankiセクションを保持し、他を削除
        $sections = $content -split "# Obsidian_to_Anki"
        
        if ($sections.Count -ge 3) {
            # 最初のセクション（メインコンテンツ）+ 最初のObsidian_to_Ankiセクション
            $firstSection = $sections[0]
            $firstObsidianSection = $sections[1]
            
            # ENDマーカーまでを抽出
            if ($firstObsidianSection -match "(.*?END)") {
                $cleanObsidianSection = $matches[1]
                $newContent = $firstSection + "# Obsidian_to_Anki" + $cleanObsidianSection + "`n"
                
                Set-Content $FilePath -Value $newContent -NoNewline -Encoding UTF8
                Write-Host "✅ 重複セクション削除完了（$($obsidianToAnkiCount-1) 個削除）" -ForegroundColor Green
                return $true
            }
        }
        
        Write-Host "❌ 重複セクション修正失敗（複雑な構造）" -ForegroundColor Red
        return $false
        
    } catch {
        Write-Host "❌ 重複セクション修正エラー: $($_.Exception.Message)" -ForegroundColor Red
        return $false
    }
}

# 使用例
# Fix-DuplicateObsidianToAnkiSections "back-背中.md"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tosisisijp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tosisisijp)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
