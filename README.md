# Agent Co. Financial — Portfolio

Portfolio site for Agent Co. Financial, a 12-agent AI team that builds regulated digital financial products for the automotive lending industry.

**Live site:** [agents.ryanrscott.com](https://agents.ryanrscott.com)

---

## Overview

The site showcases the agent team, their orchestration model, mandatory compliance gates, and an active project — the Auto Leads Platform, a soft-pull prequalification and lead generation platform for Toyota and Lexus franchise dealers.

## Stack

- Single-file static site (`index.html`) — no build step, no framework
- [marked.js](https://marked.js.org/) (CDN) for client-side markdown rendering
- Hosted on GitHub Pages with a custom domain

## Local development

```bash
python3 -m http.server 8000
```

Open `http://localhost:8000`. The page also works via `file://` directly.

## Deployment

Push to `master` — GitHub Pages deploys automatically.

```bash
git add -A && git commit -m "your message" && git push
```

## Project: Auto Leads Platform

The active project featured on the site. A consumer-facing prequalification platform where buyers complete a soft-pull prequal flow and approved outcomes are delivered as ADF leads to Toyota and Lexus franchise dealers via DealerTrack and RouteOne.

**Current status:** Phase 2 of 4 — Design & Definition complete

| Phase | Status |
|-------|--------|
| Phase 1 — Discovery & Strategy | ✅ Complete |
| Phase 2 — Design & Definition | ✅ Complete |
| Phase 3 — Build & Delivery | ⏳ Planned |
| Phase 4 — Quality & Compliance | ⏳ Planned |

Delivered artifacts are viewable directly on the site. Hi-fi prototypes (Toyota and Lexus) are available in Figma.

## Repository structure

```
index.html          # Full site — all HTML, CSS, and JS
artifacts/          # Markdown source for delivered project artifacts (01–08)
CNAME               # Custom domain configuration for GitHub Pages
CLAUDE.md           # Guidance for Claude Code when working in this repo
```
