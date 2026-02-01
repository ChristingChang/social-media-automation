# 詳細設定指南

## 📋 目錄

1. [環境需求](#環境需求)
2. [Telegram Bot 設定](#telegram-bot-設定)
3. [Google Gemini API 設定](#google-gemini-api-設定)
4. [Discord Webhook 設定](#discord-webhook-設定)
5. [n8n Workflow 設定](#n8n-workflow-設定)
6. [常見問題](#常見問題)

---

## 環境需求

### 必要工具

- **n8n**: 版本 1.0.0 以上
  - [安裝教學](https://docs.n8n.io/hosting/)
  - 可使用 n8n Cloud（免費版）
  
- **網路連線**: 穩定的網路環境

### 可選工具

- **Ollama**: 如果想使用地端 AI 模型
- **Docker**: 如果要使用 Docker 運行 n8n

---

## Telegram Bot 設定

### 步驟 1: 建立 Telegram Bot

1. 在 Telegram 搜尋 **@BotFather**
2. 發送 `/newbot`
3. 依照指示設定 Bot 名稱與用戶名
4. 完成後會收到 **Bot Token**（格式：`123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`）

### 步驟 2: 設定 Bot 權限

```
/setprivacy - 設定為 DISABLED（允許 Bot 接收所有訊息）
/setcommands - 設定指令（可選）
```

### 步驟 3: 在 n8n 設定 Credential

1. 開啟 n8n
2. 點擊 **Credentials** > **New**
3. 搜尋 **Telegram**
4. 選擇 **Telegram API**
5. 貼上您的 Bot Token
6. 儲存

---

## Google Gemini API 設定

### 步驟 1: 取得 API Key

1. 前往 [Google AI Studio](https://aistudio.google.com/app/apikey)
2. 登入 Google 帳號
3. 點擊 **Get API Key**
4. 建立新的 API Key
5. 複製 Key（格式：`AIzaSy...`）

### 步驟 2: 在 n8n 設定

由於 n8n 目前沒有內建 Gemini 節點，我們使用 **HTTP Request** 節點：

```json
{
  "method": "POST",
  "url": "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "contents": [{
      "parts": [{
        "text": "YOUR_PROMPT_HERE"
      }]
    }]
  },
  "qs": {
    "key": "YOUR_GEMINI_API_KEY"
  }
}
```

### 免費額度

- 每分鐘 60 次請求
- 完全免費使用

---

## Discord Webhook 設定

### 步驟 1: 建立 Discord Webhook

1. 開啟 Discord 伺服器
2. 選擇要發文的頻道
3. 點擊頻道設定（齒輪圖示）
4. 選擇 **整合** > **Webhook**
5. 點擊 **新增 Webhook**
6. 複製 **Webhook URL**（格式：`https://discord.com/api/webhooks/...`）

### 步驟 2: 在 n8n 設定

1. 在 Discord 節點選擇 **Webhook**
2. 貼上 Webhook URL
3. 設定訊息內容

---

## n8n Workflow 設定

### 節點配置

#### 1. Telegram Trigger

- **Trigger On**: Message
- **Credential**: 選擇您建立的 Telegram credential

#### 2. HTTP Request (Gemini AI)

**設定範例：**

```
URL: https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=YOUR_API_KEY

Method: POST

Body (JSON):
{
  "contents": [{
    "parts": [{
      "text": "你是社群媒體內容編輯。將以下內容改寫成社群貼文：\n\n規則：\n1. 150-200字\n2. 專業但親切\n3. 加2-3個hashtag\n4. 更吸引人\n\n內容：{{ $json.message.text }}\n\n只輸出最終貼文，不要有任何解釋。"
    }]
  }]
}
```

**輸出處理：**

在下一個節點使用：
```
{{ $json.candidates[0].content.parts[0].text }}
```

#### 3. Discord Webhook

- **Webhook URL**: 貼上您的 Discord Webhook URL
- **Text**: 選擇前一個節點的 AI 輸出

#### 4. Telegram (Send Message) - 可選

發送成功通知回 Telegram：

- **Chat ID**: `{{ $('Telegram Trigger').item.json.message.chat.id }}`
- **Text**: `✅ 已成功發布到 Discord！`

---

## 測試流程

### 1. 啟動 Workflow

- 點擊右上角 **Active** 開關（變成綠色）

### 2. 發送測試訊息

- 在 Telegram 發送：「測試」
- 等待 5-10 秒

### 3. 檢查結果

- Discord 應該會收到 AI 優化後的內容
- 在 n8n 的 **Executions** 查看執行記錄

---

## 常見問題

### Q1: Telegram Trigger 顯示錯誤

**錯誤訊息**: "Can't listen for test executions"

**解決方法**: 
- 不要點「Execute Workflow」測試
- 直接啟動成 Active
- 在 Telegram 真實測試

### Q2: Gemini API 回應錯誤

**可能原因**:
- API Key 錯誤
- 超過免費額度限制
- 網路連線問題

**解決方法**:
- 檢查 API Key 是否正確
- 查看 [API 使用量](https://aistudio.google.com/)

### Q3: Discord 沒收到訊息

**檢查清單**:
- ✅ Webhook URL 是否正確
- ✅ 訊息內容是否為空
- ✅ Discord 頻道權限

### Q4: AI 輸出格式錯誤

**解決方法**:
- 調整 Prompt 加上「只輸出最終結果」
- 使用 Code 節點清理輸出
- 換用其他 AI 模型（如 OpenAI）

---

## 進階配置

### 使用 Ollama（地端模型）

如果不想使用 Google API，可以改用 Ollama：

1. 安裝 Ollama
2. 下載模型：`ollama pull gemma2`
3. 在 n8n 使用 HTTP Request 呼叫：
   ```
   POST http://localhost:11434/api/generate
   Body: {
     "model": "gemma2",
     "prompt": "YOUR_PROMPT",
     "stream": false
   }
   ```

### 新增內容排程

1. 新增 **Schedule Trigger** 節點
2. 設定發文時間
3. 連接到現有 workflow

---

## 🆘 需要協助？

- 📧 Email: cc1799999@gmail.com
- 💬 在 GitHub 開 Issue

---

最後更新：2025-01-31
