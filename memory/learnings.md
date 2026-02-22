# Notebook Bridge — Learnings

Accumulated UI patterns, workarounds, and behaviors from real interactions.
Format: `- **[Topic]**: [Learning] (date: YYYY-MM-DD)`

## Verified Patterns

- **UI language**: User's NotebookLM displays in zh-TW (繁體中文). All button texts are Chinese (date: 2026-02-11)
- **Login state**: Google account [user-email] is logged in. Playwright browser carries auth cookies (date: 2026-02-11)
- **Homepage layout**: Notebook cards show emoji + title + date + source count. Click card to open notebook (date: 2026-02-11)
- **Add Source dialog**: Triggered by "新增來源" button. Four options: 上傳檔案, 網站, 雲端硬碟, 複製的文字 (date: 2026-02-11)
- **Studio tiles**: 9 content types available: 語音摘要, 影片摘要, 心智圖, 報告, 學習卡, 測驗, 資訊圖表, 簡報, 資料表 (date: 2026-02-11)
- **Chat input**: Placeholder text "開始輸入...", submit button has arrow_forward icon (date: 2026-02-11)
- **Source counter**: Shows in dialog as "N/300" format (Plus plan, 300 max) (date: 2026-02-11)
- **Existing notebooks**: KSA, 啟動未來教育, 簡報視覺識別, 揭秘帕蘭提爾, 文化大學勁風, Entities設計, 潛水風險, Untitled (date: 2026-02-11)

## Download Patterns (2026-02-11)

- **Infographic download method**: `page.context().request.get(url)` fetches the image correctly (returns PNG buffer with full auth). But Playwright MCP sandbox blocks `require('fs')`, so the buffer cannot be saved directly. Solution: extract cookies → Python `urllib.request` with critical headers (date: 2026-02-11)
- **Critical download headers**: `Referer: https://notebooklm.google.com/` and `Sec-Fetch-Dest: image` + `Sec-Fetch-Mode: no-cors` + `Sec-Fetch-Site: cross-site` are REQUIRED. Without them, `lh3.googleusercontent.com` returns HTML instead of the image (date: 2026-02-11)
- **Cookie extraction**: Use `page.context().cookies('https://lh3.googleusercontent.com')` + `page.context().cookies('https://google.com')`, deduplicate by name, build `name=value; ...` string (date: 2026-02-11)
- **Infographic image hosting**: Images at `lh3.googleusercontent.com/notebooklm/` with URL params like `=w2752-d-h1536-mp2?authuser=0` (date: 2026-02-11)
- **Failed approaches**: Playwright download event (blocked by macOS native dialog), CDP `setDownloadBehavior` (browser target not allowed), `page.evaluate(fetch())` (CORS blocks cross-origin), canvas `toDataURL` in Playwright (cross-origin taints canvas), curl without Referer/Sec-Fetch headers (returns HTML), curl with cookies for slide URLs (403 redirect loop — cookies may expire) (date: 2026-02-11)
- **Chrome AppleScript method (BEST)**: Open image URL in new Chrome tab → Chrome loads image with full auth → canvas `toDataURL()` works (same-origin after redirect to `rd-notebooklm` subdomain) → extract base64 → decode to PNG. No CORS, no cookie expiry. Batch: loop with 3s delay per image. (date: 2026-02-11)
- **Slide deck structure**: Each slide is a separate image at `lh3.googleusercontent.com/notebooklm/` with `=w1376-h768` params. Click the slide deck item in Studio to render all slides, then extract img src URLs. (date: 2026-02-11)
- **URL redirect pattern**: `lh3.googleusercontent.com/notebooklm/...` redirects to `lh3.googleusercontent.com/rd-notebooklm/...` when accessed with auth. The `rd-` prefix makes it same-origin for canvas extraction. (date: 2026-02-11)

## Functional Test Results (2026-02-11)

- **Flow A (Create notebook)**: Creating notebook auto-opens Add Source dialog with URL param `?addSource=true` (date: 2026-02-11)
- **Flow C (Add URL)**: Click "網站" → URL input with placeholder "貼上任何連結" → type URL → click "插入" button. Source processes within seconds for web pages (date: 2026-02-11)
- **URL input dialog**: Title is "網站與 YouTube 網址", has tips list (多個網址用空格分隔, 只匯入可見文字, 不支援付費文章, etc.) (date: 2026-02-11)
- **Auto-summary**: After source is added, chat panel automatically generates a Chinese summary with bold key terms and citation buttons (date: 2026-02-11)
- **Flow F (Chat/Query)**: Use `browser_type` with `submit=true` on chat textbox. Response includes formatted tables, headings, numbered citations (button format: "N: Source Name") (date: 2026-02-11)
- **Chat response actions**: 4 buttons — "將訊息儲存至記事" (keep_pin), "將模型回覆複製到剪貼簿" (copy_all), "給予回覆好評" (thumb_up), "給予回覆負評" (thumb_down) (date: 2026-02-11)
- **Flow G (Studio - Mind Map)**: Click tile → shows "正在生成心智圖... 根據 N 個來源" with sync icon → completes in ~20s → appears as clickable button with flowchart icon (date: 2026-02-11)
- **Studio generated item format**: `button "{title} {N} 個來源 · {time} 更多"` with type-specific icon + "更多" (more_vert) menu (date: 2026-02-11)
- **Toast notifications**: "開始生成「{type}」。" on start, "{type}「{title}」已就緒。" on completion (date: 2026-02-11)
