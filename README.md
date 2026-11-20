# 📚 Build with Gemini (Track 3) 全方位開發與實戰指南手冊

> 本儲存庫為 **Google Cloud · Build with Gemini World Tour (Track 3: Agent-First Apps)** 專案之全週期完整教學技術手冊，彙整了在開發以 **Gemini 3.8 Flash**、**Vertex AI Agent Platform (ADK)**、**Memory Bank**、**Firestore**、**Cloud Run** 與多模態生成模型（**Imagen & Omni Flash**）為基礎之企業級 Agent 應用程式時的所有技能、工具、架構設計與踩坑排解。

---

## 🧭 目錄導覽 (Table of Contents)

1. [📖 01. 雲端架構與 Google Cloud 服務解析](01-architecture-and-cloud-services.md)
   - 總體系統架構圖 (Web UI ➔ Cloud Run Proxy ➔ Agent Platform ➔ Memory Bank & DB & Sandbox)
   - 各 Google Cloud 託管服務詳細職責表
   - IAM 服務帳戶與權限綁定清單 (`roles/datastore.user`, `roles/storage.objectAdmin`, `roles/aiplatform.user`)
   - 區域與全域端點架構 (`us-east1` vs `global`)

2. [🛠️ 02. Skills 與 MCP 資源清單](02-skills-and-mcp-inventory.md)
   - 11 項 Antigravity 內建與自訂工作流 Skills 深入解析
   - Google 開發者知識庫 (`google-developer-knowledge`) 與 Firebase MCP 伺服器
   - 關鍵相依套件庫一覽表 (`google-adk`, `agents-cli`, `google-genai`, `a2a-sdk`, `playwright`)

3. [🚀 03. 從零到一端到端實戰教學](03-end-to-end-tutorial.md)
   - 步驟 1 至步驟 14 完整實作演練
   - 包含專案鷹架建立、Firestore CRUD 工具、Memory Bank 長期記憶整合、Agent Engine 程式碼沙盒、Imagen 心智圖生成、Google Omni Interactions API 影音生成、A2UI 結構化介面回呼、FastAPI A2A 代理、Cloud Run 部署、Playwright 自動化展示錄製與 GitHub 開源發布。

4. [⚠️ 04. 常見疑難排解與踩坑記錄](04-troubleshooting-and-pitfalls.md)
   - `ValueError: Protocol message AgentCard has no "supportedInterfaces" field` 根本原因與處置
   - `gemini-omni-flash-preview` 影音模型區域限制與專屬 Interactions API 呼叫技巧
   - 雲端特工服務帳戶 403 PERMISSION_DENIED 授權修正
   - Cloud Run 存取 Agent Runtime 403 Forbidden 排除
   - 本地 ADK Web Playground Memory Bank 連線指令參數
   - Git Push 憑證輔助程式自動化配置

5. [⚡ 05. 實用指令速查手冊 (CLI Cheatsheet)](05-cli-cheatsheet.md)
   - GCP & IAM 常用指令
   - ADK & `agents-cli` 本地啟動與雲端部署
   - FastAPI 前端本機除錯與 Cloud Run 發布
   - `ffmpeg` 高畫質超輕量循環 GIF 轉檔語法
   - GitHub CLI (gh) 裝置碼認證與儲存庫管理

---

## 🎯 核心實戰專案範例：MeetMind

本手冊各章節之技術實踐均奠基於完整實作的特工專案 **MeetMind (會議智慧與知識庫特工)**：
- **核心特工程式碼儲存庫**：[`hongyue1213/buildwithgemini-meetmind`](https://github.com/hongyue1213/buildwithgemini-meetmind)
- **技術棧特色**：
  - ADK + Gemini 3.8 Flash
  - Vertex AI Memory Bank 跨工作階段記憶
  - Cloud Firestore 會議記錄 NoSQL 知識庫
  - Agent Engine Sandbox 隔離 Python 數據運算
  - Gemini 3.1 Flash Lite 視覺心智圖生成 + GCS 公開串流上傳
  - Gemini Omni Flash 全模態 MP4 動畫精華短片生成 (Interactions API)
  - A2UI v0.8 卡片動態渲染與自訂主題風格聊天介面
  - FastAPI A2A 協定代理 + Cloud Run 無伺服器部署

---

## 📄 授權條款 (License)
Apache-2.0
