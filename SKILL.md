---
name: notebook-bridge
description: This skill should be used when the user asks to "upload to NotebookLM", "add source to notebook", "ask NotebookLM", "generate audio overview", "generate slide deck", "download from NotebookLM", "下載 NotebookLM 內容", "產生簡報", "上傳到 NotebookLM", "問 NotebookLM", "產生語音摘要", "產生資訊圖表", or discusses using NotebookLM as a knowledge base, uploading research data, generating content from notebooks, or downloading generated content.
version: 0.3.0
tools: mcp__playwright__browser_navigate,mcp__playwright__browser_snapshot,mcp__playwright__browser_click,mcp__playwright__browser_type,mcp__playwright__browser_wait_for,mcp__playwright__browser_file_upload,mcp__playwright__browser_evaluate,mcp__playwright__browser_run_code
argument-hint: <action> [notebook name] [content/URL]
---

# Notebook Bridge

Automate Google NotebookLM via Playwright as a knowledge base. Upload sources, query via
RAG chat, and generate Studio content (audio, slides, infographics, etc.).

ARGUMENTS: The action to perform, notebook name, and content/URL if applicable.

**CRITICAL**: Use Playwright tools ONLY. BrowserTools MCP cannot see Playwright's browser.
Always use `browser_snapshot` to locate elements — never hardcode refs (they change each page load).

## Config

| Parameter | Value | Used In |
|-----------|-------|---------|
| Base URL | `https://notebooklm.google.com/` | Flow A, B, auth check |
| Source processing timeout | 60s (large files: 90s) | Flow C, D, E |
| Chat response timeout | 60s (complex: 90s) | Flow F |
| Studio generation timeout | Audio/Video: 120s, others: 60s | Flow G |
| Batch operation delay | 5s between items | Batch upload |
| Rate limit retry wait | 10s | Error handling |

## Step 0 — Check Memory

```
Read: ~/.claude/skills/notebook-bridge/memory/learnings.md
```

## Step 1 — Classify Action

| Action | Trigger | Workflow |
|--------|---------|----------|
| **Create notebook** | "create notebook X" | → Flow A |
| **Open notebook** | "open notebook X" | → Flow B |
| **Add URL source** | URL provided | → Flow C |
| **Add YouTube** | YouTube URL provided | → Flow C (same as URL) |
| **Add text** | text/content provided | → Flow D |
| **Add file** | file path provided | → Flow E |
| **Chat/Query** | question about sources | → Flow F |
| **Generate content** | "generate audio/slides/..." | → Flow G |
| **Download content** | "download infographic/slides" | → Flow H |
| **Batch upload** | multiple URLs or files | → Flow C/D/E in loop |

## Step 2 — Ensure Notebook Is Open

Before any source/chat/studio action, verify a notebook is open:

1. `browser_snapshot` → check if URL contains `/notebook/`
2. If on homepage → open or create the target notebook first
3. If already in a notebook → proceed

## Flow A — Create Notebook

1. `browser_navigate` → `https://notebooklm.google.com/`
2. `browser_snapshot` → find "建立新的筆記本" button
3. `browser_click` → the create button
4. Wait for notebook to open (URL changes to `/notebook/...`)
5. If user provided a name → click the title textbox, clear it, type new name

## Flow B — Open Existing Notebook

1. `browser_navigate` → `https://notebooklm.google.com/`
2. `browser_snapshot` → scan notebook list for matching name
3. `browser_click` → the matching notebook card
4. `browser_wait_for` → wait for "來源" or "Sources" heading to appear

## Flow C — Add URL / YouTube Source

1. Ensure notebook is open (Step 2)
2. `browser_snapshot` → find "新增來源" (Add Source) button
3. `browser_click` → "新增來源"
4. `browser_snapshot` → find "網站" (Website) button in the dialog
5. `browser_click` → "網站"
6. `browser_snapshot` → find the URL input textbox
7. `browser_type` → paste the URL, then submit
8. `browser_wait_for` → wait for source to appear in source list (up to 30s)
9. `browser_snapshot` → verify source was added

**For multiple URLs:** Repeat steps 2-9 for each. NotebookLM processes one URL at a time.

## Flow D — Add Text Source (Paste)

