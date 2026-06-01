本地自架（Self-hosting）密碼管理服務是一個保護隱私的絕佳選擇。既然你希望用一個主密碼（Master Password）來解鎖所有密碼

Bitwarden 的強大功能，但家裡的伺服器（如 NAS、樹莓派）記憶體有限，**Vaultwarden** 是絕對的首選。

- **它是什麼：** 它是用 Rust 語言重寫的 Bitwarden 後端伺服器。
- **優點：**
    - **極度輕量：** 記憶體佔用通常只需幾十 MB（原版 Bitwarden 需要幾 GB）。
    - **完全相容：** 可以直接使用 Bitwarden 官方的網頁、瀏覽器擴充功能、手機 App 及桌面軟體。
    - **免費解鎖進階功能：** 直接內建了原版需要付費的組織分享、兩步驟驗證（TOTP）產生器、密碼附件等功能。

# 需要資料庫嗎
Vaultwarden 已經自帶資料庫

Vaultwarden 內建了 **SQLite** 資料庫。當你在 Mac 的 Docker 跑起 Vaultwarden 時，它會自動在容器內建立一個輕量型的資料庫檔案。對個人或家庭幾個人使用來說，SQLite 的效能與穩定度已經綽綽有餘。


```bash
mkdir -p ~/Projects/app/vaultwarden
cd ~/Projects/app/vaultwarden
```

```yaml
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true  # 啟用網頁即時同步功能
    volumes:
      # 將資料存在當前目錄下的 vw-data 資料夾
      - ./vw-data:/data
    ports:
      # 左邊的 8080 是你 Mac 瀏覽器要輸入的 Port，可以改成你喜歡的（如 80 或 8888）
      # 右邊的 80 是容器內部固定 Port，請勿修改
      - "8080:80"

```

```bash
docker compose up -d
```
# vaultwarden Insecure URL not allowed. All URLs must use HTTPS.

Vaultwarden 非常常見的「安全保護機制

**密碼管理是非常敏感的資料**，Vaultwarden 官方和現代瀏覽器（Chrome、Edge、Safari）都強制規定：**所有連接到 Vaultwarden 的連線都必須使用 HTTPS 加密。**

當你使用 Mac 的區域網路 IP（例如 `http://192.168.x.x:8080`）或是外部網址去存取 HTTP 時，系統就會直接阻擋建立帳戶。

# 使用 Nginx Proxy Manager / Caddy（純內網 SSL）

如果你只想在**家裡的區域網路**使用，絕對不對外公開，但希望手機 App 能連，你需要自己簽發憑證：

1. 可以用 Docker 加掛一個 **Nginx Proxy Manager** 或 **Caddy** 容器。
2. 搭配自簽憑證（Self-signed SSL


```yaml
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true
    volumes:
      - ./vw-data:/data

  nginx-proxy-manager:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy-manager
    restart: always
    ports:
      - '80:80'    # HTTP 流量
      - '443:443'  # HTTPS 流量 (未來你用這個連進來)
      - '81:81'    # NPM 的管理後台介面
    volumes:
      - ./npm-data:/data
      - ./npm-letsencrypt:/etc/letsencrypt
```

## 啟動

- 執行 `docker compose up -d`。
- **登入 NPM 後台：** 瀏覽器打開 `http://localhost:81`。
    - 預設帳號：`admin@example.com`
    - 預設密碼：`changeme`

## 生成給 IP 用的憑證

因為剛才的憑證是簽給 `localhost` 的，我們現在重新生成一張同時支援 `localhost`、`127.0.0.1` 和你 Mac 內網 IP 的憑證。

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout domain.key -out domain.crt \
  -subj "/CN=127.0.0.1" \
  -addext "subjectAltName = DNS:localhost, IP:127.0.0.1"
```

## 去 NPM 修改設定（關鍵步驟）

- 上傳憑證：
    - 登入 NPM (nginx-proxy-manager) 後台（`http://localhost:81`）。
    - 到 **Certificates**，把舊的刪掉，或是點 **Add SSL Certificate -> Custom** 新增一個，名稱填 `IP-SSL`，並上傳剛剛新生成的 `domain.key` 和 `domain.crt`。

## 修改 Mac 的 hosts 檔案（讓 Mac 認得這個名字）
```bash
sudo nano /etc/hosts
```

```
127.0.0.1 vaultwarden.local 
```

 按 `Control + O` 然後按 `Enter` 存檔，再按 `Control + X` 離開。
## 設定反向代理（Proxy Host）：

- 點選 **Hosts** -> **Proxy Hosts** -> **Add Proxy Host**。
- **Domain Names:** 輸入你想要的本機域名（例如 自訂假域名 `vaultwarden.local` 
- **Forward Scheme:** 選擇 `http`。
- **Forward Host名:** 輸入 `vaultwarden`（直接填 yml 裡的服務名稱）。
- **Forward Port:** 輸入 `80`。
- Proxy Host 旁邊的頁籤切換到 **SSL** 頁籤，選擇你剛剛建立的憑證，並勾選 **Force SSL**。
	- 把 **Force SSL**、**HTTP/2 Support** 和 **HSTS Enabled** 這幾個開關都勾選（開啟）


打開瀏覽器，訪問： **`https://vaultwarden.local`**

瀏覽器應該會安全地跳出「您的連線不是不公開連線」的警告，點選 **「進階」->「繼續前往 不安全）**

# ERR_SSL_UNRECOGNIZED_NAME_ALERT


無法連上這個網站 
https://localhost/ 的網頁可能暫時離線，或是已經遷移到另一個網址。 
ERR_SSL_UNRECOGNIZED_NAME_ALERT

`ERR_SSL_UNRECOGNIZED_NAME_ALERT` 是一個非常經典的 Nginx 錯誤（又叫 SSL SNI 錯誤）


如果Proxy Host Domain Name 填寫 localhost 或 127.0.0.1 高機率會報這個錯

因為 **Nginx 收到你的 HTTPS 請求了，但它認不出 `localhost` 這個名字，所以不知道該拿哪張憑證敷衍你，直接把連線拒絕了。**

NPM 內部在處理 `127.0.0.1` 或 `localhost` 時，與 Mac 本機（OrbStack）的網路對映產生了衝突
