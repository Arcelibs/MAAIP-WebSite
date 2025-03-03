---
title: 定時收訓練成果
type: docs
prev: /
next: docs/Venus_Tower/
---

## 訓練成果

分成訓練成果沒滿(TAP)跟訓練成果滿(MAX)，寫了兩種判別方式

然後基於每天第一次2小時免費收益寫了個判斷，沒什麼特別的，應該不會有人花鑽石吧...?

## Pipeline參考

```json
{
    "Click_daily": {
        "doc": "點擊每日任務,優先檢查max再檢查tap",
        "next": [
            "Click_MAX",
            "Click_tap"
        ]
    },
    "Click_MAX": {
        "doc": "點擊MAX",
        "recognition": "TemplateMatch",
        "template": "max.png",
        "action": "Click",
        "next": [
            "Check_free2hour",
            "Click_x"
        ]
    },
    "Click_tap": {
        "doc": "點選tap",
        "recognition": "TemplateMatch",
        "template": "tap.png",
        "action": "Click",
        "next": [
            "Check_free2hour",
            "Click_x"
        ]
    },
    "Check_free2hour": {
        "doc": "領取免費2小時收益",
        "recognition": "TemplateMatch",
        "template": "free2hour.png",
        "action": "Click",
        "next": "Click_x"
    },
    "Click_x": {
        "doc": "點X",
        "recognition": "TemplateMatch",
        "action": "Click",
        "template": "x.png"
    },
    "final": {
        "doc": "結束",
        "action": "DoNothing"
    }
}
```

後面的final是GPT自己加上去的，所以說寫FW還是別用GPT好一點