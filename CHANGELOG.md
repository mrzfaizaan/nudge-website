# Changelog

All notable changes to Nudge will be documented in this file.

---

## [0.2.1] — 2026-06-05

### Fixed
- **Trial hardening** — five critical bypass vectors closed: localStorage short-circuit removed (frontend always validates via Rust backend first); plain-text "LICENSED" sentinel replaced with HMAC-signed token binding status + fingerprint + timestamp; multi-location storage (shadow copy of trial state in `state.json` for tamper-resistant cross-check, restoring from shadow if `.sys_time` is deleted or tampered); system clock rollback detection (max observed timestamp tracked, backward time jumps trigger expiry); periodic license re-validation (licensed installs re-verify with Lemon Squeezy every 3 days, 7-day offline grace period, server-side revocation clears license)
- **Machine fingerprinting** — trial identity now binds to `hostname + COMPUTERNAME + USERNAME` hash, preventing trial reset via hostname change or registry drift
- **Startup performance** — removed blocking `reg.exe` process spawn; fingerprint computed instantly from environment variables with `OnceLock` caching, eliminating 5-second lag on every launch

---

## [0.2.0] — 2026-06-04

### Added
- **Right-click context menu** — glassmorphism overlay on banner right-click (Refresh, Open File, Auto-Scroll toggle, Quit) and expanded task right-click (Toggle Complete, Edit Task inline, Add Task Below, Delete Task, Multi-Select mode); `cmd_delete_task`, `cmd_edit_task_line`, `cmd_insert_task_after` Rust commands; multi-select with checkboxes, bulk delete/toggle, Select All/Deselect All; drag-and-drop reordering with smooth CSS transitions; delete animation (fade + shrink + slide-right, 250ms)
- **Inline task editing** — "Edit Task" and "Add Task Below" in context menu open inline input fields; preserves or replaces format prefix based on `[ Preserve Format | Full Line ]` setting
- **Drag task in expanded view** — users can reorder tasks via mouse drag (mousedown/mousemove/mouseup with 8px dead zone to avoid accidental drags); `cmd_move_task` Rust command moves lines in the source file; visual feedback with `.dragging` (faded) and `.drag-over` (emerald accent border) CSS
- **Settings UI overhaul** — segmented button tracks replace dropdowns and sliders for position, monitor, font size, auto-hide delay, and theme; width and opacity use stepper controls (±buttons + scroll wheel); keyboard shortcut recording UI with "Record" button and Esc-to-cancel flow
- **Global keyboard shortcut** — `Ctrl+Alt+Shift+N` (Next Item) toggles and advances the current task; works globally even when unfocused; auto-restores banner from ghost mode and re-enables auto-hide after action; configurable via settings with custom binding recording; `tauri-plugin-global-shortcut` integration
- **Theme system** — `[ System | Dark | Light ]` segmented track with live OS theme detection via `matchMedia`
- **Font size presets** — `[ Sm | Md | Lg ]` 3-segment toggle with `--font-scale` CSS variable
- **Simplified task formatting** — canonical format `[ ]` for tasks (replaces `- [ ]`); `[x]` for completed tasks; `*` for bullets; `1.` for numbered lists; `#`/`##` for headings; bare text for plain notes. LLM prompt, tutorial, and default template updated. Legacy `- [ ]` and `h1:` formats still parse for backward compatibility.
- **Auto-scroll** — play/pause button (▶/⏸) in icon row; configurable cadence `[ s | min ]` unit toggle + stepper 0.5–5.0 (±0.5) in settings; persisted as `autoscroll_value` + `autoscroll_unit`; pauses when settings/expanded/info views are open; toggle also available in right-click context menu
- **Expand button relocated** — ⛶ moved from icon row to content row, placed beside the stacked ↑↓ arrows as a task-control cluster; enlarged to 28×22px for better tap targets
- **Quick file switcher** — right-click banner to see recent files; click any to open instantly; up to 10 files tracked, deduped, most recent first; persisted as `recent_files` in state
- **Hot reloading** — `notify` crate watches open file for changes; 300ms debounced `file-changed` event triggers automatic re-parse and re-render; no manual refresh needed when editing files externally
- **Inline link parsing** — `[label](url)` in task text renders as clickable emerald links; opens URLs/files via `open_file_natively`; XSS-safe via `escapeHtml()` on plain text; LLM prompt updated with link formatting rule
- **7-day trial enclave** — offline-first trial tracker with HMAC-signed timestamp in `.sys_time` file; app opens normally for 7 days with trial badge in settings footer ("Trial · N days left · Buy a License · Enter Key"); trial badge links to `nudgehud.work`; inline license key entry during trial; license key activation via existing Lemon Squeezy `verify_license` flow; trial-expired lockout overlay reuses license wall styling; existing v0.1.4 licensed users skip trial via `localStorage.nudge_is_licensed`
- **User guide** — comprehensive `USER_GUIDE.md` replaces `tutorial.md`; bundled as resource; accessible via ✎ (when no file loaded) and "User Guide" button in Info panel; enriched with 10 detailed use-case workflows (flashcard/active recall, standup, demo/presentation, exam prep, morning routine, meeting agenda, podcast/streaming, writing/research, project milestones, shopping/errands) each with step-by-step setup instructions, file structure templates, and power-user tips

