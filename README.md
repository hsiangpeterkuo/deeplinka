# AI PRO Deep Link 中繼頁

單一靜態 HTML 頁面，用於將外部服務（例如 LINE Bank）的連結導轉到「富邦 AI PRO」App 內指定頁面；若使用者尚未安裝 App，則依裝置類型自動導向 App Store／Google Play。

不需要後端、不需要建置工具，任何靜態網站託管（GitHub Pages、Netlify、Vercel、Firebase Hosting、S3 + CloudFront、公司內部 Nginx 等）皆可直接部署 `index.html`。

## 運作方式

1. 使用者在來源網站（如 LINE Bank）點擊連結，帶到本中繼頁，網址參數即為 AI PRO 想要的 Deep Link 參數（見下方「網址參數規則」）。
2. 頁面載入後依裝置類型自動嘗試開啟 App：
   - **iOS**：導向 `fap-aipro://Launcher?...`；約 2 秒內若頁面仍在前景（代表未安裝或未成功喚起），自動導向 App Store。
   - **Android**：導向 `intent://Launcher?...#Intent;scheme=fap-aipro;package=com.fubon.securities.fubonaipro;S.browser_fallback_url=...;end`，由瀏覽器原生處理「已安裝就開 App，未安裝就導去 Google Play」，並額外加上 JS 計時器作為備援。
   - **桌機／其他裝置**：不嘗試喚起 App，顯示提示文字與下載按鈕。
3. 畫面上同時提供「手動開啟」按鈕與商店下載按鈕，避免自動跳轉在少數瀏覽器（尤其部分 iOS 版本限制未經使用者手勢觸發的 URL scheme 導轉）失效時，使用者無法繼續。

## 網址參數規則

中繼頁的 query string **直接對應**要傳給 AI PRO 的 Deep Link 參數，原樣透傳，不需要每個功能都另外設定，因此同一份頁面可支援所有 AI PRO 支援的 Action（申購簽署、資產總覽、下單、條件單…等，共 100+ 種，詳見 `AI PRO_CallBack Function` 對照表）。

```
https://<你的網域>/?Action=AGREESIGN&MARKET=SK&PAGENO=850
```

會被轉成：

```
fap-aipro://Launcher?Action=AGREESIGN&MARKET=SK&PAGENO=850
```

只要是對照表中「AI Pro 參數」欄位的組合，直接原封不動放進中繼頁的網址即可，例如：

| 用途 | 中繼頁網址 |
|---|---|
| 申購／股票申購紀錄簽署（範例） | `?Action=AGREESIGN&MARKET=SK&PAGENO=850` |
| 一般線上簽署 | `?Action=AGREESIGN` |
| 資產總覽 | `?Action=FBSASSET` |
| 台股下單帶入股號 | `?Action=SO&STOCKID=1108` |
| 期貨線上簽署帶條件單預告書 | `?Action=AGREESIGNFO&MARKET=FU&PAGENO=23` |

必要參數：`Action`（大小寫皆可，但建議照對照表原樣帶入）。缺少 `Action` 時頁面會顯示錯誤提示，不會嘗試喚起 App。

## 固定設定（如需異動請直接修改 `index.html` 內常數）

```js
APP_SCHEME_HOST   = "fap-aipro://Launcher"
IOS_STORE_URL     = "https://apps.apple.com/tw/app/id6751573234"
ANDROID_STORE_URL = "https://play.google.com/store/apps/details?id=com.fubon.securities.fubonaipro&hl=zh_TW"
ANDROID_PACKAGE   = "com.fubon.securities.fubonaipro"
```

## 部署建議

- 建議掛在獨立子網域，例如 `deeplink.fubon-aipro.example.com`，方便日後調整不影響其他系統。
- 為避免被搜尋引擎索引，`index.html` 已加上 `<meta name="robots" content="noindex, nofollow">`。
- 若來源方（LINE Bank）是在 App 內 WebView 開啟本頁，建議請對方確認該 WebView 允許導向自訂 URL scheme（`fap-aipro://`）與 `intent://`，多數主流 WebView（含 LINE 內建瀏覽器）皆支援。
- 目前僅做前端導轉，未蒐集任何個資或串接後端；如未來需要記錄點擊或轉換率，可另外掛 GA4 / 自建 log endpoint。

## 本機測試

不需任何安裝，直接用瀏覽器開啟 `index.html`，或起一個簡單靜態伺服器：

```bash
cd aipro-deeplink
python3 -m http.server 8080
```

再用手機瀏覽器連到你電腦的區網 IP（例如 `http://192.168.1.10:8080/?Action=AGREESIGN&MARKET=SK&PAGENO=850`）即可在實機上測試 Deep Link 與商店導轉行為。桌機瀏覽器僅會顯示裝置提示畫面，無法驗證實際跳轉。
