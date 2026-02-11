[English](README.md) | [繁體中文](README.zh.md)

# Notebook Bridge

A Claude Code skill that automates Google NotebookLM via Playwright — upload sources, query via RAG chat, and generate Studio content (audio overviews, slide decks, infographics, and more).

## What It Does

Notebook Bridge drives NotebookLM's browser UI programmatically through Playwright:

| Action | Description |
|---|---|
| Create / Open notebook | Create a new notebook or open an existing one by name |
| Add URL source | Import web pages or YouTube videos as sources |
| Add text source | Paste text content directly as a source |
| Add file source | Upload PDF, TXT, Markdown, images, or audio files |
| Chat / Query (RAG) | Ask questions grounded in uploaded sources |
| Generate Studio content | Audio overview, video, mind map, slides, quiz, and more |
| Batch upload | Add multiple URLs or files in sequence |

**Key design principle:** Uses Playwright tools exclusively. Always takes fresh snapshots to locate UI elements rather than relying on hardcoded selectors.

## Installation

1. Clone this repository into your Claude skills directory:

   ```bash
   git clone https://github.com/joneshong-skills/notebook-bridge.git ~/.claude/skills/notebook-bridge
   ```

2. Prerequisites:
   - Playwright MCP server configured in your Claude Code environment
   - A Google account logged in within Playwright's browser session

3. The skill activates when you mention NotebookLM actions like "upload to NotebookLM", "add source to notebook", "generate audio overview", etc.

## Usage

Just ask Claude naturally. Examples:

- *"Create a new NotebookLM notebook called 'Research Project'"*
- *"Upload https://example.com/article to my Research Project notebook"*
- *"Ask my notebook: What are the key findings from the uploaded papers?"*
- *"Generate an audio overview for my Research Project notebook"*

## Project Structure

```
notebook-bridge/
├── SKILL.md                        # Skill definition and automation workflows
├── README.md                       # This file
├── references/
│   └── ui-patterns.md              # Known UI element patterns and selectors
└── memory/
    └── learnings.md                # Accumulated UI patterns and workarounds
```

## License

MIT
