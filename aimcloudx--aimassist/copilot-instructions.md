## aimassist

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要
WPFベースのコマンドランチャー・生産性向上ツール
- 複数のプロジェクトから構成される.NETアプリケーション
- DI、Factory、Command、MVVMパターンを採用
- ユニットベースの機能拡張システム

## Common Development Commands

### Build and Test
```bash
# Build entire solution
dotnet build AimAssist.sln

# Run tests
dotnet test src/AimAssist/AimAssist.Tests/

# Run application
dotnet run --project src/AimAssist/AimAssist/

# Publish release build
dotnet publish src/AimAssist/AimAssist/ -c Release --self-contained true -r win-x64
```

### Project Structure
```
AimAssist/                    # Main WPF application
├── AimAssist.Core/          # Core interfaces and base services
├── AimAssist.Services/      # Business logic services
├── AimAssist.Units/         # Unit-based feature implementations
├── AimAssist.Plugins/       # MEF plugin system
├── AimAssist.Tests/         # xUnit tests with Moq
Common/                      # Shared functionality
Common.UI/                   # Shared UI components and WebView controls
Common.UI.Editor/            # Monaco Editor integration
Common.Dependencies/         # Consolidated NuGet dependencies
```

## Architecture Overview

### Dependency Injection System
Uses Microsoft.Extensions.DependencyInjection with 7 specialized service modules:
- **CoreServiceModule**: UnitsService, CommandService, ErrorHandling
- **ApplicationServiceModule**: Application lifecycle services
- **OptionServiceModule**: Configuration and settings (SettingManager)
- **FactoryServiceModule**: Factory pattern implementations
- **PluginServiceModule**: MEF plugin system
- **UiServiceModule**: MainWindow, PickerService, UI components
- **InitializationServiceModule**: Application startup services

Service registration in `App.xaml.cs` via `ServiceRegistration.RegisterServices()` extension method.

### Unit-Based Plugin Architecture
Core extensibility through `IUnit` interface:
```csharp
public interface IUnit {
    IMode Mode { get; }      // Operating context (WorkTools, Snippets, etc.)
    string Name { get; }     
    string Description { get; }
    string Category { get; } // For grouping and sorting
}
```

**Auto-Registration via Attributes**:
```csharp
[AutoRegisterUnit("CategoryName", Priority = 100, IsEnabled = true)]
public class MyUnit : IUnit { }
```

**Factory Pattern**: Multiple specialized factories combined via `CompositeUnitsFactory`:
- `ReflectionBasedUnitsFactory` - Attribute-based discovery
- `WorkToolsUnitsFactory` - Work productivity tools
- `CheatSheetUnitsFactory` - Help documentation
- `PluginUnitsFactory` - MEF-based external plugins

### MVVM Implementation
- `MainWindowViewModel` with `INotifyPropertyChanged`
- Command pattern via `IAppCommands` and `RelayCommand`
- Clear View/ViewModel/Model separation
- Two-way binding with ObservableCollections

### Display Order Logic
Units are sorted by: Mode display order → IUnit/IFeature type (IUnit=0, IFeature=1) → Category → Name

## Key Integrations

### Monaco Editor (VS Code Editor)
- Full Monaco Editor embedded via WebView2
- JavaScript/TypeScript with webpack build system
- Located in `Common.UI.Editor/` project
- Custom keybindings and Nord theme support

### Speech Recognition
- **Vosk**: Primary Japanese speech recognition with local models
- **Whisper.net**: Alternative recognition engine
- **NAudio**: Audio input processing

### MEF Plugin System
- Plugin directory: `./Plugins/`
- `IUnitPlugin` interface for external plugins
- Automatic composition with graceful error handling
- System.ComponentModel.Composition framework

### LLM Integration
- Microsoft Semantic Kernel integration
- Google Connectors for LLM services
- ChatGPT and Claude UI controls

## 完了したタスク
### GitHub Actions自動リリース機能
- `release.yml`: シンプルなリリースワークフロー
- `build-and-release.yml`: 高機能なビルド・リリースワークフロー
  - キャッシュ機能
  - テスト実行
  - 自動バージョニング
  - タグベース・ブランチベースリリース
  - セルフコンテインド実行ファイル生成
  - 詳細なリリースノート生成

