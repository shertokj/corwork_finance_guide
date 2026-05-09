# Claude Cowork — Finance Prospecting Guide

A practitioner's field guide for the **Corporate Cash Investment SVP** on using Claude Cowork, agent skills, scheduled routines, live artifacts, and GitHub integration to run a modern, AI-assisted prospecting practice.

Built on Claude Cowork GA, the February 2026 plugin expansion, and the May 2026 financial services agent launch.

---

## 📖 Live Guide

**[View the guide →](https://shertokj.github.io/corwork_finance_guide/)**

The guide supports two viewing modes:
- **Website mode** — full scrollable document with sticky navigation
- **Slideshow mode** — click the **Slideshow** button in the nav bar for a full-screen presentation. Arrow keys to navigate, Esc to exit. Every section, skill card, routine, and artifact becomes its own slide automatically — context slide followed by a dark terminal prompt slide.

---

## 🔧 Slideshow Builder Skill

This guide's slideshow mode was built and validated using the **slideshow-builder** Claude skill included in this repository.

**[Download slideshow-builder.skill →](https://github.com/shertokj/corwork_finance_guide/raw/main/skills/slideshow-builder.skill)**

### What it does

The skill enforces a strict decomposition contract: every content type maps to a fixed number of slides with defined content limits. Before any push, it spawns three archetype agents:

| Agent | Checks |
|---|---|
| **Skeptic** | Duplicate function definitions, missing edge case guards, document structure integrity |
| **Writing Style** | Em dashes in prose, emoji in callout titles, hype words, rhetorical questions, bullet lists |
| **Presenter** | Prompt word counts, rationale paragraph lengths, estimated slide overflow on 1080p |

Zero violations across all agents are required before deployment is permitted.

### Installation

1. Download `slideshow-builder.skill` from the link above or the `skills/` directory
2. Open Claude Desktop → **Settings → Customize → Skills**
3. Click **Install from file** and select the `.skill` file
4. The skill activates automatically on any slideshow-related task

### Repackaging

The full source is in `skills/slideshow-builder/SKILL.md`. To modify and repackage:
```bash
cd skills/
python3 -m scripts.package_skill slideshow-builder
```

---

## What's Inside the Guide

### Main Guide

| Section | Description |
|---|---|
| The Lay of the Land | What Cowork is and how its five layers work together |
| Setup | Plans, plugins, connectors, Claude in Chrome, project context |
| Workflow 1 — Contact Lists | Build targeted contact lists from scratch |
| Workflow 2 — Pre-Call Brief | Research a prospect before any conversation |
| Workflow 3 — Outreach | Draft and queue personalized first-touch emails |
| Workflow 4 — Daily Digest | Morning market intelligence brief |
| Scheduled Tasks | Recurring task architecture and scheduling logistics |
| Boundaries | What Claude won't do, LinkedIn rules, compliance notes |
| Starting Stack | Recommended tools and cost breakdown |
| First Task | The exact prompt to run today to prove the concept |

### Appendix A — Agent Skills (7 skills)

| Skill | Purpose |
|---|---|
| 01 — Treasury Contact Classifier | Tier 1/2/3 framework applied to every contact Claude surfaces |
| 02 — Pre-Call Research Protocol | Six-section structured brief with data retrieval sequence |
| 03 — Outreach Voice and Compliance Guard | Tone rules, sentence structure, auto-flagging for regulated content |
| 04 — Counterparty Credit Monitor | Materiality classification: immediate notification vs routine log |
| 05 — Conference Intelligence Extractor | Speaker directory extraction cross-referenced against coverage |
| 06 — Market Rate and Instrument Digest | Standardized rate data format with source citation rules |
| 07 — Relationship Status Tracker | Six-status pipeline taxonomy with Gmail scanning and follow-up logic |

### Appendix B — Routines Library (14 routines)

| Routine | Frequency |
|---|---|
| R01 — Morning Market Open | Daily, 6:45 AM |
| R02 — Coverage List Refresh | Weekly, Friday 4 PM |
| R03 — Pipeline Follow-Up Audit | Biweekly, Monday 8 AM |
| R04 — New Coverage Expansion | Monthly, 1st of month |
| R05 — Counterparty Credit Sweep | Monthly, 1st of month |
| R06 — Quarterly Relationship Review | Quarterly |
| R07 — Conference Season Scan | On-demand, pre-conference |
| R08 — Earnings Season Cash Watch | On-demand, earnings windows |
| R09 — Fed Decision Briefing | On-demand, FOMC days |
| R10 — Dormant Re-Engagement Campaign | On-demand, quarterly |
| R11 — New Hire Radar | Weekly, Wednesday 9 AM |
| R12 — Regulatory Watch | Weekly, Thursday 7 AM |
| R13 — Outreach Performance Review | Monthly, last Friday |
| R14 — Annual Coverage Audit | Annual, first week of January |

### Appendix C — Live Artifacts (8 artifacts)

| Artifact | Type | Purpose |
|---|---|---|
| Master Coverage Sheet | Google Sheets | Central contact database |
| Outreach Queue | Google Sheets | Draft staging with compliance flags |
| Call Brief Library | Drive folder | Pre-call research briefs |
| Active Counterparty Watchlist | Google Sheets | Credit monitoring scope |
| Credit Monitor Log | Google Doc | Continuously appended credit event log |
| Market Rate Tracker | Google Sheets | Rolling rate data with charts |
| New Hire Radar Log | Google Sheets | Hire detection with outreach timing |
| Relationship Review Report | Google Doc | Quarterly and annual template |

### Appendix D — GitHub Integration

Publishing weekly HTML analysis pages as live GitHub Pages sites, and version-controlled backup of prospect data as timestamped CSVs. Includes setup steps, four routines with prompts, Skill 08 — GitHub Integration, and compliance review gates.

### Appendix E — Slideshow Builder Skill

Documentation, download link, installation instructions, validation results, and guidance on applying the skill to your own HTML documents.

---

## Repository Structure

```
corwork_finance_guide/
├── index.html                          # The full guide (website + slideshow)
├── README.md                           # This file
├── LICENSE
├── skills/
│   ├── slideshow-builder.skill         # Packaged skill — install in Claude Desktop
│   └── slideshow-builder/
│       ├── SKILL.md                    # Skill source (decomposition contract + agents)
│       └── scripts/                    # Packaging and validation scripts
└── .github/
    └── workflows/
        └── deploy.yml                  # GitHub Actions → GitHub Pages
```

---

## Tools Referenced

| Tool | Purpose | Cost |
|---|---|---|
| Claude Desktop (Max) | Platform | $100/mo |
| Financial Analysis plugin | Core finance skills | Included |
| Sales plugin | Contact research + CRM | Included |
| Apollo | Contact data | $49/mo |
| Claude in Chrome | Browser automation | Included |
| Clay (optional) | Data enrichment | $149/mo |
| FactSet | Market data | Existing subscription |
| Google Workspace | Email, calendar, Drive | Existing |
| GitHub (Free) | Analysis publishing + data backup | Free |

---

## Resources

- [Claude Cowork documentation](https://support.claude.com)
- [Financial services plugins](https://github.com/anthropics/financial-services)
- [Claude Desktop download](https://claude.com/download)
- [Agent skill best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

---

*Last updated May 2026*
