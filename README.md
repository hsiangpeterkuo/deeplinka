# AI PRO Deep Link 中繼頁

單一靜態 HTML 頁面（`index.html`），用於將外部服務（例如 LINE Bank）的連結導轉到「富邦 AI PRO」App 內指定頁面；未安裝 App 時依裝置類型自動導向 App Store／Google Play。不需要後端、不需要建置工具。

正式測試站台：https://hsiangpeterkuo.github.io/deeplinka/

## 網址參數規則

**只接受 `?d=<base64url>` 一種格式，明文 `?Action=...` 完全無效（不解析、不理會）。**

`d` 的值是「真正要送給 AI PRO 的 query string」做 base64url 編碼後的結果。例如：

```
真正要送的內容：Action=AGREESIGN&MARKET=SK&PAGENO=850
                ↓ base64url 編碼
中繼頁網址：https://hsiangpeterkuo.github.io/deeplinka/?d=QWN0aW9uPUFHUkVFU0lHTiZNQVJLRVQ9U0smUEFHRU5PPTg1MA
                ↓ 頁面自動解碼、組成 deep link
fap-aipro://Launcher?Action=AGREESIGN&MARKET=SK&PAGENO=850
```

`Action` 之後可以帶任意參數，同一份頁面支援 AI PRO 全部 Action（申購簽署、資產總覽、下單、條件單…等 100+ 種，詳見 `AI PRO_CallBack Function` 對照表），不需要每個功能另外開一頁。

### ⚠️ 產生 `d=` 時務必遵守：絕對不要帶結尾的 `=` padding

base64 標準編碼結尾可能會有 `=` 或 `==`（padding）。這個頁面的解碼邏輯**不管有沒有 padding 都能正確解回原文**，但實測發現有些環境（訊息 App 的連結偵測、複製貼上流程等）在網址結尾是 `=` 時會出狀況，導致連結失效。

**規則：產生 `d=` 時一律去掉結尾的 `=`。** 幾乎所有語言的 base64url 編碼函式都有現成方式可以做到：

```python
import base64
d = base64.urlsafe_b64encode(raw_query.encode()).decode().rstrip("=")   # 記得 .rstrip("=")
```

```js
// JS 範例
let d = btoa(rawQuery).replace(/\+/g, "-").replace(/\//g, "_").replace(/=+$/, "");
```

只要是「值本身」（例如加密過的 `idNo`、`contactPhone` 等 token）自己帶 `=`，那個 `=` **保留不動、不要去掉**——上面說的規則只針對最外層 `d=` 這個包裝值結尾的 padding，跟參數值裡面原本就有的字元無關。

## 裝置行為

- **iOS**：導向 `fap-aipro://Launcher?...`；約 2 秒內若頁面仍在前景（代表未安裝或未成功喚起），自動導向 App Store。
- **Android**：導向 `intent://Launcher?...#Intent;scheme=fap-aipro;package=com.fubon.securities.fubonaipro;S.browser_fallback_url=...;end`，由瀏覽器原生處理「已安裝就開 App，未安裝就導去 Google Play」，並加上 JS 計時器作為備援。
- **桌機／其他裝置**：不嘗試喚起 App。顯示「您目前使用電腦裝置，請使用手機下載富邦AI PRO」，灰字倒數 3 秒後自動導向 `https://www.fubon.com/securities/aipro/index.html`（跟 fbs.com.tw 既有頁面 `e19_3.html` 的桌機行為一致）。**這個判斷在解析 `d` 之前就執行，所以桌機上不管網址帶什麼參數，行為都一樣。**

畫面本身沒有任何按鈕或提示文字（桌機那段訊息除外），純粹是自動導轉頁。

## 測試版 App（`?env=test`）

網址加上 `env=test`，deep link 的 scheme 會從正式版 `fap-aipro://` 換成測試版 `efap-aipro://`，用來對應裝在同一台裝置上的 UAT 測試版 App，避免跟正式版 scheme 衝突。

```
https://hsiangpeterkuo.github.io/deeplinka/?env=test&d=QWN0aW9uPUFHUkVFU0lHTiZNQVJLRVQ9U0s
```

不加 `env=test` 就是正式版。這個參數是明文的（不需要、也不會被包進 `d=` 裡）。

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
