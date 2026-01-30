# 30-mcv-and-modules.md

在這個專案中，每個功能模組請遵循 Domain / MCV（Model / Controller / Views）模式與固定目錄結構。 [magicui](https://magicui.design/blog/next-js-project-structure)

### 模組命名

- 模組資料夾命名採英文複數，例如：`grades`, `attendance`, `reports`。  
- `/app/(modules)` 與 `/modules` 下的模組資料夾名稱應一致。

### Domain / MCV 層概念

- 專案有一個獨立的 **Domain / MCV 層**，集中放各模組的商業邏輯與資料存取。  
  - UI 與路由：在 `/app/**`。  
  - Domain / MCV 層：在 `/modules/**`，屬於「領域層」，不直接耦合 Next.js 路由。 [nextjs](https://nextjs.org/docs/app/getting-started/project-structure)
- 以 `grades` 為例：  
  - `/modules/grades/model.ts`：Prisma + MySQL 資料操作。  
  - `/modules/grades/controller.ts`：業務邏輯與 RBAC，內部可呼叫 `modules/rbac/config.ts` 中的 `can()`。  
  - `/modules/grades/views.tsx`：共用 UI 組件（Ant + Tailwind），不執行資料存取。

### MCV 分工

- **Model（`model.ts`）**  
  - 所有 Prisma + MySQL 相關的資料操作。  
  - 提供高層使用的函式，如 `getGradesByClass`, `getGradeById`, `createGrade`, `updateGrade`, `deleteGrade`。  

- **Controller（`controller.ts`）**  
  - 封裝業務邏輯與權限判斷。  
  - 由 API route 或 server action 呼叫，不直接暴露給 UI component。  
  - 這裡可以載入：  
    - `modules/rbac/config.ts` → `can(role, module, action)`。  
    - 對應模組的 model 函式。  

- **Views（`views.tsx`）**  
  - 存放此模組共用的 React UI 元件。  
  - 使用 Ant Design + Tailwind。  
  - 不直接操作 Prisma 或 LLM，只接收 props、觸發 callback。

### UI 與 API 結構

- UI 路由放在 `/app/(modules)/<moduleName>/(ui)`：  
  ```text
  /app
    /(modules)
      /grades
        /(ui)
          page.tsx       # 列表頁
          [id]/
            page.tsx     # 詳情頁
  ```  

- API 路由放在 `/app/api/<moduleName>`：  
  ```text
  /app
    /api
      /grades
        list/route.ts    # GET: 取得列表
        [id]/route.ts    # GET/PATCH/DELETE: 操作單筆
  ```

- API handler 的職責：  
  - 解析 request。  
  - 使用 NextAuth 取得 session / role。  
  - 呼叫對應模組的 controller 函式。  
  - 回傳適當的 HTTP 狀態碼與 JSON。

### 當我要求你「新增一個模組」時，你應該這樣做

1. 向我確認：  
   - 模組名稱（英文複數）。  
   - 主要資料欄位與功能。  
   - 哪些角色可以使用哪些操作。  
2. 檢查是否需要新增／修改 Prisma schema（若需要，先描述變更並詢問是否 OK）。  
3. 在 `/modules` 下建立 `<moduleName>` 資料夾：  
   - `model.ts`  
   - `controller.ts`  
   - `views.tsx`  
4. 在 `/app/(modules)/<moduleName>/(ui)` 下建立頁面：  
   - `page.tsx`（主列表或首頁）  
   - 視需要新增 `[id]/page.tsx` 等。  
5. 在 `/app/api/<moduleName>` 下建立 API routes：  
   - `list/route.ts`  
   - `[id]/route.ts` 等。  
6. 在 `modules/rbac/config.ts` 中為新模組加入權限規則。  
7. 用文字向我總結：新增了哪些檔案、每個檔案的職責，以及是否需要我執行 migration 或設定額外環境變數。