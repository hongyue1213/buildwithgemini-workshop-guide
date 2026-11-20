# 04. 常見疑難排解與踩坑記錄 (Troubleshooting & Pitfalls)

本章節完整記錄在實戰開發、多模型整合與雲端部署過程中遇到的真實錯誤與標準解法。

---

## ⚠️ 1. `ValueError: Protocol message AgentCard has no "supportedInterfaces" field`

### 問題現象
本機前端或代理伺服器在解析後端特工回傳的 AgentCard 協定訊息時崩潰，回報：
```
ValueError: Protocol message AgentCard has no "supportedInterfaces" field
```

### 根本原因
- 背景中仍有常駐處理程序（Daemon）運行著舊版或已載入記憶體的早期套件。
- `a2a-sdk` 在 0.2.x 升級至 0.3+ 時，通訊協定 protobuf 定義調整了卡片介面欄位。

### 解決方案
1. 查詢並終止佔用 8080 連接埠的舊處理程序：
   ```bash
   lsof -i :8080 | awk 'NR>1 {print $2}' | xargs -r kill -9
   ```
2. 重新同步前端虛擬環境依賴：
   ```bash
   cd frontend
   uv pip install -r requirements.txt --python .venv
   ```
3. 重新啟動代理伺服器：
   ```bash
   .venv/bin/python main.py
   ```

---

## ⚠️ 2. Omni 影音模型 (`gemini-omni-flash-preview`) 調用失敗

### 問題現象
嘗試像一般文字或圖片模型一樣呼叫 `generate_content` 或在區域端點（如 `us-east1`）調用時，回報模型不存在或方法不支援：
```
NotFound: 404 Model gemini-omni-flash-preview not found in us-east1
或
AttributeError: 'Client' object has no attribute 'generate_video'
```

### 根本原因
- `gemini-omni-flash-preview` 屬於實驗性全模態多輪互動模型，目前**僅部署於全域端點 (`global`)**。
- 不使用傳統的 `client.models.generate_content`，而是必須透過全新的 **Interactions API** (`client.interactions.create`)。

### 解決方案
在實作影音工具時，明確指定全域區域與 API 版本：
```python
from google import genai
import base64

client = genai.Client(
    http_options={"api_version": "v1alpha"},
    location="global"  # 關鍵設定：必須為 global
)

res = client.interactions.create(
    model="gemini-omni-flash-preview",
    input="Generate a video animation highlighting milestone achievements"
)

# 影片二進位資料以 base64 封裝於 output_video.data
video_bytes = base64.b64decode(res.output_video.data)
```

---

## ⚠️ 3. 雲端部署後特工存取 Firestore / GCS 出現 403 PERMISSION_DENIED

### 問題現象
本地透過 `adk web` 執行完全正常，但使用 `agents-cli deploy` 發布至雲端後，特工執行查詢會議或上傳圖片時噴出 403 錯誤。

### 根本原因
- 本機執行時使用的是開發者個人帳號憑證（ADC），具備專案 Owner 或 Editor 權限。
- 一旦部署至 Vertex AI Agent Runtime，特工是以 **Google 代管的 Agent 服務帳戶** 身分執行，預設無權存取專案內的 Firestore 或 GCS 儲存貯體。

### 解決方案
查詢專案號碼並明確授權 Agent 服務帳戶：
```bash
PROJECT_NUM="$(gcloud projects describe <PROJECT_ID> --format='value(projectNumber)')"
AGENT_SA="service-${PROJECT_NUM}@gcp-sa-aiplatform-re.iam.gserviceaccount.com"

# 授權 Firestore 資料庫讀寫
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:${AGENT_SA}" \
  --role="roles/datastore.user"

# 授權 GCS 儲存貯體管理權限
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:${AGENT_SA}" \
  --role="roles/storage.objectAdmin"
```

---

## ⚠️ 4. Cloud Run 前端無法與後端特工通訊 (403 Forbidden)

### 問題現象
訪問 Cloud Run 前端網址送出對話時，前端回應：
```
Server returned non-JSON (HTTP 403). Permission 'aiplatform.reasoningEngines.query' denied
```

### 根本原因
Cloud Run 預設的執行身分是 Compute Engine 預設服務帳戶（`<PROJECT_NUMBER>-compute@developer.gserviceaccount.com`），該帳號預設未被賦予 Vertex AI 調用權限。

### 解決方案
為 Cloud Run 服務帳戶增加 `roles/aiplatform.user` 角色：
```bash
CR_SA="${PROJECT_NUM}-compute@developer.gserviceaccount.com"

gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:${CR_SA}" \
  --role="roles/aiplatform.user"
```

---

## ⚠️ 5. 本機 ADK Playground 未保留或載入 Memory Bank 記憶

### 問題現象
在 `uv run adk web .` 中聊天，換了一個工作階段後，特工完全遺忘了前一次對話中強調的設定。

### 根本原因
`adk web` 預設只使用暫存的記憶體儲存（in-memory），並不會自動連線到雲端的 Vertex AI Memory Bank 服務。

### 解決方案
啟動命令中必須顯式附加 `--memory_service_uri`：
```bash
uv run adk web . --port 8000 --reload_agents --memory_service_uri=agentengine://<MEMORY_BANK_ENGINE_ID>
```

---

## ⚠️ 6. Git Push 時提示需要手動輸入帳號密碼

### 問題現象
使用 `gh auth login` 登入成功後，直接執行 `git push origin main` 卻依然停在 `Username for 'https://github.com':`。

### 根本原因
Git 的全域憑證輔助程式（Credential Helper）尚未註冊 GitHub CLI 的 Token 映射。

### 解決方案
執行：
```bash
gh auth setup-git
git push origin main
```
此指令會自動將 GitHub CLI 登入憑證配置至 Git 配置中，後續推送不再需要手動輸入密碼。
