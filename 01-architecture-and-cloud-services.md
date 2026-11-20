# 01. 雲端架構與 Google Cloud 服務解析 (Architecture & Cloud Services)

本章節詳細彙整在本次 **Build with Gemini (Track 3: Agent-First Apps)** 專案中實際串接與使用的 Google Cloud 各項雲端服務、架構關係與 IAM 權限配置。

---

## 🏛️ 總體系統架構圖 (System Architecture)

```
                     ┌──────────────────────────────────────────────┐
                     │                 User Browser                 │
                     └──────────────────────┬───────────────────────┘
                                            │ HTTP / Web UI (A2UI)
                                            ▼
                     ┌──────────────────────────────────────────────┐
                     │          Frontend Proxy (Cloud Run)          │
                     │          - FastAPI + A2A Protocol SDK        │
                     │          - Native A2UI Card Renderer         │
                     └──────────────────────┬───────────────────────┘
                                            │ A2A gRPC/REST (Streaming)
                                            ▼
┌───────────────────────────────────────────────────────────────────────────────────┐
│              Google Cloud Agent Platform (Vertex AI Agent Runtime)                │
│                                                                                   │
│   ┌───────────────────────────────────────────────────────────────────────────┐   │
│   │                        Root ADK Agent (MeetMind)                          │   │
│   │                        Model: gemini-3.8-flash                            │   │
│   └──────┬────────────────┬────────────────┬────────────────┬─────────────────┘   │
└──────────┼────────────────┼────────────────┼────────────────┼─────────────────────┘
           │                │                │                │
           ▼                ▼                ▼                ▼
┌──────────────────┐ ┌──────────────┐ ┌──────────────┐ ┌───────────────────────────┐
│  Vertex AI       │ │ Cloud        │ │ Vertex AI    │ │ Multimodal Media Services │
│  Memory Bank     │ │ Firestore    │ │ Agent Engine │ │                           │
│  (Long-Term      │ │ (Native DB)  │ │ Code Sandbox │ │ 1. Image Gen              │
│   Memory)        │ │              │ │              │ │    gemini-3.1-flash-lite  │
│                  │ │ - meetings   │ │ - Python 3   │ │                           │
│ - Preload Tool   │ │ - tasks      │ │   Execution  │ │ 2. Omni Video Gen         │
│ - Generate Hook  │ │ - decisions  │ │ - Sandbox ID │ │    gemini-omni-flash-prev │
└──────────────────┘ └──────────────┘ └──────────────┘ └─────────────┬─────────────┘
                                                                     │
                                                                     ▼
                                                     ┌─────────────────────────────┐
                                                     │ Google Cloud Storage (GCS)  │
                                                     │ - Public Asset Bucket       │
                                                     │ - Direct Stream Upload      │
                                                     └─────────────────────────────┘
```

---

## ☁️ Google Cloud 服務清單與職責

| 服務名稱 | 核心角色與職責 | 本專案實作機制與呼叫方式 |
| :--- | :--- | :--- |
| **Vertex AI Agent Platform (Agent Runtime)** | 代管 ADK 特工的雲端推論與狀態執行環境 | 透過 `agents-cli deploy` 打包部署，對外暴露標準 A2A (Agent-to-Agent) 協定介面。 |
| **Vertex AI Memory Bank** | 跨對話工作階段（Cross-Session）的長期使用者偏好與背景記憶 | 建立在 Agent Engine 資源上；在模型推論前透過 `PreloadMemoryTool` 注入相關記憶，推論後觸發 `add_session_to_memory()` 提取重點。 |
| **Vertex AI Agent Engine Sandbox** | 安全隔離的雲端程式碼執行沙盒 (Code Execution) | 綁定 `AgentEngineSandboxCodeExecutor`，讓特工能在隔離容器中執行 Python 程式碼，安全進行日期運算、統計與數據分析。 |
| **Google Cloud Firestore** | 原生 NoSQL 文件資料庫，保存結構化知識庫 | 建立 `meetings` 與 `action_items` 集合，透過 Python `google-cloud-firestore` SDK 提供 6 項 CRUD 工具（增查檢修）。 |
| **Google Cloud Storage (GCS)** | 公開儲存貯體（Bucket），代管生成的多模態資源 | 建立全域公開儲存貯體，影像與影片生成後直接串流上傳至 GCS，並將公開 HTTPS 網址回傳給卡片介面渲染。 |
| **Cloud Run** | 無伺服器容器代管服務，託管前端介面與代理伺服器 | 將 FastAPI 代理程式與靜態網頁打包為容器映像檔，並賦予存取 Agent Platform 的服務帳戶權限。 |
| **Gemini 3.8 Flash** | 核心大型語言模型（Core Reasoning LLM） | 作為特工的主思考大腦，具備極高推論速度、精確的 Function Calling 工具調用與長上下文處理能力。 |
| **Gemini 3.1 Flash Lite Image** | 影像生成模型（Image Generation） | 用於生成視覺化心智圖、會議架構圖與里程碑成果海報，產出二進位圖檔並存入 Artifact。 |
| **Gemini Omni Flash Preview (`gemini-omni-flash-preview`)** | 影音多模態模型（Video/Omni Generation） | 位於 `global` 區域，專門透過 **Interactions API** 生成 MP4 動畫精華短片。 |

---

## 🔐 IAM 服務帳戶與權限配置 (Identity and Access Management)

在無伺服器與 Agentic 雲端架構中，服務帳戶（Service Account）是打通各元件權限的核心樞紐：

### 1. Deployed Agent Service Account（特工專屬服務帳戶）
- **帳號角色**：`service-<PROJECT_NUMBER>@gcp-sa-aiplatform-re.iam.gserviceaccount.com`
- **所需角色 (Roles)**：
  - `roles/datastore.user`：允許特工存取與讀寫 Cloud Firestore 資料庫。
  - `roles/storage.objectAdmin`：允許特工直接寫入並管理 Cloud Storage 儲存貯體物件。
  - `roles/aiplatform.user`：允許特工調用 Vertex AI 推論模型與 Memory Bank。

```bash
# 授權 Firestore 存取權
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:service-<PROJECT_NUMBER>@gcp-sa-aiplatform-re.iam.gserviceaccount.com" \
  --role="roles/datastore.user"

# 授權 Cloud Storage 寫入權
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:service-<PROJECT_NUMBER>@gcp-sa-aiplatform-re.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"
```

### 2. Cloud Run Service Account（前端代理服務帳戶）
- **帳號角色**：`<PROJECT_NUMBER>-compute@developer.gserviceaccount.com`
- **所需角色 (Roles)**：
  - `roles/aiplatform.user`：允許 Cloud Run 前端透過 Google 預設憑證（ADC）代表使用者身分調用後端 Vertex AI Reasoning Engine / Agent Runtime。

```bash
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:<PROJECT_NUMBER>-compute@developer.gserviceaccount.com" \
  --role="roles/aiplatform.user"
```

---

## 🌐 區域與端點設定 (Regions & Endpoints)

- **主工作區區域 (Primary Region)**：`us-east1`
  - Agent Platform Reasoning Engine
  - Agent Engine Sandbox
  - Cloud Firestore
  - Google Cloud Storage Bucket
  - Cloud Run Service
- **全球區域 (Global Region)**：`global`
  - 專用於 Google Interactions API（`gemini-omni-flash-preview` 影音模型目前僅部署於全域端點）。
