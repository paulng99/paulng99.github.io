## 50-safety-and-constraints.md

在這個專案中，你必須優先考慮安全與穩定性。

### 修改前必須先詢問我的檔案

以下檔案或設定，在修改前你一定要先問我，得到明確允許後才能修改：

- `.env*`（所有環境變數檔）
- `capacitor.config.*` 及任何原生平台設定檔
- `Dockerfile`, `docker-compose.*`
- CI/CD 設定檔（例如 `.github/workflows/**`, `.gitlab-ci.yml` 等）
- `prisma/schema.prisma`（資料庫 schema）


### 禁止行為

- 禁止在任何程式碼中硬編 API key、密碼、token 或任何敏感連線字串。
- 禁止執行具有高度破壞性的 shell 指令，例如：
    - `rm -rf /`
    - `rm -rf .git`
    - 任何會刪除大量檔案或資料庫的指令。
- 禁止自動執行 `git push` 或修改遠端分支設定。


### 終端與資料庫操作

- 可以使用的開發指令（示例，可視實際專案調整）：
    - `npm install` / `pnpm install` / `yarn`
    - `npm run dev` / `npm run build` / `npm run lint`
    - `npx prisma generate`
    - `npx prisma migrate dev`（如有需要，建議先說明將做哪些 schema 變更）[^26][^39][^40][^41][^22]
- 修改 Prisma schema 與執行 migration 之前，建議先向我簡要說明預期影響。


### 一般原則

- 優先遵守專案既定的目錄結構與 Domain / MCV 分層。
- 新增第三方套件前，請評估必要性，並在變更說明中註明用途。
- 遇到需求不清楚或可能有多種設計方案的情況，請先用中文列出幾種選項向我確認，再選擇其一實作。

***

