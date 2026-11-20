# 02. Skills 與 MCP 資源清單 (Skills & MCP Catalog)

本章節系統化收錄本次專案全週期所具備與運用的 **Antigravity Skills**、**Model Context Protocol (MCP) 伺服器** 以及關鍵程式庫工具。

> [!TIP]
> 本儲存庫已完整備份收錄所有 Skill 的**原始指導手冊 (`SKILL.md`)、自動化腳本（如 Playwright 錄影、GitHub 發布腳本）與程式碼模板（如 FastAPI 代理、A2UI Utils）**，存放於本儲存庫的 [`skills/`](skills/) 與 [`mcp/`](mcp/) 目錄下。

---

## 🛠️ Antigravity Skills 完整盤點與原始檔案

Skills 是 Antigravity 擴充 AI 能力的工作流套件，每個 Skill 均封裝了特定領域的最佳實踐指令、自動化腳本與參考範本：

### 1. 專案規劃與環境診斷
- **`troubleshoot-lab-setup`** 📂 [查看目錄](skills/workshop-skills/troubleshoot-lab-setup/) ｜ 📄 [SKILL.md](skills/workshop-skills/troubleshoot-lab-setup/SKILL.md)
  - **時機**：工作站剛啟動或遇到 403 / PERMISSION_DENIED 權限錯誤時。
  - **功能**：自動檢查 GCP 登入狀態、專案 ID 設定、`gcloud auth application-default login`、Vertex AI API 啟用狀態，以及帳號是否具備 `roles/aiplatform.user` 權限。
- **`pick-your-agent-project`** 📂 [查看目錄](skills/workshop-skills/pick-your-agent-project/) ｜ 📄 [SKILL.md](skills/workshop-skills/pick-your-agent-project/SKILL.md)
  - **時機**：專案開發之初進行架構規劃與發想時。
  - **功能**：引導設計具體 Agent 應用場景，對齊 Google 工具覆蓋度（Memory、Tools、Storage、UI、Image Gen、Sandbox），並產出結構化的 `project_brief.md`。

### 2. 專案鷹架與特工編程
- **`google-agents-cli-scaffold`** 📂 [查看目錄](skills/google-agents-cli-skills/google-agents-cli-scaffold/) ｜ 📄 [SKILL.md](skills/google-agents-cli-skills/google-agents-cli-scaffold/SKILL.md)
  - **時機**：初始化 ADK 專案結構時。
  - **功能**：執行 `agents-cli scaffold create`，產生包含 `agents-cli-manifest.yaml`、`app/agent.py`、虛擬環境與設定檔的標準專案目錄。
- **`google-agents-cli-adk-code`** 📂 [查看目錄](skills/google-agents-cli-skills/google-agents-cli-adk-code/) ｜ 📄 [SKILL.md](skills/google-agents-cli-skills/google-agents-cli-adk-code/SKILL.md)
  - **時機**：撰寫 Agent 程式邏輯、自訂 Tool、狀態管理（State）與回呼函式（Callbacks）時。
  - **功能**：提供 Google ADK（Agent Development Kit）的最佳程式碼模式，涵蓋 `@tool` 裝飾器、`ToolContext`、`after_model_callback` 與非同步執行。
- **`google-agents-cli-workflow`** 📂 [查看目錄](skills/google-agents-cli-skills/google-agents-cli-workflow/) ｜ 📄 [SKILL.md](skills/google-agents-cli-skills/google-agents-cli-workflow/SKILL.md)
  - **時機**：掌握 ADK 完整生命週期與編程規範時。

### 3. 進階雲端能力整合
- **`setup-memory-bank` (Memory Bank Setup)** 📂 [查看目錄](skills/workshop-skills/setup-memory-bank/) ｜ 📄 [SKILL.md](skills/workshop-skills/setup-memory-bank/SKILL.md)
  - **時機**：需要特工具備跨對話階段長期記憶能力時。
  - **功能**：指導在 Vertex AI 上建立託管的 Memory Bank 執行個體，並將其透過 `PreloadMemoryTool` 與 `generate_memories_callback` 注入至 ADK Agent。