1. Ensure notebook is open (Step 2)
2. `browser_snapshot` → find "新增來源" button
3. `browser_click` → "新增來源"
4. `browser_snapshot` → find "複製的文字" (Paste text) button
5. `browser_click` → "複製的文字"
6. `browser_snapshot` → find the title input and text area
7. `browser_type` → enter title, then paste content into text area
8. `browser_snapshot` → find and click the submit/save button
9. `browser_wait_for` → source appears in list

**Text length limit:** 500,000 words per source. For very long content, split into multiple sources.

## Flow E — Add File Source (Upload)

1. Ensure notebook is open (Step 2)
2. `browser_snapshot` → find "新增來源" button
3. `browser_click` → "新增來源"
4. `browser_snapshot` → find "上傳檔案" (Upload file) button
5. `browser_click` → "上傳檔案"
6. `browser_file_upload` → provide absolute file path(s)
7. `browser_wait_for` → source appears in list (may take 30-60s for large files)

**Supported formats:** PDF, TXT, Markdown, images, audio (MP3/WAV). Max 200 MB per file.

## Flow F — Chat / Query (RAG)

1. Ensure notebook is open (Step 2)
2. `browser_snapshot` → find chat textbox (placeholder "開始輸入..." or "Start typing...")
3. `browser_type` → type the question, set `submit: true`
4. `browser_wait_for` → wait for response (up to 60s for complex queries)
5. `browser_snapshot` → extract the latest response text from the chat panel
6. Return the response text to the user, noting any citation references

**Tips:**
- Select/deselect specific sources via checkboxes before querying for focused RAG
- Responses include citation numbers linking to source passages

## Flow G — Generate Studio Content

1. Ensure notebook is open (Step 2)
2. `browser_snapshot` → find the Studio panel ("工作室" heading)
3. Locate the target tile button by name:

| Content Type | Button Text (zh-TW) | Has Customize |
|-------------|---------------------|---------------|
| Audio Overview | 語音摘要 | Yes (自訂語音摘要) |
| Video Overview | 影片摘要 | Yes (自訂影片摘要) |
| Mind Map | 心智圖 | No |
| Report | 報告 | No |
| Flashcards | 學習卡 | Yes (自訂學習卡) |
| Quiz | 測驗 | Yes (自訂測驗) |
| Infographic | 資訊圖表 | Yes (自訂資訊圖表) |
| Slide Deck | 簡報 | Yes (自訂簡報) |
| Data Table | 資料表 | Yes (自訂資料表) |

4. If user wants customization → click "自訂..." button → fill in prompt/options
5. Otherwise → `browser_click` the tile button directly
6. `browser_wait_for` → wait for generation (Audio: 30-120s, others: 15-60s)
7. `browser_snapshot` → verify content appeared in Studio panel
8. If download requested → find download/share button and click

## Flow H — Download Generated Content

Download infographics (PNG), slide decks, and other generated content to local disk.

**Why not just click "下載"?** Playwright cannot interact with macOS native save dialogs,
and `page.waitForEvent('download')` times out on NotebookLM's download mechanism.

### Strategy: Extract URL → Cookies → Python urllib

1. Ensure the generated content is visible in the Studio panel
2. **Find the content image URL** via `browser_run_code`:
   ```js
   // For infographics: find large images in the page
   await page.evaluate(() => {
     const imgs = [...document.querySelectorAll('img')];
     return imgs.filter(i => i.naturalWidth > 500)
       .map(i => ({ src: i.src, w: i.naturalWidth, h: i.naturalHeight }));
   });
   ```
   Infographic images are hosted on `lh3.googleusercontent.com/notebooklm/`.

3. **For slide decks**: Click on the generated slide deck in Studio → each slide is
   rendered as an image. Extract all slide image URLs from the DOM.

4. **Extract cookies** via `browser_run_code`:
   ```js
   const cookies = await page.context().cookies('https://lh3.googleusercontent.com');
   const googCookies = await page.context().cookies('https://google.com');
   // Deduplicate and build cookie string
   ```

