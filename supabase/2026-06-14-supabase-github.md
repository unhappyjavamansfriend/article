## 流程

- **去 GitHub (Developer Settings)：** 建立一個 OAuth App ➔ 取得 **Client ID** & **Client Secret**。
    
- **填入 Supabase (Auth -> Providers -> GitHub)：** 把上面這兩個東西填進去，並打開開關 ➔ 同時複製 Supabase 給你的 `Redirect URL`。
    
- **回 GitHub：** 把這條 `Redirect URL` 貼進 GitHub 的 "Authorization callback URL" 欄位。

##  1：進入 GitHub 開發者設定

1. 登入你的 [GitHub](https://github.com/) 帳號。
    
2. 點擊右上角的**個人頭像**，在下拉選單中選擇 **Settings（設定）**。
    
3. 進入設定頁面後，將左側選單滑到**最下方**，點擊 **<> Developer settings（開發者設定）**。
    

##  2：建立 OAuth 應用程式

1. 在左側選單中，點選 **OAuth Apps**。
    
2. 點擊右上角的 **New OAuth App（新建 OAuth 應用程式）** 按鈕。
    

## 3：填寫應用程式資訊（填寫 Callback URL 的地方）

這時候會出現一個表單，請依序填入：

- **Application name：** 你的專案名稱（例如：`My Supabase App`，可自由命名）。
    
- **Homepage URL：** 你的網站主頁網址（開發階段可以先填 `http://localhost:3000` 或你的前端網址）。
    
- **Application description：** 應用程式描述（可留空）。
    
- **Authorization callback URL：** ✨ **就是這裡！** 請在這裡貼上你從 Supabase 複製的 Callback URL。
    
    - 格式為：`https://<你的專案ID>.supabase.co/auth/v1/callback`
        

填寫完畢後，點擊下方的 **Register application（註冊應用程式）**。

## 4：取得 Client ID 與 Client Secret 回填至 Supabase

註冊成功後，頁面會跳轉到該 App 的管理介面，你會看到：

1. **Client ID：** 直接複製這一串字串。
    
2. **Client Secret：** 預設不會顯示，你需要點擊 **Generate a new client secret（生成新的客戶端密鑰）** 按鈕，這時候會出現一串長密鑰。


## 為什麼不需要確認已授權的 JavaScript 來源

GitHub 的 OAuth 設計相對單純。GitHub 認為「只要請求過後的 **Authorization callback URL（重新導向 URL）**與後台設定的完全一致，那這個請求就是安全的」。

因此，GitHub 在建立 OAuth App 時，**只需要填寫 Homepage URL 和 Authorization callback URL** 即可，並不需要像 Google 一樣分開宣告 JavaScript 來源。