
# 第一步：準備 GCP 基礎設施

我們需要建立一個儲存桶（Bucket）並準備好 IAM 權限。

## 1. **建立 GCS 儲存桶**：
    
    - 在 GCP Console 建立一個儲存桶（例如命名為：`my-memos-data-bucket`）。
        
    - 建議選擇與你 Cloud Run 相同的區域（例如 `asia-east1` 台灣），以降低延遲與流量成本。

[[2026-06-02-Cloud Storage (GCS)]]


## 2. **準備或確認服務帳戶（Service Account）**：
    
    - Cloud Run 執行時需要一個身份去讀寫 GCS。你可以使用預設的 Compute Engine 服務帳戶，或者單獨建立一個。
        
    - 確保該服務帳戶擁有該儲存桶的 **儲存空間物件管理員 (Storage Object Admin)** 權限。

[[2026-06-02-IAM]]
# 第二步：設定 Cloud Run 部署



Memos 的官方 Image 已經非常標準，我們直接在 Cloud Run 建立服務。

## 1. 建立服務基本設定

- **服務名稱**：`memos`
    
- **部署管道**：選擇「從現有容器映像檔部署一個修訂版本」。
    
- **映像檔網址**：`ghcr.io/usememos/memos:latest` （或者你可以先拉到你的 Artifact Registry 中再引流）。
    

## 2. 進階設定：容量與通訊埠

- **容器通訊埠 (Container Port)**：改為 **`5230`**（Memos 預設埠）。
    
- **CPU 分配**：為了防止 SQLite 寫入時遇到硬體卡頓，建議選擇「一律分配 CPU」以獲得更穩定的回應，或者人少時選「僅在要求處理期間分配 CPU」來省錢。
    

## 3. 進階設定：掛載 Cloud Storage (關鍵步驟 )

滑到下方的 **「磁碟區 (Volumes)」** 分頁：

1. 點擊 **「新增磁碟區 (Add Volume)」**。
    
2. **磁碟區類型**：選擇 **Cloud Storage 儲存桶**。
    
3. **磁碟區名稱**：自訂（例如 `memos-storage`）。
    
4. **選取儲存桶**：選擇你剛剛建立的 `my-memos-data-bucket`。
    

接著往下一點，找到 **「容器掛載項目 (Container Mounts)」**：

1. 點擊 **「掛載磁碟區 (Mount Volume)」**。
    
2. **選擇磁碟區**：選剛才建立的 `memos-storage`。
    
3. **掛載路徑 (Mount Path)**：輸入 **`/var/opt/memos`**。
    

> **💡 為什麼是 `/var/opt/memos`？** 因為這是 Memos 容器內部預設尋找 SQLite 資料庫 (`memos_prod.db`) 的地方。透過 Cloud Run FUSE，Memos 讀寫這個資料夾時，實際上是直接背後同步到你的 GCS 儲存桶中！

[[2026-06-02-Cloud Run]]
# 第三步：部署與訪問

點擊 **「建立 (Create)」**，等待 Cloud Run 部署完成。

- 部署完成後，GCP 會直接發配給你一組 **HTTPS 的官方網址**（例如 `[https://memos-xxxx-de.a.run.app](https://memos-xxxx-de.a.run.app)`）。
    
- 直接點擊該網址，就能看到高顏值的 Memos 初始化註冊畫面了！

# 第四步：安全建立日常帳號

「完全不公開/私密基地」狀態，請用 Host 身分登入後，立刻進行以下操作：

- 點擊左下角的設定（齒輪）。
    
- 進入 **「管理 (Administration)」** ➔ **「成員 (Members)」**。
    
- 檢查目前列表，確保**只有你這一個帳號**。
    
- 點擊 **「系統 (System)」** 分頁，找到 **「允許訪客註冊 (Allow sign up)」** 並將它**關閉（Turn Off）**。
	

| **設定項目**                                      | **建議狀態** | **為什麼要這樣設定？**                                                                                    |
| --------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------ |
| **Disallow user registration**<br>(禁止使用者註冊)   | 開啟 (ON)  | 大門口就不會出現「註冊」按鈕，任何陌生人都無法自己建立帳號跑進你的系統。                                                             |
| **Disallow password auth**<br>(禁止密碼登入)        | 關閉 (OFF) | **請保持關閉。** 如果你開啟了這個（禁止密碼登入），系統會強制要求使用像是 GitHub OAuth 等第三方單一登入（SSO）。既然你是個人用，保持關閉才能讓你繼續用自訂的帳號密碼登入。 |
| **Disallow changing username**<br>(禁止變更使用者名稱) | 開啟 (ON)  | 為了防止以後如果你有開帳號給信任的朋友，他們去亂改登入用的帳號（Username），開啟它可以鎖定帳號結構，提升系統穩定性。你自己如果想改，隨時回來關掉就能改。                 |
| **Disallow changing nickname**<br>(禁止變更暱稱)    |          | 暱稱（Nickname）只是顯示在筆記畫面上的名字（例如：Uncle）。開啟就會鎖死大家不能改名。通常個人使用維持預設即可。                                   |

 外面的人無法建立帳號，此時只有你這個 `Host` 可以在後台「內定」新成員：


#### 用 Host 身分登入，手動建立新帳號

1. 點擊 Memos 左下角的 **「設定 (Settings)」** ➔ 進入 **「成員 (Members)」**。
    
2. 點擊右上角的 **「新增成員 (Add)」** 按鈕。
    
3. 輸入日常分身的名字（例如 `userdaily`）、暱稱、以及一組密碼。

# 工程師的架構優化與避坑指南

## 1. **Cloud Run 的最大實例數 (Max Instances) 限制**： 

因為 Memos 預設使用的是 **SQLite**，SQLite **不支援多個容器同時寫入（會觸發 Database Locked）**。

- **強烈建議**：在 Cloud Run 的「調整調整 (Scaling)」設定中，將 **最大實例數限制為 `1`**。這樣能確保同一時間只有一個 Memos 容器在跑，完美保護 SQLite 的資料一致性。
    
- 如果未來你的社群媒體真的做大、並發量極高，請在 GCP 補一組 **Cloud SQL (PostgreSQL)**，並透過環境變數將 Memos 導向 Cloud SQL，屆時 Cloud Run 就能開到 100 台進行橫向擴充。

## 2. **省錢冷啟動 (Cold Start)**： 

如果你把「最小實例數」設為 `0`，當完全沒人造訪時，GCP 會把容器關掉（此時完全不收費）。當第一個使用者連入時，啟動大約需要 2~3 秒的時間（冷啟動），對於個人日記或小社群而言，這點代價換取**接近免費的託管成本**非常划算。
    
## 3. **備份超簡單**： 

既然資料都進了 GCS，你只需要在 GCS 上開啟 **「版本控管 (Versioning)」** 或者是設定 **「生命週期管理 (Lifecycle)」** 來做定期備份，就再也不用擔心資料庫遺失的問題了！
