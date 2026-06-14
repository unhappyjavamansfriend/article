# 在 Supabase 後台啟用驗證方法

1. 登入 [Supabase Dashboard](https://www.google.com/search?q=https://supabase.com/dashboard) 並進入你的專案。
    
2. 在左側側邊欄點選 **Authentication**（鎖頭圖示）。
    
3. 點選 **Providers**。
    
4. **第三方登入（例如 Google / GitHub）：**
    
	- 點開對應的 Provider，將其切換為 **ON**。
	    
	- 你需要到該平台的開發者後台（如 Google Cloud Console）申請 `Client ID` 和 `Client Secret` 並填入。
	    
	- 複製 Supabase 提供給你的 `Redirect URL`，填回該平台的開發者後台。

## 如何進行

1. **GitHub / Google 控制台：** 在這裡建立一個應用程式，告訴它們：「我是某某 App，我想要借用你們的用戶登入功能。當用戶登入成功後，請把資料送去我的 Supabase 回呼網址（Callback URL）。」
    
2. **Supabase Dashboard：** 你要把 GitHub / Google 給你的 **Client ID** 和 **Client Secret（密鑰）** 填進去。這樣 Supabase 才有憑有據，能代表你的 App 去跟 Google / GitHub 要資料。
    
3. **前端程式碼：** 只需要初始化 `URL` 和 `ANON_KEY`。當你想登入時，只要給一個簡單的指令（如 `provider: 'github'`），其餘複雜的安全性確認和憑證交換，全部交由 Supabase 後端在幕後搞定。

## 取得 Client ID 與 Client Secret 回填至 Supabase

[[2026-06-14-supabase-github]]
[[2026-06-14-supabase-google-oauth]]

## 取得前端程式碼需要的keys

1. 登入 [Supabase Dashboard](https://www.google.com/search?q=https://supabase.com/dashboard) 並點選進入你的專案。
    
2. 看看左側的側邊欄，最下方有一個 **Settings (齒輪圖示)**，點擊它。
    
3. 在 Settings 的選單中，找到並點選 **API**。
    
4. 進入 API 設定頁面後，你就會在 **Project API keys** 區塊看到這兩個欄位：
	- Legacy anon, service_role API keys
		- anon public
	    
	- **`Project URL`**：點專案就看得到 https://supabase.com/dashboard/project/xxxxx。