#### 設定内容
- mainブランチpush時に自動リリース
- vタグpush時に正式リリース
- Windows x64向けセルフコンテインド実行ファイル
- 自動バージョニング（日付ベース + ビルド番号）
- ZIP形式でのアーティファクト配布

### AimAssistCommandsControl 動的コマンドボタン生成機能
- IAppCommandsインターフェースのすべてのコマンドを自動検出
- リフレクションを使用してコマンドプロパティを動的に取得
- 各コマンドに対応するボタンを自動生成
- 日本語表示名のマッピング機能
- シンプルなデフォルトスタイル

#### 実装内容
- ShutdownAimAssist、ToggleMainWindow、ShowPickerWindowの3つのコマンドをサポート
- 動的ボタン生成により新しいコマンド追加時の保守性向上
- シンプルなデフォルトボタンスタイルで統一
- ScrollViewer内での適切なレイアウト

### MainWindow DI インターフェース化
- IMainWindowインターフェースの作成
- MainWindowクラスでのインターフェース実装
- UIServiceModuleでのインターフェースベース登録
- ServiceProviderからインターフェース経由でMainWindow取得可能に

#### 実装内容
- IMainWindowインターフェースで主要メソッドを定義
- テスタビリティとコードの疎結合化を実現
- 既存コード互換性のため具象クラス登録も維持
- インターフェース経由での依存性注入が可能

## TODO
- なし

## 最新の変更
### MainWindowViewModel IUnit/IFeature表示順序修正
- **LoadUnitsForMode**: IUnitとIFeatureが混在している場合にIUnitが先に表示されるように並び替え順序を修正
- **並び替え優先度**: Mode表示順→IUnit/IFeature種別(IUnit=0, IFeature=1)→Category→Name順に変更
- **混在表示**: 同一モード内でIUnitとIFeatureが混在する場合の表示順序を統一
- **ユーザー体験**: より直感的な順序でユニットとフィーチャーが表示されるように改善

### IUnit/IFeature順序統一修正
- **IUnitsService**: RegisterUnitsとRegisterFeaturesメソッドの定義順序をIUnit→IFeature順に統一
- **UnitsService**: メソッド実装順序をIUnit→IFeature順に統一
- **FactoryInitializationService**: RegisterUnitsとRegisterFeaturesの呼び出し順序をIUnit→IFeature順に統一
- **WorkToolsUnitsFactory**: インターフェース継承順序をIUnitsFactory→IFeaturesFactory順に統一
- **メソッド定義**: IUnitsFactoryのGetUnitsメソッドを追加してCreateUnitsとの一貫性を確保
- **コード一貫性**: すべてのファクトリーとサービスでIUnit関連処理がIFeature関連処理より先に実行される構造に統一

### Unit表示順序の修正
- **Mode→Category→Name順序**: Unitの表示順序をMode表示順→Category→Name順に統一
- **ReflectionBasedUnitsFactory修正**: Priority降順からMode表示順→Category→Name順に変更
- **MainWindowViewModel修正**: CategorySortKey→Name順からMode表示順→Category→Name順に変更
- **統一された並び順**: すべてのUnit表示において一貫したソート順序を実現

### ComputerView 超軽量化 - タイマー完全削除
- **タイマー完全削除**: 定期的な自動更新機能を削除してCPU使用率を最小化
- **オンデマンド読み込み**: ユーザーがタブを選択した時のみ最新データを取得
- **リソース最小化**: Timer、CancellationToken、各種フラグ変数を削除
- **シンプル構造**: コンストラクタ→タブ選択→データ取得の最小限フロー
- **最新データ保証**: タブ切り替えごとに必ず最新データを取得
- **メモリ効率**: 不要なオブジェクト管理やライフサイクル処理を排除
- **パフォーマンス重視**: バックグラウンド処理の完全分離でUI応答性最大化
- **コード簡素化**: 200行以上のコード削減で保守性向上
- **即応性**: View表示から操作可能まで最短時間を実現

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AimCloudX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
