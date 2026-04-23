## docai-backend-rails

> **目的**：本 FAQ 文檔旨在為新加入「AI ENGLISH」項目的後端工程師提供快速指引，解答在熟悉項目過程中可能遇到的常見問題。對於更詳細的信息，請參考本文檔中鏈接的其他相關內部文檔。

# AI ENGLISH 後端項目 - 新工程師常見問題解答 (FAQ)

**目的**：本 FAQ 文檔旨在為新加入「AI ENGLISH」項目的後端工程師提供快速指引，解答在熟悉項目過程中可能遇到的常見問題。對於更詳細的信息，請參考本文檔中鏈接的其他相關內部文檔。

---

**一、項目概覽與核心業務**

**Q1: 「AI ENGLISH」項目的核心定位和主要功能是什麼？**
A: 「AI ENGLISH」是一個專注於英語學習的平台，通過多種類型的互動練習和 AI 自動評分，幫助用戶提升英語技能。核心功能圍繞 `GeneralUser` (用戶)、`EssayAssignment` (作業/練習任務) 和 `EssayGrading` (評分/練習記錄) 展開。
*   詳細信息請參閱：`[.cursor/rules/100-project-overview.mdc](mdc:.cursor/rules/100-project-overview.mdc)` 的 "1. 項目核心定位" 和 "2. 核心功能模塊" 部分。

**Q2: AI English 支持哪些英語學習功能？**
A: 主要包括作文練習 (`essay`)、閱讀理解 (`comprehension`)、口說短文 (`speaking_essay`)、口說對話 (`speaking_conversation`)、句子構建 (`sentence_builder`) 和發音練習 (`speaking_pronunciation`)。這些定義在 `[app/models/general_user.rb](mdc:app/models/general_user.rb)` 的 `VALID_AI_ENGLISH_FEATURES` 中。

**Q3: 誰是這個項目的主要用戶？項目是否支持學校或機構使用？**
A: 主要用戶是英語學習者。系統通過 `GeneralUser` 模型中的 `meta` 字段區分「AI English 用戶」並管理其功能權限。項目也支持教育機構場景，用戶可以關聯到 `School` (學校) 和 `SchoolAcademicYear` (學年)。
*   用戶角色和權限詳見：`[.cursor/rules/100-project-overview.mdc](mdc:.cursor/rules/100-project-overview.mdc)` 的 "GeneralUser (用戶)" 部分。
*   學校相關功能可參考 `[app/models/school.rb](mdc:app/models/school.rb)` (假設存在) 和 `[app/services/schools/](mdc:app/services/schools)` 目錄下的服務。

**Q4: 項目的主要業務流程是怎樣的？例如學生如何完成一次練習？**
A: 簡要流程：
    1.  教師/管理員創建 `EssayAssignment` (作業)。
    2.  學生完成作業並提交，系統創建 `EssayGrading` (評分記錄)。
    3.  AI 根據作業類型自動評分 (調用 Dify Workflow 或發音 API)。
    4.  學生和教師查看評分結果和報告。
*   詳細業務流程請參閱：`[.cursor/rules/100-project-overview.mdc](mdc:.cursor/rules/100-project-overview.mdc)` 的 "3. 主要業務流程" 部分。
*   數據如何在系統中流動，可參考：`[.cursor/rules/400-architecture.mdc](mdc:.cursor/rules/400-architecture.mdc)` 的 "5. 關鍵功能的數據流示例" 部分。

---

**二、技術棧與關鍵組件**

**Q5: 項目使用的主要技術框架是什麼？**
A: 核心框架是 Ruby on Rails 7。
*   關於項目中 Rails 的具體應用，請參閱：`[.cursor/rules/300-rails-development-guide.mdc](mdc:.cursor/rules/300-rails-development-guide.mdc)`。

**Q6: 項目依賴哪些關鍵的外部 AI 服務？**
A: 主要依賴兩個外部服務：
    1.  **Dify Platform API** (`https://aienglish-dify.docai.net/v1/workflows/run`): 用於多數核心 AI 功能，如作文批改、句子示例生成等。相關服務封裝在 `[app/services/dify_service.rb](mdc:app/services/dify_service.rb)`。
    2.  **發音評測 API** (`https://pronunciation.m2mda.com/pinyin`): 用於發音練習的評分。
*   更多外部依賴信息，請參閱：`[.cursor/rules/100-project-overview.mdc](mdc:.cursor/rules/100-project-overview.mdc)` 的 "4. 關鍵技術組件與外部依賴" 部分。

