
classDiagram
direction TB

class 使用者 {
  +使用者ID: String
  +姓名: String
  +帳號: String
  +密碼: String
  --
  +登入(): Boolean
  +登出(): Void
}

class 課程 {
  +課程ID: String
  +課程名稱: String
  +授課教師: String
  +上課地點: String
  +上課時間: String
}

class 課表 {
  +學期: String
  +課程清單: List~課程~
  --
  +新增課程(課程)
  +刪除課程(課程)
  +產生提醒(): String
}

class 成績 {
  +課程ID: String
  +分數: Float
  --
  +計算等第(): String
}

class 成績管理 {
  +成績清單: List~成績~
  --
  +新增成績(成績)
  +計算平均(): Float
}

使用者 "1" -- "1" 課表 : 擁有
使用者 "1" -- "1" 成績管理 : 擁有
課表 "1" -- "多" 課程 : 包含
成績管理 "1" -- "多" 成績 : 管理
課程 "1" -- "1" 成績 : 對應

sequenceDiagram
actor 使用者
participant 系統介面
participant 資料庫

使用者 ->> 系統介面: 輸入帳號與密碼
系統介面 ->> 資料庫: 查詢帳號與密碼
資料庫 -->> 系統介面: 回傳驗證結果
系統介面 -->> 使用者: 顯示登入成功畫面

flowchart TD
A[開始] --> B[輸入帳號密碼]
B --> C{驗證成功?}
C --是--> D[進入主畫面]
C --否--> E[顯示錯誤訊息]
E --> B
D --> F[結束]

