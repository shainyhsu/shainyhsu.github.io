# 01_CURRENT_STATE.md（SSSM 語義狀態空間快照）

- 最後更新：2026-11-24 13:39（台灣時間）
- 標記：〔Node 實測〕＝抽出步程試算 CORE（index.html L936–2145）以 Node 22 執行確認；〔程式碼核對〕＝讀 main 分支原始碼確認。兩者都不是瀏覽器實機測試。

## 1. 逐字任務目標（Verbatim Objective & Done Criteria）

- 當前唯一原子任務：〔待 Shainy 指定〕
- 完成驗收條件：〔待 Shainy 指定〕
- 候選（Claude 2026-11-24 盤點時提出，未經 Shainy 確認；擇一後改寫上兩行並刪除其餘）：
  1. TrailJSON 轉換器：在 CORE 內新增 toTrailJSON()，由 normSections()＋runModelV3() 換算 00 第四節欄位（fatigueIndex 先留空）。驗收：Node 載入 CORE 對測試 GPX 印出符合契約的陣列，頁面計算結果完全不變。
  2. 刪除 bind() 內體重綁定死碼：mainBodyKg 兩段重複（L5597–5615）與 pBody（L5624–5631），兩個元素已不存在；中間的 pLightKg（L5616–5623）仍在使用，保留。驗收：行為不變，grep 無 mainBodyKg／pBody 殘留。
  3. JSZip 改為讀 KMZ 時才載入（比照路線剖面比較 loadJSZip()），移除 L64 的同步 `<script>`。驗收：只讀 GPX／KML 時不請求 cdnjs，KMZ 仍可讀。

## 2. 已驗證事實（Verified Facts）

- 步程試算現行 V112（2026-11-24 09:31，commit 4a95c9f）；附件寫的「穩定版 v109」已過時（V109 為 2026-11-23 19:52）。〔程式碼核對〕
- 速度基準 Tobler 與 DIN 33466 各半：wTob 0.5；Tobler 倍率 F 0.734；DIN 平地 3.62 km/h、上坡 468 m/h、下坡 791 m/h；300 m 以上每升高 1,000 m 上坡慢 6.3%。〔Node 實測〕
- 基準體重 bodyKg 50、校準負重 calKg 13、輕裝 lightKg 2；applyInputs() 每次重設 bodyKg = 50（L4750）。〔Node 實測＋程式碼核對〕
- 分段可調：normSections() 每段獨立 terr／heavy／rain／run，未設定區間補「一般步道・重裝・晴」。〔Node 實測〕
- 勞侖茲擬合：有效紀錄 ≥3 筆且樣本 ≥20 才呼叫 fitLorentz()（histDone() 內 L5448）；樣本 19 個時 fitLorentz() 回傳 null。〔Node 實測＋程式碼核對〕
- 時間餘裕 MARGIN 1.1，畫面勾選框 marginOn 預設勾選。〔Node 實測＋程式碼核對〕
- 實走紀錄分析區分移動與停留（actData() 的 stop／gap）；V112 起上傳過去 GPX 會量測隊伍每小時短休，可套用到計畫。〔程式碼核對〕
- 路線剖面比較 V19（2026-11-13）、上山籌碼評估 V35（2026-11-17）、總首頁 V9（2026-11-17）已上線。〔程式碼核對〕
- 三頁皆無 Service Worker、manifest；未用 IndexedDB／OPFS、`<dialog>`、Popover；狀態存 localStorage。〔程式碼核對〕

## 3. 未驗證想法與待確認假設（Unverified，不得當作已實作）

- [Unverified] 三工具合併為單一離線 App；未測大於 5 MB GPX 在手機 IndexedDB 的讀寫延遲。
- [Unverified] TrailJSON 單向資料流（00 第四節）尚未實作；fatigueIndex 需 Shainy 定義。
- [Unverified] SSSM 能省 token 費用：arXiv:2611.05306 證實短上下文下準確率大致不降，未證實總費用下降。第三方分析：論文主實驗（上下文滿額才壓縮）的名目費用，GPT 22 組設定有 21 組、Opus 22 組有 7 組高於不壓縮，主因提示快取失效；單一動作壓縮的費用未量測。

## 4. 已套用變更與下一步唯一動作（Applied Changes & Immediate Next Action）

- 已修改檔案（2026-11-24）：shainyhsu.github.io 新增 .ai/00–03、AGENTS.md、CLAUDE.md；另 6 個公開工具 repo 與 1 個私人 repo 新增 AGENTS.md、CLAUDE.md 入口檔；網頁與程式碼未動。
- 下一步唯一動作：Shainy 從第 1 節候選擇一（或另定），改寫「當前唯一原子任務」與「完成驗收條件」。