### Changed
- **Info panel restyled** — glass-styled prompt container replaces terminal dark block; labeled sections (Format, What this does, Prompt) match settings/expanded pattern; Copy Prompt button styled as segment button with emerald hover accent; single-format dropdown hidden
- **LLM prompt improved** — added plain-text output format spec, use case context, line conciseness rule, filler removal rule, code fence prohibition, and input→output example for better LLM compliance
- **Accent color** — global accent changed from purple (`#6c5ce7`) to emerald (`#10B981`) across both themes (segment buttons, checkboxes, progress bar, links)
- **Settings layout** — reorganized top-to-bottom with sectioned labels; scrollable with emerald scrollbar; height matches expanded view (500px)
- **Segmented button selection** — active state now uses emerald underline (`box-shadow: inset 0 -2px 0`) instead of solid emerald block, matching the drag-and-drop drop target styling and glassmorphism aesthetic
- **Icon styling** — Settings (⚙) and Info (ⓘ) top-row buttons now render as monochrome symbols via `Segoe UI Symbol` font + `text-shadow` silhouette, matching the other icon-row buttons
- **Tutorial carousel** — enriched to 17-step walkthrough covering all v0.2 features; progress bar + step counter ("4 / 17") appear after scrolling past welcome step; monochrome icons via `font-variant-emoji: text` + `Segoe UI Symbol` fallback; live checkbox demo at step 4 with auto-advance
- **App version** — settings footer now reads version from `env!("CARGO_PKG_VERSION")` via `cmd_get_version` command; during trial shows trial badge, after license shows "v0.2.0 | Licensed"
- **Monitor restore** — `populateMonitorTrack()` called on startup to restore correct monitor before positioning
- **Progress bar** — light theme progress bar now uses emerald accent (`#10B981`)
- **Heading display** — headings with no tasks now appear as section headers in the expanded view; `>` prefix visually indicates sub-heading depth; section ordering fixed (parent before child); `--heading-color` CSS var improves heading legibility
- **Auto-scroll pins banner** — pressing ▶ auto-pins the banner (if auto-hide is enabled) so tasks scroll visibly; pressing ⏸ restores auto-hide. Manual pin toggle during play clears the auto-pin flag.
- **Settings scrollability** — `overflow: hidden` restored on view containers; `overflow-y: auto` scoped to `:not(.view-closed)` so scrollbars only appear when views are open

