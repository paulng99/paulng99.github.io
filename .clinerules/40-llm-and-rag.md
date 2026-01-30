# 40-llm-and-rag.md

在這個專案中，LLM / RAG 是輔助功能，請遵守以下規則。

### LLM 服務（OpenRouter, fal.ai）

- 所有 LLM 相關金鑰一律放在 `.env`：  
  - `OPENROUTER_API_KEY`  
  - `FAL_AI_API_KEY` 等。 [datacamp](https://www.datacamp.com/tutorial/openrouter)
- 呼叫 LLM 時：  
  - 優先在 server side（Next.js API route 或 FastAPI）呼叫，不要在瀏覽器端直接使用金鑰。  
  - 遵循各服務官方 API 風格正確設定 headers / body（例如 OpenRouter 的 model id、fal.ai 的 endpoint）。 [docs.litellm](https://docs.litellm.ai/docs/providers/fal_ai)

### RAG 與 Faiss（`python-services/rag-api/main.py`）

- RAG 的後端採用 **FastAPI + Faiss**，文件與檢索邏輯集中於：  
  - `python-services/rag-api/main.py` 及其子模組。 [c-sharpcorner](https://www.c-sharpcorner.com/article/advanced-rag-in-python-with-fastapi-multi-source-retrieval-and-evaluation/)

建議結構：

```text
/python-services
  /rag-api
    main.py          # FastAPI app 入口，掛載路由
    rag.py           # RAG pipeline：切分、向量化、Faiss 檢索、組合上下文
    models.py        # Pydantic models：Request / Response schema
    faiss_store.py   # Faiss index 封裝與存取
```

- `main.py` 應建立 FastAPI app 並 include 對應 router，例如：  
  ```py
  from fastapi import FastAPI
  from .endpoints import router

  app = FastAPI(title="RAG API")
  app.include_router(router)
  ```  

- 建議實作一個 `POST /rag/query` endpoint：  
  - Request：user query + 選擇性 context（例如模組名稱、角色）。  
  - Response：LLM 回答 + 引用片段。  

- Next.js 端不直接操作 Faiss，只透過 HTTP 呼叫 `rag-api`（例如 `POST /rag/query`），由 RAG API 負責向量檢索與組合結果。 [datacamp](https://www.datacamp.com/tutorial/building-a-rag-system-with-langchain-and-fastapi)

### LLM 功能的模組化

- 若 LLM 是獨立模組（例如 AI 助教）：  
  - 建立 `/modules/ai-assistant`（model/controller/views）與 `/app/(modules)/ai-assistant/(ui)`。  
  - API route（如 `/app/api/ai-assistant/ask/route.ts`）負責：  
    - 讀取 session / role。  
    - 呼叫 `python-services/rag-api` 或直接呼叫 OpenRouter / fal.ai。  
- 若 LLM 嵌入其他模組（例如成績模組的「自動產生評語」）：  
  - 可以在該模組建立 `llm.ts` 或在 controller 中封裝 LLM 呼叫邏輯。

### 隱私與安全

- 不得將包含學生個資、敏感資訊的完整原始資料直接送到第三方 LLM。  
  - 如有需要，請先進行脫敏或只傳必要摘要。  
- API 回應中不得回傳任何金鑰、token 或內部環境變數值。