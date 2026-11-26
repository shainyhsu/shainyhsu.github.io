# 00_INTERFACE_CONTRACT.md（生態系黑盒契約與選擇性讀取索引）

- 專案擁有者：Shainy Hsu（https://github.com/shainyhsu，網站 https://shainyhsu.github.io/）
- 索引核對：2026-11-26，步程試算依 V119 分支；其他頁依 main（theme-meta 於 09:35 合併）。總首頁 V10、步程試算 V119（分支 v119-act-best-day，PR #24 待合併；行號依 V119 重算）、路線剖面比較 V21、上山籌碼評估 V37。本次修改都在同一行內，行號沒有位移。
- 短上下文紅線（SSSM）：單次對話峰值 16K–32K tokens。不整檔貼上大型單檔，一律「函數定位＋局部片段（≤150 行／6,000 字元）」。
- 行號會隨改版漂移：以函數名稱為準。改版後用 `grep -n "function 名稱" index.html` 重新定位，再更新本索引。

## 一、Repo 與頁面

| 模組 | Repo | 主檔（行數） | 版本 |
|---|---|---|---|
| 總首頁（工具清單） | shainyhsu.github.io | index.html（33） | V10 |
| Hike Time Estimator & Walk Record Analysis（步程試算 與 實走紀錄分析） | class-hiking-calculator | index.html（6,273） | V119（PR #24 待合併） |
| Multi-Route Profile Comparison（路線剖面比較） | multi-route-profile-comparison | index.html（2,239） | V21 |
| Leverage Check（上山籌碼評估） | Leveraged-Outdoor-Strategy | index.html（2,320）、member.html（361）、apps-script/Code.gs | V37 |
| BTC & Gold Live Rates、Jet A Fuel Calculator、File to Markdown | markets、fuel、file2md | 見各自 repo 的 .ai/00 | V21、V16、V19 |

頁面網址一律為 https://shainyhsu.github.io/<repo>/。

## 二、關鍵函數索引（Manifest）

附件範本中的 parseGPX()、calcSegmentPace() 在程式中不存在，對應的實際函數為 parseTrackText()、runModelV3()。

### 1. 步程試算：class-hiking-calculator/index.html

核心模組 CORE（L952–2186）：`var CORE = (function () {…})()`，L2186 為 `module.exports = CORE`。抽出 L949–2186 即可用 Node `require` 測試，不需瀏覽器。

- 軌跡解析：parseTrackText() L1003–1063 → buildTrack() L1066–1080、buildTrackRaw() L1104–1139、cleanElevationByDistance() L1084–1103
- 參數：DEFAULTS 物件 L1365 起（Tobler 與 DIN 33466 各半、bodyKg 50）
- 分段：normSections() L1596–1613、secAt() L1614–1617、secTerr() L1594、secHeavy() L1595
- 速度：tobler() L1626、lightHours() L1629–1637（Pandolf 1977 輕裝換算）、kayPace() L1641–1645（跑步）、runHours() L1647–1666、stepHours() L1677–1721、refSpeeds() L1724–1730、stretchHours() L1732–1745
- 主模型：runModelV3() L1748–1800 → applyBuffer() L1818–1828（短休）→ scheduleV3() L1835–1846；clockAtV3() L1847–1855、rainAtV3() L1830–1834
- 勞侖茲：extractLorentzSamples() L1453–1487、normLorentzSamples() L1493–1514、fitLorentz() L1516–1553（Gauss-Newton；樣本 <20 回傳 null）
- 過去紀錄校準：countHeightJumps() L1876–1887、settleStart() L1891–1911、timeGrid() L1912–2046、matchRoute() L2060–2113、calibrate() L2118–2169

頁面層（L2189–5982）：

- 載入檔案：handleFile() L4951–4986（V118：中途車程變搭車路段）、vehicleInfo() L2560–2608、rideSecs() L2612–2622、readKmz() L4943–4950、applyInputs() L4918–4935
- 實走紀錄分析：actGrid() L3979–3985、actParts() L3987–4000、actPick() L4001–4016、actWhen() L4017–4020（V119：跨日或多段的實走紀錄自動選和路線對得最好的整份或一段）、rideGaps() L4021–4032（V115：車程當紀錄中斷，V118 起校準也用）、actData() L4046–4151、actView() L4153–4217、renderAct() L4237–4259、handleActFile() L4287–4299
- 單筆校準框：renderCal() L2843–2909、calBrk() L2910–2950（V113：小休、大休改用實走分析單獨讀這筆紀錄；有「只算其中一段」時只算該段）
- 過去 GPX 擬合（bind() 內）：histAdd() L5528–5555、histBrk() L5564–5570、histTeam() L5588–5596、histDone() L5599–5653
- 畫面：renderNow() L3084–3116、renderSections() L3205–3262、renderSummary() L3265–3305、walkStats() L2514–2526（V118：距離、爬升只算步行）、renderProfile() L3381–3555、renderTable() L3567–3591
- 圖卡與匯出：buildPicDoc0() L4327–4673、makePictures() L4689–4702、sharePicture() L4862–4877、buildGpx() L4803–4849
- 儲存：save() L2304–2315、loadSaved() L2378–2402（localStorage）
- 事件綁定：bind() L5060–5912（852 行，只讀需要的片段）、init() L5914–5978
- 分享短連結（L5983–6270）：seal() L6045–6049、unseal() L6050–6054、shortLink() L6085–6102

