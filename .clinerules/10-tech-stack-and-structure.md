# 10-tech-stack-and-structure.md

在這個專案中，你必須遵守以下技術與結構規則。

### 技術棧

- 使用 Next.js App Router（`/app`）作為主要框架。 [nextjs](https://nextjs.org/docs/app/getting-started/project-structure)
- UI 以 Ant Design 為主，Tailwind CSS 只用於 layout、spacing、顏色等 utility class。不要主動引入其他 UI 框架或 CSS-in-JS（如 MUI、Chakra、styled-components），除非我明確要求。 [dev](https://dev.to/codeparrot/nextjs-and-tailwind-css-2025-guide-setup-tips-and-best-practices-2f6h)
- 認證使用 NextAuth（Auth.js）搭配 Google Provider。 [authjs](https://authjs.dev/guides/role-based-access-control)
- 資料庫使用 Prisma + MySQL。 [axelerant](https://www.axelerant.com/blog/how-to-setup-role-based-access-control-with-next-auth)
- LLM / RAG 使用：  
  - LLM 服務：OpenRouter、fal.ai。  
  - 向量檢索：Faiss，必要時搭配 FastAPI 實作 RAG backend。 [linkedin](https://www.linkedin.com/pulse/turning-company-chaos-instant-insights-my-rag-journey-yassine-koubaa-irsxe)
- 若需要 Python 生態（大量運算、RAG pipeline），在 `python-services/` 底下使用 FastAPI，由 Next.js 透過 HTTP 呼叫。 [datacamp](https://www.datacamp.com/tutorial/building-a-rag-system-with-langchain-and-fastapi)
- 專案會用 Capacitor 打包成 iOS / Android App。

### 環境變數與金鑰（重要）

- 所有金鑰與敏感設定 **一律集中在專案根目錄 `.env`**：  
  - 包含 DB（MySQL）、NextAuth、Google OAuth、OpenRouter、fal.ai、FastAPI URL、Faiss / RAG 相關設定等。 [nextjs](https://nextjs.org/docs/pages/guides/environment-variables)
- 程式碼中禁止硬編任何 key / secret / token，只能透過環境變數讀取：  
  - Next.js 端使用 `process.env.*`。 [nextjs](https://nextjs.org/docs/app/guides/environment-variables)
  - Python / FastAPI 端使用標準環境變數讀取。  
- `.env` 不應 commit 到版本控制，如需範例請建立 `.env.example`，僅列出變數名稱與用途註解，不放真實值。 [nextjsstarter](https://nextjsstarter.com/blog/nextjs-environment-variables-secure-management-guide/)

### 專案與目錄結構總覽

本專案採用 Domain / MCV 層的結構。 [giancarlobuomprisco](https://giancarlobuomprisco.com/next/a-scalable-nextjs-project-structure)

- UI 與路由：放在 `/app/**`（Next.js App Router）。  
- Domain / MCV 層：放在 `/modules/**`，作為領域層（domain layer），不直接耦合 Next.js 路由。  
- Python / RAG：放在 `/python-services/**`，例如 `python-services/rag-api/main.py`。

以 `grades` 模組為例：

```text
/app
  /(modules)
    /grades
      /(ui)
        page.tsx           # 成績列表頁
        [id]/
          page.tsx         # 學生成績詳情頁
  /api
    /grades
      list/route.ts        # GET 列表
      [id]/route.ts        # GET/PATCH/DELETE 單筆

/modules
  /grades
    model.ts               # 資料存取（Prisma）
    controller.ts          # 業務邏輯 + RBAC
    views.tsx              # 共用 UI 元件 (Ant + Tailwind)

/modules/rbac
  config.ts                # 集中 RBAC 規則與 can() 函式
  types.ts                 # Role 型別定義

/python-services
  /rag-api
    main.py                # FastAPI 入口
    rag.py                 # RAG pipeline（Faiss + LLM）
    models.py              # Pydantic models
    faiss_store.py         # Faiss index 封裝
```

原則：

- UI 與 API 必須清楚分開：  
  - UI：`/app/**` + `/modules/*/views.tsx`。  
  - API：`/app/api/**` 和（如有）`/python-services/**`。  
- 不要在 UI component 中直接操作 Prisma 或 LLM，改由 API / controller 處理。

### Capacitor 相關

- 與 Capacitor 有關的設定檔（例如 `capacitor.config.*`、原生平台設定檔），修改前一律先詢問我是否可以修改，以及修改的原因與內容。