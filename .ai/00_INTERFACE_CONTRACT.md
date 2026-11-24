# 00_INTERFACE_CONTRACT.md（生態系黑盒契約與選擇性讀取索引）

- 專案擁有者：Shainy Hsu（https://github.com/shainyhsu，網站 https://shainyhsu.github.io/）
- 索引核對：2026-11-24，各 repo main 分支。步程試算 V112（4a95c9f）、路線剖面比較 V19（47cb95b）、上山籌碼評估 V35（64f8943）、總首頁 V9（2ecb565）
- 短上下文紅線（SSSM）：單次對話峰值 16K–32K tokens。不整檔貼上大型單檔，一律「函數定位＋局部片段（≤150 行／6,000 字元）」。
- 行號會隨改版漂移：以函數名稱為準。改版後用 `grep -n "function 名稱" index.html` 重新定位，再更新本索引。

## 一、Repo 與頁面

| 模組 | Repo | 主檔（行數） | 版本 |
|---|---|---|---|
| 總首頁（工具清單） | shainyhsu.github.io | index.html（33） | V9 |
| Hike Time Estimator & Walk Record Analysis（步程試算 與 實走紀錄分析） | class-hiking-calculator | index.html（6,100） | V112 |
| Multi-Route Profile Comparison（路線剖面比較） | multi-route-profile-comparison | index.html（2,239） | V19 |
| Leverage Check（上山籌碼評估） | Leveraged-Outdoor-Strategy | index.html（2,320）、member.html（361）、apps-script/Code.gs | V35 |
| BTC & Gold Live Rates、Jet A Fuel Calculator、File to Markdown | markets、fuel、file2md | 本索引未涵蓋 | — |

頁面網址一律為 https://shainyhsu.github.io/<repo>/。

## 二、關鍵函數索引（Manifest）

附件範本中的 parseGPX()、calcSegmentPace() 在程式中不存在，對應的實際函數為 parseTrackText()、runModelV3()。

### 1. 步程試算：class-hiking-calculator/index.html

核心模組 CORE（L939–2145）：`var CORE = (function () {…})()`，L2145 為 `module.exports = CORE`。抽出 L936–2145 即可用 Node `require` 測試，不需瀏覽器。

- 軌跡解析：parseTrackText() L990–1050 → buildTrack() L1053–1067、buildTrackRaw() L1091–1126、cleanElevationByDistance() L1071–1090
- 參數：DEFAULTS 物件 L1352 起（Tobler 與 DIN 33466 各半、bodyKg 50）
- 分段：normSections() L1583–1600、secAt() L1601–1604、secTerr() L1581、secHeavy() L1582
- 速度：tobler() L1613、lightHours() L1616–1624（Pandolf 1977 輕裝換算）、kayPace() L1628–1632（跑步）、runHours() L1634–1653、stepHours() L1664–1708、refSpeeds() L1711–1717、stretchHours() L1719–1732
- 主模型：runModelV3() L1735–1787 → applyBuffer() L1805–1815（短休）→ scheduleV3() L1822–1833；clockAtV3() L1834–1842、rainAtV3() L1817–1821
- 勞侖茲：extractLorentzSamples() L1440–1474、normLorentzSamples() L1480–1501、fitLorentz() L1503–1540（Gauss-Newton；樣本 <20 回傳 null）
- 過去紀錄校準：countHeightJumps() L1863–1874、settleStart() L1878–1898、timeGrid() L1899–2005、matchRoute() L2019–2072、calibrate() L2077–2128

頁面層（L2148–5809）：

- 載入檔案：handleFile() L4782–4814、readKmz() L4774–4781、applyInputs() L4749–4766
- 實走紀錄分析：actData() L3879–3983、actView() L3985–4049、renderAct() L4069–4090、handleActFile() L4118–4130
- 過去 GPX 擬合（bind() 內）：histAdd() L5355–5382、histBrk() L5391–5397、histTeam() L5415–5423、histDone() L5426–5480
- 畫面：renderNow() L2976–3008、renderSections() L3094–3150、renderSummary() L3153–3193、renderProfile() L3269–3441、renderTable() L3453–3477
- 圖卡與匯出：buildPicDoc0() L4158–4504、makePictures() L4520–4533、sharePicture() L4693–4708、buildGpx() L4634–4680
- 儲存：save() L2263–2274、loadSaved() L2337–2361（localStorage）
- 事件綁定：bind() L4888–5739（852 行，只讀需要的片段）、init() L5741–5805
- 分享短連結（L5810–6097）：seal() L5872–5876、unseal() L5877–5881、shortLink() L5912–5929

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