- **`build-rag` (RAG Engine Setup)** 📂 [查看目錄](skills/workshop-skills/build-rag/) ｜ 📄 [SKILL.md](skills/workshop-skills/build-rag/SKILL.md)
  - **時機**：需要特工掛載大規模文件檢索、建立企業知識庫時。
  - **功能**：引導建置 Serverless Vertex AI RAG Engine Corpus，配置文件解析提示詞，並封裝為純 Function Tool。
- **`enable-a2ui` (Agent-to-UI)** 📂 [查看目錄](skills/workshop-skills/enable-a2ui/) ｜ 📄 [SKILL.md](skills/workshop-skills/enable-a2ui/SKILL.md) ｜ 🐍 [a2ui_utils.py 模板](skills/workshop-skills/enable-a2ui/template/a2ui_utils.py)
  - **時機**：需要特工輸出豐富結構化介面（卡片、圖表、排版）而非單純文字時。
  - **功能**：引入 A2UI v0.8 規格與 `a2ui_utils.py`，透過攔截模型回呼將特工生成的 JSON 轉換為前端可渲染的 A2UI Data Parts。

### 4. 前端建置與雲端部署
- **`build-agent-frontend`** 📂 [查看目錄](skills/workshop-skills/build-agent-frontend/) ｜ 📄 [SKILL.md](skills/workshop-skills/build-agent-frontend/SKILL.md) ｜ 🌐 [前端與 Proxy 完整模板](skills/workshop-skills/build-agent-frontend/template/)
  - **時機**：為特工打造 Web 聊天介面並連接雲端 Agent 時。
  - **功能**：提供輕量 FastAPI 代理伺服器範本，透過 **A2A Protocol (Agent-to-Agent)** 與部署於 Agent Platform 的後端安全通訊，內建 A2UI 渲染器。
- **`google-agents-cli-deploy`** 📂 [查看目錄](skills/google-agents-cli-skills/google-agents-cli-deploy/) ｜ 📄 [SKILL.md](skills/google-agents-cli-skills/google-agents-cli-deploy/SKILL.md)
  - **時機**：將特工部署上雲端生產環境時。
  - **功能**：執行 `agents-cli deploy`，自動打包程式碼上傳至 Vertex AI Agent Runtime，配置環境變數與推論設定。
- **`google-agents-cli-eval`** 📂 [查看目錄](skills/google-agents-cli-skills/google-agents-cli-eval/) ｜ 📄 [SKILL.md](skills/google-agents-cli-skills/google-agents-cli-eval/SKILL.md)
  - **時機**：對 Agent 進行品質評估與測試評分時。
- **`google-agents-cli-observability`** 📂 [查看目錄](skills/google-agents-cli-skills/google-agents-cli-observability/) ｜ 📄 [SKILL.md](skills/google-agents-cli-skills/google-agents-cli-observability/SKILL.md)
  - **時機**：配置 Cloud Trace、Log 記錄與 Agent Analytics 監控時。
- **`google-agents-cli-publish`** 📂 [查看目錄](skills/google-agents-cli-skills/google-agents-cli-publish/) ｜ 📄 [SKILL.md](skills/google-agents-cli-skills/google-agents-cli-publish/SKILL.md)
  - **時機**：將特工註冊發布至 Gemini Enterprise 或 Agent Registry 時。

### 5. 展示錄製與開源分享
- **`record-demo`** 📂 [查看目錄](skills/workshop-skills/record-demo/) ｜ 📄 [SKILL.md](skills/workshop-skills/record-demo/SKILL.md) ｜ 📜 [record-agent.js 錄影腳本](skills/workshop-skills/record-demo/record-agent.js)
  - **時機**：專案開發完成，需要錄製展示影片時。
  - **功能**：使用 Playwright 驅動無周邊瀏覽器（Headless Chromium），自動在前端輸入多輪情境提示詞，並錄製為高品質 `.webm` 影片。
