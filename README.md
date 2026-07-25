# 福岡＆九州旅遊景點地圖

福岡與九州的旅遊景點整理網站，包含參考景點地圖與實際行程地圖。

目前版本：`1.0.0`

## 使用方式

本專案是純前端靜態網站，沒有 npm 相依套件或建置流程。由於頁面會透過
`fetch()` 讀取 JSON，請在專案目錄啟動本機伺服器：

```bash
python3 -m http.server 8000
```

接著開啟 <http://localhost:8000>。

景點與行程資料分別位於：

- `assets/reference_data.json`：參考景點
- `assets/itinerary_data.json`：實際行程

更完整的協作方式請參考 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 版本規則

本專案使用 [Semantic Versioning](https://semver.org/)（SemVer），版本格式為
`大版本.中版本.小版本`（`MAJOR.MINOR.PATCH`）：

- 大版本（MAJOR）：景點大調整
- 中版本（MINOR）：行程微調整
- 小版本（PATCH）：debug 或行程抓錯修復

例如：

- `1.0.0` → `2.0.0`：景點大調整
- `1.0.0` → `1.1.0`：行程微調整
- `1.0.0` → `1.0.1`：debug 或行程抓錯修復

目前版本同時記錄在根目錄的 `VERSION` 檔案與 Git tag（格式為 `v1.0.0`）。
