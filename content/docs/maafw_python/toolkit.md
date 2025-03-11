---
title: Toolkit模組
type: docs
sidebar:
  open: true
---

## MaaFramework Toolkit 接口說明

MaaFramework 的 **Toolkit** 模組主要提供與外部環境交互、設備搜索以及自定義擴展（如自定義識別和自定義動作）相關的接口。本文件將用簡單易懂的語言逐一介紹 toolkit.py 中各個接口的用途和參數的含義。

---

## 1. 初始化配置選項

### `init_option(user_path, default_config)`

**用途：**  
初始化工具箱的配置信息。這個接口會根據用戶指定的路徑與預設配置初始化相關環境，為後續操作做準備。

**參數說明：**  
- **user_path** (str 或 Path)：用戶存放配置文件或工作數據的目錄路徑。  
- **default_config** (dict, 可選)：預設配置的字典（例如默認參數、環境變量等），默認為空字典。

**返回值：**  
- 返回一個布林值，若初始化成功則返回 `True`，否則返回 `False`。

---

## 2. 搜尋 ADB 設備

### `find_adb_devices(specified_adb=None)`

**用途：**  
搜索通過 ADB (Android Debug Bridge) 連接的 Android 設備。這個接口可以自動發現可用設備，便於進行設備管理或自動化測試。

**參數說明：**  
- **specified_adb** (str 或 Path, 可選)：若提供，則指定某個 ADB 執行檔的路徑來進行設備搜索；若未指定，則使用默認的 ADB 工具進行搜索。

**返回值：**  
- 返回一個 `AdbDevice` 物件列表，每個物件包含：  
  - **name**：設備名稱。  
  - **adb_path**：ADB 執行檔所在路徑（以 Path 物件表示）。  
  - **address**：設備地址或標識。  
  - **screencap_methods**：用於截屏的方法（以整數形式表示各種方法的位標識）。  
  - **input_methods**：模擬輸入的方式。  
  - **config**：設備相關的其他配置參數，以字典形式存儲。

---

## 3. 搜尋桌面窗口

### `find_desktop_windows()`

**用途：**  
查找當前桌面上的所有窗口。該接口主要用於桌面自動化或者窗口管理相關的應用場景。

**參數說明：**  
- 無參數。

**返回值：**  
- 返回一個 `DesktopWindow` 物件列表，每個物件包含：  
  - **hwnd**：窗口的句柄（用 ctypes.c_void_p 表示）。  
  - **class_name**：窗口的類名。  
  - **window_name**：窗口的標題或名稱。

---

## 4. 註冊自定義識別接口

### `pi_register_custom_recognition(name, recognition, inst_id=0)`

**用途：**  
通過該接口，使用者可以向 MaaFramework 項目接口中註冊自定義識別邏輯。這通常用於圖像識別、特徵檢測等任務中。

**參數說明：**  
- **name** (str)：為自定義識別接口命名，必須是唯一標識。  
- **recognition** (CustomRecognition)：繼承自 `CustomRecognition` 抽象類的實例，其內部實現了識別算法。  
- **inst_id** (int, 可選)：實例 ID，默認為 0；在多實例環境下用於區分不同實例。

**返回值：**  
- 返回布林值，若註冊成功則返回 `True`，否則返回 `False`。

*註：內部會把傳入的 recognition 實例保存在一個靜態字典中，避免被垃圾回收。*

---

## 5. 註冊自定義動作接口

### `pi_register_custom_action(name, action, inst_id=0)`

**用途：**  
通過該接口，使用者可以向 MaaFramework 項目接口中註冊自定義動作。自定義動作常用於在識別結果基礎上執行特定操作（例如點擊、拖拽或其它自定義行為）。

**參數說明：**  
- **name** (str)：為自定義動作接口命名，必須唯一。  
- **action** (CustomAction)：繼承自 `CustomAction` 抽象類的實例，其內部實現了動作執行的邏輯。  
- **inst_id** (int, 可選)：實例 ID，默認為 0。

**返回值：**  
- 返回布林值，若註冊成功則返回 `True`，否則返回 `False`。

*同樣，該接口也會將 action 實例保存在靜態字典中以避免被垃圾回收。*

---

## 6. 透過 CLI 執行項目接口

### `pi_run_cli(resource_path, user_path, directly=False, notification_handler=None, inst_id=0)`

**用途：**  
該接口用於以命令行模式啟動 MaaFramework 的項目接口，通常用於批量處理或腳本化執行。通過此接口，可以指定資源路徑與用戶配置目錄，同時可設置是否直接執行以及通知處理器。

**參數說明：**  
- **resource_path** (str 或 Path)：資源包所在路徑。  
- **user_path** (str 或 Path)：用戶工作目錄路徑。  
- **directly** (bool, 可選)：是否直接執行，默認為 `False`。  
- **notification_handler** (NotificationHandler, 可選)：可選的通知處理器實例，用於接收運行過程中的各種通知。  
- **inst_id** (int, 可選)：實例 ID，默認為 0。

**返回值：**  
- 返回布林值，若 CLI 模式啟動成功則返回 `True`，否則返回 `False`。

---

## 7. 私有接口：設置 API 屬性

### `_set_api_properties()`

**用途：**  
這是一個內部方法，用於設置 toolkit 模組中所有對底層函數庫的接口原型（包括返回值類型和參數類型）。利用 Python 的 ctypes 模組來正確調用 C 語言庫中的函數。

**參數說明：**  
- 無參數，此方法在每個公有接口被調用前都會檢查並初始化（只會初始化一次）。

**返回值：**  
- 無返回值，僅用於內部配置。

---

## 補充：內部數據結構

### AdbDevice

這是一個數據類，封裝了 ADB 設備的相關信息。主要屬性包括：
- **name**：設備名稱。  
- **adb_path**：ADB 執行檔的路徑。  
- **address**：設備地址或標識。  
- **screencap_methods**：支持的截屏方法，通常以位掩碼形式表示。  
- **input_methods**：支持的輸入方法，同樣以整數表示。  
- **config**：包含設備其他配置的字典。

### DesktopWindow

這是一個數據類，封裝了桌面窗口的相關信息。主要屬性包括：
- **hwnd**：窗口句柄（用於操作系統級的窗口控制）。  
- **class_name**：窗口類名。  
- **window_name**：窗口的標題或名稱。

### 靜態保留結構

- **_pi_custom_recognition_holder** 與 **_pi_custom_action_holder**：用於存放自定義識別和動作對象的字典，防止它們被 Python 垃圾回收。  
- **_pi_notification_handler**：保存通知處理器的引用，便於在 CLI 運行時回調通知。

---

# 總結

**Toolkit** 模組通過一系列靜態方法提供了：
- 配置初始化與管理（`init_option`）；
- 設備與窗口搜索（`find_adb_devices`、`find_desktop_windows`）；
- 自定義識別與動作接口的註冊（`pi_register_custom_recognition`、`pi_register_custom_action`）；
- 以命令行方式啟動項目接口（`pi_run_cli`）；
- 以及內部的 API 屬性設置方法（`_set_api_properties`）。

這些接口使 MaaFramework 能夠靈活地與外部設備交互、自定義圖像識別和動作邏輯，同時通過 CLI 模式實現自動化運行，從而滿足各種黑盒測試和自動化操作需求。

---