- **`publish-to-github`** 📂 [查看目錄](skills/workshop-skills/publish-to-github/) ｜ 📄 [SKILL.md](skills/workshop-skills/publish-to-github/SKILL.md) ｜ 🐚 [publish.sh 發布腳本](skills/workshop-skills/publish-to-github/publish.sh)
  - **時機**：將實驗室專案永久保存至個人 GitHub 帳號並填寫完賽表單時。
  - **功能**：利用 GitHub CLI Device Code 流程進行個人身分授權，自動生成 `.gitignore`、進行秘密資訊掃描（Secret Scan）、建立 Public 儲存庫並產生預先填妥的 Google Form 連結。

---

## 🔌 Model Context Protocol (MCP) 伺服器與 Schema 備份

本環境內建兩個強大的 MCP 伺服器，允許特工直接查詢 Google 官方文件庫與 Firebase 資源。所有 Tool Schema JSON 均已備份於 [`mcp/`](mcp/)：

### 1. `google-developer-knowledge` (開發者知識庫) 📂 [查看 Schema 目錄](mcp/google-developer-knowledge/)
提供 Google 最新產品與 SDK 的權威技術文件查詢：
- **`search_documents`** ([schema](mcp/google-developer-knowledge/search_documents.json))：在 Google 開發者知識庫中搜尋相關主題文件。
- **`answer_query`** ([schema](mcp/google-developer-knowledge/answer_query.json))：針對技術問題進行深入分析並產出官方建議答覆。
- **`get_documents`** ([schema](mcp/google-developer-knowledge/get_documents.json))：直接獲取指定技術文檔的原始全文。
- **實戰案例**：在開發 `generate_meeting_video` 時，透過此 MCP 查明 Omni 影音模型（`gemini-omni-flash-preview`）不支援標準的 `generate_content`，必須透過全域區域的 **Interactions API** (`client.interactions.create`) 調用。

### 2. `firebase` (Firebase 雲端整合工具) 📂 [查看 Schema 目錄](mcp/firebase/)
支援專案與應用程式層級的即時操作：
- **`firebase_get_project` / `firebase_list_projects`**：檢查與取得 Firebase 專案設定。
- **`firebase_get_security_rules`** ([schema](mcp/firebase/firebase_get_security_rules.json))：檢視 Firestore 與 Storage 的安全規則。
- **`firebase_get_sdk_config`** ([schema](mcp/firebase/firebase_get_sdk_config.json))：產出 Web 與行動端 SDK 初始設定值。
- **`firebase_deploy` / `firebase_init`**：初始化與部署 Firebase 資源。

---

## 📦 關鍵核心程式庫清單 (Core Libraries)

| 套件名稱 | 版本/用途 | 核心功能 |
| :--- | :--- | :--- |
| **`google-adk`** | Agent Development Kit | Google 官方 Agent 架構，負責狀態、工具裝飾、回呼與特工編排。 |
| **`agents-cli`** | Google Agent CLI 工具 | 負責專案初始化、評估、打包、部署至 Vertex AI Agent Runtime。 |
| **`google-genai`** | 0.8+ (New Unified SDK) | Google 全新世代 GenAI SDK，支援 Gemini 3.x、Imagen 與 Interactions API。 |
| **`google-cloud-firestore`** | 2.19+ | 原生連線 Google Cloud Firestore，執行即時文件存取與檢索。 |
| **`google-cloud-storage`** | 2.18+ | 儲存貯體物件串流上傳、中繼資料管理與公開存取控制。 |
| **`a2a-sdk`** | 0.3+ | Google Agent-to-Agent 通訊協定，處理瀏覽器與代理伺服器之間的串流事件。 |
| **`fastapi` / `uvicorn`** | 0.115+ | 輕量、高效能的非同步 Web 伺服器，提供 A2A 代理端點。 |
| **`playwright`** | 1.49+ | 自動化端對端瀏覽器測試與高畫質展示錄影工具。 |
