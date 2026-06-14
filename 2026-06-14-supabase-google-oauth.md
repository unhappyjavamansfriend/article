# 流程

- **去 Google Cloud Console：** 建立憑證 (OAuth 2.0 用戶端 ID) ➔ 取得 **Client ID** & **Client Secret**。
    
- **填入 Supabase (Auth -> Providers -> Google)：** 填入對應欄位並啟用 ➔ 複製 Supabase 的 `Redirect URL`。
    
- **回 Google：** 把這條 URL 填入 Google 的 "已授權的重新導向 URI"。

# 已授權的 JavaScript 來源要填什麼？

這個欄位的作用是告訴 Google：「只有從這些網址發出的登入請求，你才可以受理。」這是一道安全防線。

因為在使用 Supabase 時，你的前端網頁會直接呼叫 Google 的 SDK 或觸發跳轉，因此你需要把**前端專案運行的網址**填進去：

- **本地開發測試：** 填入 `http://localhost:3000`（或是你 Vite/Next.js 跑出來的 `http://localhost:5173` 等埠號）。
    
- **線上生產環境：** 如果你的網站已經部署，請填入你的正式網域（例如 `https://www.yourdomain.com`）。
    
- **Supabase 專案網址（保險起見）：** 建議也把你的 Supabase Project URL（例如 `https://project-id.supabase.co`）一起加進去。
