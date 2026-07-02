# AI 關鍵字攻防 - 三驗企劃書

資訊營AI組 周宗穎
GitHub連結：https://github.com/mcjksieu005/prompt_injection_battle

---

## 測試網站連結

- 記分板：http://ws1.csie.ntu.edu.tw:6767/prompt_battle/scoreboard/
- 後台：http://ws1.csie.ntu.edu.tw:6767/prompt_battle/admin/
    - 帳號：admin
    - 密碼：6767
- 紅隊：http://ws1.csie.ntu.edu.tw:6767/prompt_battle/team?team=red
    - 帳號：red
    - 密碼：114514
- 藍隊：http://ws1.csie.ntu.edu.tw:6767/prompt_battle/team?team=blue
    - 帳號：blue
    - 密碼：1919810
---

## 遊戲簡介

該遊戲改編自李宏毅老師課程 GenAI 2025 HW4 和 ML 2026 HW1，讓小隊員體驗 prompt injection，嘗試誘導對方模型輸出不該輸出的關鍵字，同時加強自己模型防禦使得關鍵字不被輸出。

---

## 遊戲配置

| 項目 | 描述 |
|------|------|
| 參與人員 | 20名小隊員分為 Team A、Team B 對抗，可自行分工撰寫 defense prompt 或 attack prompt |
| 題目形式 | 關主提供關鍵字清單 (3~5個)，由兩組各自撰寫 defense prompt 和 attack prompt |
| 得分方式 | 一條 attack prompt 使對方模型輸出幾個關鍵字 (不重複) 便得幾分 |
| 使用模型 | 一個強度適中的 LLM 模型，聽得進防禦指令，也可以被攻破 (測試版使用 qwen2.5:7b-instruct) |
| 設備需求 | 總共5台電腦：主持人1台、兩隊各2台電腦，可進行基礎分工；網路、投影機 (放映比分與戰況，optional) |

---

## 遊戲流程

整體時長預計30分鐘

1. **遊戲講解**：講解 LLM 的 prompt 運作原理、如何撰寫 attack 跟 defense prompt，以及一些範例 (參考ML作業題目)
2. **Prompt撰寫**：計時10分鐘，讓各組自行分工，撰寫 defense prompt 跟第一輪的 attack prompt，defense prompt 有字數上限 (暫定為1000 token)，attack prompt最多10條
3. **第一次攻擊**：雙方把 defense prompt 放上自己的模型，並把自己的 attack prompt 餵給對方，並宣讀第一輪結果與得分
4. **第二次攻擊**：計時10分鐘，雙方可以編寫新的attack prompt攻擊對方的模型 (15秒 CD)，也可以修改自己的 defense prompt (寫好點擊儲存後適用於後續的attack prompt)，
5. **遊戲結算**：停止攻擊，並公布雙方分數，宣讀結果，若同分則 defense prompt 的字數少者獲勝 (防禦效率)，若依然相同則attack prompt 較少者獲勝 (攻擊強度)，若依然相同就判平手

---

## 運行架構與部署說明 (以下是AI生的)

這是一個基於 FastAPI 與 WebSockets 開發的即時 AI 提示詞攻防遊戲系統。專為營隊活動設計，具備低延遲廣播、雙隊獨立介面、關主後台管理與大螢幕記分板功能。

### 📂 專案架構 (基於 FastAPI + Vanilla JS)

*   `backend.py`: 系統後端核心 (狀態機、WebSocket 廣播中心、API 路由)。
*   `static/`: 前端靜態資源資料夾。
    *   `login.html`: 中央登入大廳 (具備 Cookie 身分驗證)。
    *   `admin.html`: 關主控制台 (賽程切換、設定同步)。
    *   `team.html`: 雙方小隊操作終端 (紅藍兩隊共用，依 URL 參數動態渲染)。
    *   `scoreboard.html`: 大螢幕投影專用記分板。
*   `old(gradio)/`: 舊版概念驗證 (POC) 程式碼備份。

### 💻 環境建置與啟動流程

建議使用 Python 3.10 以上版本。

**1. 建立並啟動虛擬環境**
```bash
python -m venv venv

# Windows
venv\Scripts\activate
# Mac/Linux
source venv/bin/activate
```

**2. 安裝依賴套件**
```bash
pip install -r requirements.txt
```

**3. 設定環境變數**
請複製 .env.example 並重新命名為 .env，接著填入你的設定值：
```Ini
OPENROUTER_API_KEY=你的_API_KEY
BASE_PATH=/prompt_battle
ADMIN_PWD=admin123
RED_PWD=red123
BLUE_PWD=blue123
```

**4. 啟動伺服器**
```bash
python backend.py
```
伺服器啟動後，將會在 http://0.0.0.0:6767 運行。

### 🔗 系統存取路徑

所有使用者請先進入登入大廳，系統會依據身分自動導向：

- 入口網站 (登入大廳)：http://localhost:6767/prompt_battle/login
- 強制跳轉目標網址 (供參考)：
    - 關主後台：.../prompt_battle/admin
    - 記分板：.../prompt_battle/scoreboard
    - 紅隊終端：.../prompt_battle/team?team=red
    - 藍隊終端：.../prompt_battle/team?team=blue

### 🛡️ 安全性設計

- Cookie 攔截：未經登入無法存取任何遊戲頁面。
- 路由隔離：小隊員無法透過修改 URL 參數進入敵對陣營或關主後台。
- 動態禁用：鎖定階段時，前端輸入框將強制觸發系統層級的 disabled，防堵任何鍵盤穿透操作。

---
