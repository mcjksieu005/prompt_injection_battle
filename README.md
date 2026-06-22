資訊營AI組 周宗穎

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

本系統專為多人即時對抗設計，目前採用 Python 原生全端框架進行快速開發與概念驗證，並已規劃後續的進階重構路線。

### 🛠️ 當前實作方法
目前的架構採用 **FastAPI + Gradio** 的混合模式，兼具輕量與即時性：
* **狀態管理 (State Machine)**：所有遊戲狀態（包含分數、防禦指令、倒數計時）皆儲存於後端記憶體的 `match_state` 字典中。
* **併發安全 (Concurrency Control)**：採用 `threading.RLock()` 可重入鎖，確保雙方小隊同時送出指令或背景計時器觸發時，資料不會發生 Race Condition。
* **單頁應用驅動 (SPA-like UI)**：捨棄傳統的分頁重新整理，前端透過 `gr.Timer` 配合後端狀態，利用 `gr.update(visible=...)` 強制同步雙方的遊戲畫面與階段切換。
* **模型推理解耦**：使用 OpenRouter API 呼叫雲端 `qwen-2.5-7b-instruct` 模型，將繁重的 LLM 推理運算外包，確保本地伺服器/工作站只需專注於處理 Web 請求。

### 💻 環境建置與啟動流程

請確保運行環境已安裝 Python 3.10 以上版本。

**1. 建立並啟動虛擬環境**
```bash
python -m venv venv
# Windows 系統:
venv\Scripts\activate
# macOS/Linux 系統:
source venv/bin/activate
```

**2. 安裝依賴套件**
```bash
pip install -r requirements.txt
```

**3. 設定環境變數**
在專案根目錄複製一份 .env.example 並重新命名為 .env，填入你的 API 密鑰：
```Ini
OPENROUTER_API_KEY=sk-or-v1-你的真實金鑰
```

**4. 啟動伺服器**
```bash
python app.py
```
啟動後，伺服器將運行於 http://0.0.0.0:6767 。參與者可透過瀏覽器進入 /team/red、/team/blue 等對應路由進行遊戲。

---

## 預期改進方向

為了符合營隊美宣需求並提供更極致的電競級體驗，本專案預計於後續進行前後端分離 (Frontend-Backend Separation) 重構：

後端架構升級 (Flask / FastAPI + WebSocket)：
保留現有的核心遊戲邏輯 (game_logic.py)，將通訊協定從 Gradio 輪詢全面升級為 Socket.IO (WebSocket) 雙向連線，以降低伺服器負載並達成毫秒級延遲。

前端介面重寫 (原生 HTML/JS/CSS)：
徹底移除 Gradio 的 UI 限制，改用原生網頁語言撰寫，以便完全無縫套用營隊的「海島風情」視覺主題。

加入共編提示互動 (Presence UI)：
藉由 WebSocket 的優勢，實作類似 Google Docs 的「正在輸入...」與游標鎖定提示，解決多台電腦操作同一隊伍畫面時的 UX 衝突問題。

---
