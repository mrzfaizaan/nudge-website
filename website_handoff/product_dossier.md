# Nudge — Product Dossier

> Generated from source codebase `todo-banner` for marketing website construction.

---

## Product Identity

**Name:** Nudge
**Tagline:** *The Ambient HUD for High-Stakes Focus*
**Identifier:** `com.mrzfaizaan.nudge`
**Version:** 0.1.0

### Core Philosophy

> *"Eliminating the friction of context switching."*

Nudge is a persistent heads-up display that lives at the top of your monitor and keeps you on track while you work in other windows. It is **not** a to-do app. It is a **teleprompter for your life.**

Most productivity tools demand your full attention. Nudge demands nearly none. It occupies **2px of vertical space** when idle and expands to a readable banner when you need it. The current task is always in frame. The hierarchy is always visible. You never switch contexts to check *"what's next."*

The logo — **Progressive Stream** — is a set of horizontal bars transitioning from dense to sparse, representing raw data flowing through the parser and emerging as structured, actionable tasks. Completion tames the stream.

### Voice / Brand Tone

- Concise, opinionated, technical
- No hype, no gamification, no notifications
- Frames itself as a *cognitive anchor* not a *productivity tool*
- Describes itself with industrial precision (absolute Cartesian coordinates, 4-pass state machine, 3-tier node system)

---

## Technical Flex

| Dimension | Specification |
|---|---|
| **Runtime** | Tauri v2 (Rust binary + webview, no Electron) |
| **Frontend** | Vanilla TypeScript + Vite, zero framework runtime |
| **Binary footprint** | ~4 MB on disk (release build) |
| **Frontend payload** | ~46 KB gzipped (HTML + CSS + JS) |
| **Window mode** | Frameless, transparent, always-on-top, non-resizable |
| **Default window** | 800 × 85 px, expands to 120–500 px per view |
| **Security** | CSP disabled (null) for local file access |
| **Parsing engine** | 3-Tier Node Parser in Rust (regex-based state machine) |
| **File formats** | `.txt`, `.md`, `.json` |

### Multi-Monitor Spatial Windowing

Nudge uses **absolute Cartesian coordinates** to manage window placement across multiple displays:

- Each monitor's physical position, size, and scale factor are read independently via `available_monitors()` (Tauri window API)
- Position presets (top-left, top-center, top-right) are calculated in **physical pixels** from the target monitor's origin, not logical units
- The user selects which monitor via a dropdown populated dynamically from the monitor enumeration
- Width is set as a percentage of the target monitor's width (default 35%), clamped to real pixel boundaries
- Banner height is purely logical (120 px banner, 420 px settings, 500 px expanded view) and scaled by the monitor's `scaleFactor`
- Auto-hide shrinks the window to 2 px logical height (effectively invisible) with a configurable delay (default 0.5 s)
- On hover, the window restores to its full logical height and re-positions using the stored monitor/preset

This means **each monitor preserves its own geometry contract**. Moving the banner to monitor 2 with "top-right" selected correctly places it at `(monitor2.x + monitor2.width - bannerWidth, monitor2.y)`.

---

## The Engine: 3-Tier Node Parser

The parser is a Rust state machine in `src-tauri/src/lib.rs` that runs 4 classification passes over the input file and assigns one of three `display_mode` values to each line.

### The Three Node Types

| Node Type | Prefix | `display_mode` | UI Rendering | Toggle-able? |
|---|---|---|---|---|
| **Action** | `[ ]`, `[x]`, `- [ ]`, `- [x]` | `"action"` | Checkbox □ / ☑ with strikethrough on completion | Yes |
| **Bullet** | `* ` (asterisk-space) | `"bullet"` | Neutral bullet dot •, no completion style | No |
| **Plain** | `- ` (dash-space BUT NOT `- [ ]`) | `"plain"` | No checkbox, text only | No |

### How It Works (4 Passes)

