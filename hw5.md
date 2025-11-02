classDiagram
%% ========== 主系統類別 ==========
class CampusHelperSystem {
    -String 系統名稱
    -String 版本
    -Date 當前日期
    +初始化()
    +關閉()
    +取得系統狀態()
}

%% ========== 登入/使用者 ==========
class AuthModule {
    -boolean 已登入
    +登入(email, 密碼)
    +登出()
    +驗證權限(token)
}
class 使用者 {
    -UUID 使用者ID
    -String 姓名
    -String email
    -String 學號
    +更新基本資料()
    +啟用提醒(分鐘)
}

%% ========== 課表管理模組 ==========
class ScheduleModule {
    -List~ScheduleEntry~ 課表清單
    +新增課程(課程)
    +編輯課程(課程)
    +刪除課程(課程ID)
    +檢查時間重疊(課程): boolean
}
class 課程 {
    -UUID 課程ID
    -String 名稱
    -String 老師
    -String 地點
    +設定時間(時段)
}
class 時段 {
    -String 星期
    -String 開始時間
    -String 結束時間
}
class ScheduleEntry {
    -UUID 項目ID
    -課程 課程
    -時段 時段
}

%% ========== 成績管理模組 ==========
class GradeModule {
    -List~GradeEntry~ 成績清單
    +新增成績(課程ID, 分數, 權重)
    +更新成績(成績ID, 分數, 權重)
    +刪除成績(成績ID)
    +計算總平均(): float
    +計算加權平均(): float
}
class GradeEntry {
    -UUID 成績ID
    -課程 課程
    -float 分數
    -float 權重
    -String 學期
}

%% ========== 提醒排程模組 ==========
class ReminderModule {
    -ReminderSettings 設定
    -List~Reminder~ 今日提醒
    +產生今日提醒(課表)
    +發送提醒(提醒)
    +重試發送(提醒)
}
class ReminderSettings {
    -int 提前分鐘
    -boolean 啟用
    +更新(分鐘, 啟用)
}
class Reminder {
    -UUID 提醒ID
    -DateTime 時間
    -String 訊息
    -String 狀態  "PENDING|SENT|FAILED"
}

%% ========== 視覺化/報表 ==========
class VisualizationModule {
    -Dashboard 儀表板
    +載入平均結果(平均資料)
    +顯示長條圖(資料)
    +顯示圓餅圖(資料)
    +即時更新()
}
class Dashboard {
    -String 標題
    -int 重新整理頻率
    -boolean 自動更新
    +初始化()
    +重新整理()
    +顯示圖表(圖表)
}
class Chart {
    -String 類型
    -String 標題
    -List~DataPoint~ 資料點
    +建立折線圖()
    +建立長條圖()
    +更新資料(資料)
    +渲染()
}
class DataPoint {
    -float x值
    -float y值
    -String 標籤
    -Date 時間戳記
}

%% ========== 資料匯出/備份 ==========
class ExportModule {
    -String 輸出目錄
    -String 檔案格式
    +匯出課表CSV(課表)
    +匯出成績CSV(成績)
    +匯出統計報告CSV(平均)
    +備份所有資料(zip檔名)
}
class OutputFile {
    -String 檔名
    -String 路徑
    -String 類型
    -Date 建立日期
    -long 檔案大小
    +建立()
    +寫入(資料)
    +關閉()
    +取得路徑()
}

%% ========== 外部系統（推播/郵件等） ==========
class 通知服務 {
    +推播(訊息, 接收者)
    +寄信(主旨, 內容, 收件者)
}

%% ========== 關聯 ==========
CampusHelperSystem "1" --> "1" AuthModule : 包含
CampusHelperSystem "1" --> "1" ScheduleModule : 包含
CampusHelperSystem "1" --> "1" GradeModule : 包含
CampusHelperSystem "1" --> "1" ReminderModule : 包含
CampusHelperSystem "1" --> "1" VisualizationModule : 包含
CampusHelperSystem "1" --> "1" ExportModule : 包含

AuthModule "1" --> "1" 使用者 : 目前使用者
使用者 "1" --> "1" ReminderSettings : 設定

ScheduleModule "1" --> "*" ScheduleEntry : 管理
ScheduleEntry "1" --> "1" 課程 : 參照
課程 "1" --> "1" 時段 : 使用

GradeModule "1" --> "*" GradeEntry : 管理
GradeEntry "*" --> "1" 課程 : 對應

ReminderModule "1" --> "1" ReminderSettings : 讀取
ReminderModule "1" --> "*" Reminder : 產生
ReminderModule --> 通知服務 : 發送

VisualizationModule "1" --> "1" Dashboard : 使用
VisualizationModule --> Chart : 生成
Chart "1" --> "*" DataPoint : 包含

ExportModule "1" --> "*" OutputFile : 產生
ExportModule --> ScheduleModule : 讀取課表
ExportModule --> GradeModule : 讀取成績
ExportModule --> VisualizationModule : 讀取統計
