[English](README.md) | [繁體中文](README.zh.md)

# Notebook Bridge

一個 Claude Code 技能，透過 Playwright 自動化操作 Google NotebookLM — 上傳來源、透過 RAG 對話查詢，以及產生工作室內容（語音摘要、簡報、資訊圖表等）。

## 功能特色

Notebook Bridge 透過 Playwright 程式化驅動 NotebookLM 的瀏覽器介面：

| 動作 | 說明 |
|---|---|
| 建立 / 開啟筆記本 | 建立新筆記本或依名稱開啟現有筆記本 |
| 新增 URL 來源 | 匯入網頁或 YouTube 影片作為來源 |
| 新增文字來源 | 直接貼上文字內容作為來源 |
| 新增檔案來源 | 上傳 PDF、TXT、Markdown、圖片或音訊檔案 |
| 對話 / 查詢（RAG） | 針對已上傳來源提問 |
| 產生工作室內容 | 語音摘要、影片、心智圖、簡報、測驗等 |
| 批次上傳 | 依序新增多個 URL 或檔案 |

**核心設計原則：** 僅使用 Playwright 工具。始終取得最新快照來定位 UI 元素，而非依賴寫死的選擇器。

## 安裝

1. 將此倉庫 clone 到 Claude 技能目錄：

   ```bash
   git clone https://github.com/joneshong-skills/notebook-bridge.git ~/.claude/skills/notebook-bridge
   ```

2. 前置需求：
   - Claude Code 環境中已設定 Playwright MCP 伺服器
   - Playwright 瀏覽器中已登入 Google 帳號

3. 當您提到 NotebookLM 相關操作如「上傳到 NotebookLM」、「新增來源到筆記本」、「產生語音摘要」等時，技能會自動啟動。

## 使用方式

直接自然地詢問 Claude。範例：

- *「建立一個叫『研究專案』的 NotebookLM 筆記本」*
- *「上傳 https://example.com/article 到我的研究專案筆記本」*
- *「問我的筆記本：上傳論文的主要發現是什麼？」*
- *「幫我的研究專案筆記本產生語音摘要」*

## 專案結構

```
notebook-bridge/
├── SKILL.md                        # 技能定義及自動化工作流程
├── README.md                       # 英文說明
├── README.zh.md                    # 繁體中文說明（本檔案）
├── references/
│   └── ui-patterns.md              # 已知 UI 元素模式和選擇器
└── memory/
    └── learnings.md                # 累積的 UI 模式和解決方案
```

## 授權

MIT
