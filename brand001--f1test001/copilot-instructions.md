## f1test001

> 請產出 Azure DevOps PR 內容, 產出 PR 的結果請用 "英文" 輸出，


請產出 Azure DevOps PR 內容, 產出 PR 的結果請用 "英文" 輸出，

step1.
先取得 azure/master 的 commit SHA，
然後檢查當前 git branch 是否包含該 commit SHA。

若有則將格式內代碼，請將

- [ ] Already contains azure/master codebase
      變成
- [x] Already contains azure/master codebase

；若無則停下動作 提醒需先合併 azure/master

step2.
執行 git branch --show-current | sed -E 's/[-\_][^-_]\*$//'
其 output 結果視為 "票號variable" 並自動替換

step3.
執行 `git diff azure/master "當前gitBranchName"`，基於這個 git diff 生成 PR 內容。
檢查過大小寫、註釋、引用正確。

若一切正確則將格式內，請將

- [ ] That typing, comment and reference are correct
      變成
- [x] That typing, comment and reference are correct

若修改行數小於180行，請將

- [ ] These code-changes of this PR are less than 150-180 lines
      變成
- [x] These code-changes of this PR are less than 150-180 lines

final step.
輸出 PR 內容為 markdown, 請遵循以下格式, 以英文輸出：

## JIRA ticket

