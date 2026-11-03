# 雪國東京 湯澤滑雪＋東京 App

2027/1/28–2/4 五人家庭旅遊（東京 1 晚 → 越後湯澤 3 晚 → 東京 3 晚）的手機網頁 App。
架構與視覺完全沿用「越來越南」河內版（https://coookiepan.github.io/hanoi-trip/）。部署在 GitHub Pages：
https://coookiepan.github.io/yuzawa-trip/

## 為什麼這樣做
上一版是用傳 HTML 檔的方式給大家，手機用 `file://` 開，分頁連結、JSON 讀取、定位全部失效，只看得到首頁。
這版的三個原則不要動：**放在 HTTPS 網址上、整個 App 只有一個 index.html、頁面切換用 hash 路由不換檔案**。

## 技術
- 純 HTML / CSS / JS，沒有框架、沒有打包工具、沒有 npm。改完直接 push 就上線（Pages 約 1 分鐘）。
- PWA：`manifest.json` 加入主畫面，`sw.js` 離線快取。
- 路由：`#/`、`#/itinerary/<day>`、`#/history`、`#/history/<id>`、`#/nearby`、`#/reading`、`#/guide`、`#/guide/<id>`、`#/place/<id>`。
- 不要加 `type="module"`、不要拆成多個 html、不要用相對路徑跳頁。

## 檔案
| 檔案 | 用途 |
|---|---|
| `index.html` | 外殼：頂欄、`#view`、底部膠囊導覽列（SVG 圖標） |
| `app.js` | 路由、各頁 render、當地時間（trip.timezone）、營業中判斷、定位與距離 |
| `style.css` | 樣式，所有顏色在 `:root` |
| `data.json` | **全部內容**。改內容只動這個檔 |
| `sw.js` | 離線快取。改了任何檔案要把第 2 行 `yuzawa-vN` 版本號加 1；新版裝好後 app.js 會自動重載一次 |
| `icons/` | App 圖示 |

## data.json 結構
```
trip      { title, short, headline[2], start "2027-01-28", end, timezone, clockLabel, summary,
            historyLabel, historyDesc, readingDesc, itineraryNote, bases[{placeId,label}] }
historyMeta { title, notice, offCityLabel, more, sitesTitle }   ← 這版把「歷史速查」當「雪場速查」用
days[]    { day, date "1/28", weekday "六", area, stay, items[] }
  items[] { time, type, title, note, status, placeId? }
            type:   交通 | 住宿 | 餐飲 | 景點 | 其他
            status: 已訂 | 固定 | 選項 | 未訂
places[]  { id, name, nameLocal, category, lat, lng, hours, price, intro, recommendedBy, links[], area, needsBooking? }
            category: 景點 | 小吃 | 逛街 | 住宿（住宿不會出現在附近推薦）
            hours:    {"mon-sun":"08:00-17:00"} 或 {"mon-sat":"11:30-14:00,18:00-22:00","sun":"休"}
history[] { id, era, years, title, subtitle, takeaway, body, sites[] }
            body:   Markdown，app.js 認 ### 小標、**粗體**、`程式碼`、> 引用、- 與 1. 清單（可縮排）、| 表格 |、``` 區塊、網址、⚠
            sites[] { name, nameLocal, note, offCity?, placeId? }
                    有 placeId 連到地點頁，沒有就連 Google Maps 搜尋 nameLocal
reading[] { group, items[{ title, url }] }
guide     { title, tagline, searchHint, quick[[sectionId, 標籤]], notes[], parts[] }                ← 基礎知識，首頁方格下方的橫幅進入
  parts[] { id, num "一", title, intro[], sections[] }
    sections[] { id, num "1", title, body }          body 是 Markdown，### 小標會變成段落快速導航
```
行程目前是草案，機票（樂桃，比價中）與住宿都還沒訂，item 的 status 多為「未訂／選項」。原始查證資料與組裝腳本不在 repo 裡，之後直接改 data.json。

## 視覺規範（參考一張 fintech App 設計圖，色碼從圖上取樣）
- 深藍 `#112433`（導覽列、文字）、米白 `#fcf9f2`（背景）、紫 `#977bf2`、萊姆綠 `#dafb62`、天藍 `#8ccdeb`、珊瑚 `#f96e59`
- 狀態色：已訂＝萊姆綠、固定＝天藍、選項＝紫、未訂＝珊瑚；用餐列淡綠底
- 大圓角（22px）、膠囊鈕、粗體大標（Inter 800，退回系統字體）、實心 SVG 圖標，不用 emoji
- 深色模式已處理，新增元件要同時看兩種模式

## 目前狀態與待辦
- 行程總表：8 天草案，全部待確認
- 雪場速查：總覽、滑雪實務、8 個雪場（2026-27 雪季資料為主，部分為 2025-26，標 ⚠ 待確認）
- 附近推薦：湯澤 13 處、東京 24 處；東京座標未逐一校對，起點用越後湯澤站、上野站，訂好住宿後換成飯店
- 基礎知識：21 節＋這趟專屬 4 節（成田往返、住宿區域、湯澤交通、節分）＋資料來源
- 延伸閱讀：官網＋查證來源，可補家人找到的文章
- 地點頁按鈕：Google Maps 導航、電車怎麼搭（Google Maps 大眾運輸）

## 本機預覽
```
python3 -m http.server 8080
```
開 http://localhost:8080 。定位在 localhost 可以用，其他 http 網址不行。

## 測試
改完至少在 iPhone Safari 和 Android Chrome 各開一次，確認五個頁面都進得去。
不要在 LINE 內建瀏覽器測定位。