```
Pass 1 — classify_line()
  Regex-based classification into:
    • HeadingA (h1:/h2:/h3: tags)
    • Decorative (====)
    • Dashes (----)
    • TaskAC / TaskD ([ ] / - [ ] action nodes)
    • Bullet (* ) 
    • Plain (-  or unrecognized text)
    • HeadingD (Markdown # syntax)
    • Plain (generic text fallthrough)

Pass 2 — resolve_format_c()
  Dashes immediately preceded by a plain line → Format C heading
  (e.g. "My Heading\n----" becomes a HeadingC)

Pass 3 — resolve_format_b()
  Plain lines immediately followed by Bullet items → Format B heading
  (e.g. "Tasks\n* Item" makes "Tasks" a HeadingB)

Pass 4 — build_tasks()
  Assigns display_mode based on which LineType matched:
    • TaskAC / TaskD → display_mode: "action"
    • Bullet         → display_mode: "bullet"
    • Plain          → display_mode: "plain"
  Maintains heading_stack for Markdown hierarchy breadcrumbs
```

### Toggle Behavior

Only `"action"` nodes trigger file mutation. `toggle_line_in_file()` rewrites the line on disk, swapping `[ ]` ↔ `[x]` or `- [ ]` ↔ `- [x]`. Bullet and plain nodes silently return `Ok(false)` with no file modification.

---

## Target Audience & Use Cases

### Primary Users

| Persona | Why Nudge |
|---|---|
| **Engineers / Developers** | Keeps standup talking points, code review queues, or deployment steps visible without alt-tabbing |
| **Researchers** | Literature review beats, chapter milestones, data collection steps — visible during writing |
| **Presenters / Performers** | Slide talking points, podcast outlines, live demo scripts — the audience sees the content, not the notes |
| **Sales & Client-facing** | Call agendas, objection handling sequences, meeting talking points — no paper shuffling |
| **Clinical & Therapy** | Speech exercises, CBT prompts, physiotherapy step sequences — persistent until done |
| **Personal OS** | Morning routines, medication sequences, habit stacks — visible all day without phone notifications |

### Workflow Example 1: Live Presentation Nudging

A presenter runs Nudge on their primary monitor with a Markdown file:

```
# Keynote — Q4 Review
- [ ] Opening: revenue growth story (2 min)
- [ ] Product team: shipping velocity (3 min)
- [ ] Engineering: migration status (2 min)
* Note: mention the new hire
- [ ] Q&A buffer — watch the clock
```

While screen-sharing the slide deck on a secondary monitor, Nudge displays the current talking point at the top of the presenter's screen. The audience never sees it. Clicking the checkbox advances to the next slide's notes. The `* Note: mention the new hire` renders as a passive bullet — visible but not toggle-able, so it can't accidentally mark a task "done."

### Workflow Example 2: Engineering Standup

```
## Yesterday
- [ ] Finished auth refactor
- [ ] Reviewed PR #342
- [ ] Documented API changes
## Today
- [ ] Deploy auth to staging
* Investigate prod latency spike
- Coordinate with QA on test plan
## Blockers
- Awaiting DevOps for staging access
```

Action items (`- [ ]`) track commitments. Bullets (`*`) mark investigatory tasks that don't need completion tracking. Plain items (`- `) record blockers or notes. The heading stack shows `Yesterday > ...` or `Today > ...` context in the breadcrumb tooltip.

### Workflow Example 3: Podcast / Live Stream Outline

```
# Episode 42 — The Future of Edge Computing
## Cold Open (0:00–3:00)
- [ ] Intro music fade
- [ ] Welcome + sponsor read
- [ ] Tease: "we're joined by someone who literally wrote the book"
## Interview (3:00–22:00)
- [ ] Guest intro + background
- [ ] How edge differs from cloud
* Key stat: 75% of enterprise data will be processed at edge by 2027
- [ ] Biggest misconception
- [ ] What scares you most about the industry
## Wrap (22:00–25:00)
- [ ] Where to find the guest
- [ ] Outro + next episode tease
- [ ] Music fade
- [ ] Stop recording
```