[票號variable](https://arcadie.atlassian.net/browse/票號variable)

## Related Link(s)

- e.g. [若需動多個代碼庫，請提供關聯PR](mdc:the pr link)
- e.g. [title](mdc:https://dev.azure.com/ogglobal/FE-Engineering/_git/J1-APP-CODE/pullrequest)

## Essential Code-changes List

- e.g. 串接xxx api
- e.g. 根據api回傳新增下拉選單

## The effect scope of code-changes

- e.g. `src/components/Button.tsx` - 優化按鈕樣式
- e.g. `pages/index.tsx` - 新增首頁 Banner

## Rollback Solution

- e.g. revert PR
- e.g. feature flag 關閉

## Performance/Security check

- e.g. 無明顯效能影響

## PR check-list

```bash
# 創建臨時 PR 內容文件
cat > pr-body.md <<EOF
${generatedContent}
EOF

# 使用 Azure DevOps CLI 創建 PR
az repos pr create --source-branch ${currentBranch} --target-branch master --title "${generatedTitle}" --description "$(cat pr-body.md)"

# 強制刪除臨時文件（無論 PR 創建是否成功）
rm -f pr-body.md
```

7. **Step 7: 更新 JIRA 狀態**

```javascript
// 自動更新 JIRA
await jira.transitionIssue(ticketNumber, 'In Review');
await jira.addComment(ticketNumber, `PR created: ${prUrl}`);
```

8. **Step 8: 清理驗證**

```bash
# 確保臨時文件已被刪除
if [ -f pr-body.md ]; then
    echo "⚠️ Warning: pr-body.md still exists, forcing removal..."
    rm -f pr-body.md
fi
```

## 📝 標準 PR 格式模板

```markdown
## JIRA票號

[${jiraTicket}](mdc:https:/arcadie.atlassian.net/browse/${jiraTicket})

**票號摘要**: ${jiraSummary}
**狀態**: ${jiraStatus}
**指派給**: ${jiraAssignee}
**優先級**: ${jiraPriority}

## 相關連結 <!-- 若無，可刪除或留白 -->

${generateRelatedLinks()}

## 請條列修改重點 <!-- 異動的文件名稱, 與異動的內容 -->

${generateChangesSummary()}

## 此次修改影響範圍 <!-- 影響的文件名稱與路徑，需具體 -->

${generateImpactAnalysis()}

## Rollback方案 <!-- 建議補充：若上線出問題，如何回滾 -->

- revert PR #${prNumber}
- 若有 feature flag，請關閉相關功能開關
  ${generateSpecificRollbackPlan()}

## 效能/安全性考量 <!-- 建議補充：有無潛在效能瓶頸或安全疑慮 -->

${generatePerformanceSecurityAnalysis()}

## 提交PR前請檢查以下是否已完成：

- [${masterMerged}] 已合併azure/master最新代碼
- [${codeReviewed}] 已自行檢查過大小寫、註釋、引用正確
- [x] PR 有明確標題、內容描述與commit
- [${codeLineCheck}] 已留意這張PR代碼變更未超過150-180行(asset增加不算)
```

## 🔍 自動化內容生成邏輯

### generateChangesSummary()

基於 git diff 分析：

```javascript
const changes = {
	documentation: [], // .cursor/rules, README 等
	coreLogic: [], // 主要功能邏輯
	components: [], // UI 組件
	redux: [], // Redux 相關
	config: [], // 配置文件
};

// 按類別分組顯示，並標註行數變更
```

### generateRelatedLinks()

自動收集和整理相關連結：

```javascript
const relatedLinks = {
	jiraTicket: `https://arcadie.atlassian.net/browse/${jiraTicket}`,
	relatedPRs: [], // 相關 PR
	designAssets: [], // Figma/設計稿
	apiDocumentation: [], // API 文件
	confluencePages: [], // Confluence 頁面
	testingLinks: [], // 測試環境連結
	deploymentLinks: [], // 部署相關連結
};

// 自動搜尋邏輯：
// 1. JIRA 票號中的描述和評論連結
// 2. 分支名稱模式匹配相關 PR
// 3. 代碼中的 TODO/FIXME 註釋連結
// 4. package.json 或配置文件中的文件連結
// 5. README 或文件中提及的連結
```

**相關連結範例格式：**

```markdown
## 相關連結

### 📋 JIRA 相關

- [CXPC-1860: 主要票號](https://arcadie.atlassian.net/browse/CXPC-1860)
- [CXPC-1850: 相關票號](https://arcadie.atlassian.net/browse/CXPC-1850)

### 🔗 相關 PR

- [PR #123: 前置 PR](https://dev.azure.com/ogglobal/FE-Engineering/_git/J1-APP-CODE/pullrequest/123)
- [PR #456: 相依 PR](https://dev.azure.com/ogglobal/FE-Engineering/_git/J1-APP-CODE/pullrequest/456)

### 🎨 設計資源

- [Figma 設計稿](https://figma.com/file/xxx)
- [UI 規範文件](https://confluence.company.com/ui-guidelines)

### 📚 API 文件

- [支付 API 文件](https://api-docs.company.com/payments)
- [用戶認證 API](https://api-docs.company.com/auth)

### 🔧 技術文件

- [Confluence 技術規範](https://confluence.company.com/tech-specs)
- [架構決策記錄](https://confluence.company.com/adr-123)

### 🌐 測試環境

- [Staging 環境](https://staging.company.com)
- [測試數據](https://test-data.company.com)
```

### generateImpactAnalysis()

列出具體影響的文件和模組：

```javascript
const impactAreas = {
	architecture: [], // 架構層面影響
	payments: [], // 支付模組
	ui: [], // 使用者介面
	data: [], // 資料流/狀態管理
};
```

### generatePerformanceSecurityAnalysis()

自動檢查：

- Bundle size 影響分析
- API 安全性檢查
- 記憶體洩漏風險評估
- 敏感資訊暴露檢查
- 效能瓶頸識別

### generateSpecificRollbackPlan()

根據變更類型生成具體回滾方案：

- Redux 狀態重置方法
- 功能開關操作指引
- 資料庫回滾步驟（如適用）

## ⚡ 整合 JIRA MCP 功能

### 自動獲取 JIRA 資訊

```javascript
const jiraInfo = await jira.getIssue(ticketNumber);
const jiraLinks = jiraInfo.description.match(/https:\/\/[^\s]+/g);

// 自動收集相關連結
const relatedLinks = await collectRelatedLinks(jiraInfo);

async function collectRelatedLinks(jiraInfo) {
	const links = {
		jira: [],
		pr: [],
		figma: [],
		api: [],
		confluence: [],
		testing: [],
	};

	// 1. 從 JIRA 描述和評論中提取連結
	const allText = jiraInfo.description + jiraInfo.comments.join(' ');

	// Figma 連結
	links.figma = allText.match(/https:\/\/(?:www\.)?figma\.com\/[^\s]+/g) || [];

	// Azure DevOps PR 連結
	links.pr = allText.match(/https:\/\/dev\.azure\.com\/[^\/]+\/[^\/]+\/_git\/[^\/]+\/pullrequest\/\d+/g) || [];

	// API 文件連結
	links.api = allText.match(/https:\/\/[^\/]*api[^\/]*\..*?\/[^\s]+/g) || [];

	// Confluence 連結
	links.confluence = allText.match(/https:\/\/[^\/]*confluence[^\/]*\..*?\/[^\s]+/g) || [];

	// 2. 從代碼註釋中搜尋連結
	const codeLinks = await searchCodeForLinks();
	links.pr.push(...codeLinks.pr);
	links.api.push(...codeLinks.api);

	// 3. 從相關 JIRA 票號中搜尋
	const relatedJiraTickets = await findRelatedJiraTickets(jiraInfo);
	links.jira.push(...relatedJiraTickets);

	return deduplicateLinks(links);
}

async function searchCodeForLinks() {
	// 搜尋代碼中的 TODO、FIXME 註釋
	const todoComments = await executeCommand('grep -r "TODO\\|FIXME" src/ --include="*.js" -n');

	// 提取註釋中的連結
	const linkRegex = /https?:\/\/[^\s\)]+/g;
	const foundLinks = todoComments.match(linkRegex) || [];

	return categorizeLinks(foundLinks);
}
```

### 自動更新 JIRA 狀態

PR 創建後自動：

- 更新 JIRA 狀態為 "In Review"
- 添加 PR 連結到 JIRA 註釋
- 通知相關 assignee 和 reviewer

## 🎯 PR 標題生成規則

根據 JIRA 票號和變更類型自動生成：

```
${jiraTicket}: ${commitType}(${scope}): ${description}

例如：
CXPC-1860: refactor(payment): enhance GoTransHistory data flow and add project development guidelines
```

### 自動判斷 commitType:

- `feat`: 新增功能
- `fix`: 錯誤修復
- `refactor`: 重構優化
- `docs`: 文件更新
- `style`: 樣式調整
- `perf`: 效能優化

## 📊 代碼變更行數檢查

自動計算並分類：

```bash
# 總行數
git diff azure/master --numstat | awk '{sum+=$1+$2} END {print sum}'

# 排除文件類型
git diff azure/master --numstat | grep -v -E '\.(md|json|lock)$' | awk '{sum+=$1+$2} END {print sum}'
```

**檢查邏輯**:

- 超過 150-180 行：⚠️ 提醒分拆 PR
- Asset/文件不計入：✅ 正常範圍
- 實際程式碼行數：重點關注

## 🔄 完整執行流程

### Phase 1: 預覽生成

當用戶輸入「幫我創建 PR」：

1. **前置檢查** ✅

   - ✅ 檢查 master 合併狀態
   - ✅ 提取 JIRA 票號
   - ✅ 驗證代碼品質

2. **資訊收集** 📊

   - ✅ 獲取 JIRA 詳情
   - ✅ 收集相關連結 (PR/Figma/API doc)
   - ✅ 分析代碼變更
   - ✅ 計算影響範圍

3. **內容生成** 📝

   - ✅ 生成 PR 標題
   - ✅ 填充 PR 模板
   - ✅ 檢查清單狀態

4. **預覽展示** 👀
   - 📋 顯示完整 PR 內容
   - 🤔 列出需確認項目
   - ⏳ 等待用戶確認

### Phase 2: 確認創建

用戶確認後：

1. 產生 PR 內容 markdown 檔案（如 `pr-body.md`）。
2. 執行 `az repos pr create --source-branch <feature-branch> --target-branch <target-branch> --title "..." --description "$(cat pr-body.md)"` 建立 PR。
3. PR 建立成功後，立即自動刪除 `pr-body.md`，避免檔案殘留。

#### 範例腳本

```bash
cat > pr-body.md <<EOF
# ...你的 PR 內容 markdown...
EOF

az repos pr create --source-branch <feature-branch> --target-branch <target-branch> --title "..." --description "$(cat pr-body.md)"

# 強制刪除臨時文件（使用 -f 參數確保刪除）
rm -f pr-body.md

# 驗證文件已被刪除
if [ -f pr-body.md ]; then
    echo "⚠️ Warning: pr-body.md still exists, forcing removal..."
    rm -f pr-body.md
fi
```

## 🛠️ 錯誤處理

### 常見情況處理

- **JIRA 票號不存在**: 提示檢查分支命名規則
- **master 未合併**: 停止並提供合併指令
- **代碼檢查失敗**: 列出具體問題清單
- **PR 創建失敗**: 檢查權限和分支狀態
- **用戶取消**: 保存草稿，支援後續修改

### 用戶確認選項

```
✅ 確認創建 PR
📝 需要修改內容
❌ 取消此次操作
💾 保存草稿供後續使用
```

## 💡 使用範例

### 第一階段：生成預覽

用戶說：`"幫我創建 PR"`

系統回應：

````
## 📋 PR 內容預覽

## JIRA票號
[CXPC-1860](https://arcadie.atlassian.net/browse/CXPC-1860)

**票號摘要**: 優化支付流程和數據處理邏輯
**狀態**: In Progress
**指派給**: Albert Chen
**優先級**: High

## 相關連結

### 📋 JIRA 相關
- [CXPC-1860: 主要票號](https://arcadie.atlassian.net/browse/CXPC-1860)
- [CXPC-1850: 前置需求](https://arcadie.atlassian.net/browse/CXPC-1850)

### 🎨 設計資源
- [支付流程 Figma 設計稿](https://figma.com/file/abc123/payment-flow)
- [UI 組件庫規範](https://figma.com/file/def456/ui-components)

### 📚 API 文件
- [支付 API v2 文件](https://api-docs.company.com/payments/v2)
- [錢包 API 規範](https://api-docs.company.com/wallet)

### 🔧 技術文件
- [支付系統架構](https://confluence.company.com/payments-architecture)

[其餘 PR 內容...]

## 🤔 需要你確認的項目：
1. 代碼行數: 142行 (✅ 符合建議範圍)
2. PR 標題: CXPC-1860: refactor(payment): enhance GoTransHistory data flow
3. Target Branch: master
4. 相關連結: 自動找到相關連結(如果有符合建議範圍)

**確認後請告訴我是否要創建此 PR！** 🚀
`階段：確認創建
用戶回出"`

系統執行：
- ✅ 創建 Azure DevOps PR
- ✅ 更新 JIRA 狀態
- ✅ 提供 PR 連結

## 🎛️ 自訂選項

### 支援的確認指令
- `"確認創建"` / `"送出"` / `"創建 PR"` → 執行創建
- `"修改標題: [新標題]"` → 更新標題後創建
- `"修改內容"` → 進入編輯模式
- `"取消"` / `"不要"` → 取消操作

### 進階功能
- 支援草稿模式 (`draft: true`)
- 自動指派 reviewer
- 標籤自動添加
- Milestone 關聯

## PR 自動化流程

1. 依照需求產生 PR 內容（建議先寫成 markdown 檔案，如 `pr-body.md`）。
2. 用 `az repos pr create --source-branch <feature-branch> --target-branch <target-branch> --title "..." --description "$(cat pr-body.md)"` 建立 PR，確保內容格式正確。
3. **強制刪除臨時文件**：PR 建立後，無論成功與否都必須刪除 `pr-body.md`，避免檔案殘留在專案。

### 範例腳本
```bash
cat > pr-body.md <<EOF
# ...你的 PR 內容 markdown...
EOF

az repos pr create --source-branch <feature-branch> --target-branch <target-branch> --title "..." --description "$(cat pr-body.md)"

# 強制刪除臨時文件（使用 -f 參數確保刪除）
rm -f pr-body.md

# 驗證文件已被刪除
if [ -f pr-body.md ]; then
    echo "⚠️ Warning: pr-body.md still exists, forcing removal..."
    rm -f pr-body.md
fi
```

### 重要提醒
- **必須使用 `rm -f pr-body.md`** 而不是 `rm pr-body.md`，確保即使文件不存在也不會報錯
- **PR 創建後立即刪除**：無論 PR 創建是否成功，都必須刪除臨時文件
- **添加驗證步驟**：確保臨時文件確實被刪除，避免檔案殘留`

參考專案規範 @README.md 和 PR 生成規則 @pr-generate.mdc。
````

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brand001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brand001)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
