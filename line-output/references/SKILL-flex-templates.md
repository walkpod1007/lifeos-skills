# SKILL-flex-templates.md
# 龍蝦 Flex Message 模板系統

## 1. 模板位置

所有模板存放於：
workspace/projects/line-experience-lab/templates/

| 檔名 | 用途 |
|------|------|
| ai-reply.json | 一般 AI 回覆（header + body） |
| confirm.json | 確認/取消二選一 |
| info-card.json | 資訊卡（資訊列 + 按鈕） |
| error.json | 錯誤提示（固定紅色 tag，無按鈕） |
| success.json | 成功提示（固定綠色 tag，可選按鈕） |
| image-result.json | 帶 hero image 的結果卡 |

---

## 2. 各模板適用場景

### ai-reply.json
- 適用：任何一般問答、AI 回覆包裝
- 場景：用戶問問題 → 我的回答用 Flex 卡包裝

### confirm.json
- 適用：需要用戶二選一確認的場景
- 場景：「開新對話？」「確認刪除？」「要切換語言嗎？」
- 必填變數：TAG, TAG_COLOR, TITLE, BODY, CONFIRM_LABEL, CONFIRM_DATA, CANCEL_LABEL, CANCEL_DATA

### info-card.json
- 適用：展示結構化資訊 + 操作按鈕
- 場景：語音辨識結果、PDF 摘要、OCR 結果、系統狀態
- 資訊列最多 5 列，不用的列填空字串 ""
- 必填變數：TAG, TAG_COLOR, TITLE，至少一列 ROW

### error.json
- 適用：操作失敗、API 錯誤、格式錯誤
- Tag 固定為「⚠️ 錯誤」，顏色固定 #DC2626
- 必填變數：MESSAGE；SUGGESTION 可選（填空字串則不顯示）

### success.json
- 適用：操作成功確認
- Tag 固定為「✓ 完成」，顏色固定 #06C755
- 可選一個按鈕；不需要按鈕時把 BTN_LABEL 和 BTN_ACTION 填空
- 必填變數：TITLE, MESSAGE

### image-result.json
- 適用：圖片生成結果、圖片分析結果展示
- Hero image 規格：url 必須是公開 https，aspectRatio 1.51:1
- 必填變數：IMAGE_URL, TAG, TAG_COLOR, TITLE

---

## 3. 變數替換規則

所有變數用 {{DOUBLE_BRACE}} 格式。替換方式：

### 用 sed 替換（shell 腳本）
```
sed 's/{{TITLE}}/你的標題/g; s/{{BODY}}/你的內文/g' templates/ai-reply.json
```

### 用 python3 替換（推薦，處理特殊字元安全）
```
python3 -c "
import json, sys
t = open('templates/info-card.json').read()
t = t.replace('{{TAG}}', '語音辨識')
t = t.replace('{{TAG_COLOR}}', '#2563EB')
t = t.replace('{{TITLE}}', '辨識完成')
print(t)
"
```

### 手動替換（用於 curl 直接發送）
直接在 curl -d 的 JSON 字串內把 {{VAR}} 換成對應值。

---

## 4. 發送方式（curl）

### 發送到群組（push）
```
curl -X POST https://api.line.me/v2/bot/message/push \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $LINE_TOKEN" \
  -d '{
    "to": "GROUP_ID",
    "messages": [
      {
        "type": "flex",
        "altText": "卡片替代文字（純文字環境顯示）",
        "contents": <模板 JSON>
      }
    ]
  }'
```

### LINE_TOKEN（已在 openclaw.json 設定，可直接讀取）
```
LINE_TOKEN="2fXs0Ntx+B48rdWkQP7rXrkeeBiEnC8sOpjEI/ioBg98LsX1ryCqj7UGNMHKrGOXLyH0M9kTi1nJlbMQL1h3ltJUf9r9Ap9GbGadbmbcWpVXXwQElZUY7BXKeo9tWXO2Ox5mnJBAcOUuAFtX+vWGnwdB04t89/1O/w1cDnyilFU="
```

### 常用群組 ID
- LINE Experience Lab 群組：<LINE_GROUP_ID>

---

## 5. 顏色系統速查表

| 名稱 | 十六進位 | 用途 |
|------|----------|------|
| green | #06C755 | 預設、成功、LINE 官方綠 |
| blue | #2563EB | 資訊、語音辨識、中性操作 |
| orange | #D97706 | 警告、待確認 |
| red | #DC2626 | 錯誤、危險操作 |
| purple | #7C3AED | AI 生成、創意功能 |

選色原則：
- 成功/確認 → green
- 資訊展示 → blue
- 需注意但非錯誤 → orange
- 錯誤/刪除 → red
- AI 生成內容 → purple

---

## 6. 完整範例：收到「開新對話」→ 用 confirm.json 回覆

### 情境
用戶傳來：「開新對話」

### 步驟

1. 識別意圖：用戶要清除目前對話歷史、開始新 session
2. 選模板：confirm.json（需要二選一確認）
3. 填入變數：

| 變數 | 值 |
|------|----|
| TAG | 💬 對話管理 |
| TAG_COLOR | #2563EB |
| TITLE | 開始新對話？ |
| BODY | 目前對話紀錄將清除，確定要開新對話嗎？ |
| CONFIRM_LABEL | 確認開新 |
| CONFIRM_DATA | action=new_session |
| CANCEL_LABEL | 取消 |
| CANCEL_DATA | action=cancel |

4. 用 curl 發送：

```
curl -X POST https://api.line.me/v2/bot/message/push \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 2fXs0Ntx+B48rdWkQP7rXrkeeBiEnC8sOpjEI/ioBg98LsX1ryCqj7UGNMHKrGOXLyH0M9kTi1nJlbMQL1h3ltJUf9r9Ap9GbGadbmbcWpVXXwQElZUY7BXKeo9tWXO2Ox5mnJBAcOUuAFtX+vWGnwdB04t89/1O/w1cDnyilFU=" \
  -d '{
    "to": "<LINE_GROUP_ID>",
    "messages": [{
      "type": "flex",
      "altText": "開始新對話？",
      "contents": {
        "type": "bubble",
        "header": {
          "type": "box", "layout": "vertical", "backgroundColor": "#F8F8F8",
          "contents": [
            {"type": "text", "text": "💬 對話管理", "size": "xs", "color": "#2563EB", "weight": "bold"},
            {"type": "text", "text": "開始新對話？", "size": "md", "weight": "bold", "color": "#111111"}
          ]
        },
        "body": {
          "type": "box", "layout": "vertical",
          "contents": [{"type": "text", "text": "目前對話紀錄將清除，確定要開新對話嗎？", "wrap": true, "size": "sm", "color": "#444444"}]
        },
        "footer": {
          "type": "box", "layout": "horizontal", "spacing": "sm",
          "contents": [
            {"type": "button", "style": "primary", "color": "#06C755", "height": "sm",
             "action": {"type": "postback", "label": "確認開新", "data": "action=new_session"}},
            {"type": "button", "style": "secondary", "color": "#AAAAAA", "height": "sm",
             "action": {"type": "postback", "label": "取消", "data": "action=cancel"}}
          ]
        }
      }
    }]
  }'
```

---

## 7. 使用原則

- 純文字訊息（簡短問答）不需要 Flex，保持輕量
- 有結構資料（OCR / 語音辨識 / 摘要）→ 一律用 info-card
- 需要用戶決策 → 用 confirm
- 操作成功/失敗 → 用 success / error
- 有圖片輸出 → 用 image-result
