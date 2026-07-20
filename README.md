# 真的假的啦（fake-news）

單頁 HTML 假訊息辨識遊戲，針對台灣長輩常見的 LINE 詐騙／謠言設計。

- **正式站**：https://jinchen123456.github.io/fake-news/
- **GitHub Repo**：https://github.com/jinchen123456/fake-news
- **部署方式**：GitHub Pages（`main` 分支根目錄）
- **技術棧**：純 HTML + CSS + JavaScript，**單一 `index.html` 檔**（無 build step、無依賴）

## 專案結構

```
fake-news/
├── index.html      ← 所有東西都在這裡（HTML + CSS + JS + 題庫）
├── README.md       ← 你正在看
└── MAINTENANCE.md  ← 常見維運任務手冊
```

## 快速開始

1. **本機預覽**
   ```bash
   # 用任意 static server；推薦 python 內建 http.server
   cd fake-news
   python3 -m http.server 8000
   # 打開 http://localhost:8000
   ```

2. **修改**
   直接編輯 `index.html`，用瀏覽器重新整理即可看到結果。

3. **部署**
   ```bash
   git add index.html
   git commit -m "your message"
   git push origin main
   # GitHub Pages 會自動 build，約 30 秒～2 分鐘生效
   ```

   若遇 Pages build 失敗（偶發），推空 commit 重試：
   ```bash
   git commit --allow-empty -m "chore: trigger pages redeploy"
   git push
   ```

   確認 build 狀態：
   ```bash
   gh api repos/jinchen123456/fake-news/pages/builds --jq '.[0] | {commit, status, updated_at}'
   ```

## 遊戲玩法

1. 玩家共答 10 題（4 簡單＋4 中等＋2 困難，隨機抽選）。
2. 判斷每條訊息是「真的」還是「假的」，每題 15 秒。
3. **答錯 3 題結束遊戲**（不是分數歸零）。
4. 連答 2 題正確會觸發「加分挑戰」（惡魔小遊戲）。
5. 完成後顯示分數、等級（大師 / 高手 / 玩家 / 挑戰者）與分享卡片。

## 檔案內部結構（`index.html`）

| 行號區間 | 內容 |
|--------|-----|
| 1–7 | `<head>` metadata / viewport |
| 8–53 | **CSS 設計 tokens** — 主要顏色／圓角／陰影變數 |
| 54–820 | CSS 樣式 |
| 821–1018 | HTML 各畫面（intro / game / feedback / demonScreen / gameOver / final / leaderboard） |
| 1022–2172 | **題庫 `const QS`**（目前 365 題） |
| 2552–2600 | 常數、trick／category 對應表、工具函式 |
| 2606–3703 | 遊戲主邏輯（session 建立、答題、惡魔、結算、排行榜、分享） |

## 常見維運任務

詳見 [`MAINTENANCE.md`](./MAINTENANCE.md)。速查：

- **新增題目** → `MAINTENANCE.md`「題庫管理」
- **調整字級／顏色／間距** → `MAINTENANCE.md`「設計系統」
- **改文案** → `Grep` 現有文字直接改
- **改遊戲規則**（題數、時間、扣分） → `MAINTENANCE.md`「遊戲參數」

## 聯絡

- 產品負責人：jinchen@cw.com.tw（天下雜誌）
- 遇到問題先讀 `MAINTENANCE.md`，再看 git log（`git log --oneline` 有記錄近期決策）
