# 03. 從零到一端到端實戰教學 (End-to-End Tutorial)

本手冊提供建置類似 **MeetMind** 的全功能 AI Agent 應用程式的完整實作流程，從環境配置、特工撰寫、資料庫掛載、多模態影音整合，到雲端部署與展示。

---

## 📋 流程總覽 (Step Overview)

```
[1. 環境驗證] ──▶ [2. 專案鷹架] ──▶ [3. Firestore 建置] ──▶ [4. Memory Bank 整合]
                                                                     │
[8. A2UI 卡片] ◀── [7. Omni 影音] ◀── [6. 影像與 GCS] ◀── [5. 程式沙盒]
      │
      ▼
[9. 後端部署] ──▶ [10. IAM 授權] ──▶ [11. 前端建置] ──▶ [12. Cloud Run 部署]
                                                                     │
                                        [14. 開源發布] ◀── [13. 展示錄製]
```

---

## 步驟 1：環境驗證與 GCP 初始化

確認 Google Cloud 登入狀態、專案設定與應用程式預設憑證（ADC）：

```bash
# 1. 登入 Google Cloud CLI
gcloud auth login

# 2. 登入應用程式預設憑證（讓本機 Python SDK 具備 GCP 權限）
gcloud auth application-default login

# 3. 設定預設專案
export GCP_PROJECT="<YOUR_GCP_PROJECT_ID>"
gcloud config set project $GCP_PROJECT

# 4. 啟用必要 API
gcloud services enable \
  aiplatform.googleapis.com \
  firestore.googleapis.com \
  storage.googleapis.com \
  run.googleapis.com \
  cloudbuild.googleapis.com
```

---

## 步驟 2：使用 agents-cli 初始化 ADK 專案

使用 `agents-cli` 鷹架建立標準專案結構：

```bash
# 建立專案
agents-cli scaffold create \
  --name meetmind \
  --base-template adk \
  --deployment-target agent_runtime \
  --region us-east1

cd meetmind
```

專案目錄結構如下：
- `agents-cli-manifest.yaml`：宣告特工進入點與雲端設定
- `app/agent.py`：Root Agent 定義
- `pyproject.toml`：Python 套件依賴管理

---

## 步驟 3：掛載 Cloud Firestore 結構化知識庫

建立 `app/firestore_tools.py`，封裝會議記錄（`meetings`）與待辦項目（`action_items`）之查詢與寫入工具：

```python
from google.cloud import firestore
import uuid

db = firestore.Client()

def list_meetings():
    """列出歷史會議清單"""
    docs = db.collection("meetings").order_by("created_at", direction=firestore.Query.DESCENDING).limit(10).stream()
    return [{"id": d.id, **d.to_dict()} for d in docs]

def search_meetings(keyword: str):
    """搜尋會議重點與決策"""
    docs = db.collection("meetings").stream()
    results = []
    for d in docs:
        data = d.to_dict()
        if keyword.lower() in str(data).lower():
            results.append({"id": d.id, **data})
    return results

def list_action_items(owner: str = None, status: str = None):
    """查詢待辦事項，可依負責人或狀態過濾"""
    query = db.collection("action_items")
    if owner:
        query = query.where("owner", "==", owner)
    if status:
        query = query.where("status", "==", status)
    return [{"id": d.id, **d.to_dict()} for d in query.stream()]
```

---

## 步驟 4：整合 Vertex AI Memory Bank 長期記憶

讓特工能跨對話階段記住使用者偏好與團隊背景資訊：

1. **建立託管 Memory Bank 資源**（透過 Agent Engine 建立）。
2. 在 `app/agent.py` 中掛載記憶工具與回呼：

```python
from google.adk.tools.memory import PreloadMemoryTool

async def generate_memories_callback(callback_context):
    """在對話回合結束後，自動將內容提取至長期記憶庫"""
    await callback_context.add_session_to_memory()

root_agent = Agent(
    model=Gemini(model="gemini-3.8-flash"),
    tools=[PreloadMemoryTool(), ...],
    after_model_callback=generate_memories_callback,
)
```

