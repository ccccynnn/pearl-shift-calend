# Cyn’s Work Calendar — enhanced HTML/PWA

## 已加入
- 保留原本班表、假日、心情、備註、外觀等功能。
- 在「規則」頁新增「雲端同步設定」。
- Supabase email magic-link 登入。
- 將 `cynShiftOverrides`（含自訂日班/夜班/休假/加班/請假覆寫）、`pearlShiftNotes`、`pearlShiftMoods` 同步到同一使用者的雲端 JSON state。
- 本機與雲端初次合併，按日期/項目合併，既有本機值優先；雲端資料不會被整體清空。
- Supabase Realtime 更新監聽與手動同步按鈕。
- PWA manifest、圖示與 service worker。
- 保留原本 `.ics` 匯出功能；不會掃描、修改或刪除使用者其他既有行事曆事件。

## 尚未完成（請勿誤認為已上線）
- Google Calendar API OAuth 與真正的雙向事件同步尚未實作。
- 原本 `.ics` 匯出仍屬匯入式流程，不等於雙向同步。
- 雲端同步目前使用整包 JSON state，適合單人初版試用；正式使用前建議改為逐事件資料表、同步佇列、衝突提示、刪除 tombstones 和自動重試。
- service worker 只快取前端檔案；離線期間的新修改不保證會自動排隊並在重連後可靠送出。
- 使用者自訂外觀與上傳背景圖仍只在本機。
- 跨午夜班次與現有班表算法維持原檔行為。

## 啟用雲端同步
1. 建立 Supabase 專案。
2. 在 SQL Editor 執行 `supabase-schema.sql`。
3. 在 Authentication 設定啟用 Email 登入，設定 Site URL 與 Redirect URLs 為實際 HTTPS 網址。
4. 部署整個資料夾到支援 HTTPS 的靜態主機（例如 Netlify、Cloudflare Pages、GitHub Pages）。
5. 用 iPhone Safari 開啟部署網址，進入「規則」頁的「雲端同步設定」。
6. 填入 Supabase Project URL 與 anon/publishable key，輸入 email 並寄送登入連結；在同一裝置開啟連結完成登入。
7. 電腦或另一台裝置開啟同一網址，使用同一帳號登入。

重要：只能放 Supabase anon/publishable key，絕不可把 `service_role` key 放入 HTML。RLS policy 必須保持啟用。

## iPhone 主畫面
1. 將整個資料夾部署到 HTTPS。
2. iPhone 使用 Safari 開啟網址。
3. 分享 → 加入主畫面。

## iPhone 內建行事曆雙向同步：下一個開發階段
最簡單的使用體驗通常是用 Google Calendar 作為橋接：
1. 在 iPhone「設定 → 行事曆 → 帳號 → 加入帳號」加入 Google 帳號，開啟行事曆同步。
2. 建立專用 Google 日曆，例如「輪班與加班」，不要選取或批次修改其他既有日曆。
3. 下一階段需在 Google Cloud Console 啟用 Calendar API、設定 OAuth consent screen 與 redirect URI。
4. 以伺服器端（例如 Supabase Edge Functions）處理 OAuth code、refresh token 與 Calendar API；refresh token 不得暴露在前端。
5. 將本 App 管理的輪班與加班事件映射到專用 Google 日曆，保存 `google_calendar_id`、`google_event_id` 及 sync token。
6. 透過 Google Calendar API 增量同步外部變更，並加入衝突處理、重試與刪除標記。其他日曆事件不應被掃描或修改。

在這個版本，雲端跨裝置同步已加到 HTML，但 Google 日曆雙向同步仍需伺服器端實作與 Google OAuth 部署設定。
