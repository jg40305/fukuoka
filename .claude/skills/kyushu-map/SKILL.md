---
name: fukuoka-kyushu-map-maintainer
description: Maintain and extend the Fukuoka/Kyushu travel spots site (index.html, fukuoka_kyushu_map.html, itinerary.html + assets/*.json), a static Leaflet + OpenStreetMap site hosted on GitHub Pages. Use this skill whenever the user asks to add, remove, recolor, or reclassify a location on either map, asks about the JSON data schema, the north-to-south color gradient logic, or how a new prefecture/city group or itinerary day should be inserted. Trigger on phrases like "加個景點到地圖", "把這個排進第X天行程", "幫我查座標放進地圖", "這個點位歸哪一類", "地圖顏色怎麼排的", or any request referencing fukuoka_kyushu_map.html, itinerary.html, reference_data.json, or itinerary_data.json.
---

# 福岡九州景點地圖維護

這是一個三頁的靜態網站（`index.html` 首頁 + 兩個 Leaflet 地圖頁），資料全部外置在 `assets/` 底下的 JSON 檔，頁面用 `fetch()` 讀取。跨工具的 baseline 說明在 repo 根目錄的 `AGENTS.md`，這份 SKILL.md 補充 Claude 在實際動手改資料時要遵守的細節規則。

## 兩份資料源，用途不同

| 檔案 | 對應頁面 | 分組欄位 | 用途 |
|---|---|---|---|
| `assets/reference_data.json` | `fukuoka_kyushu_map.html` | `group`（縣市） | 所有蒐集到、未篩選的參考景點 |
| `assets/itinerary_data.json` | `itinerary.html` | `day`（Day 1, Day 2…） | 真的排進行程的景點 |

兩份 schema 幾乎一樣，差在分組欄位是 `group` 還是 `day`：

```js
// reference_data.json
{group, name, lat, lng, google_maps_url, note}
// itinerary_data.json
{day, name, lat, lng, google_maps_url, note}
```

## 新增景點的標準流程

1. **查座標**：web_search 或已知知識找經緯度，抓小數 5-6 位精度
2. **產生 google_maps_url**：固定用 `https://www.google.com/maps/search/?api=1&query={lat},{lng}` 組出來，不要嘗試產生 `maps.app.goo.gl` 短連結（那需要 Google 系統逐一生成，做不到批次處理）
3. **判斷 `group`（參考地圖）**：
   - 對照該景點實際行政區屬於哪個縣市
   - 如果一個景點常被行程/影片歸類在「隔壁縣市」的行程裡，兩種分類都可以各留一筆，不要因為看起來重複就只留一筆
   - 查不到精確座標的景點（餐食、活動、非獨立地標）用鄰近最相關景點的座標代替，`note` 明確寫「座標為推測」或「座標取自鄰近的OO」
4. **判斷 `day`（實際行程）**：直接問使用者這個景點排第幾天，不要自己猜
5. 寫入對應 JSON 的陣列，`note` 欄位一律存在，沒內容填 `""`

## 顏色分配邏輯

**參考地圖（`fukuoka_kyushu_map.html`）**：`groupColors` 物件的 key 插入順序 = 側邊清單顯示順序 = 地理位置由北到南排序，顏色做藍（北）到紅（南）的漸層。新增縣市分類時：

1. 算出該分類底下幾個景點的平均緯度
2. 對照 `groupColors` 裡每行後面的緯度註解，找到該插入的位置
3. 顏色取前後兩個分類顏色的中間值
4. 絕對不要塞在物件最後面，除非緯度真的是最北或最南

**實際行程地圖（`itinerary.html`）**：顏色邏輯不一樣，是依 Day 出現順序做淺藍到深紫的漸層（`dayColor()` 函式自動計算），**不需要手動維護顏色表**，新增 Day 會自動接在漸層最後。

## 常見誤區

- 不要把兩份 JSON 的 schema 搞混（`group` vs `day`）
- 不要因為兩筆資料的 `lat`/`lng` 相同就自動合併或刪除其中一筆，先跟使用者確認
- 不要幫已存在的 `groupColors` 隨便改顏色，除非使用者要求重新排漸層
- 不要用 localStorage 或任何瀏覽器儲存 API — 資料設計上就是寫死在 JSON 檔裡，改動都要進 git commit 才算數
- 修改後提醒使用者用 `python3 -m http.server` 起本機伺服器確認一次（直接雙擊開 HTML 會被瀏覽器擋 `fetch`），再 push 上 GitHub Pages
