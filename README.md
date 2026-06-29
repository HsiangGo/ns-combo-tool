# 組合圖產生器 — 開發交接文件

> 給接手的員工/工程師。看完這份就能改、能上線。

## 一、這是什麼
電商商品「組合圖」工具：把商品圖丟進去 → 自動去背 + 排版 → 下載 **1000×1000 PNG** 上架蝦皮/露天。
支援多商品、AI 去背、**賣場樣板**（LOGO / 標題文字 / 促銷紅標）。

- 線上網址：https://hsianggo.github.io/ns-combo-tool/
- GitHub repo：https://github.com/HsiangGo/ns-combo-tool

## 二、技術概況
- **整個工具就是一個 `index.html`**：純前端，無框架、無 build、無後端、無資料庫。
- 用瀏覽器 Canvas 2D 繪圖。HTML / CSS / JavaScript 全寫在這一個檔裡。
- **唯一外部相依**：AI 去背用動態 `import('https://esm.sh/@imgly/background-removal')`，只有使用者按「AI」時才載；其餘功能全離線。
- **隱私**：所有圖片處理都在使用者自己的瀏覽器，**不上傳任何伺服器**。樣板存在瀏覽器 `localStorage`（key：`combo_templates_v1`）。

## 三、怎麼改、怎麼測（最簡單）
1. 用 VS Code（或任何編輯器）打開 `index.html`。
2. 直接編輯。
3. **雙擊 `index.html` 用 Chrome 開** → 改完存檔 → 重整瀏覽器就看到效果。不用安裝任何東西。
4. 出錯時：按 `F12` → Console 看紅色錯誤訊息。

## 四、程式架構導覽（改之前先讀這段）
- **圖層資料 `layers[]`**：畫面上每個東西都是一個物件，靠 `type` 區分：
  - `product`（商品圖）、`logo`（LOGO）、`text`（標題文字）、`badge`（促銷紅標）
  - 共同：`id, type, x, y, name`
  - 影像類（product/logo）：`base`(裁切後原圖) `pc`(去背後畫布) `scale, bw, bh, rm`(去背模式) `tol`(強度)
  - 文字：`text, size, color, weight`；紅標：`text, size, color`
- **去背**：`floodFill()`（快速去白底＝洪水填充演算法）、`aiRemove()`（AI 模型）、`process()`（依 `L.rm` 分派 快速/AI/不去背）
- **繪圖**：`draw()` 主迴圈 → `drawLayer()` 畫單一圖層；`boxOf()` 算每種元素的外框（拖曳/選取要用）；`drawGrid()` 畫座標格線
- **控制面板 UI**：`buildList()` 依每個圖層動態產生控制卡（去背、強度、大小、位置、文字內容…）
- **加入元素**：`addImageLayer(file,type)`、`addText()`、`addBadge()`
- **自動排版**：`arrange()` 把所有 product 等高排成一排
- **賣場樣板**：`serializeDeco()`（把 LOGO/標題/紅標打包）、`applyTemplate()`（套用）、`localStorage` 存取、匯出/匯入 JSON
- **裁白邊**：`trim()` 在加入圖片時自動裁掉四周白邊
- **下載**：`#dl` 的 onclick，用 `cv.toDataURL('image/png')` 匯出

## 五、怎麼上線（部署）
本工具掛在 **GitHub Pages**，**push 到 `master` 分支就自動上線**（約 1 分鐘，網址不變、已設定禁止快取，使用者重整即最新版）。

**路線 A：員工直接接手部署**（需要 repo 權限，跟阿翔要把你加成 collaborator）
```
git clone https://github.com/HsiangGo/ns-combo-tool
# 改 index.html
git add .
git commit -m "說明你改了什麼"
git push
# 等約 1 分鐘 → https://hsianggo.github.io/ns-combo-tool/ 自動更新
```

**路線 B：不想碰 GitHub**
改好 `index.html` → 把檔案傳回給阿翔 → 由阿翔這邊 push 上線。

## 六、功能演進紀錄（changelog）
```
新增：賣場樣板系統（LOGO/標題文字/紅標圖層 + 建立/套用/管理/匯出匯入）
新增：加入時自動裁白邊；位置改座標格線 + X/Y 顯示
改版：版面簡化，每個商品獨立卡片（去背/強度預設0/大小/位置）
修正：單個去背改用按鈕，避免清單重繪時選不出來
去背：逐個商品可設定（快速/AI/不去背），保留整批套用
新增：AI 精準去背（瀏覽器端模型）
新增：下載前可自訂檔名
v1：多商品自動去背組合圖工具
```

## 七、已知限制 / 可加值（待辦）
- AI 去背用免費瀏覽器模型，非白底照的邊緣可能不夠完美 → 可換更強模型。
- 樣板存在各自瀏覽器（localStorage）。要全公司共用：在工具裡「匯出 JSON」→ 改成全站內建（寫進 `BUILTIN` 陣列），或改接後端資料庫。
- 背景目前只有純色，沒有「整張底圖圖片」。
- 尚未做：員工內部限定（密碼）、一鍵輸出多種尺寸。

---
有問題可回報給阿翔，或看 GitHub repo 的 commit 紀錄了解每次改了什麼。