本機測試時啟動指令需帶入記憶庫 URI：
```bash
uv run adk web . --port 8000 --memory_service_uri=agentengine://<MEMORY_BANK_ID>
```

---

## 步驟 5：啟用 Agent Engine 雲端程式碼沙盒

讓特工能安全執行 Python 程式碼以計算統計數據：

```python
from google.adk.code_executors import AgentEngineSandboxCodeExecutor

sandbox = AgentEngineSandboxCodeExecutor(
    sandbox_resource_name="projects/<PROJECT_NUMBER>/locations/us-east1/reasoningEngines/<ENGINE_ID>/sandboxEnvironments/<SANDBOX_ID>"
)

root_agent.code_executor = sandbox
```

---

## 步驟 6：多模態影像生成與 GCS 串流

建立 `app/image_tools.py`，使用 `gemini-3.1-flash-lite-image` 生成心智圖，並串流上傳至公開 GCS 儲存貯體：

```python
from google import genai
from google.genai import types
from google.cloud import storage
import uuid

BUCKET_NAME = "<YOUR_PUBLIC_BUCKET_NAME>"

async def generate_meeting_visual(prompt: str, tool_context) -> str:
    client = genai.Client()
    res = client.models.generate_images(
        model="gemini-3.1-flash-lite-image",
        prompt=prompt,
        config=dict(number_of_images=1, output_mime_type="image/png")
    )
    img_bytes = res.generated_images[0].image.image_bytes
    
    # 1. 儲存為 ADK Session Artifact
    filename = f"visual_{uuid.uuid4().hex[:8]}.png"
    await tool_context.save_artifact(filename=filename, artifact=types.Part.from_bytes(data=img_bytes, mime_type="image/png"))
    
    # 2. 直接串流上傳至 GCS（不寫入本地磁碟）
    gcs_client = storage.Client()
    bucket = gcs_client.bucket(BUCKET_NAME)
    blob = bucket.blob(filename)
    blob.upload_from_string(img_bytes, content_type="image/png")
    
    return f"https://storage.googleapis.com/{BUCKET_NAME}/{filename}"
```

---

## 步驟 7：Google Omni 影音短片生成 (Interactions API)

建立 `app/video_tools.py`，使用 Google 最新 Omni 影音模型生成短片：

> ⚠️ **關鍵技巧**：`gemini-omni-flash-preview` 僅支援全域區域 (`global`)，且必須透過 **Interactions API** (`client.interactions.create`) 調用：

```python
from google import genai
from google.genai import types
from google.cloud import storage
import base64, uuid

BUCKET_NAME = "<YOUR_PUBLIC_BUCKET_NAME>"

async def generate_meeting_video(topic: str, tool_context) -> str:
    # 必須指定 location="global"
    client = genai.Client(http_options={"api_version": "v1alpha"}, location="global")
    
    res = client.interactions.create(
        model="gemini-omni-flash-preview",
        input=f"Generate a short video animation highlighting: {topic}"
    )
    video_bytes = base64.b64decode(res.output_video.data)
    
    filename = f"video_{uuid.uuid4().hex[:8]}.mp4"
    # 保存 Artifact
    await tool_context.save_artifact(filename=filename, artifact=types.Part.from_bytes(data=video_bytes, mime_type="video/mp4"))
    
    # 上傳至 GCS
    gcs_client = storage.Client()
    blob = gcs_client.bucket(BUCKET_NAME).blob(filename)
    blob.upload_from_string(video_bytes, content_type="video/mp4")
    
    return f"https://storage.googleapis.com/{BUCKET_NAME}/{filename}"
```

---

## 步驟 8：掛載 A2UI 結構化介面

透過 `a2ui_utils.py` 攔截模型輸出，將結構化 JSON 自動包裝為 A2UI 卡片規格：

```python
from app.a2ui_utils import A2uiSchemaManager, a2ui_callback

schema_manager = A2uiSchemaManager()

root_agent = Agent(
    model=Gemini(model="gemini-3.8-flash"),
    tools=[...],
    after_model_callback=[generate_memories_callback, a2ui_callback],
)
```

---

