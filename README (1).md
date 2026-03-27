# Modern Art — 線上版開發記錄

> 基於 Reiner Knizia《Modern Art》第三版規則（base23.com 英文規則書）  
> 參考：Gabriel Rocklin PC 版 v1.1.8（2006，Neural Network + Fuzzy Logic AI）

---

## 檔案版本對照

| 檔案 | 說明 |
|------|------|
| `modern_art_demo.html` | v5 原始 demo，v1–v5 迭代基礎 |
| `modern_art_v6.html` | AI 引擎 v10 全面升級版 |
| `modern_art_v7.html` | Bug 修正 + 畫家更名 + Call Round 調整 |
| `modern_art_beta09.html` | **Beta 0.9 對外發佈版**（簡化 UI，EV 隱藏）|

---

## 版本歷程

### v5 → v6（AI 引擎 v10 升級）

**方向一：出價策略**
- `aiMax`：LV1 改為「友善出價」（約 bidEV × 0.38–0.52），讓新手玩家更容易買到牌。LV2 加入 18% 情緒化超標機率。LV3–5 導入 halfValue 損失感知公式（p×gain/(1-p)）
- 新增 `buyerThreatAdjust()`：LV4+ 感知「這個畫家的其他對手持有量」，若威脅對手持有 2+ 張，買家出價上限微降最多 12%（低權重設計，僅微調）
- `aiFixedPrice`：LV1–3 維持原有公式。LV4–5 改為估算每位對手 aiMax，找「至少讓 1 人買的最高售價」；LV5 額外考慮「賣給哪個對手威脅最小」

**方向二：手牌管理 / 序列**
- Double 時機重設計（LV3–5）：拆解為五個判斷條件——補牌能力、場上時機視窗（2–3 張最佳）、投資充足度、出後是否達 4 張、對手手上有無補牌能力。三張時出 Double（→4 張）標為高風險
- 一步前瞻序列評估（LV4+）：評分時加入「出這張後下一步最佳選擇」× 0.35 折扣，讓 Open→Double 組合比直接出 Double 分數更高
- 新增 `myTotalGain_approx()` / `snapshot_approx()` 輔助函數

**方向三：Call Round**
- `oppGain` 動態加權：對手持有確定性高（3+ 張或場上 4 張）係數 0.45；一般情況 0.18。LV1–2 維持固定 0.30
- `wastedTurnCost` 動態化（LV3–5）：依手牌剩餘 rawEV 計算，手牌差代價低、手牌好代價高。第 4 回合上限 7
- `isLeadAtRisk`（LV4+）：領先差距 ≤ 1 且對手手上有同畫家牌 → +10 積極加分

**方向五（中期）：截斷時機**
- Genius 專屬：最佳截斷視窗（場上 3–4 張 + 對手持有 ≥ 2 + 自己 ≤ 1），計算截斷代價 vs 阻止對手收益的淨效益，正值才執行

---

### v6 → v7（Bug 修正 + 調整）

**Bug：Double 市場板顯示錯誤**
- 症狀：出 Double 後市場板顯示 N+1，應顯示 N+2
- 原因：`initiatePlay` 在第一張 `rOut++` 後立即 `renderMarket()`，第二張補牌尚未計數
- 修法：`initiatePlay` 中對 double 暫時 `rOut+1` 供 render 用，render 後立即還原；等 `playSecondCard` 真正 `++` 時自然正確

**畫家名字更新**（現代化、國際化）
| 舊名 | 新名 | 縮寫 | 國籍感 |
|------|------|------|--------|
| Lite Metal | Mira Voss | MV | 德裔 |
| Yoko | Kenji | KJ | 日裔 |
| Cristin P | Cleo Stark | CS | 中性英語 |
| Karl Gitter | Ran Ortiz | RO | 西語裔 |
| Krypto | Domu | DM | 非洲裔 |

**Call Round 觸發率提升（約 +15%）**
- LV1：機率 30% → 35%
- LV2：callMult 提升，觸發閾值 crv > 0 → crv > -5
- LV3：callMult 略升，負值抑制降低
- LV4：加入「第四輪手牌 ≤ 2 張」手牌不足加成 +12；「持有 ≥ 2 張且場上 3 張」位置優勢 +8
- LV5：加入「對手資金領先 ≥ 40」截斷動機 +10；「第四輪持有量最多」主導加分 +10；手牌不足 +8