5. **Download via Python** (Bash tool) — this is the reliable method:
   ```python
   import urllib.request, ssl
   req = urllib.request.Request(url, headers={
       'User-Agent': 'Mozilla/5.0 ...',
       'Accept': 'image/avif,image/webp,image/apng,image/*,*/*;q=0.8',
       'Referer': 'https://notebooklm.google.com/',
       'Cookie': cookie_string,
       'Sec-Fetch-Dest': 'image',
       'Sec-Fetch-Mode': 'no-cors',
       'Sec-Fetch-Site': 'cross-site',
   })
   with urllib.request.urlopen(req, context=ssl.create_default_context()) as resp:
       data = resp.read()
       with open(target_path, 'wb') as f:
           f.write(data)
   ```

6. Verify the download: check file type with `file` command, confirm PNG/PDF header.

### Critical Headers

These headers are **required** — without them the server returns HTML instead of the image:

| Header | Value | Required |
|--------|-------|----------|
| `Referer` | `https://notebooklm.google.com/` | Yes |
| `Sec-Fetch-Dest` | `image` | Yes |
| `Sec-Fetch-Mode` | `no-cors` | Yes |
| `Sec-Fetch-Site` | `cross-site` | Yes |
| `Cookie` | Full cookie string from Playwright | Yes |
| `User-Agent` | Chrome-like UA string | Recommended |

### What Does NOT Work

| Approach | Why It Fails |
|----------|-------------|
| `page.waitForEvent('download')` | macOS native dialog blocks the event |
| `fs.writeFileSync` in `browser_run_code` | Playwright MCP sandboxes Node.js — no `require('fs')` |
| `page.evaluate(() => fetch(url))` | CORS blocks cross-origin fetch from notebooklm.google.com |
| Canvas `toDataURL` | Cross-origin image taints the canvas |
| `curl` without `Referer`/`Sec-Fetch-*` | Returns HTML instead of image |
| CDP `Browser.setDownloadBehavior` | Playwright MCP blocks CDP browser-level targets |

### Alternative A: Chrome AppleScript (Recommended when Playwright unavailable)

When Playwright MCP is disconnected but Chrome is logged into Google:

1. **Find image URLs** via AppleScript JS in the NotebookLM tab:
   ```applescript
   execute theTab javascript "
       Array.from(document.querySelectorAll('img'))
       .filter(i => i.src.includes('lh3.googleusercontent.com/notebooklm'))
       .map(i => i.src)
   "
   ```

2. **For each URL**, open in new Chrome tab → wait → canvas extract → close:
   ```python
   # Open tab (via osascript)
   # Wait 3s for image load
   # Extract: execute javascript "canvas.toDataURL('image/png')"
   # Close tab
   # Decode base64 → save to file
   ```

   Chrome carries full Google auth cookies, bypassing CORS and cookie extraction.
   This method never has cookie expiry issues.

3. **Batch download**: Loop through all URLs with 3s delay per image.

### Alternative B: Verify with context.request (Playwright)

Before extracting cookies, verify the URL is valid using `browser_run_code`:
```js
const resp = await page.context().request.get(imgUrl);
// Returns status 200, content-type: image/png, body as Buffer
```
`page.context().request` carries all cookies automatically and bypasses CORS.
If this returns the correct content-type and size, the URL and auth are valid.

## Source Limits Quick Reference

| Plan | Notebooks | Sources/NB | Queries/day |
|------|-----------|------------|-------------|
| Free | 100 | 50 | 50 |
| Plus | 500 | 300 | 500 |

## Error Handling

Quick reference for common issues. See `references/troubleshooting.md` for detailed
symptoms, solutions, and step-by-step recovery procedures.

- **Login required**: Inform user to log in manually in Playwright's browser, then retry
- **Source processing stuck**: Wait up to 90s, snapshot to check for error messages
- **Dialog not found**: Take fresh snapshot — UI may have changed. Record new pattern to memory
- **Rate limit**: Wait 10s and retry once. Check daily limits in Source Limits table above

## Step 3 — Record Learnings

After each interaction, if any UI pattern changed, new behavior discovered, or workaround needed:

```
Edit: ~/.claude/skills/notebook-bridge/memory/learnings.md
```

Format: `- **[Topic]**: [What was learned] (date: YYYY-MM-DD)`

## Additional Resources

### Reference Files
- **`references/troubleshooting.md`** — Detailed error symptoms, solutions, and recovery procedures
- **`references/ui-patterns.md`** — Known UI element patterns and selectors

### Memory Files
- **`memory/learnings.md`** — Accumulated UI patterns and workarounds