### Fixed
- **Auto-hide/reveal freeze** — untracked inner 300ms resize timer in `startHoverTimer()` made cancelable via `hoverResizeTimer`; `onMouseEnterWindow()` now removes `.banner-faded` before resizing (matching `toggleSettings`/`toggleExpanded`/`toggleInfo` pattern), eliminating WebView2 stale backdrop-filter snapshot
- **Heading hierarchy** — `#`/`##` section ordering now correct (parent before child) via file-ordered heading registry; sub-headings visually indented and dimmer; banner shows root heading as title with drill-down breadcrumb; `↳` icon restyled as emerald pill badge; empty section drop zones use real file line numbers
- **Drag into empty sections** — `.section-drop-zone` renders on empty sections; `heading_line + 1` ensures task lands under correct heading; `onAddTask()` positions cursor to new heading section after heading creation
- **Opacity setting** — backgrounds now use `--bg-alpha` CSS variable instead of `document.body.style.opacity`; text stays fully opaque at all levels; `applyOpacity()` sets alpha on `:root` via `setProperty`
- **Add-task input persistence** — expanded view restructured as flex column: add-task row sits as fixed header, only the task list scrolls (no overlay feel, no sticky hacks); scrollbar moved to `#expanded-task-list`
- **Expanded-view scroll** — clicking a task in expanded view no longer snaps to top; scroll position saved before DOM rebuild and restored after
- **Auto-updater** — updater plugin was registered in Rust but never triggered from JavaScript; added `check()` call from `@tauri-apps/plugin-updater` in `initApp()` after window shown

### Removed
- **"Always hide (show on hover)" checkbox** — redundant with pin toggle in icon row
- **Toggle Visibility shortcut** — redundant with auto-hide and pin behavior

---

## [0.1.4] — 2026-05-17

### Added
- **Auto-update support** — `tauri-plugin-updater` integrated with `Builder::new().build()`; pubkey in `tauri.conf.json` under `plugins.updater`; endpoint at `https://nudgehud.work/updater.json`; minisign signing via `tauri signer sign`
- **License activation system** — Lemon Squeezy License API integration; `verify_license` Tauri command calls `POST /v1/licenses/activate` with license key and machine hostname as `instance_name`; 10-second connection timeout via `ureq`
- **License paywall UI** — full-screen blocking modal on first launch (400×380px centered on monitor); emerald accent (`#10B981`) for button and link; transparent overlay wrapper, white card with rounded corners and drop shadow; checks `localStorage.nudge_is_licensed` before booting main app
- **`hostname` crate** — machine identifier bound to license instances for activation tracking

### Changed
- **Accent color** — license flow button and link changed from purple (`#6c5ce7`) to emerald (`#10B981`)
- **Tauri plugin registration** — updater plugin added to `tauri::Builder` chain in `src-tauri/src/lib.rs`
- **Cargo dependencies** — added `ureq` (with `json` feature), `hostname`, `tauri-plugin-updater`; removed `reqwest` to keep dep tree light
- **npm dependencies** — added `@tauri-apps/plugin-updater`

### Fixed
- **License API payload format** — switched from `send_json` (JSON) to `send_form` (URL-encoded) to match Lemon Squeezy License API requirements
- **"Box-in-box" visual artifact** — license wall parent div stripped to `background: transparent`, leaving only `.license-card` as the visible container

---

## [0.1.3] — 2026-04-30

### Added
- **"Create New" button** (📄 in icon row) — opens native save dialog, writes a markdown template with formatting instructions to the chosen path
- **"Add Task" input field** at the bottom of expanded view — type and press Enter or click `+` to append a task under the current heading section
- **`classifyInput()` auto-classification** — detects `- [ ]` (task), `* ` (bullet), `- ` (plain), `#`/`##` (heading), numbered lists; defaults to `- [ ]` action task
- **Numbered list support** — `1. Item` syntax parsed as `display_mode: "numbered"`, rendered without checkbox or bullet
- **Pin toggle (📌)** — repurposed Hide button toggles between auto-hide and pinned mode with visible icon swap (`●` ↔ `📌`)
- **`cmd_create_file` Rust command** — writes the default markdown template to disk and returns the parsed `TodoDocument`
- **`cmd_append_task` Rust command** — inserts a formatted task line after the last task under the currently-viewed heading section, re-parses, and returns the updated document
- **`find_insert_line()` Rust helper** — scans parsed entries to locate the correct insert point per heading context

