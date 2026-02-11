# NotebookLM UI Patterns Reference

Observed UI patterns from Playwright snapshots. These are guidelines — always use
`browser_snapshot` to find actual element refs at runtime.

Last verified: 2026-02-11

## Homepage (notebooklm.google.com/)

### Key Elements
- **Create notebook**: button text "建立新的筆記本" (with add icon)
- **Notebook cards**: each card has a button with the notebook title, e.g.
  `button "KSA 職能判斷..." [cursor=pointer]`
- **Tab filters**: radiogroup with "全部", "我的筆記本", "精選筆記本", "與我共用"
- **Sort**: button "選取專案篩選器類型" with dropdown (最新, etc.)
- **View toggle**: "格狀檢視" / "清單檢視"

### Notebook Card Structure
```
generic [cursor=pointer]
  ├── button "{notebook title}"
  ├── generic (emoji icon)
  ├── button "專案動作選單" (more_vert icon)
  ├── generic "{title text}"
  └── generic "{date}·{N} 個來源"
```

## Notebook Interior (notebooklm.google.com/notebook/{id})

### Three-Panel Layout
1. **Sources panel** (left) — heading "來源"
2. **Chat panel** (center) — heading "對話"
3. **Studio panel** (right) — heading "工作室"

### Sources Panel
- **Add source button**: `button "新增來源"` (with add icon)
- **Discover sources search**: `textbox` with placeholder "在網路上搜尋新來源"
- **Source list**: each source has checkbox + name
- **Select all**: `checkbox "選取所有來源"`

### Add Source Dialog
Triggered by clicking "新增來源". Dialog title: "用文件 製作語音和影片摘要"

Buttons inside dialog:
| Button | Text (zh-TW) | Icon |
|--------|-------------|------|
| Upload file | "上傳檔案" | upload |
| Website/URL | "網站" | link + video_youtube |
| Google Drive | "雲端硬碟" | drive |
| Paste text | "複製的文字" | content_paste |

- **Close dialog**: `button "關閉"` (close icon)
- **Source counter**: shows "N/300" (current/max)
- **Discover search**: same textbox with "網路" and "Fast Research" toggles

### URL Sub-Dialog (after clicking "網站")
- **Back button**: `button "返回"` (arrow_back icon)
- **Title**: "網站與 YouTube 網址"
- **URL input**: `textbox "輸入網址"` with placeholder "貼上任何連結"
- **Submit button**: `button "插入"` (disabled until URL entered)
- **Tips list**: 多個網址用空格或換行分隔, 只匯入可見文字, 不支援付費文章, etc.

### Chat Response Actions (updated from live test)
- `button "將訊息儲存至記事"` (keep_pin icon)
- `button "將模型回覆複製到剪貼簿"` (copy_all icon)
- `button "給予回覆好評"` (thumb_up icon)
- `button "給予回覆負評"` (thumb_down icon)

### Studio Generation Progress
- During generation: `button "正在生成{type}... 根據 {N} 個來源"` [disabled] (sync icon)
- Toast start: "開始生成「{type}」。"
- Toast complete: "{type}「{title}」已就緒。"

### Chat Panel
- **Chat input**: `textbox "查詢方塊"` with placeholder "開始輸入..."
- **Submit button**: `button "提交"` (disabled until text entered, arrow_forward icon)
- **Source count indicator**: shows "N 個來源" near chat input
- **Suggested questions**: appear as clickable buttons below the summary
- **Response structure**: headings + paragraphs with citation buttons like `button "1: {source}"`
- **Response actions**: "儲存至記事" (save), copy, thumbs up/down

### Studio Panel
Tile buttons (top row):
| Content | Button Text | Customize Button |
|---------|------------|-----------------|
| Audio Overview | "語音摘要" | "自訂語音摘要" |
| Video Overview | "影片摘要" | "自訂影片摘要" |
| Mind Map | "心智圖" | — |
| Report | "報告" | — |
| Flashcards | "學習卡" | "自訂學習卡" |
| Quiz | "測驗" | "自訂測驗" |
| Infographic | "資訊圖表" | "自訂資訊圖表" |
| Slide Deck | "簡報" | "自訂簡報" |
| Data Table | "資料表" | "自訂資料表" |

Generated content appears below tiles as clickable items:
```
button "{title} {N} 個來源 · {days} 天前 更多"
  ├── icon (type-specific: tablet for slides, stacked_bar_chart for infographic, etc.)
  ├── generic (title + metadata)
  └── button "更多" (more_vert)
```

- **Add note**: `button "新增記事"` (at bottom of Studio panel)

### Notebook Title
- Editable: `textbox` containing the notebook title (in top header bar)
- Click to edit, type new name

### Settings & Actions (top bar)
- "建立筆記本" — create new notebook from within a notebook
- "數據分析" — analytics
- "共用筆記本" / "共用" — share
- "設定" — settings

## URL Detection

| URL Pattern | Location |
|------------|----------|
| `/` | Homepage |
| `/notebook/{uuid}` | Inside a notebook |
| `/notebook/{uuid}?addSource=true` | Add source dialog open |

## Language Notes

The UI language follows the user's Google account language setting.
Current user: zh-TW (繁體中文).

If the user switches language, button texts will change. Always find elements by
snapshot analysis rather than hardcoded text.
