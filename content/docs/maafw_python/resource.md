---
title: Resource模組
type: docs
sidebar:
  open: true
---

## MaaFramework Resource 接口說明

**Resource** 類是 MaaFramework 中用於管理和操作資源（如待測試項目、模型、配置等）的核心組件。通過 Resource 類，使用者可以提交資源包、查詢資源狀態、設置推理設備以及註冊自定義的識別與動作回調，以滿足自動化測試和資源管理的需求。

---

## 1. Resource 實例的創建與銷毀

### `__init__(notification_handler=None, handle=None)`

**用途：**  
初始化一個 Resource 實例，創建或接管底層 MaaResourceHandle。

**參數說明：**  
- **notification_handler** (NotificationHandler, 可選)：用於處理資源操作過程中的各種通知事件。  
- **handle** (MaaResourceHandle, 可選)：如果已存在底層資源句柄，則直接使用並標記為非擁有者；否則，通過底層函數 MaaResourceCreate 自行創建資源並成為擁有者。

**內部機制：**  
- 調用 `_set_api_properties()` 來初始化 C API 接口屬性（僅初始化一次）。  
- 若資源創建失敗則拋出 RuntimeError。  
- 同時初始化兩個字典：`_custom_action_holder` 和 `_custom_recognition_holder`，用於保存自定義動作和識別對象，防止它們被垃圾回收。

### `__del__()`

**用途：**  
在 Resource 實例銷毀時，若該實例是資源的擁有者（_own 為 True），則調用 MaaResourceDestroy 釋放底層資源。

---

## 2. 資源操作

### `post_bundle(path) -> Job`

**用途：**  
上傳或提交一個資源包到框架中進行處理。通常用於加載待測試項目或模型文件。

**參數說明：**  
- **path** (str 或 Path)：資源包文件所在的路徑。

**返回值：**  
- 返回一個 `Job` 對象，使用者可以通過該對象等待任務完成或查詢上傳狀態。

---

### `loaded` 屬性

**用途：**  
檢查當前資源是否已成功加載。

**返回值：**  
- 返回布林值，`True` 表示資源已加載成功。

---

### `clear() -> bool`

**用途：**  
清除當前資源的內部狀態或緩存。

**返回值：**  
- 返回布林值，成功清除返回 `True`。

---

## 3. 設置推理設備（Inference Options）

Resource 提供多種方法設置資源運行時的推理設備或運行模式：

### `use_cpu() -> bool`

**用途：**  
強制資源使用 CPU 進行推理。

**返回值：**  
- 設置成功則返回 `True`。

### `use_directml(device_id: int = MaaInferenceDeviceEnum.Auto) -> bool`

**用途：**  
設置資源使用 DirectML 作為推理執行提供者，支持指定 GPU ID（默認自動選擇）。

**參數說明：**  
- **device_id** (int)：設備 ID，默認為自動。

**返回值：**  
- 成功則返回 `True`。

### `use_coreml(coreml_flag: int = MaaInferenceDeviceEnum.Auto) -> bool`

**用途：**  
設置資源使用 CoreML 進行推理（主要針對蘋果設備）。

**參數說明：**  
- **coreml_flag** (int)：CoreML 參數，默認為自動。

**返回值：**  
- 成功則返回 `True`。

### `use_auto_ep() -> bool`

**用途：**  
設置資源自動選擇最佳的推理執行提供者。

**返回值：**  
- 成功則返回 `True`。

> *備註：另外還提供了 `set_gpu()`, `set_cpu()`, `set_auto_device()` 等已標記為棄用的方法，建議使用上述對應方法。*

### 私有方法：`set_inference(execution_provider, device_id) -> bool`

**用途：**  
內部方法，根據指定的推理執行提供者與設備 ID，通過 MaaResourceSetOption 分別設置 **InferenceExecutionProvider** 與 **InferenceDevice**。

**參數說明：**  
- **execution_provider** (int)：推理執行提供者（例如 CPU、DirectML、CoreML 等）。  
- **device_id** (int)：設備 ID 或旗標值。

**返回值：**  
- 同時設置成功返回 `True`。

---

## 4. 自定義識別與動作的註冊

Resource 支持用戶通過自定義識別與動作接口擴展框架功能，通常用於自定義圖像識別算法或動作邏輯。

### 自定義識別相關接口

#### `custom_recognition(name)`

**用途：**  
作為裝飾器，便於用戶在函數定義時註冊自定義識別邏輯。  
**參數說明：**  
- **name** (str)：自定義識別接口的名稱。

使用此裝飾器的函數需返回一個 CustomRecognition 的實例，內部會調用 `register_custom_recognition`。

#### `register_custom_recognition(name, recognition) -> bool`

**用途：**  
註冊一個自定義識別對象到框架中。

**參數說明：**  
- **name** (str)：識別接口的唯一標識名稱。  
- **recognition** (CustomRecognition)：實現了自定義識別邏輯的對象。

**返回值：**  
- 返回布林值，註冊成功返回 `True`。  
- 同時將識別對象存入內部字典 `_custom_recognition_holder`，防止被垃圾回收。

#### `unregister_custom_recognition(name) -> bool` 與 `clear_custom_recognition() -> bool`

**用途：**  
分別用於卸載單個或全部自定義識別接口。

---

### 自定義動作相關接口

#### `custom_action(name)`

**用途：**  
作為裝飾器，便於用戶在定義自定義動作時自動進行註冊。  
**參數說明：**  
- **name** (str)：自定義動作接口的名稱。

返回的函數需返回一個 CustomAction 的實例，內部會調用 `register_custom_action`。

#### `register_custom_action(name, action) -> bool`

**用途：**  
註冊一個自定義動作對象。

**參數說明：**  
- **name** (str)：自定義動作接口的標識名稱。  
- **action** (CustomAction)：實現自定義動作邏輯的對象。

**返回值：**  
- 成功註冊返回 `True`，同時將動作對象存入 `_custom_action_holder`。

#### `unregister_custom_action(name) -> bool` 與 `clear_custom_action() -> bool`

**用途：**  
分別卸載單個或全部自定義動作接口。

---

## 5. 資源哈希值

### `hash` 屬性

**用途：**  
獲取資源的唯一哈希值，用於驗證資源的一致性或進行版本控制。

**內部實現：**  
- 調用 MaaResourceGetHash 函數，並通過 StringBuffer 讀取返回的字符串。

**返回值：**  
- 返回一個字符串表示的資源哈希值。

---

## 總結

Resource 模塊為 MaaFramework 提供了資源管理的基本能力，其主要功能包括：

- **資源創建與銷毀：** 根據是否提供句柄決定創建新資源或接管現有資源，並在實例銷毀時自動釋放資源。  
- **任務提交：** 通過 `post_bundle` 將資源包提交到框架中進行處理。  
- **資源狀態查詢：** 如 `loaded` 屬性和 `clear()` 方法用於檢查和管理資源狀態。  
- **推理設備設置：** 通過多種方法設置推理運算的設備與模式，保證運行時選擇合適的硬體。  
- **自定義擴展：** 提供識別與動作接口的註冊功能，支持用戶自定義圖像識別和交互動作，並利用內部字典保存回調對象防止被回收。  
- **資源哈希：** 方便資源的版本管理和一致性校驗。

這些接口共同構成了 Resource 模塊，使 MaaFramework 能夠靈活管理待測試項目及其相關配置，並支持豐富的自定義擴展功能。

---
