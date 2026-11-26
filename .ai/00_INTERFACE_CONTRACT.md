# 00_INTERFACE_CONTRACT.md（生態系黑盒契約與選擇性讀取索引）

- 專案擁有者：Shainy Hsu（https://github.com/shainyhsu，網站 https://shainyhsu.github.io/）
- 索引核對：2026-11-26，main（步程試算 V121 於 14:34 合併）。總首頁 V10、步程試算 V121（357774b，2026-11-26 14:34，行號已重算）、路線剖面比較 V21、上山籌碼評估 V37。本次修改都在同一行內，行號沒有位移。
- 短上下文紅線（SSSM）：單次對話峰值 16K–32K tokens。不整檔貼上大型單檔，一律「函數定位＋局部片段（≤150 行／6,000 字元）」。
- 行號會隨改版漂移：以函數名稱為準。改版後用 `grep -n "function 名稱" index.html` 重新定位，再更新本索引。

## 一、Repo 與頁面

| 模組 | Repo | 主檔（行數） | 版本 |
|---|---|---|---|
| 總首頁（工具清單） | shainyhsu.github.io | index.html（33） | V10 |
| Hike Time Estimator & Walk Record Analysis（步程試算 與 實走紀錄分析） | class-hiking-calculator | index.html（6,268） | V122 |
| Multi-Route Profile Comparison（路線剖面比較） | multi-route-profile-comparison | index.html（2,239） | V21 |
| Leverage Check（上山籌碼評估） | Leveraged-Outdoor-Strategy | index.html（2,320）、member.html（361）、apps-script/Code.gs | V37 |
| BTC & Gold Live Rates、Jet A Fuel Calculator、File to Markdown | markets、fuel、file2md | 見各自 repo 的 .ai/00 | V21、V16、V19 |

頁面網址一律為 https://shainyhsu.github.io/<repo>/。

## 二、關鍵函數索引（Manifest）

附件範本中的 parseGPX()、calcSegmentPace() 在程式中不存在，對應的實際函數為 parseTrackText()、runModelV3()。

### 1. 步程試算：class-hiking-calculator/index.html

核心模組 CORE（L951–2185）：`var CORE = (function () {…})()`，L2185 為 `module.exports = CORE`。抽出 L948–2185 即可用 Node `require` 測試，不需瀏覽器。

- 軌跡解析：parseTrackText() L1002–1062 → buildTrack() L1065–1079、buildTrackRaw() L1103–1138、cleanElevationByDistance() L1083–1102
- 參數：DEFAULTS 物件 L1364 起（Tobler 與 DIN 33466 各半、bodyKg 50）
- 分段：normSections() L1595–1612、secAt() L1613–1616、secTerr() L1593、secHeavy() L1594
- 速度：tobler() L1625、lightHours() L1628–1636（Pandolf 1977 輕裝換算）、kayPace() L1640–1644（跑步）、runHours() L1646–1665、stepHours() L1676–1720、refSpeeds() L1723–1729、stretchHours() L1731–1744
- 主模型：runModelV3() L1747–1799 → applyBuffer() L1817–1827（短休）→ scheduleV3() L1834–1845；clockAtV3() L1846–1854、rainAtV3() L1829–1833
- 勞侖茲：extractLorentzSamples() L1452–1486、normLorentzSamples() L1492–1513、fitLorentz() L1515–1552（Gauss-Newton；樣本 <20 回傳 null）
- 過去紀錄校準：countHeightJumps() L1875–1886、settleStart() L1890–1910、timeGrid() L1911–2045、matchRoute() L2059–2112、calibrate() L2117–2168

頁面層（L2188–5977）：

- 載入檔案：handleFile() L4973–5008（V118：中途車程變搭車路段）、vehicleInfo() L2559–2607、rideSecs() L2611–2621、readKmz() L4966–4972、applyInputs() L4929–4945
- 實走紀錄分析：actGrid() L3978–3984、actParts() L3986–3999、actOnRoute() L4000–4008（V121：在路線 80 m 內實際往前走的長度）、actPick() L4009–4027、actWhen() L4028–4031（V119：跨日或多段的實走紀錄自動選和路線對得最好的一段；V120：有一段對得到就不用整份）、rideGaps() L4032–4043（V115：車程當紀錄中斷，V118 起校準也用）、actData() L4057–4162、actView() L4164–4228、renderAct() L4248–4270、handleActFile() L4298–4310
- 單筆校準框：renderCal() L2842–2908、calBrk() L2909–2949（V113：小休、大休改用實走分析單獨讀這筆紀錄；有「只算其中一段」時只算該段）
- 過去 GPX 擬合（bind() 內）：histAdd() L5550–5577、histBrk() L5586–5592、histTeam() L5610–5618、histDone() L5621–5675
- 畫面：renderNow() L3083–3115、renderSections() L3204–3261、renderSummary() L3264–3304、walkStats() L2513–2525（V118：距離、爬升只算步行）、renderProfile() L3380–3554、renderTable() L3566–3590
- 圖卡與匯出：buildPicDoc0() L4338–4684、makePictures() L4700–4713、sharePicture() L4873–4888、buildGpx() L4814–4860
- 儲存：save() L2303–2314、loadSaved() L2377–2401（localStorage）
- 事件綁定：bind() L5082–5907（825 行，只讀需要的片段）、init() L5909–5973
- 分享短連結（L5978–6265）：seal() L6040–6044、unseal() L6045–6049、shortLink() L6080–6097

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
