# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file static portfolio (`index.html`) for Agent Co. Financial — a 12-agent AI team showcase. No build step, no framework, no dependencies beyond two CDN scripts (Google Fonts, marked.js). Open `index.html` directly in a browser or serve it locally.

## Local development

```bash
# Serve locally (avoids file:// restrictions)
python3 -m http.server 8000
# then open http://localhost:8080
```

The page works when opened via `file://` — the only exception is that `fetch()` calls fail on `file://`. All artifact markdown content is therefore **embedded directly in the HTML** as a JS object (`ARTIFACT_CONTENT`) rather than loaded at runtime.

## Deploying

```bash
git add -A && git commit -m "..." && git push
```

GitHub Pages auto-deploys from `master`. Live at `agents.ryanrscott.com` (CNAME → `rysco78.github.io`). SSL is managed by GitHub/Let's Encrypt automatically.

## Architecture

Everything lives in `index.html`. Key sections in order:

1. **CSS custom properties** — all design tokens (`--blue`, `--gold`, `--t1/t2/t3`, etc.) defined at `:root`
2. **Section CSS** — nav, hero, concept, team, workflow/graph, gates, projects, modals, footer
3. **Responsive breakpoints** — `1080px`, `820px`, `520px` in a `/* ─── RESPONSIVE */` block near the end of `<style>`
4. **Nav drawer** — hamburger menu hidden above 820px; `.nav-drawer` + `.nav-hamburger` toggled by `toggleDrawer()` JS
5. **Agent data** — `AGENTS` array (inline JS) drives the team grid; each entry has `id, name, type, emoji, tagline, expertise[], owns[], doesnt[]`
6. **Agent modal** — `openModal(id)` / `closeModal()` reads from `AGENTS` array
7. **SVG node graph** — `initAgentGraph()` renders an interactive SVG with 12 nodes and 27 edges; hover highlights connected nodes
8. **`ARTIFACT_CONTENT` object** — ~350KB embedded `<script>` block injected via Python before the main script; keys are `artifact_01` through `artifact_08`; content is the raw markdown of each delivered artifact
9. **Artifact modal** — `openArtifactModal(path, title)` extracts the artifact number from `path`, looks up `ARTIFACT_CONTENT[key]`, renders via `marked.parse()`
10. **Project detail accordion** — `toggleDetail()` animates `max-height` on `.project-detail`

## Updating embedded artifact content

When artifact markdown files in `artifacts/` change, re-inject the embedded data:

```bash
python3 - << 'EOF'
import json, os

artifacts_dir = 'artifacts'
html_path = 'index.html'

data = {}
for i in range(1, 9):
    fname = next((f for f in os.listdir(artifacts_dir) if f.startswith(f'{i:02d}-')), None)
    if fname:
        with open(os.path.join(artifacts_dir, fname)) as f:
            data[f'artifact_{i:02d}'] = f.read()

js_block = f'<script>\nconst ARTIFACT_CONTENT = {json.dumps(data, ensure_ascii=False)};\n</script>\n'

with open(html_path) as f:
    html = f.read()

# Remove existing block if present
import re
html = re.sub(r'<script>\nconst ARTIFACT_CONTENT = \{.*?\};\n</script>\n', '', html, flags=re.DOTALL)

marker = '<!-- ── ARTIFACT MODAL'
html = html[:html.find(marker)] + js_block + html[html.find(marker):]

with open(html_path, 'w') as f:
    f.write(html)
print('Done')
EOF
```

## Section anchor IDs

| Nav label | Section ID |
|-----------|-----------|
| Concept | `#how-it-works` |
| Team | `#team` |
| Orchestration | `#workflow` |
| Controls | `#gates` |
| Projects | `#projects` |
