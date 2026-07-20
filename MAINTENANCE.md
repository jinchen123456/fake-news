# 維運手冊

給接手同事的常見任務操作指南。所有變更都在 `index.html` 這一個檔案裡。

---

## 目次

1. [題庫管理](#題庫管理)
2. [設計系統](#設計系統)
3. [遊戲參數](#遊戲參數)
4. [畫面結構](#畫面結構)
5. [關鍵 JS 函式](#關鍵-js-函式)
6. [歷次改版脈絡](#歷次改版脈絡)
7. [已知限制與注意事項](#已知限制與注意事項)

---

## 題庫管理

題庫是 `index.html` 裡的 `const QS = [ ... ]`（約行 1022 起）。

### 題目格式

```javascript
{
  diff: 2,                          // 1=簡單 / 2=中等 / 3=困難
  sender: '兒子',                   // 訊息發送者顯示名
  av: '👨',                         // 頭像 emoji
  time: '下午 07:30',               // 時間標籤
  cat: '健康資訊',                  // 分類顯示名
  catCls: 'cat-govt',               // 分類 CSS class（見下方）
  msg: '媽，你的血壓最好每天量一次…', // 訊息內容
  real: true,                       // true=真訊息 / false=假訊息
  why: '正確！…',                   // 答對／答錯後解說
  tip: '💡 …',                      // 小提示（用途、查證管道）
}
```

### 分類 `catCls` 對應

| catCls | 顯示名 | 顏色 | 用途 |
|--------|-------|------|------|
| `cat-health` | 健康謠言／健康資訊 | 綠 | 醫療、營養、疫苗 |
| `cat-scam` | 詐騙訊息 | 紅 | 明顯的詐騙話術 |
| `cat-fake` | 假新聞／政策謠言 | 橘 | 政治／政策謠言 |
| `cat-govt` | 政府公告／政策資訊 | 藍 | 真的政府資訊（多為 `real:true`） |
| `cat-custom` | 習俗謠言 | 紫 | 民俗迷信類 |

### 難度分佈

抽題邏輯（見 `buildSession()`，約行 2606）：
- 4 題 diff:1（簡單）
- 4 題 diff:2（中等）
- 2 題 diff:3（困難）
- 共 10 題

### 現行題數與比例

用以下指令即時查詢：

```bash
NODE_OPTIONS= node -e "const h=require('fs').readFileSync('index.html','utf8');const m=h.match(/const QS = \[([\s\S]*?)\n\];/);const a=eval('(['+m[1]+'])');const t={true:0,false:0};a.forEach(q=>t[q.real]++);console.log('total',a.length,'T/F',t.true+'/'+t.false);const d={};a.forEach(q=>{const k=q.diff;if(!d[k])d[k]={t:0,f:0};d[k][q.real?'t':'f']++});console.log(d)"
```

目前約 365 題，真假比例約 47/53（產品要求「均衡但略偏假」，讓玩家有東西可以抓）。

### 新增題目建議

1. **維持真假比例**：加 3 題假的就配 3 題真的。
2. **時效性內容更新**：資訊會過時（如稅率、免稅額、疫苗時程），每年檢查一次。
3. **貼近真實用語**：模仿 LINE 家人訊息／詐團實際腳本，避免書面語太重。
4. **`why` 要寫清為什麼真／為什麼假**，`tip` 要給查證管道或判斷訣竅。
5. **參考來源**：
   - 165 反詐儀表板：https://165dashboard.tw/fraud-method
   - MyGoPen 事實查核：https://www.mygopen.com
   - 台灣事實查核中心：https://tfc-taiwan.org.tw
   - 衛福部食藥署闢謠：https://www.fda.gov.tw

### 移除／修改題目

直接編輯，注意：
- 逗號結尾（除了最後一個元素也可留逗號，JS 允許 trailing comma）
- 用單引號時，內文用「」與『』代替直引號避免衝突
- 表情符號可直接貼入

---

## 設計系統

### 顏色 tokens（`:root` 開頭，約行 8）

```css
:root {
  --bg: #fbfaf6;          /* 頁面底色 */
  --bg-mid: #f2f0ea;      /* 中間層 */
  --bg-card: #ffffff;     /* 卡片底 */
  --bg-hi: #eef2ec;       /* 淺綠 hint 底 */

  --positive: #5aa572;    /* 淡綠 — 一般 UI 綠 */
  --negative: #c76666;    /* 淡紅 — 一般 UI 紅 */
  --brand: #C8161C;       /* 天下雜誌品牌紅 */
  --brand-d: #9D0F14;     /* 品牌深紅 */

  --text: #1e2429;        /* 主要文字 */
  --text-mid: #4a5259;    /* 次要文字 */
  --text-soft: #8b8f92;   /* 輔助文字 */
  --border: #ebe8df;      /* 邊框 */

  --r-sm: 8px; --r-md: 16px; --r-lg: 24px; --r-full: 999px;
  --shadow-sm: 0 2px 8px rgba(30,36,41,.05);
  --shadow-md: 0 8px 24px rgba(30,36,41,.06);
}
```

### 需要飽和一點的顏色（單獨定義，不走 token）

- `.btn-real`（真的按鈕）: `#2fa856`（飽和綠）
- `.btn-fake`（假的按鈕）: `#e0392e`（飽和紅）
- `.btn-next-q`、`.btn-fight`（feedback 繼續按鈕）: 同上

若整體要重新調色，記得這幾個按鈕也要一起改。

### 字級

桌面基準字級都往大調過（產品明確要求）。改字級時注意：

- **CSS 有三層 media query 覆蓋**：`max-height:780px`、`max-width:340px`、`max-height:600px`。
- 改字級不要只改桌面基準，行動端要同步改對應 media query 內的值，否則手機字會變回小。
- 數字類欄位（分數、時間）務必加 `font-variant-numeric: tabular-nums`。

### 圓角與陰影

- 統一用 `var(--r-sm/md/lg/full)`，不要再直接寫 `border-radius: 12px`。
- 陰影用 `var(--shadow-sm/md)` 或 rgba 微調（不要用純黑）。

---

## 遊戲參數

以下是最常被要求調整的常數，位置在 `index.html` 各處：

| 參數 | 位置 | 說明 |
|-----|-----|-----|
| `TOTAL = 10` | `buildSession` 附近 | 每局題數 |
| `QUESTION_TIME = 15` | `startQTimer` | 每題秒數 |
| `wrongCount >= 3` | `nextQ()`、`applyWrong()` | 結束條件（錯 3 題） |
| `+10 / -10` | `answer()` 內 | 答對／答錯分數 |
| `DMN_TIME = 13` | 約行 2552 | 加分挑戰惡魔時間 |
| `FINAL_BONUS_TIME = 12` | 約行 2864 | 最終加分挑戰時間 |
| `CHAIN_RADIUS = 68` | 約行 2863 | 惡魔連鎖爆炸範圍 |

**產品規則**：遊戲結束條件目前是「錯 3 題」，**不是**「分數歸零」。之前有過分數歸零觸發，已在 `ae0c5ea` 移除，不要加回來。

---

## 畫面結構

畫面切換由 `show(id)` 函式控制（約行 2633）。所有畫面都是同一頁的 `<div class="screen">`，靠 `.hidden` class 切換顯示。

```
intro         — 首頁（開始遊戲、輸入名字、看排行榜）
game          — 答題主畫面（訊息＋真的／假的按鈕）
feedback      — 答完顯示解說＋小提示＋按鈕
demonScreen   — 加分挑戰惡魔小遊戲
gameOver      — 錯 3 題結束畫面
final         — 完成 10 題結果頁（成績、分享、天下雜誌 CTA）
leaderboard   — 排行榜
```

### `final` 頁按鈕結構（產品指定）

順序不要亂：
1. 分數／勳章／星星／統計
2. `share-card`（可複製分享文字）
3. `.final-actions`（`再試一次` + `分享朋友` 並排）
4. `.btn-lb-link`（`查看排行榜` 小型底線文字）
5. `.btn-cta-cw`（**天下雜誌 CTA，紅色主 CTA，放最下面**）

主 CTA 只留一個（天下雜誌），其他都是次要樣式。

---

## 關鍵 JS 函式

| 函式 | 位置 | 職責 |
|-----|-----|-----|
| `buildSession()` | 2606 | 從 QS 抽 10 題出來成為 `sessionQ` |
| `nextQ()` | 2690 | 進入下一題／觸發加分挑戰／結束 |
| `answer(userReal)` | 2768 | 玩家點按鈕的入口 |
| `showFeedback(ok, timeout)` | 2801 | 答完顯示解說卡 |
| `afterFeedback()` | 2848 | 從解說卡離開的路由（下題／加分挑戰／結束） |
| `launchDemons()` | 2894 | 加分挑戰惡魔生成 |
| `launchFinalBonus()` | 3276 | 最終加分挑戰入口 |
| `showFinal()` | 3563 | 結果頁組裝（分數、勳章、分享文字） |
| `showGameOver()` | 3604 | 錯 3 題結束頁 |
| `saveScore` / `showLeaderboard` | 3626 / 3643 | 排行榜寫入／顯示 |
| `copyShare()` | 3683 | 複製分享文字到剪貼簿 |
| `restart()` | 3693 | 回到首頁重玩 |

排行榜用 `localStorage`（key = `fgScores_v1`），僅存本機。

---

## 歷次改版脈絡

`git log --oneline` 可看完整紀錄。近期重要決策：

- **`54dca71`** — 題庫從 318 → 365，補 47 題正確衛教／政府資訊，真假比例從 39/61 → 47/53
- **`2a5ff83`** — 依 165 反詐儀表板加 50 題現代長輩詐騙（假投資、假檢警、AI 換臉、殺豬盤等）
- **`448e41d`** — 結果頁只留天下雜誌一個主 CTA，其他按鈕改為次要樣式
- **`32ab05b`** — 移除「防騙免疫力分析」統計圖，分享文字去除網址，分享按鈕改中性色
- **`4fe3701`** — 縮小加分挑戰彈窗、真假按鈕加飽和度、結果頁去 emoji＋兩按鈕並排
- **`ae0c5ea`** — 遊戲結束條件簡化為「錯 3 題」
- **`fda44f3`** — 全站字級調大（含所有 media query breakpoint）
- **`765a0e5`** — 收斂色彩到 3 色系統＋去主視覺 emoji＋統一節奏／圓角

---

## 已知限制與注意事項

- **單檔限制**：目前所有東西都塞在 `index.html`。若要拆檔（例如把 QS 抽成 JSON），要維持零依賴／零 build（GitHub Pages 純 static 部署）。
- **`document.querySelector` 依賴 CSS class**：改按鈕 class 名字時（例如把 `.btn-copy` 改成 `.btn-final-share`），記得同步改 JS 裡的 selector；上一輪就有踩過這個坑（`copyShare` 曾經 reference 已刪除的 class）。
- **`buildDots()` 曾經 crash**：因為引用了不存在的 element。加新畫面元素時記得檢查所有相依函式的 null guard。
- **測試主要靠肉眼**：沒有自動化測試。改動後至少要跑一輪完整遊戲（開始 → 答題 → 加分挑戰 → 結束 → 分享 → 排行榜）。手機／桌面都要看。
- **GitHub Pages 偶爾 build 失敗**：推空 commit 重試即可。
- **題庫時效性**：稅率、免稅額、疫苗政策等會變，建議每年 1 月與 7 月各檢查一次 `cat-govt` 類型的題目。
- **`localStorage` key**：`fgScores_v1`。若排行榜資料結構要改（加欄位），記得 bump 到 `_v2` 避免舊資料格式衝突。
- **不要加 build step**：產品要求零依賴，直接編輯即可。加了打包工具會讓維運複雜度上升，且非同步部署難處理。

---

## 需要 Claude Code 幫忙？

這份文件與最近的改版都是 Claude Code 產生的。若要繼續用 AI 協作維運：

- 直接在 repo 目錄 `claude` 起互動視窗
- 常見任務可以直接說「幫我加 10 題假冒國稅局的題目」或「把倒數計時字級調小一點」
- Claude 會直接改 `index.html`，commit 前會給你看 diff
