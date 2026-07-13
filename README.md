# AI 關鍵字攻防 - 正式版

台大資訊營AI組 周宗穎

GitHub連結：https://github.com/mcjksieu005/prompt_injection_battle

營隊網站連結：https://csiecamp.csie.org/prompt_battle/

---

## 遊戲簡介

該遊戲改編自李宏毅老師課程 GenAI 2025 HW4 和 ML 2026 HW1，讓小隊員體驗 Prompt Injection (提示注入)，嘗試誘導對方模型輸出不該輸出的關鍵字，同時加強自己模型防禦使得關鍵字不被輸出。

---

## 遊戲配置

| 項目 | 描述 |
|------|------|
| 參與人員 | 小隊員分為紅隊、藍隊進行對抗，可自行分工撰寫 defense prompt 或 attack prompt |
| 題目形式 | 一個共同的關鍵字清單 (一個或多個)，由兩組各自撰寫 defense prompt 和 attack prompt |
| 得分方式 | 一條 attack prompt 使對方模型輸出幾個關鍵字 (不重複) 便得幾分 |
| 使用模型 | 一個強度適中的 LLM 模型，聽得進防禦指令，也可以被攻破 (admin.html中預設有多個模型可選) |
| 設備需求 | 至少3台電腦：主持人1台、兩隊至少各1台電腦，有多台電腦則可進行分工；網路、投影機 (放映比分與戰況，optional) |

---

## 遊戲流程

整體時長預計30分鐘，各階段時長皆可調整

1. **遊戲講解**：講解 LLM 的 prompt 運作原理、如何撰寫 attack 跟 defense prompt，以及一些範例 (參考ML作業題目)
2. **決定關鍵字**：可由關主決定關鍵字，也可由兩組互相提出
3. **Prompt撰寫**：計時7分鐘，讓各組自行分工，撰寫 defense prompt 跟第一輪的 attack prompt，其中attack prompt最多10條
4. **第一次攻擊**：雙方把 defense prompt 放上自己的模型，並把自己的 attack prompt 餵給對方，並宣讀第一輪結果與得分
5. **第二次攻擊**：計時10分鐘，雙方可以編寫新的attack prompt攻擊對方的模型 (7秒 CD)，也可以修改自己的 defense prompt (寫好點擊儲存後適用於後續的attack prompt)，
6. **遊戲結算**：停止攻擊，並公布雙方分數，宣讀結果，若同分則attack prompt 較少者獲勝 (攻擊強度)，若依然相同則 defense prompt 較少者獲勝 (防禦效率)，若依然相同就判平手

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
*   `archived/`: 包含三驗企劃書、二驗企劃書及 Gradio 舊版程式碼備份。

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
# 核心系統設定
OPENROUTER_API_KEY=sk-or-v1-你的API金鑰......
BASE_PATH=/prompt_battle
PASSWORD_SALT=CSIE_CAMP_PROMPT_BATTLE_SUPER_SECRET_SALT_999

# 關主 (Admin) 登入設定 (隱蔽式帳號，防猜測)
ADMIN_USERNAME=admin6767
ADMIN_PWD=HereIsMySuperSecretAdminPassword

# 第一局系統預設小隊密碼 (第二局後可由關主後台隨機產生覆蓋)
RED_PWD=red67
BLUE_PWD=blue67
```

**4. 啟動伺服器**
```bash
python backend.py
```
伺服器啟動後，將會在 http://0.0.0.0:6767 運行。

---

## 🔗 系統存取路徑與營運操作

所有使用者均需透過**中央登入大廳**進入，系統會依據 Cookie 身分嚴格攔截並自動導向：

- **🚪 登入大廳**：`http://[你的IP]:6767/prompt_battle/login`

### 🎮 營運標準作業流程 (SOP)
1. 關主前往登入大廳，輸入 `.env` 中的 `ADMIN_USERNAME` 與 `ADMIN_PWD` 進入後台。
2. 關主點擊 **「📺 另開大螢幕記分板」** 並將該視窗拖曳至投影機放映。
3. 第一場比賽開始前，小隊員使用預設密碼 (`red67`, `blue67`) 登入。
4. 比賽結束後，關主點擊 **「📥 一鍵下載完整賽局戰報」** 留存紀錄。
5. 下一場比賽交接時，關主點擊 **「🔄 產生新回合隨機密碼」**，大螢幕與後台會顯示 6 碼新密碼，上一場的小隊員會被瞬間踢下線。關主將新密碼抄給新上台的小隊員。

---

## 🌟 系統特色與核心技術

本系統針對高強度「駭客級」小隊員進行了全方位的安全與體驗重構：

1. **清單化即時攻防 (CRUD 完整支援)**
   - 防禦規則與第一輪攻擊指令全面「清單化」。支援新增、無縫編輯、刪除、拖曳排序（防禦規則）與單條啟用/停用。徹底消滅多人協作時的 Race Condition 覆蓋問題。
2. **拋棄式動態密碼 (OTP) 與實體隔離**
   - 伺服器啟動後，關主可於後台「一鍵刷新」紅藍兩隊的隨機 4 碼登入密碼，並寫在實體紙條上發給上台的小隊員。
   - **強制驅逐機制 (Session Invalidation)**：刷新密碼的瞬間，舊局玩家的 Cookie 立即失效並被系統強制踢下線，杜絕跨回合偷登與偷看敵方陣型的可能。
3. **鐵壁級後端防護 (Zero-Trust Architecture)**
   - **嚴格身分校驗 (RBAC)**：後端完全不信任前端傳遞的 `team` 參數，強行解析 `HttpOnly` Cookie，徹底阻絕橫向越權攻擊 (IDOR)。
   - **硬體級防爆刷 (Hard Rate Limit)**：第二輪即時熱戰的 CD 時間在後端以 $O(1)$ 進行阻斷，完美防禦腳本洪水攻擊 (Flood Attack) 保護 API 額度。
   - **防洗版機制 (Anti-Spam)**：可開啟重複指令攔截，強制小隊員動腦編寫新咒語。
4. **全方位 XSS 防護與介面優化**
   - 戰報與日誌全面實作 `escapeHTML`，防止惡意腳本劫持大螢幕。
   - **手機版專屬響應式設計 (RWD)**：自動加高輸入框防止 iOS 破版，長文閱讀自動觸發半透明「懸浮視窗 (Modal)」，維持戰鬥畫面的極度乾淨。

---
