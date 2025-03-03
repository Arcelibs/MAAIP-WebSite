---
title: main_tower
type: docs
prev: docs/Venus_Tower/
---

## 開發邏輯

點訓練→Venus天塔→主塔→層數確定→自動編隊→開始→等小偶像跳完→NEXT→回首頁

## MAA Pipeline邏輯

這裡運用到了一個等待迴圈的方式，參考如下

```json
 "start_venus_main": {
        "doc": "點擊開始",
        "recognition": "TemplateMatch",
        "template": "start.png",
        "pre_delay": 2000,
        "post_delay": 2000,
        "action": "Click",
        "interrupt": [
            "stack1"
        ],
        "next": [
            "download_check",
            "next_toend"
        ]
    },
    "download_check": {
        "doc": "如果要下載就確認",
        "recognition": "TemplateMatch",
        "template": "download_check.png",
        "pre_delay": 2000,
        "post_delay": 2000,
        "action": "Click",
        "interrupt": [
            "stack1"
        ],
        "next": [
            "next_toend"
        ]
    },
    "stack1": {
        "post_delay": 1000
    },
```

這是因為在點了開始後可能會要下載歌曲，或者開始播MV，時間很久

為了怕FW等到TimeOUT所以設計了一個中斷迴圈，直到出現NEXT的按鈕
