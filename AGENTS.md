# AGENTS.md

給任何 AI coding agent（Claude Code、Codex、Cursor、Gemini CLI…）看的專案說明。人類協作者請看 `CONTRIBUTING.md`。

## 這個專案是什麼

福岡＆九州旅遊景點地圖，純前端靜態網站，部署在 GitHub Pages。沒有建置流程、沒有 npm、沒有後端，改完檔案 push 上去就生效。

## 檔案結構

```
index.html                     首頁，連到下面兩個地圖
fukuoka_kyushu_map.html        參考景點地圖（讀 assets/reference_data.json）
itinerary.html                 實際安排景點地圖（讀 assets/itinerary_data.json，依 Day 分組）
assets/
  reference_data.json          參考景點資料源
  itinerary_data.json          實際行程資料源
.claude/skills/                Claude Code 專用的 skill，細節見裡面的 SKILL.md
CONTRIBUTING.md                給人類協作者的操作指南
```

兩個地圖頁面都是用 `fetch()` 讀對應的 JSON，畫面本身不含資料，改資料只需要改 JSON，不用碰 HTML。

## 資料格式

**`assets/reference_data.json`**（陣列，每筆一個景點）：
```json
{"group":"福岡市區", "name":"天神地下街", "lat":33.589571899999996, "lng":130.3997484,
 "google_maps_url":"https://www.google.com/maps/search/?api=1&query=33.589571899999996,130.3997484",
 "note":""}
```

**`assets/itinerary_data.json`**（陣列，每筆一個景點，`group` 換成 `day`）：
```json
{"day":"Day 1", "name":"太宰府天滿宮", "lat":33.5213697, "lng":130.5348239,
 "google_maps_url":"https://www.google.com/maps/search/?api=1&query=33.5213697,130.5348239",
 "note":""}
```

`google_maps_url` 統一用 `https://www.google.com/maps/search/?api=1&query=緯度,經度` 這個格式產生，不是 `maps.app.goo.gl` 那種需要 Google 系統逐一產生的短連結。

## 修改規則

1. 新增景點前先查座標，`lat`/`lng` 抓小數 5-6 位精度即可
2. `note` 欄位一定要存在，沒有內容就填 `""`，不要省略這個 key
3. `reference_data.json` 裡的 `group` 必須對應到 `fukuoka_kyushu_map.html` 裡 `groupColors` 物件已定義的縣市名稱；新增全新縣市才需要同步在 `groupColors` 加一行
4. `groupColors` 的顏色是依各分類平均緯度由北到南排成藍到紅漸層，插入新分類要照緯度排在對的位置，顏色取前後兩色的中間值，不要塞在最後面
5. 同一個景點在不同分類脈絡下重複出現是刻意保留的設計，不要因為看起來重複就砍掉一筆
6. 查不到精確座標的景點（餐食、活動、非獨立地標）用鄰近景點座標代替，並在 `note` 註明「座標為推測」
7. 不要用 localStorage 或任何瀏覽器儲存 API，資料一律寫進 JSON 檔並進 git commit

## 驗證方式

改完 JSON 後：
```bash
python3 -m json.tool assets/reference_data.json > /dev/null   # 確認合法 JSON
python3 -m http.server 8000                                    # 本機起服務，瀏覽器開 http://localhost:8000
```
直接雙擊打開 HTML（`file://`）會因為瀏覽器擋 `fetch()` 而讀不到 JSON，兩個頁面都有寫 fallback（讀不到就退回內建備用資料或顯示空狀態），但正式測試還是建議用本機伺服器或直接看 GitHub Pages 上的版本。