**Fixed Price 定價微調**
- hard：adjFactor 0.85 → 0.90
- normal：adjFactor 0.72 → 0.74

---

### v7 → beta 0.9（對外發佈版調整）

> 所有 beta 修改都標有 `// BETA:` 注釋，還原方式寫在注釋裡

**人數固定 4 人**
- 位置：setup HTML（`#num-row` 加 `display:none`）+ `startGame()` 加 `C.num=4`
- 還原：移除 `display:none`，移除 `startGame()` 中的 `C.num=4`

**難度描述改為挑戰導向文字**
- 位置：setup HTML，五個 `.diff-item` 的文字
- 還原：換回原文字（見 v7）

**進階設定（AI 思考延遲）隱藏**
- 位置：setup HTML，整個進階設定 `<div>` 加 `display:none`
- 預設：AI 思考延遲維持開啟（`C.aiThink:true`）
- 還原：移除外層 `display:none`

**EV 開關隱藏，改為三連擊解鎖**
- 位置：topbar `.ev-row` 加 `display:none`；退出按鈕左側加透明觸發區 `#ev-secret`
- 機制：連點三次（900ms 內），EV 開關出現並短暫閃爍金框提示
- JS：`_secretTaps`, `_secretTimer`, `handleSecretTap()` 三個變數/函數
- 還原：移除透明觸發區，移除 `.ev-row` 的 `display:none`，移除三個 beta 變數/函數

---

## 核心規則備忘

- **Double**：出牌者出第一張，自己或下家可補第二張成為新拍賣人（收取全款）；全員 Pass 則出牌者取回，免費
- **第 5 張觸發回合結束**：第 5 張「出牌即結束」，不進行拍賣
- **Double 出牌後市場板計數**：顯示 N+2（含即將補的第二張），非 N+1
- **Seller blocking**：賣家不能在自己的拍賣中競標（Open / Once Around）
- **金錢流向**：買家付賣家；若賣家自購，付給銀行；Free get（無人出價）不付錢

## 設計原則備忘

- `bidEV`：買家本輪結算能拿回的錢（不含未來輪次），僅供出價邏輯用
- `rawEV`：含未來輪次潛力，僅供出牌選擇評分用
- `aiMax` 天花板：`cv + 30`（本輪最大可能回收），任何情況不超過 ceiling × 0.90
- LV1 友善出價設計目的：新手玩家體驗更好，容易買到牌，感受遊戲節奏
- Call Round `oppGain` 係數設計：動態加權（確定性高 0.45 / 低 0.18），非固定值

---

## 待辦 / 長期規劃

- [ ] Hand Inference Engine（LV4+，從對手出價行為推算手牌）
- [ ] 兩步前瞻 / MCTS 替代 LV5 啟發式評分
- [ ] 回合間跨輪手牌規劃完整版（LV3+，動態調整權重）
- [ ] 模擬器 Open/Once 反競標完整互動迴圈
- [ ] 挑戰模式（10 關，固定牌序，可重試）
- [ ] 成就 / 稱號系統
- [ ] 畫作圖片素材（AI 生成）
- [ ] 局後回放 + EV 最優解比對
- [ ] React/TypeScript 重構（Zustand + Framer Motion + Web Audio API）

---

## 免費版 vs 付費版規劃

| 功能 | 免費版 | 付費版 |
|------|--------|--------|
| 人數設定 | 固定 4 人 | 3–5 人自選 |
| 難度 | 全 AI 同一難度 | 各座位獨立難度 |
| 出牌順序 | 玩家固定第一 | 自訂 / 隨機順位 |
| 挑戰模式 | 前 3 關試玩 | 完整 10 關 |
| 特殊挑戰 | — | 設計手牌關卡 |
| 局後回放 | — | 完整回放 + EV 比對 |
| 成就系統 | — | 解鎖 / 稱號 |
| 視覺主題 | 標準 | 額外主題（夜場等）|
| 儲存配置 | — | 最多 3 組 |
| 統計頁 | — | 完整個人統計 |
