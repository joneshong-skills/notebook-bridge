# Troubleshooting Guide

Common errors, their symptoms, and solutions for NotebookLM automation.

## Table of Contents

- [Authentication Issues](#authentication-issues)
- [Source Processing Failures](#source-processing-failures)
- [UI Element Not Found](#ui-element-not-found)
- [Chat / Query Issues](#chat--query-issues)
- [Studio Generation Failures](#studio-generation-failures)
- [Rate Limiting](#rate-limiting)

---

## Authentication Issues

### Symptom: Snapshot shows Google login page

The Playwright browser session has no valid cookies, or cookies have expired.

**Solution:**
1. `browser_navigate` → `https://notebooklm.google.com/`
2. Inform user: "Please log in to your Google account in the Playwright browser window."
3. `browser_wait_for` → wait for text "建立新的筆記本" or "Create new notebook" (up to 120s)
4. Once logged in, cookies persist for the session. Proceed with the original task.

### Symptom: Redirected to accounts.google.com mid-operation

Session expired during a long operation (>30 min idle).

**Solution:**
1. Stop current flow
2. `browser_navigate` → `https://notebooklm.google.com/`
3. If login page appears → follow login procedure above
4. Re-open the target notebook and retry the failed operation

### How to verify login state

1. `browser_navigate` → `https://notebooklm.google.com/`
2. `browser_snapshot` → check for:
   - **Logged in**: "建立新的筆記本" button visible, notebook list present
   - **Not logged in**: Google login form, "Sign in" text, or redirect URL contains `accounts.google.com`

---

## Source Processing Failures

### Symptom: Source stuck in "processing" state

After adding a URL or file, the source doesn't appear in the list within 60s.

**Solutions (try in order):**
1. `browser_snapshot` → check if an error message appeared in the dialog
2. If no error → wait another 30s (large files can take up to 90s)
3. If still processing → close the add-source dialog, refresh the page, check source list
4. If source is missing → retry adding it

### Symptom: "Unable to process this source" error

The URL is paywalled, geo-restricted, or returns non-HTML content.

**Solutions:**
- Paywalled articles: Copy the article text manually, use Flow D (paste text) instead
- PDF URLs: Download the file first, use Flow E (file upload) instead
- YouTube: Verify the video is public and not age-restricted

### Symptom: File upload fails silently

`browser_file_upload` completes but no source appears.

**Solutions:**
1. Verify file path is **absolute** (not relative)
2. Check file size (max 200 MB)
3. Check file format: PDF, TXT, MD, MP3, WAV, images are supported
4. For unsupported formats → convert to PDF or TXT first

---

## UI Element Not Found

### Symptom: Expected button/dialog not in snapshot

The most common issue. NotebookLM updates its UI frequently.

**Solutions:**
1. Take a fresh `browser_snapshot` — refs change every page load
2. Look for **similar text** (Google may A/B test different labels)
3. Check if the page is fully loaded: look for the three-panel layout (來源 / 對話 / 工作室)
4. If a dialog is expected but missing → the previous click may not have registered. Retry the click.
5. **Record the new pattern** to `memory/learnings.md` for future reference

### Symptom: Language changed from zh-TW

If user changed their Google account language, all button texts change.

**Solutions:**
- Use `browser_snapshot` and match by element structure rather than exact text
- Common mappings: "來源" = "Sources", "新增來源" = "Add source", "工作室" = "Studio"
- Update `references/ui-patterns.md` if persistent language change detected

---

## Chat / Query Issues

### Symptom: Chat response takes very long (>60s)

Complex queries across many sources can be slow.

**Solutions:**
1. Wait up to 90s before timing out
2. If still no response → check if the response is streaming (partial text visible in snapshot)
3. Reduce scope: deselect irrelevant sources via checkboxes before querying
4. Simplify the question

### Symptom: Response is generic / doesn't cite sources

NotebookLM couldn't find relevant information in the loaded sources.

**Solutions:**
- Verify the right sources are selected (checked) in the Sources panel
- Rephrase the question to match terminology used in the sources
- Check if sources were fully processed (not still in "processing" state)

---

## Studio Generation Failures

### Symptom: Generation button is disabled

Not enough sources loaded, or sources are still processing.

**Solutions:**
1. Ensure at least 1 source is fully processed
2. For Audio/Video Overview: some source types may not be supported
3. `browser_snapshot` → check if there's a tooltip or error message on the disabled button

### Symptom: Generation starts but fails

Toast message "生成失敗" or content doesn't appear after waiting.

**Solutions:**
1. Wait the full timeout (Audio: up to 120s, others: up to 60s)
2. Check source quality — very short or low-content sources may cause failures
3. Retry once. If it fails again, try with different/fewer sources selected
4. For custom generation ("自訂...") → simplify the customization prompt

---

## Rate Limiting

### Symptom: Actions are throttled or return errors

NotebookLM has daily query limits and may throttle rapid successive operations.

**Limits:**
| Plan | Queries/day | Source adds/day |
|------|-------------|-----------------|
| Free | ~50 | ~50 |
| Plus | ~500 | ~300 |

**Solutions:**
1. Wait 10-15 seconds between operations
2. For batch uploads: add a 5s delay between each source
3. If hard limit reached → inform user, suggest continuing tomorrow or upgrading to Plus
