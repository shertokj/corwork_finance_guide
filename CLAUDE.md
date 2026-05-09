# Claude Code Project — corwork_finance_guide

## What This Is
This repository contains a single-page HTML guide: **Claude Cowork for Finance Prospecting** — a practitioner's field guide for a Corporate Cash Investment SVP using Claude Cowork for prospecting, outreach, and market intelligence.

**Live site:** https://shertokj.github.io/corwork_finance_guide/

## Repository Structure
```
index.html          — The entire guide (single self-contained HTML file, ~3100 lines)
README.md           — GitHub-formatted documentation
.github/
  workflows/
    deploy.yml      — GitHub Actions: auto-deploys index.html to GitHub Pages on every push to main
```

## The Single-File Architecture
Everything lives in `index.html`: all HTML content, CSS (inline `<style>`), and JavaScript (inline `<script>` before `</body>`). No build tools, no npm, no dependencies beyond Google Fonts loaded via CDN.

### CSS Variable System
All colors reference CSS custom properties defined at `:root`:
- `--ink` `#0f0f0e` — primary dark (masthead, step numbers, dark cards)
- `--paper` `#f5f2ec` — page background
- `--cream` `#ede9e0` — card backgrounds, nav bar
- `--gold` `#c8a84b` — accent color (labels, borders, slideshow button)
- `--gold-light` `#e8d49a` — italic emphasis in headings
- `--slate` `#3d4a5c` — secondary text, info callout borders
- `--muted` `#8a8578` — nav text, footer text
- `--rule` `#d4cfc4` — divider lines, card borders

### Typography
Three typefaces, all from Google Fonts:
- **DM Serif Display** — all headings (`h1`, `h2`, `h3`, `.masthead h1`, etc.)
- **DM Sans** — body text
- **DM Mono** — labels, badges, code, terminal prompts, nav links

## Key Components

### Content Sections
The guide has 6 main sections + 4 appendices:
- `#overview` — The Eight Layers (Part One)
- `#setup` — Setup steps (Part Two)
- `#workflows` — Four prospecting workflows (Part Three)
- `#scheduled` — Scheduled tasks (Part Four)
- `#caveats` — Boundaries (Part Five)
- `#stack` — Recommended stack + first task (Part Six)
- `#appendix-skills` — 7 agent skills with creation prompts
- `#appendix-routines` — 14 named routines
- `#appendix-artifacts` — 8 live artifacts
- `#appendix-github` — GitHub integration guide

### CSS Component Classes
| Class | Purpose |
|---|---|
| `.masthead` | Dark hero header |
| `.masthead-slideshow-btn` | Gold CTA button in hero |
| `.toc-bar` | Sticky nav bar |
| `.body-wrap` | Main content container (max-width: 900px) |
| `.stack-grid` | 4×2 grid of layer cards |
| `.steps` | Numbered step list |
| `.skill-card` | Appendix A skill cards |
| `.routine-card` | Appendix B routine cards |
| `.artifact-card` | Appendix C artifact cards |
| `.workflow-header/.workflow-body` | Workflow sections |
| `.prompt-block` | Terminal-style prompt UI (transformed by JS) |
| `.callout-info/.warn/.tip` | Callout boxes |
| `.pricing-table` | Tables (stack, schedule) |
| `.appendix-divider` | Dark full-width appendix title cards |
| `.appendix-body` | Appendix content container |
| `.ss-*` | All slideshow overlay components |

### JavaScript (at bottom of file, inside `<script>`)
Two systems:

**1. Prompt Block Transformer** (runs immediately at parse time)
Queries all `.prompt-block` elements and replaces their HTML with a macOS-style terminal UI: traffic light dots, centered filename tab, Copy button. Copy button uses `navigator.clipboard` with `execCommand` fallback.

**2. Slideshow Engine** (functions defined at parse time, triggered by button click)
- `enterSlideshow()` — called by the gold CTA button `onclick`. Calls `buildSlides()` then activates the overlay.
- `buildSlides()` — reads the live DOM and constructs a slide deck. Runs once (guarded by `SS.built`). Slide types: `title`, `section`, `appendix`, `content`.
- `goToSlide(n)` — handles transition animation (forward = slide left, backward = slide right using `requestAnimationFrame`).
- `slideshowNav(dir)` — called by Prev/Next buttons.
- `exitSlideshow()` — called by Exit button and Escape key.
- Keyboard handler on `document` for ← → Space arrows and Esc.

## Deployment
Push to `main` → GitHub Actions workflow triggers → Pages deploys within ~30 seconds.

```bash
git add index.html README.md
git commit -m "Description of change"
git push origin main
```

The workflow file (`.github/workflows/deploy.yml`) uses:
- `actions/checkout@v4`
- `actions/configure-pages@v5`
- `actions/upload-pages-artifact@v3`
- `actions/deploy-pages@v4`

Pages source is set to `workflow` (not legacy branch/folder).

## Common Edit Patterns

### Adding a new skill to Appendix A
Find the last `.skill-card` inside `#appendix-skills .appendix-body` and add a new one following the same structure:
```html
<div class="skill-card">
  <div class="skill-card-header">
    <span class="skill-badge">Skill 0N</span>
    <h3>Skill Name</h3>
  </div>
  <div class="skill-card-body">
    <div class="skill-meta-row">...</div>
    <p>Description</p>
    <div class="prompt-block">
      <div class="prompt-label">→ Skill Creation Prompt</div>
      <p>Prompt text here</p>
    </div>
  </div>
</div>
```
The JS will automatically include it in the slideshow deck.

### Adding a new routine to Appendix B
Same pattern using `.routine-card` with `.routine-index`, `.routine-header`, `.routine-body`.

### Changing the color scheme
All colors are CSS variables at `:root` in the `<style>` block. Change `--gold` to update all gold accents simultaneously.

### Updating the Eight Layers grid
The grid is in `#overview` and uses `.stack-grid` with `.stack-card` children. Currently 8 cards in a 4×2 layout (`grid-template-columns: repeat(4, 1fr)`).

## Content Notes
- All prose follows a specific writing style: no em dashes, no hype language, formal but direct, no bullet-point defaults
- Prompt blocks use `<div class="prompt-label">→ Label</div><p>Text</p>` — the JS transforms this into terminal UI
- The slideshow automatically picks up all `.skill-card`, `.routine-card`, `.artifact-card`, and `[id^="workflow-"]` elements — no manual registration needed