**Q7: 項目中使用了哪些重要的 Gems？它們分別起了什麼作用？**
A: 項目使用了許多 Gems，例如：
    *   `devise` & `devise-jwt`: 用戶認證與 API Token。
    *   `sidekiq`: 後台異步任務處理。
    *   `rswag`: API 文檔 (Swagger/OpenAPI)。
    *   `ros-apartment`: 數據庫多租戶。
    *   `prawn` / `pdfkit`: PDF 報告生成。
    *   `kaminari` / `api-pagination`: 數據分頁。
*   詳細的 Gem 列表及其作用，請參閱 `[Gemfile](mdc:Gemfile)` 和 `[.cursor/rules/300-rails-development-guide.mdc](mdc:.cursor/rules/300-rails-development-guide.mdc)` 的 "3. 關鍵 Gems 及其在項目中的角色與配置指導" 部分。

**Q8: 什麼是多租戶 (Multi-tenancy)？AI ENGLISH 項目是如何實現的？**
A: 多租戶允許不同客戶（例如不同學校）的數據在共享的應用程序和數據庫基礎設施中保持邏輯上的隔離。AI ENGLISH 項目使用 `ros-apartment` Gem 來實現，可能採用 Schema-per-tenant 的方式。
*   **極其重要**：理解多租戶對數據訪問、數據庫遷移和後台任務的影響至關重要。
*   詳細說明請參閱：`[.cursor/rules/300-rails-development-guide.mdc](mdc:.cursor/rules/300-rails-development-guide.mdc)` 的 "Apartment (`ros-apartment`, `ros-apartment-sidekiq`)" 部分，以及 `[.cursor/rules/400-architecture.mdc](mdc:.cursor/rules/400-architecture.mdc)` 的 "6. 架構上的重要考量" 中關於多租戶的說明。

**Q9: 項目如何處理後台任務？**
A: 項目使用 `sidekiq` Gem 進行後台異步任務處理，例如 `SentenceBuilderExampleJob` 用於為句子構建作業生成例句。
*   詳見：`[.cursor/rules/300-rails-development-guide.mdc](mdc:.cursor/rules/300-rails-development-guide.mdc)` 的 "Sidekiq" 和 "後台處理 (Sidekiq)" 部分。

---

**三、架構與代碼組織**

**Q10: 項目的整體架構風格是怎樣的？**
A: 項目是一個基於 Ruby on Rails 7 的單體應用，內部通過服務對象和後台任務實現模塊化，遵循 MVC 模式，並採用了多租戶架構。
*   詳細描述請參閱：`[.cursor/rules/400-architecture.mdc](mdc:.cursor/rules/400-architecture.mdc)` 的 "2. 整體架構風格" 部分。

**Q11: AI English 功能的核心代碼主要在哪裡？**
A:
    *   **模型**: `[app/models/general_user.rb](mdc:app/models/general_user.rb)`, `[app/models/essay_assignment.rb](mdc:app/models/essay_assignment.rb)`, `[app/models/essay_grading.rb](mdc:app/models/essay_grading.rb)`。
    *   **控制器**: `[app/controllers/api/v1/essay_assignments_controller.rb](mdc:app/controllers/api/v1/essay_assignments_controller.rb)`, `[app/controllers/api/v1/essay_gradings_controller.rb](mdc:app/controllers/api/v1/essay_gradings_controller.rb)`。
    *   **服務**: `[app/services/](mdc:app/services)` 目錄下與 AI、Dify、Essay Grading、Sentence Builder 相關的服務。
    *   **路由**: `[config/routes.rb](mdc:config/routes.rb)` 中 `api/v1` 命名空間下的路由。
*   更詳細的目錄和組件職責說明，請參閱：`[.cursor/rules/400-architecture.mdc](mdc:.cursor/rules/400-architecture.mdc)` 的 "3. 核心目錄結構與 AI English 重點" 和 "4. AI English 核心組件及其職責" 部分。

**Q12: 什麼是服務對象 (Service Objects)？項目中是如何使用的？**
A: 服務對象用於封裝複雜的業務邏輯、涉及多個模型或外部服務調用的操作，以保持控制器的輕量化。項目中 `app/services/` 目錄下包含了諸如 `EssayGradingService`, `SentenceBuilderExampleService` 等服務對象。
*   關於服務對象的建議，請參閱：`[.cursor/rules/200-coding-style.mdc](mdc:.cursor/rules/200-coding-style.mdc)` 的 "8. Rails 最佳實踐" 中關於「瘦控制器，胖模型/服務」的討論。

