classDiagram
    direction LR

    class 使用者 {
      +使用者ID: UUID
      +姓名: string
      +email: string
      +登入()
    }

    class 課程 {
      +課程ID: UUID
      +名稱: string
      +時間: 時段
      +地點: string
      +老師: string
    }

    class 課表 {
      +課表ID: UUID
      +學期: string
    }

    class 成績 {
      +成績ID: UUID
      +分數: float
      +權重: float
      +學期: string
    }

    class 提醒設定 {
      +提前分鐘: int
      +啟用: bool
    }

    class 提醒 {
      +提醒ID: UUID
      +時間: datetime
      +訊息: string
      +狀態: enum
    }

    class 課表服務 {
      +新增課程(u:使用者, c:課程)
      +編輯課程(c:課程)
      +刪除課程(c:課程)
    }

    class 成績服務 {
      +記錄成績(u:使用者, c:課程, 分數, 權重)
      +計算平均(u:使用者): float
    }

    class 提醒服務 {
      +建立排程(課表, 提醒設定)
      +發送提醒(u:使用者, 提醒)
    }

    class 通知服務
    class 登入驗證系統

    %% 樣式：外部系統以灰底表示
    classDef external fill:#eee,stroke:#999,color:#333
    class 通知服務,登入驗證系統 external

    使用者 "1" o-- "1" 課表 : 擁有
    課表 *-- "0..*" 課程 : 包含
    使用者 "1" o-- "0..*" 成績 : 產生
    課程 "1" o-- "0..*" 成績 : 對應
    使用者 "1" o-- "1" 提醒設定 : 設定
    使用者 "1" o-- "0..*" 提醒 : 接收

    課表服務 --> 課表
    課表服務 --> 課程
    成績服務 --> 成績
    成績服務 --> 課程
    提醒服務 --> 提醒
    提醒服務 --> 通知服務 : 發送

    使用者 --> 登入驗證系統 : 驗證

@startuml
skinparam classFontName 微軟正黑體
skinparam shadowing false
left to right direction

' === 核心系統（模組視角） ===
class 課課到系統 as System {
  +系統名稱: string
  +版本: string
  +啟動()
}

class 課表管理模組 as ScheduleModule
class 成績管理模組 as GradeModule
class 提醒排程模組 as ReminderModule
class 視覺化模組 as VisualizationModule

System *-- ScheduleModule : 包含
System *-- GradeModule : 包含
System *-- ReminderModule : 包含
System *-- VisualizationModule : 包含

' === 實體/資料類別 ===
class 課表 {
  +課表ID: UUID
  +學期: string
}
class 課程 {
  +課程ID: UUID
  +名稱: string
  +時間: 時段
  +地點: string
  +老師: string
}
class 成績 {
  +成績ID: UUID
  +分數: float
  +權重: float
}
class 提醒 {
  +提醒ID: UUID
  +時間: datetime
  +訊息: string
}
class 使用者 {
  +使用者ID: UUID
  +姓名: string
  +email: string
}

' === 外部系統 ===
class 通知服務 <<external>>
class 登入驗證系統 <<external>>

' === 模組與資料/外部互動 ===
ScheduleModule *-- 課表 : 包含
課表 *-- 課程 : 包含
GradeModule --> 成績 : 使用
GradeModule --> 課程 : 參照
ReminderModule --> 提醒 : 產生
ReminderModule ..> 通知服務 : 發送
VisualizationModule --> 成績 : 讀取
VisualizationModule --> 課程 : 讀取

使用者 --> 登入驗證系統 : 登入/驗證
使用者 --> ScheduleModule : 管理課程
使用者 --> GradeModule : 輸入成績
使用者 --> ReminderModule : 設定提醒
使用者 --> VisualizationModule : 檢視圖表

@enduml

sequenceDiagram
    autonumber
    actor 使用者
    participant 前端 as 前端介面
    participant 後端 as 後端API
    participant Svc as 課表服務
    participant DB as 資料庫

    使用者->>前端: 新增/編輯/刪除課程
    前端->>後端: /api/schedule (課程資料)
    後端->>Svc: 驗證與處理(課程資料)
    Svc->>DB: 讀取現有課表
    DB-->>Svc: 課表/課程清單
    Svc->>Svc: 檢查時間重疊
    alt 時間重疊
        Svc-->>後端: 回傳錯誤(時間衝突)
        後端-->>前端: 422 Unprocessable
        前端-->>使用者: 提示：課程時間重疊
    else 無重疊
        Svc->>DB: 寫入課程異動
        DB-->>Svc: OK
        Svc-->>後端: 更新後課表
        後端-->>前端: 回傳最新課表
        前端-->>使用者: 顯示課表
    end
flowchart LR
    A[開始] --> B[輸入課程資料]
    B --> C{時間是否重疊?}
    C -- 是 --> D[提示錯誤並返回編修]
    C -- 否 --> E[更新資料庫]
    E --> F[回傳最新課表]
    F --> G[結束]
    D --> B

sequenceDiagram
    autonumber
    actor 使用者
    participant 前端 as 前端介面
    participant 後端 as 後端API
    participant Gsvc as 成績服務
    participant DB as 資料庫

    使用者->>前端: 輸入成績(分數、權重)
    前端->>後端: /api/grades (成績資料)
    後端->>Gsvc: 驗證與寫入
    alt 格式錯誤
        Gsvc-->>後端: 回傳錯誤(格式不正確)
        後端-->>前端: 400 Bad Request
        前端-->>使用者: 提示重新輸入
    else 格式正確
        Gsvc->>DB: 儲存成績
        DB-->>Gsvc: OK
        Gsvc->>DB: 查詢使用者所有成績
        DB-->>Gsvc: 成績清單
        Gsvc->>Gsvc: 計算總平均/加權平均
        Gsvc-->>後端: 成績與平均
        後端-->>前端: 回傳結果
        前端-->>使用者: 顯示成績與平均
    end
flowchart LR
    A[開始] --> B[輸入成績]
    B --> C{格式有效?}
    C -- 否 --> D[提示錯誤並重填]
    C -- 是 --> E[寫入資料庫]
    E --> F[讀取所有成績]
    F --> G[計算平均]
    G --> H[顯示結果]
    H --> I[結束]
    D --> B

sequenceDiagram
    autonumber
    participant 排程器 as 提醒排程
    participant 提醒服務
    participant DB as 資料庫
    participant 通知 as 通知服務
    actor 使用者

    排程器->>提醒服務: 觸發(分鐘/時段)
    提醒服務->>DB: 讀取課表與提醒設定
    DB-->>提醒服務: 課程清單/設定
    提醒服務->>提醒服務: 生成今日提醒
    提醒服務->>通知: 發送提醒(訊息, 時間)
    通知-->>使用者: 推播/通知
    note over 提醒服務,DB: 若課程被修改 → 下次排程時重新計算
flowchart LR
    A[開始(排程觸發)] --> B[讀取課表與提醒設定]
    B --> C{有即將開始的課程?}
    C -- 否 --> H[結束]
    C -- 是 --> D[生成提醒內容]
    D --> E[呼叫通知服務發送]
    E --> F{發送成功?}
    F -- 否 --> G[記錄失敗並重試排程]
    F -- 是 --> H[結束]