### Fixed
- **Trackpad scroll sensitivity** — debounce (300ms quiet window) plus synchronous navigation lock eliminates multi-task skipping from momentum scroll events
- **Navigation stuck at heading boundaries** — lock now released by `onNavigate()` completion instead of a fixed 450ms timeout, eliminating the race window between animation end and lock expiry
- **Bullet point UI** — bullets now render as clean dots (`•`) without checkbox borders, both in the banner and expanded view
- **macOS white shadow/border during auto-hide shrink** — removed `border` and `box-shadow` on `.banner-faded` class
- **macOS Next/Previous navigation buttons hidden** — `flex-shrink: 0` on `#main-task-row`, `min-width: 44px` and `z-index: 1` on `.nav-arrows`, `position: relative` on `.content-row`
- **macOS autohide not reopening on mouse hover** — `HOVER_IDLE_H` increased from `2px` to `6px` for reliable mouse event detection

### Changed
- **LLM prompt** (Info panel) — rule 1 now instructs the model to scan input and auto-generate `#` and `##` headings (previously left heading usage ambiguous)
- **Hide button repurposed** — `●` button now toggles Pin mode (`📌`). When pinned, auto-hide is suppressed and the banner stays visible. Visual icon swap communicates state clearly.
- **Scroll navigation lock** — renamed `isNavigating` to `navigateLocked`; lock set synchronously at wheel handler entry instead of inside async animation functions
- **Banner view** — `#main-task-row` now uses `flex-shrink: 0` and `.content-row` uses `position: relative` for layering stability

---

## [0.1.2] — 2026-04-29

### Changed
- **Product branding** — name changed to "Nudge"; integrated brand identity across icons and window title

### Added
- **GitHub Actions CI/CD workflow** — automated release builds for macOS (aarch64-apple-darwin) and Windows (x86_64-pc-windows-msvc) triggered by version tags

---

## [0.1.1] — 2026-04-28

### Changed
- **Rust parser upgraded** — tasks now classified into 3-tier display modes: `action` (checkbox), `bullet` (dot), `plain` (no indicator). Previously all tasks rendered as checkboxes.

---

## [0.1.0] — 2026-04-27

### Added
- **Initial release** — Nudge ambient HUD
- **Tauri v2 shell** — Rust backend with vanilla TypeScript frontend, no framework runtime
- **4-pass task parser** — supports 4 mixed formats: A (classic brackets `h1:`/`[ ]`), B (asterisk `*`), C (underlined `----`), D (markdown `#`/`- [ ]`)
- **4 views** — banner (always-on-top HUD), settings (theme/opacity/position/width/monitor), expanded (full task list), info (LLM prompt library)
- **14-step tutorial system** — barrel-roll animated walkthrough shown when no file is loaded
- **Auto-hide** — banner shrinks to 2px after configurable delay; restores on mouse hover
- **Multi-monitor support** — select target monitor, position presets (top-left/center/right) calculated in absolute physical coordinates
- **Light/dark themes** — 22 CSS custom properties per theme with glassmorphism via `backdrop-filter`
- **Progress bar and task counter** — optional overlay showing completed/total
- **LLM prompt library** — expandable format templates with copy-to-clipboard
- **File I/O** — toggle tasks in-place (`[ ]` ↔ `[x]`, `- [ ]` ↔ `- [x]`), open in native editor
- **State persistence** — `state.json` in app data directory, saved on every setting change
