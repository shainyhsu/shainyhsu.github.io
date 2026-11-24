# 00_INTERFACE_CONTRACT.md（生態系黑盒契約與選擇性讀取索引）

- 專案擁有者：Shainy Hsu（https://github.com/shainyhsu，網站 https://shainyhsu.github.io/）
- 索引核對：2026-11-24，各 repo main 分支。步程試算 V115（5abe905，2026-11-24 21:32）、路線剖面比較 V19（47cb95b）、上山籌碼評估 V35（64f8943）、總首頁 V9（2ecb565）
- 短上下文紅線（SSSM）：單次對話峰值 16K–32K tokens。不整檔貼上大型單檔，一律「函數定位＋局部片段（≤150 行／6,000 字元）」。
- 行號會隨改版漂移：以函數名稱為準。改版後用 `grep -n "function 名稱" index.html` 重新定位，再更新本索引。

## 一、Repo 與頁面

| 模組 | Repo | 主檔（行數） | 版本 |
|---|---|---|---|
| 總首頁（工具清單） | shainyhsu.github.io | index.html（33） | V9 |
| Hike Time Estimator & Walk Record Analysis（步程試算 與 實走紀錄分析） | class-hiking-calculator | index.html（6,186） | V115 |
| Multi-Route Profile Comparison（路線剖面比較） | multi-route-profile-comparison | index.html（2,239） | V19 |
| Leverage Check（上山籌碼評估） | Leveraged-Outdoor-Strategy | index.html（2,320）、member.html（361）、apps-script/Code.gs | V35 |
| BTC & Gold Live Rates、Jet A Fuel Calculator、File to Markdown | markets、fuel、file2md | 本索引未涵蓋 | — |

頁面網址一律為 https://shainyhsu.github.io/<repo>/。

## 二、關鍵函數索引（Manifest）

附件範本中的 parseGPX()、calcSegmentPace() 在程式中不存在，對應的實際函數為 parseTrackText()、runModelV3()。

### 1. 步程試算：class-hiking-calculator/index.html

核心模組 CORE（L940–2174）：`var CORE = (function () {…})()`，L2174 為 `module.exports = CORE`。抽出 L937–2174 即可用 Node `require` 測試，不需瀏覽器。

- 軌跡解析：parseTrackText() L991–1051 → buildTrack() L1054–1068、buildTrackRaw() L1092–1127、cleanElevationByDistance() L1072–1091
- 參數：DEFAULTS 物件 L1353 起（Tobler 與 DIN 33466 各半、bodyKg 50）
- 分段：normSections() L1584–1601、secAt() L1602–1605、secTerr() L1582、secHeavy() L1583
- 速度：tobler() L1614、lightHours() L1617–1625（Pandolf 1977 輕裝換算）、kayPace() L1629–1633（跑步）、runHours() L1635–1654、stepHours() L1665–1709、refSpeeds() L1712–1718、stretchHours() L1720–1733
- 主模型：runModelV3() L1736–1788 → applyBuffer() L1806–1816（短休）→ scheduleV3() L1823–1834；clockAtV3() L1835–1843、rainAtV3() L1818–1822
- 勞侖茲：extractLorentzSamples() L1441–1475、normLorentzSamples() L1481–1502、fitLorentz() L1504–1541（Gauss-Newton；樣本 <20 回傳 null）
- 過去紀錄校準：countHeightJumps() L1864–1875、settleStart() L1879–1899、timeGrid() L1900–2034、matchRoute() L2048–2101、calibrate() L2106–2157

頁面層（L2177–5895）：

- 載入檔案：handleFile() L4868–4900、readKmz() L4860–4867、applyInputs() L4835–4852
- 實走紀錄分析：actGrid() L3936–3951（V115：車程當紀錄中斷）、actData() L3965–4069、actView() L4071–4135、renderAct() L4155–4176、handleActFile() L4204–4216
- 單筆校準框：renderCal() L2806–2872、calBrk() L2873–2913（V113：小休、大休改用實走分析單獨讀這筆紀錄；有「只算其中一段」時只算該段）
- 過去 GPX 擬合（bind() 內）：histAdd() L5441–5468、histBrk() L5477–5483、histTeam() L5501–5509、histDone() L5512–5566
- 畫面：renderNow() L3047–3079、renderSections() L3165–3221、renderSummary() L3224–3264、renderProfile() L3340–3512、renderTable() L3524–3548
- 圖卡與匯出：buildPicDoc0() L4244–4590、makePictures() L4606–4619、sharePicture() L4779–4794、buildGpx() L4720–4766
- 儲存：save() L2292–2303、loadSaved() L2366–2390（localStorage）
- 事件綁定：bind() L4974–5825（852 行，只讀需要的片段）、init() L5827–5891
- 分享短連結（L5896–6183）：seal() L5958–5962、unseal() L5963–5967、shortLink() L5998–6015

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
- Baseline 目標：Baseline Widely Available（MWG 1.0 預設）。新 UI 優先用 `<dialog>`、Popover API、CSS Grid／Subgrid；目前三頁都還沒用到 `<dialog>`／Popover。

## 四、單向資料流契約（Standard TrailJSON）〔目標契約，尚未實作〕

- 步程試算解析 GPX 後輸出分段陣列：`{ segmentId, distKm, elevGainM, elevLossM, gradePct, terrainFactor, weatherFactor, loadFactor, estMinutes, stopMinutes }`，與摘要 `{ totalDist, totalGain, maxSlope, estTotalTime, fatigueIndex }`。
- 路線剖面比較與籌碼評估只消費上述陣列或摘要，不得在各自 repo 重寫 GPX 解析或步程計算。
- 現況差距（2026-11-24）：
  - 路線剖面比較自有 parseXml()，與步程試算 parseTrackText() 重複。
  - 籌碼評估目前不讀路線資料，需求軸由使用者評分。
  - 步程試算現有分段為 normSections() 輸出 `{ a, b, real, terr, heavy, rain, run, scale, off }`（a、b 為 GPX 距離 m）；runModelV3() 回傳 `{ x, cum, xr, rs, rn, bd, total, realTotal, … }`。TrailJSON 欄位由這兩者換算。
  - fatigueIndex 無現成對應（疲勞池已於 v58 移除），須 Shainy 先定義。
