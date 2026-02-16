# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static website for Rice University's Liu Idea Lab (Lilie) undergraduate entrepreneurship resources. Displays courses, programs, and events sourced from a Google Sheet.

## Architecture

**No build tools, no package manager, no framework.** The entire frontend is two files:

- `index.html` — Single-page app with embedded CSS and vanilla JavaScript (IIFE pattern)
- `data.json` — Content data, auto-generated from a Google Sheet via Apps Script + OpenAI

**Data pipeline:**
```
Google Sheet → Code.gs (Apps Script) → OpenAI API (gpt-4o-mini) → structured JSON → GitHub API → data.json → index.html (fetch + DOM render)
```

- `Code.gs` — Google Apps Script that converts each sheet to a markdown table, sends it to OpenAI with schema instructions, validates the response, and pushes `data.json` to GitHub. Not tracked in git.
- `SETUP.md` — Deployment/setup guide. Not tracked in git.

**Code.gs key functions:**
- `callOpenAI(systemPrompt, userContent)` — generic OpenAI API wrapper with retry on 429
- `sheetToMarkdownTable(sheet)` — converts sheet data to markdown, pre-formatting Date objects
- `formatCellForPrompt(val)` — handles Date objects (→ "Month Day, Year") and 1899 sentinel times (→ "H:MM AM/PM")
- `parseCourses/parsePrograms/parseEvents` — each sends its sheet as markdown + a system prompt defining the JSON schema
- `parseLLMResponse(raw, sectionName)` — validates the LLM returned valid JSON

**Script Properties required:**
- `GITHUB_TOKEN`, `GITHUB_OWNER` (`liu-idea-lab`), `GITHUB_REPO` (`lilie-ug-guide`), `GITHUB_BRANCH` (`main`)
- `OPENAI_API_KEY` — OpenAI API key
- `OPENAI_MODEL` — optional, defaults to `gpt-4o-mini`

## Development

Serve locally with any HTTP server (fetch won't work from `file://`):
```
python3 -m http.server 8000
```

No tests, no linting, no build step. Edit `index.html` directly and refresh the browser.

## Data Format

The OpenAI-based parser in Code.gs produces clean data, but the frontend retains safety-net workarounds for robustness:

1. **Header row filter**: `isHeaderRow()` in index.html still filters rows where name="Event Name" etc., in case the LLM misses one.

2. **Date/time formatters**: `formatDateStr()` and `formatTimeStr()` in index.html handle both pre-formatted strings (from the new pipeline) and raw Date.toString() strings (legacy fallback).

3. **NRLC extraction**: The frontend still checks for NRLC events in `vdw.workshops` as a fallback, though the LLM prompt instructs correct routing to `nrlc.rounds`.

4. **Registration links with notes**: Now split server-side into `registrationLink` and `registrationNote` fields.

5. **Programs may be empty**: `data.json` may have `"programs": []` if the Programs sheet isn't populated yet.

## CSS Conventions

- CSS custom properties defined in `:root` — colors (`--navy`, `--blue`, `--cyan`, `--red`), fonts (`--body`, `--mono`), spacing
- Single breakpoint at 640px for mobile
- No CSS framework — plain flexbox/grid layouts
- XSS prevention via `esc()` function using `document.createTextNode()`

## Publishing Content Updates

Content is published from Google Sheets via the "Publish > Publish to Website" menu, which runs `publishToGitHub()` in Code.gs. This converts each sheet to markdown, calls OpenAI for structured parsing, and pushes the resulting `data.json` to the `main` branch. GitHub Pages auto-deploys. Cost is ~$0.003 per publish with gpt-4o-mini.
