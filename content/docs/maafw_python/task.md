---
title: Task模組
type: docs
sidebar:
  open: true
---

# MaaFramework Tasker 接口說明

**Tasker** 類負責管理任務（Task）的創建、狀態查詢、等待與資源綁定等操作，是 MaaFramework 中任務調度的核心模組。通過 Tasker，使用者可以提交新的任務、停止當前任務、查詢任務狀態以及設置全局選項（如日誌路徑、截圖保存等）。

---

## 1. Tasker 的初始化與銷毀

### `__init__(notification_handler=None, handle=None)`

**用途：**  
初始化 Tasker 實例。  
- 若傳入現有的 handle，則 Tasker 為非擁有者（_own=False），不負責銷毀該 handle。  
- 若未傳入 handle，則 Tasker 會調用底層函數 MaaTaskerCreate 並傳入通知處理器參數（通過 NotificationHandler._gen_c_param 生成），同時標記自己為擁有者。

**參數說明：**  
- **notification_handler** (NotificationHandler, 可選)：用於處理框架內部各種通知。  
- **handle** (MaaTaskerHandle, 可選)：若已有底層 handle，則直接使用，否則 Tasker 自行創建。

**異常處理：**  
- 若創建 handle 失敗，則拋出 RuntimeError。

### `__del__()`

**用途：**  
在 Tasker 實例被銷毀時，如果 Tasker 是擁有者（_own 為 True），則調用 MaaTaskerDestroy 釋放底層資源。

---

## 2. 綁定資源與控制器

### `bind(resource, controller) -> bool`

**用途：**  
將資源（Resource）和控制器（Controller）與 Tasker 進行綁定，以便後續任務執行時可以使用這些對象。

**參數說明：**  
- **resource** (Resource)：待綁定的資源對象，通常包含待測試項目或設備信息。  
- **controller** (Controller)：待綁定的控制器對象，用於模擬操作或截屏操作。

**返回值：**  
- 返回布林值，若同時成功綁定資源和控制器則返回 `True`。

**注意：**  
- 為避免垃圾回收，綁定後會將 resource 和 controller 分別保存在 `_resource_holder` 與 `_controller_holder` 中。

---

## 3. 任務與狀態查詢

### 屬性 `resource` 與 `controller`

**用途：**  
- **resource**：返回 Tasker 當前綁定的 Resource 實例。  
- **controller**：返回 Tasker 當前綁定的 Controller 實例。

**異常處理：**  
- 如果未能成功獲取對應的 handle，則拋出 RuntimeError。

### 屬性 `inited` 與 `running`

**用途：**  
- **inited**：檢查 Tasker 是否已初始化。  
- **running**：檢查是否有任務正在運行。

這兩個屬性均通過調用底層函數 MaaTaskerInited 與 MaaTaskerRunning 返回相應狀態（布林值）。

### `post_task(entry, pipeline_override={}) -> JobWithResult`

**用途：**  
提交一個新任務。  
- `entry` 參數指定了任務的入口腳本或配置。  
- `pipeline_override` 為可選的管線配置覆蓋，用於調整任務執行過程中的某些參數或流程。

**返回值：**  
- 返回一個 `JobWithResult` 實例，用於後續等待任務完成、查詢任務狀態或獲取結果。

### `post_stop() -> Job`

**用途：**  
提交一個停止命令，用於中止當前任務。  
**返回值：**  
- 返回一個 `Job` 實例，使用者可通過該對象等待停止操作完成。

### `get_latest_node(name) -> Optional[NodeDetail]`

**用途：**  
查詢最新的節點信息（NodeDetail），通常用於獲取最新一次任務執行過程中某個節點的詳細狀態或結果。

**參數說明：**  
- **name** (str)：指定節點名稱。

**返回值：**  
- 若找到對應節點，則返回 NodeDetail；否則返回 None。

### `clear_cache() -> bool`

**用途：**  
清理 Tasker 的內部緩存（例如任務狀態緩存等）。

**返回值：**  
- 成功則返回 `True`，否則返回 `False`。

---

## 4. 全局配置設置（靜態方法）

Tasker 提供了多個靜態方法，用於設定全局選項，這些選項會影響 MaaFramework 的運行行為。

### `set_log_dir(path) -> bool`

**用途：**  
設置全局日誌目錄。  
**參數說明：**  
- **path** (str 或 Path)：日誌存放目錄路徑。  
**返回值：**  
- 成功設置返回 `True`。

### `set_save_draw(save_draw) -> bool`

**用途：**  
設置是否保存繪製結果（如截圖中標記的區域等）。  
**參數說明：**  
- **save_draw** (bool)：是否保存繪製，`True` 表示保存。  
**返回值：**  
- 成功返回 `True`。

### `set_recording(recording) -> bool`

**用途：**  
設置是否記錄所有截圖及操作流程（錄製模式）。  
**參數說明：**  
- **recording** (bool)：`True` 表示啟用錄製。  
**返回值：**  
- 成功返回 `True`。

### `set_stdout_level(level) -> bool`

**用途：**  
設定輸出到標準輸出（stdout）的日誌級別。  
**參數說明：**  
- **level** (LoggingLevelEnum)：指定的日誌級別。  
**返回值：**  
- 成功返回 `True`。

### `set_show_hit_draw(show_hit_draw) ->
