## go-table-driven-tests

> Todo TUIプロジェクトでテストを追加する際は、GoのTable Driven Tests（TDT）パターンに従ってください。

# Go Table Driven Tests (TDT) ガイドライン

Todo TUIプロジェクトでテストを追加する際は、GoのTable Driven Tests（TDT）パターンに従ってください。

## 📚 参考実装例

既存のテストファイルでTDTが適切に実装されている例：
- [pkg/ui/list_test.go](mdc:pkg/ui/list_test.go) - styleTaskContentInternal関数のテスト
- [pkg/domain/task_test.go](mdc:pkg/domain/task_test.go) - ShouldMoveToCompleted関数のテスト  
- [pkg/ui/model_test.go](mdc:pkg/ui/model_test.go) - cyclePriority関数のテスト
- [pkg/ui/config_test.go](mdc:pkg/ui/config_test.go) - ExpandHomePath関数のテスト

## 🔧 TDT基本構造

### 標準的なTDTパターン

```go
func TestFunctionName(t *testing.T) {
    tests := []struct {
        name        string
        input       InputType
        expected    ExpectedType
        description string // 日本語でテストケースの説明
    }{
        {
            name:        "test_case_name",
            input:       inputValue,
            expected:    expectedValue,
            description: "テストケースの説明",
        },
        // 追加のテストケース...
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := FunctionToTest(tt.input)
            if result != tt.expected {
                t.Errorf("FunctionToTest() = %v, expected %v for %s", 
                    result, tt.expected, tt.description)
            }
        })
    }
}
```

## 📋 TDTルール

### 1. テストケース命名
- `name`フィールドには**snake_case**を使用
- 具体的で理解しやすい名前を付ける
- 例: `"complete_incomplete_task"`, `"priority_A_to_B"`

### 2. 構造体フィールド
- `name`: テストケース名（必須）
- `description`: 日本語での説明（必須）
- 入力パラメータ用のフィールド
- `expected`: 期待される結果
- 必要に応じて追加のコンテキストフィールド

### 3. 日本語コメント
- `description`フィールドには日本語で分かりやすい説明を記載
- テストの意図が明確になるようにする

### 4. エラーメッセージ
```go
t.Errorf("FunctionName() = %v, expected %v for %s", 
    actual, expected, tt.description)
```

### 5. 境界値テスト
```go
{
    name:        "boundary_condition_max",
    input:       maxValue,
    expected:    expectedResult,
    description: "最大値での境界テスト",
},
{
    name:        "boundary_condition_min", 
    input:       minValue,
    expected:    expectedResult,
    description: "最小値での境界テスト",
},
```

## ⚠️ 注意点

### エラーケースのテスト
```go
{
    name:           "invalid_input_error",
    input:          invalidInput,
    expectedError:  true,
    expectedResult: nil,
    description:    "無効な入力でエラーが発生する",
},
```

### 複雑なケース
時間や日付に関するテストの場合：
```go
{
    name:           "time_based_test",
    input:          inputValue,
    now:            time.Date(2025, 5, 31, 21, 0, 0, 0, time.UTC),
    expected:       expectedValue,
    description:    "特定時刻での処理テスト",
},
```

## 🚀 実装時のチェックリスト

- [ ] 正常系のテストケースを含む
- [ ] 異常系のテストケースを含む  
- [ ] 境界値のテストケースを含む
- [ ] 各テストケースに適切な`name`と`description`がある
- [ ] エラーメッセージに`tt.description`を含める
- [ ] `t.Run(tt.name, ...)`でサブテストを作成
- [ ] 必要に応じてヘルパー関数を作成

## 📖 ベストプラクティス

1. **明確なテストケース名**: テストの内容が名前から分かるようにする
2. **包括的なカバレッジ**: 正常系、異常系、境界値を網羅
3. **読みやすさ**: 他の開発者が理解しやすい構造にする
4. **保守性**: 新しいテストケースを簡単に追加できる構造にする
5. **実用的なコメント**: `description`フィールドで意図を明確にする

このガイドラインに従って、一貫性のある高品質なテストコードを維持してください。

---
> Source: [yuucu/todotui](https://github.com/yuucu/todotui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
