## 系統環境圖 (DFD)

```mermaid
flowchart LR
    %% === 外部實體 ===
    User([使用者])
    Notif([通知服務])
    Auth([登入驗證系統])
    DB[(系統資料庫)]

    %% === 系統邊界 ===
    subgraph System["課課到，分分算 系統"]
        direction TB
        SYS[課表與成績管理系統]
    end

    %% === 外部資料流 ===
    User -->|登入帳號 / 密碼| Auth
    Auth -->|登入驗證結果| SYS

    User -->|輸入課表 / 成績資料| SYS
    SYS -->|上課提醒 / 成績結果| User

    SYS -->|通知排程| Notif
    Notif -->|發送課程提醒| User

    SYS -->|讀取 / 儲存資料| DB
