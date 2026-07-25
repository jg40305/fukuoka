# 協作說明

給人類協作者看的操作指南。如果你是用 AI 助理（Claude Code、Codex…）幫忙改這個 repo，AI 會自己讀 `AGENTS.md`（跨工具通用）或 `.claude/skills/fukuoka-kyushu-map-maintainer/SKILL.md`（Claude Code 專用），不用把這份文件餵給它。

## 專案結構

```
index.html                     首頁，連到下面兩個地圖
fukuoka_kyushu_map.html        參考景點地圖
itinerary.html                 實際安排景點地圖（依 Day 分組，目前是空的）
assets/
  reference_data.json          參考地圖的資料源
  itinerary_data.json          實際行程的資料源
AGENTS.md                      給任何 AI coding agent 看的專案說明
.claude/skills/                Claude Code 專用 skill
```

純前端靜態網站，沒有建置流程、沒有 npm。改完檔案，git push 到有開 GitHub Pages 的 branch 就會自動更新。

## 開始之前

1. Clone 這個 repo
2. 本機測試建議起一個小型伺服器，不要直接雙擊打開 HTML（瀏覽器會擋 `fetch()` 讀 JSON）：
   ```bash
   python3 -m http.server 8000
   ```
   然後瀏覽器開 `http://localhost:8000`
3. 沒有伺服器也能看：兩個地圖頁面在讀不到 JSON 時會顯示備用內容或空狀態，但看到的資料可能是舊的，不建議拿來確認新增的點位對不對

## 加一個「參考景點」

打開 `assets/reference_data.json`，在陣列裡加一筆：

```json
{
  "group": "福岡市區",
  "name": "景點名稱",
  "lat": 33.5929546,
  "lng": 130.41045889999998,
  "google_maps_url": "https://www.google.com/maps/search/?api=1&query=33.5929546,130.41045889999998",
  "note": ""
}
```

- `lat` / `lng`：Google 搜尋「景點名稱 座標」，或在 Google Maps 上對該位置點右鍵會顯示經緯度
- `google_maps_url`：固定格式 `https://www.google.com/maps/search/?api=1&query=緯度,經度`，把剛剛查到的 lat/lng 填進去即可，這不是 `maps.app.goo.gl` 那種真正的短網址（那個需要 Google 系統一個一個產生，沒辦法手動批次做），但點了一樣會直接開到那個位置
- `group`：**必須**是 `fukuoka_kyushu_map.html` 裡 `groupColors` 物件已經定義好的縣市名稱之一。拼錯或用新名稱會導致這個點位顯示不出顏色，也不會出現在側邊清單
- `note`：需要提醒別人的資訊（例如「座標為推測」「實際位於OO市」），沒有就填空字串 `""`，這個 key 不要省略

存檔、重新整理瀏覽器，確認新標記出現在正確位置、顏色正確、側邊清單點得到。

## 加一個「實際安排景點」

打開 `assets/itinerary_data.json`，格式幾乎一樣，只是把 `group` 換成 `day`：

```json
{
  "day": "Day 1",
  "name": "太宰府天滿宮",
  "lat": 33.5213697,
  "lng": 130.5348239,
  "google_maps_url": "https://www.google.com/maps/search/?api=1&query=33.5213697,130.5348239",
  "note": ""
}
```

`day` 直接寫「Day 1」「Day 2」這種字串即可，順序依照它在陣列裡第一次出現的位置決定，不用照日期排序整個陣列。這邊的顏色是自動依 Day 數算出淺藍到深紫的漸層，不需要另外維護顏色對照表。

檔案目前是空陣列 `[]`，第一次加東西進去之後，`itinerary.html` 會自動從「還沒安排景點」的提示畫面切換成正常地圖。

## 新增分類的配色規則（只在加參考地圖的新縣市時才需要）

`fukuoka_kyushu_map.html` 裡的 `groupColors` 顏色不是隨便選的，是照**各分類景點的平均緯度，由北到南排成藍到紅的漸層**：

- 緯度越高（越北）→ 越偏藍
- 緯度越低（越南）→ 越偏紅

新增一個縣市分類時：

1. 算出這個縣市裡幾個景點的大概平均緯度
2. 對照 `groupColors` 裡現有分類的緯度註解，找到這個新分類該插在哪個位置
3. 顏色取前後兩個分類顏色的中間值
4. 插入後，其他分類順序不用重排，JS 物件會照插入順序渲染側邊清單

如果插入後發現顏色明顯斷層、看起來不連貫，跟大家說一聲，可能需要把全部分類的顏色重新算一輪。

## 重複景點的處理原則

同一個景點如果被不同分類引用（例如「鳥栖Premium Outlets」同時出現在「北九州市」的備註裡，也出現在「佐賀縣」的正確歸屬裡），**兩筆都保留**，不要因為看起來重複就刪掉一筆。原始資料來源（旅行社行程、YouTuber 介紹）本來就會用不同的地理框架介紹同一個點，保留兩筆可以讓查閱的人知道這個點在不同脈絡下都出現過。

## Commit 規範

沒有硬性規定，但建議寫清楚加了哪些點，方便之後回溯，例如：

```
新增：實際行程 Day 3（太宰府天滿宮、竈門神社）
```

## GitHub Pages 部署

如果 repo 已經開了 GitHub Pages（Settings → Pages → 選 main branch），push 到 main 之後幾分鐘內網站會自動更新，不需要額外操作。