**Q13: 項目中存在一些看起來與 AI English 不直接相關的舊代碼嗎？如何識別和處理？**
A: 是的，項目中存在大量與 "Documents", "Tags", "Storage", "FormSchema", "Projects", "Chatbots" 等主題相關的路由和代碼，這些很可能是待清理的舊功能。
*   識別來源主要是 `[config/routes.rb](mdc:config/routes.rb)` 和 `app/` 目錄下的相關組件。
*   處理建議是明確依賴關係，逐步剝離無關代碼，並充分測試。
*   詳細討論請參閱：`[.cursor/rules/100-project-overview.mdc](mdc:.cursor/rules/100-project-overview.mdc)` 的 "6. 關於遺留代碼的識別與處理建議" 和 `[.cursor/rules/400-architecture.mdc](mdc:.cursor/rules/400-architecture.mdc)` 的 "7. 遺留模塊的識別與管理" 部分。

---

**四、開發流程與規範**

**Q14: 項目有統一的編碼風格規範嗎？在哪裡可以找到？**
A: 是的，項目推薦遵循一套編碼風格規範，以提高代碼質量和團隊協作效率。
*   詳細規範請參閱：`[.cursor/rules/200-coding-style.mdc](mdc:.cursor/rules/200-coding-style.mdc)`。
*   強烈建議引入並配置 `RuboCop` 作為代碼風格檢查工具。

**Q15: API 設計應遵循哪些原則？如何編寫 API 文檔？**
A: API 設計應遵循 RESTful 原則，使用 `Api::V1` 進行版本控制，並返回標準的 HTTP 狀態碼和結構化的 JSON 響應。項目使用 `RSwag` (Swagger/OpenAPI) 生成 API 文檔。
*   API 設計指南詳見：`[.cursor/rules/300-rails-development-guide.mdc](mdc:.cursor/rules/300-rails-development-guide.mdc)` 的 "4. API 設計與開發" 和 "RSwag" 部分。
*   編碼風格中也有 API 設計的建議：`[.cursor/rules/200-coding-style.mdc](mdc:.cursor/rules/200-coding-style.mdc)` 的 "API 設計" 部分。

**Q16: 如何處理數據庫遷移，尤其是在多租戶環境下？**
A: 數據庫遷移應保持可逆。在多租戶 (`ros-apartment`) 環境下，需要注意 `rake db:migrate` (針對 public schema) 和 `rake apartment:migrate` (針對所有租戶 schemas) 的區別。
*   詳細指南請參閱：`[.cursor/rules/300-rails-development-guide.mdc](mdc:.cursor/rules/300-rails-development-guide.mdc)` 的 "數據庫與 Active Record 實踐" 中關於數據遷移和多租戶的部分。

**Q17: 項目推薦的測試策略是怎樣的？**
A: 鼓勵為核心業務邏輯編寫清晰、有效的單元測試和集成測試，並關注測試覆蓋率。
*   相關建議請參閱：`[.cursor/rules/200-coding-style.mdc](mdc:.cursor/rules/200-coding-style.mdc)` 和 `[.cursor/rules/300-rails-development-guide.mdc](mdc:.cursor/rules/300-rails-development-guide.mdc)` 中的測試部分。

---

**五、環境配置與調試**

**Q18: 如何配置本地開發環境？敏感信息（如 API Key）如何管理？**
A: 項目使用 `dotenv` Gem 管理環境變量。敏感信息應存儲在 `.env` 文件中（該文件不應提交到版本控制），並通過 `ENV['YOUR_KEY']` 的方式在代碼中訪問。
*   配置管理指南請參閱：`[.cursor/rules/300-rails-development-guide.mdc](mdc:.cursor/rules/300-rails-development-guide.mdc)` 的 "7. 配置與環境管理" 部分。

**Q19: 常用的調試工具有哪些？**
A: Gemfile 中包含了 `debug`, `pry-byebug`, `better_errors` 等開發和調試工具。Rails console (`bin/rails c`) 也是一個強大的調試和交互工具。

---

希望這份 FAQ 能幫助您更快地融入「AI ENGLISH」項目！如果您有其他問題，請隨時提出。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/docai-app) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