The producer runs Nudge on a separate machine or secondary display. The guest and audience see the stream — not the outline. The `* Key stat` bullet is visible as a reminder but won't get inadvertently toggled off.

---

## File Reference (Key Source Documents)

| Document | Path in Repo | Contents |
|---|---|---|
| README | `README.md` | Full product pitch, tech architecture, use-case grid, development workflow |
| Architecture Map | `ARCHITECTURE.md` | C4-style module map, component breakdown, data flow table, format reference |
| Tutorial | `tutorial.md` | End-user quick start guide, icon reference, use-case catalog, settings reference |
| Tauri Config | `src-tauri/tauri.conf.json` | Window geometry, bundle config, security policy, build pipeline |
| Rust Backend | `src-tauri/src/lib.rs` | 3-tier parser, file I/O, state manager, 12 Tauri commands, monitor API |
| Frontend | `src/main.ts` | View manager, window geometry, auto-hide, tutorial system, settings persistence |
| Styles | `src/styles.css` | Dark/light theme via CSS custom properties, glassmorphism, roll animations |
| Python Reference | `file_handler.py` | Original Python prototype of the parser (documentation / cross-reference) |

---

## Asset Manifest

The following visual assets were copied into `website_handoff/assets/`:

| File | Type | Resolution | Source |
|---|---|---|---|
| `logo1.svg` | Vector (SVG) | — | Root — Progressive Stream logo |
| `icon.png` | Raster (PNG) | ~512×512 | `src-tauri/icons/` — primary app icon |
| `32x32.png` | Raster (PNG) | 32×32 | `src-tauri/icons/` |
| `64x64.png` | Raster (PNG) | 64×64 | `src-tauri/icons/` |
| `128x128.png` | Raster (PNG) | 128×128 | `src-tauri/icons/` |
| `128x128@2x.png` | Raster (PNG) | 256×256 | `src-tauri/icons/` |
| `Square30x30Logo.png` | Raster (PNG) | 30×30 | `src-tauri/icons/` — Windows Store |
| `Square44x44Logo.png` | Raster (PNG) | 44×44 | `src-tauri/icons/` — Windows Store |
| `Square71x71Logo.png` | Raster (PNG) | 71×71 | `src-tauri/icons/` — Windows Store |
| `Square89x89Logo.png` | Raster (PNG) | 89×89 | `src-tauri/icons/` — Windows Store |
| `Square107x107Logo.png` | Raster (PNG) | 107×107 | `src-tauri/icons/` — Windows Store |
| `Square142x142Logo.png` | Raster (PNG) | 142×142 | `src-tauri/icons/` — Windows Store |
| `Square150x150Logo.png` | Raster (PNG) | 150×150 | `src-tauri/icons/` — Windows Store |
| `Square284x284Logo.png` | Raster (PNG) | 284×284 | `src-tauri/icons/` — Windows Store |
| `Square310x310Logo.png` | Raster (PNG) | 310×310 | `src-tauri/icons/` — Windows Store |
| `StoreLogo.png` | Raster (PNG) | — | `src-tauri/icons/` — Windows Store |
| `icon.ico` | Icon (ICO) | Multi-res | `src-tauri/icons/` — Windows |
| `icon.icns` | Icon (ICNS) | Multi-res | `src-tauri/icons/` — macOS |
| `vite.svg` | Vector (SVG) | — | `src/assets/` — build tool logo |
| `tauri.svg` | Vector (SVG) | — | `src/assets/` — framework logo |
| `typescript.svg` | Vector (SVG) | — | `src/assets/` — language logo |

### Notes for Website Build

- **Primary hero asset:** `logo1.svg` — the Progressive Stream logo is the brand mark
- **App icon (favicon/social):** `icon.png` or `Square310x310Logo.png` (highest res square)
- **Tech stack badges:** `tauri.svg`, `typescript.svg`, `vite.svg` are available for a "Built with" section
- No screenshots or mockups exist yet — these will need to be created for the website
- The banner UI has dark and light themes — consider showing both in a split screenshot
