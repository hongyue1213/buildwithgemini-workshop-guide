# 05. 實用指令速查手冊 (CLI Cheatsheet)

本手冊彙整在專案全流程中最高頻使用的終端機指令，涵蓋 GCP 設定、ADK 開發、前端建置、媒體處理解析與 Git 發布。

---

## ☁️ Google Cloud & IAM 指令

```bash
# 登入與設定專案
gcloud auth login
gcloud auth application-default login
gcloud config set project <PROJECT_ID>

# 查看目前設定與專案號碼
gcloud config list
PROJECT_NUM=$(gcloud projects describe $(gcloud config get-value project) --format='value(projectNumber)')

# 查詢已啟用之 API
gcloud services list --enabled | grep -E "aiplatform|run|firestore|storage"

# IAM 權限綁定指令範本
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member="serviceAccount:<SERVICE_ACCOUNT_EMAIL>" \
  --role="<ROLE_NAME>"
```

---

## 🤖 ADK & agents-cli 開發指令

```bash
# 建立全新 ADK 專案
agents-cli scaffold create --name <PROJECT_NAME> --base-template adk --deployment-target agent_runtime --region us-east1

# 本機啟動 ADK Web Playground（帶長期記憶庫連線）
uv run adk web . --port 8000 --reload_agents --memory_service_uri=agentengine://<MEMORY_BANK_ID>

# 部署 Agent 至 Vertex AI Agent Platform
agents-cli deploy --target agent_runtime --region us-east1

# 部署並更新環境變數（如 API 金鑰）
agents-cli deploy --target agent_runtime --region us-east1 --update-env-vars="CUSTOM_KEY=value"
```

---

## 🌐 前端代理與 Cloud Run 部署

```bash
# 本機安裝依賴並啟動 FastAPI 前端 (port 8080)
cd frontend
uv pip install -r requirements.txt --python .venv
AGENT_ENGINE_RESOURCE_NAME="projects/<PROJECT_NUM>/locations/us-east1/reasoningEngines/<ENGINE_ID>" \
AGENT_DIRECTORY="app" \
PORT="8080" \
.venv/bin/python main.py

# 部署前端服務至 Cloud Run
gcloud run deploy <SERVICE_NAME> \
  --source frontend \
  --region us-east1 \
  --allow-unauthenticated \
  --set-env-vars="AGENT_ENGINE_RESOURCE_NAME=<RESOURCE_NAME>,AGENT_DIRECTORY=app"

# 查看 Cloud Run 即時日誌
gcloud run services logs tail <SERVICE_NAME> --region us-east1
```

---

## 🎥 影音轉換與最佳化 (ffmpeg)

```bash
# 將 Playwright 錄製之 .webm 轉為高畫質、循環播放且體積小於 1MB 的 GIF
ffmpeg -y -i input.webm \
  -vf "fps=10,scale=720:-1:flags=lanczos,split[s0][s1];[s0]palettegen=max_colors=128[p];[s1][p]paletteuse=dither=bayer" \
  -loop 0 output.gif

# 檢查影音檔案詳細編碼中繼資料
ffprobe -v quiet -print_format json -show_format -show_streams input.webm
```

---

## 🐙 GitHub CLI (gh) & 開源發布指令

```bash
# 檢查目前登入狀態
gh auth status

# 使用裝置授權碼進行瀏覽器登入（免密碼與 SSH Key）
printf 'y\n' | gh auth login --hostname github.com --git-protocol https --web

# 自動設定 Git 憑證助手（避免 Push 時再要求輸入帳密）
gh auth setup-git

# 建立全新 Public 儲存庫並立即推送當前程式碼
gh repo create <REPO_NAME> --public --source=. --remote=origin --push

# 查詢當前儲存庫網址
gh repo view --json url -q .url
```