### 2. 路線剖面比較：multi-route-profile-comparison/index.html

- 軌跡解析：parseXml() L431–471、readFile() L483–492、loadJSZip() L473–482（讀 KMZ 才載入）
- DEM 補缺：getTile() L546–564、demAt() L565–571、ensureDtm() L1519–1531
- 路線資料：newRoute() L613–624、base() L625–672、prof() L688–780
- A4 圖卡：buildDoc() L800–1292、drawPreview() L1537–1551、setDpi() L1806–1814、makeJpgs() L1819–1831、saveBlob() L1840–1850
- 剖面編輯：drawEdit() L1566–1631、edWire() L1704–1793
- 儲存：saveState() L1860–1863、loadState() L1864–1874
- 分享短連結（L1971–2225）：與步程試算同構

### 3. 上山籌碼評估：Leveraged-Outdoor-Strategy

- index.html：radarSVG() L1248–1272、radarCards() L1278–1309、renderDemandStatic() L1364–1374、demandRow() L1375–1390、memberCard() L1425–1444、updateMembersDynamic() L1469–1496、renderLevers() L1511–1525、ensureTrip() L1543–1553、pullForm() L1559–1579、printWaiver() L1584–1632、a4Pages() L1645–1957、svgToJpg() L1958–1970、shareA4() L1982–1999、loadState() L1207–1212、saveState() L1213
- member.html：render() L236–261、loadTeam() L304–315、send() L316–348
- apps-script/Code.gs：doPost() L37–47、create() L92–111、submit() L121–148、pull() L166–174、team() L176–186、purgeDaily() L199–202

## 三、核心架構與現代網頁原則（MWG 1.0）

- 純前端運算：所有計算在瀏覽器完成，無自架伺服器。對外連線僅：DEM 圖磚（osmhacktw.github.io）、JSZip（cdnjs）、Umami 計數、分享短連結（瀏覽器端加密後存 Google 表單／試算表，金鑰留在網址 # 之後）、籌碼評估隊員填表（Google Apps Script）。
- 離線優先：〔目標，未實作〕目前無 Service Worker、無 manifest；狀態存 localStorage，未用 IndexedDB／OPFS。
- Baseline 目標：Baseline Widely Available（MWG 1.0 預設）。新 UI 優先用 `<dialog>`、CSS Grid／Subgrid；Popover API 2025-01-27 才進入 Newly available、2027-07-27 才進入 Widely，採用時須照 MWG 指南附備援；目前三頁都還沒用到 `<dialog>`／Popover。
- 外部腳本：一律鎖版本並加 SRI（integrity＋crossorigin="anonymous"＋referrerpolicy="no-referrer"），雜湊須與 npm 套件原檔逐位元一致。2026-11-25 起：步程試算 JSZip（靜態）、路線剖面比較 JSZip（讀 KMZ 才動態載入）、markets Chart.js 4.6.2、file2md 五個函式庫。
- 深淺色：`<meta name="color-scheme">` 預設 dark，開頁腳本與切換鈕同步更新；CSS 仍以 data-theme 決定顏色。改用 :has(meta) 對應與 light-dark()，要等各頁 Safari 最低版本到 15.4／17.5。

## 四、單向資料流契約（Standard TrailJSON）〔目標契約，尚未實作〕

- 步程試算解析 GPX 後輸出分段陣列：`{ segmentId, distKm, elevGainM, elevLossM, gradePct, terrainFactor, weatherFactor, loadFactor, estMinutes, stopMinutes }`，與摘要 `{ totalDist, totalGain, maxSlope, estTotalTime, fatigueIndex }`。
- 路線剖面比較與籌碼評估只消費上述陣列或摘要，不得在各自 repo 重寫 GPX 解析或步程計算。
- 現況差距（2026-11-24）：
  - 路線剖面比較自有 parseXml()，與步程試算 parseTrackText() 重複。
  - 籌碼評估目前不讀路線資料，需求軸由使用者評分。
  - 步程試算現有分段為 normSections() 輸出 `{ a, b, real, terr, heavy, rain, run, scale, off }`（a、b 為 GPX 距離 m）；runModelV3() 回傳 `{ x, cum, xr, rs, rn, bd, total, realTotal, … }`。TrailJSON 欄位由這兩者換算。
  - fatigueIndex 無現成對應（疲勞池已於 v58 移除），須 Shainy 先定義。
