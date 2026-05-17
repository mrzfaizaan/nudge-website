# Changelog

All notable changes to Nudge will be documented in this file.

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

## [Unreleased] — v0.1.3

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
