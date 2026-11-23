# 核心訓練 Core Coach

Claude Code 專案筆記：每個 session 開始時會自動讀取這份檔案。
使用者用繁體中文溝通，請用繁體中文回覆。

## 專案是什麼

手機用的核心肌群訓練網頁 App（PWA，可加到主畫面）。
跟著節拍數八拍做動作，可以一邊播 YouTube 影片（例如動畫）一邊運動。

- 純靜態網站，沒有建置步驟、沒有套件：直接打開 `index.html` 就能跑。
- 網站從 `main` 分支部署（GitHub Pages）。推上 `main` 約 1～2 分鐘後生效。
- Repo：`ry149/core-coach`；網址 https://rolandyang.com/core-coach/（ry149 帳號的自訂網域，ry149.github.io/core-coach 會轉到這裡）。
  2026-11-21 從 `ro3378/core-coach` 複製過來，舊 repo 不再更新。
- 使用者主要在手機上用，直式和橫式都要顧到。

## 檔案

| 檔案 | 用途 |
|---|---|
| `index.html` | 全部的 HTML、CSS、JS 都在這一個檔案裡 |
| `lottie/*.json` | 每個動作的 Lottie 動畫 |
| `manifest.json` | PWA 設定（名稱、icon、全螢幕、方向） |
| `sw.js` | Service worker，network-first，離線時用快取 |
| `icon.svg` | App icon 原始檔 |
| `icon-192.png`, `icon-512.png` | 從 `icon.svg` 輸出的 PNG |

## index.html 重點位置

- `:root` 顏色 token：`--bg #14161B`、`--panel #1E2128`、`--line #2D313A`、`--text #F2F0EA`、
  `--mute #9AA0AB`、`--accent #FF8A4C`（橘）、`--accent2 #FFC24C`（黃）、`--ok #5BD49A`（綠）。
- `const EX`：所有動作（名稱、等級、bpm、口令 cues、YouTube 搜尋字、動畫檔）。
- `const COURSES`：課程（核心入門／強化／挑戰／全部），休息秒數、八拍數。長度由設定 `S.mins`（15／20／30，預設 20）決定，`buildQueue()` 讓核心動作接近這個長度，伸展另外加在前後。
- `const WARM`：每堂課前後的 5 個伸展（暖身／收操），不算在課程長度內。`const BLOCK`：側棒式－棒式－鳥狗式，永遠排在一起。
- 停留動作（`hold` 秒數，`sides:true` 左右各一次）倒數秒數、不跟拍子；伸展是 `stretch:true`。
- YouTube 續播：`ytSave()` 把目前影片和秒數存在 localStorage `core-yt`，下次同網址從那裡接著播。
- 訓練畫面是 `#run`；有影片時會加上 `.withvideo` class。
- 版面三種情況，CSS 分開寫：
  1. 直式（預設規則）
  2. 橫式、沒影片：`@media (orientation:landscape) and (max-height:600px)` 裡的 `#run:not(.withvideo)`
  3. 橫式、有影片：同一個 media query 裡的 `#run.withvideo`，影片全螢幕，右下角半透明面板
- 節拍顯示：`#beat`（大數字）、`#dots`（8 個節拍點）、`#eight`（「第 N / M 個八拍」）。
  休息倒數和停留倒數是兩位數時，`#beat` 會加上 `.two` class 縮小字級。

## 目前狀態（最近完成的工作）

影片模式的節拍區，已經跟使用者來回調整好：

- **直式**：大數字約 320px（`clamp(160px,min(78vw,36vh),340px)`），左側動作圖佔 40% 寬。
  數字、節拍點、八拍文字排成一組，彼此置中對齊，整組靠右、上下置中。
  節拍點用 `justify-content:space-between` 拉開到跟八拍文字同寬，所以左右兩邊一定切齊。
- **橫式**：右下面板數字約 200px（`clamp(110px,48vh,220px)`），靠右貼上方，跟點點之間留空隙。
  八拍文字（25px／18px）跟節拍點一樣寬。
- **App icon**：深色底、橘色進度環（線寬 34），中間是棒式火柴人（身體線寬 30、手臂 22、頭半徑 23），下方細地板線。

## 使用者的偏好

- 喜歡大而清楚的數字，不介意影片被遮住多一點。
- 很在意對齊：節拍點要對準數字，文字要跟節拍點同寬。
- 每次改完會用手機截圖確認，給的回饋很具體，常會在截圖上畫箭頭標示。
- 改好之後會要求推到 `main`，好在手機上看到。

## 工作方式

- 每次推上 `main` 前，把 `index.html` 的 `VERSION`（v1.N，N 加 1）和 `VERSION_DATE` 改成當天日期；首頁最底下會淡淡地顯示。

- 改版面後，用 Playwright 截圖檢查直式（412×915）和橫式（915×412）兩種尺寸。
  要看影片模式時，在頁面上把 `#run` 加上 `withvideo` class 就好，不用真的載入 YouTube。
- 要量對齊時，用 `getBoundingClientRect()` 比較 `#beat`、`#dots`、`#eight` 的左右邊界。
- 改 icon：編輯 `icon.svg`，再輸出成 192 和 512 的 PNG。
- 已經加到主畫面的 App，要先移除再重新加入，才會換成新 icon。