## 步驟 9：部署 Agent 至 Vertex AI Agent Platform

使用 `agents-cli` 打包並部署：

```bash
agents-cli deploy \
  --target agent_runtime \
  --region us-east1
```
部署完成後會產生 `deployment_metadata.json`，記錄專屬的 Reasoning Engine 資源名稱（例如 `projects/.../locations/us-east1/reasoningEngines/2734449134395195392`）。

---

## 步驟 10：設定後端服務帳戶 IAM 權限

雲端特工執行環境需要存取 Firestore 與 GCS，必須綁定專屬角色：

```bash
PROJECT_NUM="$(gcloud projects describe $GCP_PROJECT --format='value(projectNumber)')"
AGENT_SA="service-${PROJECT_NUM}@gcp-sa-aiplatform-re.iam.gserviceaccount.com"

# 授權 Firestore 存取
gcloud projects add-iam-policy-binding $GCP_PROJECT \
  --member="serviceAccount:${AGENT_SA}" \
  --role="roles/datastore.user"

# 授權 GCS 儲存貯體寫入
gcloud projects add-iam-policy-binding $GCP_PROJECT \
  --member="serviceAccount:${AGENT_SA}" \
  --role="roles/storage.objectAdmin"
```

---

## 步驟 11：建置 Web 前端與 A2A 代理伺服器

在 `frontend/` 目錄建立 FastAPI 應用程式，使用 `a2a-sdk` 與雲端 Agent Runtime 溝通：

```python
# frontend/main.py 精簡示意
from fastapi import FastAPI
from a2a import Client

app = FastAPI()
client = Client()

@app.post("/chat")
async def chat_endpoint(payload: dict):
    response_stream = await client.send_message(
        agent_engine_resource=os.environ["AGENT_ENGINE_RESOURCE_NAME"],
        message=payload["message"]
    )
    return {"parts": response_stream.parts}
```

搭配 `static/index.html` 提供美觀的對話介面與 A2UI 原生卡片渲染。

---

## 步驟 12：部署前端至 Cloud Run

將前端容器化並上傳部署至 Cloud Run：

```bash
gcloud run deploy meetmind-frontend \
  --source frontend \
  --region us-east1 \
  --allow-unauthenticated \
  --set-env-vars="AGENT_ENGINE_RESOURCE_NAME=<RESOURCE_NAME>,AGENT_DIRECTORY=app"

# 授權 Cloud Run Service Account 調用 Vertex AI
CR_SA="${PROJECT_NUM}-compute@developer.gserviceaccount.com"
gcloud projects add-iam-policy-binding $GCP_PROJECT \
  --member="serviceAccount:${CR_SA}" \
  --role="roles/aiplatform.user"
```

---

## 步驟 13：自動化展示錄製與 GIF 最佳化

使用 Playwright 驅動無周邊瀏覽器進行端對端操作錄影：

```bash
# 1. 執行錄影腳本
node .agents/skills/record-demo/record-agent.js --url "http://localhost:8080" --output demo.webm

# 2. 使用 ffmpeg 轉換為高品質最佳化循環 GIF
ffmpeg -y -i demo.webm \
  -vf "fps=10,scale=720:-1:flags=lanczos,split[s0][s1];[s0]palettegen=max_colors=128[p];[s1][p]paletteuse=dither=bayer" \
  demo.gif
```

---

## 步驟 14：透過 Device Flow 安全發布至 GitHub

使用 `publish-to-github` 流程發布：

```bash
# 1. 環境準備與敏感資訊掃描
bash .agents/skills/publish-to-github/publish.sh prep

# 2. GitHub CLI Device Code 登入
gh auth login --hostname github.com --git-protocol https --web

# 3. 提交本地 Git
bash .agents/skills/publish-to-github/publish.sh commit

# 4. 建立公開儲存庫並推送
gh repo create buildwithgemini-meetmind --public --source=. --remote=origin --push

# 5. 取得預先填妥之完賽登記表單
bash .agents/skills/publish-to-github/publish.sh formlink "https://github.com/<USERNAME>/buildwithgemini-meetmind"
```
