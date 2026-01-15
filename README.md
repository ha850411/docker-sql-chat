# Docker SQL Chat with LiteLLM (Gemini Support)

這是一個將 **SQL Chat** 與 **Google Gemini** 模型整合的 Docker Compose 專案。

原本 SQL Chat 主要是設計給 OpenAI GPT 模型使用，本專案透過引入 **LiteLLM** 作為中間代理（Proxy），將 OpenAI 格式的請求即時轉換為 Google Gemini 的格式，讓您可以使用免費或高效的 Gemini 模型來驅動 SQL Chat。

## 架構說明

系統由三個主要容器組成：

1.  **sqlchat_app**: [SQL Chat](https://github.com/sqlchat/sqlchat) 網頁介面，負責與使用者互動。
2.  **sqlchat_db**: Postgres 資料庫，用於儲存 SQL Chat 的連線資訊與對話紀錄。
3.  **litellm_proxy**: [LiteLLM](https://docs.litellm.ai/) 代理伺服器。它偽裝成 OpenAI API，接收來自 sqlchat 的請求，並轉發給 Google Gemini API。

## 前置需求

*   [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)
*   一組 [Google Gemini API Key](https://aistudio.google.com/app/apikey)

## 快速開始

### 1. 取得專案
將本專案下載至本地端。

### 2. 設定環境變數
請複製 `.env.example` 為 `.env`（如果尚未建立），並填入您的 API Key。

```bash
cp .env.example .env
```

編輯 `.env` 檔案，最重要的設定如下：

```dotenv
# 你的 Google Gemini API Key (必填)
GEMINI_API_KEY=AIzaSy...

# 指定給 SQL Chat 看的模型名稱 (維持 gpt-3.5-turbo 即可，LiteLLM 會自動轉譯)
OPENAI_CHAT_MODEL=gpt-3.5-turbo

# NextAuth 的加密密鑰 (隨意輸入一串長字串)
NEXTAUTH_SECRET=changeme123456

# 網路設定 (通常不需要改)
OPENAI_API_ENDPOINT=http://litellm:4000
NETWORK_SUBNET=192.168.101.0/24
NETWORK_GATEWAY=192.168.101.1
```

### 3. 模型對應設定 (litellm_config.yaml)
專案預設將 `gpt-3.5-turbo` 的請求轉發給 `gemini/gemini-flash-latest`。如果需要更改使用的 Gemini 模型，請修改 `litellm_config.yaml`：

```yaml
model_list:
  - model_name: gpt-3.5-turbo  # SQL Chat 呼叫的假名稱
    litellm_params:
      model: gemini/gemini-flash-latest # 實際使用的 Gemini 模型
      api_key: os.environ/GEMINI_API_KEY
```
* **常用模型**：`gemini/gemini-pro`, `gemini/gemini-1.5-flash`

### 4. 啟動服務
執行以下指令啟動所有容器：

```bash
docker-compose up -d
```

### 5. 使用 SQL Chat
開啟瀏覽器訪問 [http://localhost:3000](http://localhost:3000)。
您現在可以用自然語言與資料庫進行對話，背後的 AI 模型是 Google Gemini。

## 常見問題 (Troubleshooting)

*   **litellm.NotFoundError / Vertex_ai_betaException**:
    *   這通常表示指定的模型名稱在當前的 API 版本中不存在，或是您的 API Key 沒有權限存取該模型。
    *   解決方法：檢查 `litellm_config.yaml` 中的 `model` 名稱是否正確（建議使用 `gemini/gemini-flash-latest` 或 `gemini/gemini-pro`）。

*   **Quota exceeded / 429 Too Many Requests**:
    *   表示您的 API Key 免費額度已用完。
    *   解決方法：等待額度重置，或切換至 `gemini-1.5-flash` 等較輕量的模型。

*   **SQL Chat 無法連線**:
    *   確保所有容器都已正常啟動 (`docker-compose ps`)。
    *   檢查 `sqlchat_app` 的 logs 用於除錯 (`docker logs sqlchat_app`)。

## 授權
本專案配置檔僅供教學與開發使用，各元件授權請參考原專案。

*   SQL Chat: Apache-2.0 license
*   LiteLLM: MIT license
