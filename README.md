# 📚 Build with Gemini (Track 3) 全方位開發與系統架構實戰手冊

> 本儲存庫為 **Google Cloud · Build with Gemini World Tour (Track 3: Agent-First Apps)** 專案之全週期技術手冊與架構白皮書。系統化彙整以 **Gemini 3.8 Flash**、**Vertex AI Agent Platform (ADK)**、**Memory Bank**、**Firestore**、**Cloud Run** 與多模態生成模型（**Imagen & Omni Flash**）為基礎之企業級 Agent 應用程式的建置原理、目標價值、優缺點評估與完整實作手冊。

---

## 🏛️ 系統架構原理 (Architectural Principles)

整套架構遵循 **「分層解耦、無狀態代理、有狀態推論、外掛增強」** 的現代特工設計哲學，主要由五大核心層級構成：

```
[前端互動層]  Modern Web UI (Tailored Theme & Dialogue Layout)
      │      ▲
      │ A2UI │ (動態宣告式介面渲染)
      ▼      │
[協定代理層]  FastAPI Proxy (A2A Protocol / Cloud Run)
      │      ▲
      │ gRPC │ (串流雙向事件通訊)
      ▼      │
[大腦推論層]  Vertex AI Agent Runtime (ADK + Gemini 3.8 Flash)
      │
      ├─▶ [短期上下文] ─▶ ReAct 迴圈 (Thought ➔ Action ➔ Observation)
      ├─▶ [長期記憶庫] ─▶ Vertex AI Memory Bank (Preload 注入 / Session Hook 提煉)
      ├─▶ [程式執行器] ─▶ Agent Engine Sandbox (隔離容器執行 Python 數學/統計)
      ├─▶ [持久知識庫] ─▶ Cloud Firestore (NoSQL 結構化會議與待辦 CRUD)
      └─▶ [多模態生成] ─▶ Gemini Image & Omni Video ➔ GCS Public 串流直存
```

### 1. 推論引擎與工具迴圈（ReAct + Function Calling）
- **核心大腦**：採用推論速度極快、具備長上下文理解力的 `gemini-3.8-flash`。
- **動態執行（Function Calling）**：特工依據 Prompt 自主推論決策，能連續發起多階段 Tool 呼叫（如先透過 Firestore 檢索會議，再根據決策項目生成摘要圖表）。

### 2. 雙重記憶架構（Dual-Memory Mechanism）
- **工作記憶（Working Memory）**：由當前對話 Session 保持，維持數輪對話的即時上下文。
- **長期記憶（Long-Term Memory）**：透過 **Vertex AI Memory Bank**。在模型推論前透過 `PreloadMemoryTool` 注入使用者歷史偏好與團隊背景；對話結束後透過 `after_model_callback` 非同步提取新知識回存，實現「越用越聰明」。

### 3. 多模態串流管線（Multimodal Streaming Pipeline）
- 生成影像（Imagen / Flash-Lite-Image）或影片（Omni Flash Preview）時，特工在記憶體中處理二進位資料（Bytes），**不落磁碟**直接上傳至 Google Cloud Storage 公開儲存貯體，並同步登記至 ADK 的 Session Artifacts，兼顧展示速度與資產保存。

### 4. 介面解耦協議（A2A 與 A2UI）
- **A2A（Agent-to-Agent）**：提供後端特工與前端之間的標準化串流對話協定，前端無需關心後端是哪一個 LLM。
- **A2UI（Agent-to-User Interface）**：特工可以在回覆中夾帶宣告式 UI 規格（Card、Column、Image），由前端輕量渲染器轉成原生元件，擺脫傳統「純文字對話」的單調侷限。

---

## 🎯 達成目的與業務價值 (Core Objectives Achieved)

1. **破除資訊孤島與斷層**：將散落於語音對話、零碎文字的雜亂資訊，自動收斂為標準結構化知識庫（Firestore），隨問隨查。
2. **閉環行動化（Action-Oriented）**：特工不再只是聊天機器人，具備指派任務、更新狀態、精確計算進度（Sandbox）、繪製心智圖（Image Gen）、生成回顧影片（Omni Video）的**實體執行力**。
3. **消除模型幻覺（Grounded & Deterministic）**：會議事實依賴資料庫檢索（Grounding），日期差與統計數據強制交由 Python 沙盒運算，杜絕大模型算術胡謅。
4. **雲端企業級就緒（Production-Ready）**：藉助 Cloud Run 與 Vertex AI 代管平台，應用具備身分認證（IAM）、自動擴展與高可用性，達到可商用原型標準。

