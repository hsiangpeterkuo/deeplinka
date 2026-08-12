# AI PRO Deep Link 中繼頁

單一靜態 HTML 頁面（`index.html`），用於將外部服務（例如 LINE Bank）的連結導轉到「富邦 AI PRO」App 內指定頁面；未安裝 App 時依裝置類型自動導向 App Store／Google Play。不需要後端、不需要建置工具。

正式測試站台：https://hsiangpeterkuo.github.io/deeplinka/

## 網址參數規則

中繼頁的 query string **直接對應**要傳給 AI PRO 的 Deep Link 參數，明文原樣透傳，不需要額外編碼。例如：

```
中繼頁網址：https://hsiangpeterkuo.github.io/deeplinka/?Action=AGREESIGN&MARKET=SK&PAGENO=850
                ↓ 頁面自動組成 deep link
fap-aipro://Launcher?Action=AGREESIGN&MARKET=SK&PAGENO=850
```

`Action` 之後可以帶任意參數，同一份頁面支援 AI PRO 全部 Action（申購簽署、資產總覽、下單、條件單…等 100+ 種，詳見 `AI PRO_CallBack Function` 對照表），不需要每個功能另外開一頁。必要參數：`Action`，缺少時頁面不會有任何動作（空白頁）。

> 曾經試過用 `?d=<base64url>` 把 query string 包起來做輕度混淆，但拿掉了——base64 包裝對「有心人用 DevTools 看」沒有實質防護效果（瀏覽器最終還是要解碼成明文才能執行），純粹增加除錯與對接的複雜度，所以改回明文。

## 裝置行為

- **iOS**：導向 `fap-aipro://Launcher?...`；約 2 秒內若頁面仍在前景（代表未安裝或未成功喚起），自動導向 App Store。
- **Android**：導向 `intent://Launcher?...#Intent;scheme=fap-aipro;package=com.fubon.securities.fubonaipro;S.browser_fallback_url=...;end`，由瀏覽器原生處理「已安裝就開 App，未安裝就導去 Google Play」，並加上 JS 計時器作為備援。
- **桌機／其他裝置**：不嘗試喚起 App。顯示「您目前使用電腦裝置，請使用手機下載富邦AI PRO」，灰字倒數 3 秒後自動導向 `https://www.fubon.com/securities/aipro/index.html`（跟 fbs.com.tw 既有頁面 `e19_3.html` 的桌機行為一致）。**這個判斷在解析參數之前就執行，所以桌機上不管網址帶什麼參數，行為都一樣。**

畫面本身沒有任何按鈕或提示文字（桌機那段訊息除外），純粹是自動導轉頁。

## 測試版 App（`?env=test`）

網址加上 `env=test`，deep link 的 scheme 會從正式版 `fap-aipro://` 換成測試版 `efap-aipro://`，用來對應裝在同一台裝置上的 UAT 測試版 App，避免跟正式版 scheme 衝突。

```
https://hsiangpeterkuo.github.io/deeplinka/?env=test&Action=AGREESIGN&MARKET=SK
```

不加 `env=test` 就是正式版。`env` 是保留字，不會被轉發到最終的 deep link 裡。

## 固定設定（如需異動請直接修改 `index.html` 內常數）

```js
SCHEME_NAME       = "fap-aipro"          // 正式版 scheme
TEST_SCHEME_NAME  = "efap-aipro"         // 測試版 scheme（?env=test 時使用）
IOS_STORE_URL     = "https://apps.apple.com/tw/app/id6751573234"
ANDROID_STORE_URL = "https://play.google.com/store/apps/details?id=com.fubon.securities.fubonaipro&hl=zh_TW"
ANDROID_PACKAGE   = "com.fubon.securities.fubonaipro"
DESKTOP_URL       = "https://www.fubon.com/securities/aipro/index.html"
```

## 部署

- 目前部署在 GitHub Pages（`DEPLOY.md`），僅供測試。
- 之後要換到公司網域可參考 `FBS_DEPLOY.md`（部署在 fbs.com.tw，比照現有 `e19_3.html` 的模式，已驗證不會被過濾）、`WCM_DEPLOY.md`、`INTERNAL_DEPLOY.md`。