---

## ⚖️ 好處與潛在挑戰分析 (Pros & Cons Evaluation)

### 🌟 核心好處 (Key Advantages)
- **全 Google 原生生態閉環**：Vertex AI、Gemini、Cloud Run、Firestore、GCS 與 IAM 深度整合，架構統一且穩定。
- **高安全性與無金鑰安全**：所有元件通訊皆以 GCP IAM 服務帳戶身分運作，**無明文 API Key 外洩風險**；Python 程式碼在隔離的 Agent Engine 沙盒中運行，保障核心系統安全。
- **高度模組化擴展性**：ADK 採用 `@tool` 裝飾器設計，未來欲擴充新工具（如 Jira、Slack 或 Google Calendar），只需新增函式並掛載，無需重構核心。
- **豐富的人機互動（Rich UI）**：A2UI 技術讓聊天介面可直接渲染動態卡片、排版與串流影片，大幅降低使用者閱讀長篇純文字的認知負荷。
- **全代管無伺服器架構**：低流量時不佔用伺服器資源，突發流量時自動擴展，幾乎免去基礎設施維運負擔。

### ⚠️ 潛在挑戰與代價 (Trade-offs & Limitations)
- **高度廠商鎖定 (Vendor Lock-in)**：深度依賴 Vertex AI Agent Platform、ADK、Memory Bank 與 A2A 協定。若要遷移至其他雲端平台需重寫部分 Agent 骨架。
- **鏈路延遲疊加 (Cumulative Latency)**：複雜任務涉及「LLM 推論 ➔ 資料庫檢索 ➔ 多模態生成 ➔ GCS 上傳 ➔ A2UI 包裝」，累積等待時間較長，需仰賴前端載入狀態（如動態打字動畫）緩解焦慮。
- **多維度計費成本**：包含 LLM Token、Reasoning Engine 運行、Cloud Run、Firestore 讀寫與影音 API 費用，高併發環境需建立快取與用量配額機制。
- **新興協定迭代風險**：A2A 與 A2UI 處於高速演進階段，依賴套件升級時需注意版本與欄位相容性。

---

## 🧭 章節導覽 (Detailed Documentation)

| 章節 | 文檔名稱 | 涵蓋重點 |
| :--- | :--- | :--- |
| **01** | [📖 雲端架構與服務解析](01-architecture-and-cloud-services.md) | 系統詳細架構圖、GCP 託管服務角色表、IAM 服務帳戶綁定、區域劃分 (`us-east1` vs `global`) |
| **02** | [🛠️ Skills 與 MCP 資源清單](02-skills-and-mcp-inventory.md) | 11 項 Antigravity Skills 全解析、Google 開發者知識庫 MCP、Firebase MCP 與核心套件清單 |
| **03** | [🚀 從零到一端到端實戰教學](03-end-to-end-tutorial.md) | 14 步完整演練：鷹架建置、Firestore、Memory Bank、Sandbox、Imagen、Omni Video、A2UI、Cloud Run、Demo 錄製到開源 |
| **04** | [⚠️ 常見疑難排解與踩坑記錄](04-troubleshooting-and-pitfalls.md) | Protocol 錯誤排解、Omni Interactions API 呼叫技巧、403 PERMISSION_DENIED 修正、記憶庫參數踩坑 |
| **05** | [⚡ 實用指令速查手冊 (CLI Cheatsheet)](05-cli-cheatsheet.md) | GCP & IAM、ADK/agents-cli、FastAPI、Cloud Run、ffmpeg 輕量 GIF 轉檔與 GitHub CLI 高頻指令 |

---

## 🎯 核心實戰專案範例：MeetMind

本手冊各章節之技術實踐均奠基於完整實作的特工專案 **MeetMind (會議智慧與知識庫特工)**：
- **核心特工程式碼儲存庫**：[`hongyue1213/buildwithgemini-meetmind`](https://github.com/hongyue1213/buildwithgemini-meetmind)
- **實作技術棧**：
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
