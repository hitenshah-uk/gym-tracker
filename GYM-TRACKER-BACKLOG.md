# Gym Tracker — Version History & Backlog

## Current Version: 2.9.0

---

## Version History

### v2.9.0 — Cardio equipment presets + History session delete (deployed)
- `CARDIO_EQUIPMENT` constant: ['Elliptical','Treadmill','Bike']
- `_cardioOtherMode` flag — tracks when Other is selected vs a preset; reset in `selectDay()`
- `buildEquipmentHTML()` — renders seg-btn row (Elliptical / Treadmill / Bike / Other); Other mode reveals a free-text input; correctly detects previously saved non-preset values on load
- `setCardioMachine(val)` — updates `logState.cardioActual.machine` and re-renders the `#cardio-equipment-section` div in-place (no full log re-render, preserves drum positions)
- Cardio Actual section restructured: equipment presets full-width on top, Duration + Avg HR side by side below
- `_hDeleteConfirmId` state variable added to history state
- `requestDeleteSession(id)` / `confirmDeleteSession(id)` / `cancelDeleteSession()` — two-tap delete flow; confirm removes session from localStorage and syncs to GitHub
- `toggleH()` updated: clears `_hDeleteConfirmId` when session is toggled closed
- History expanded session: "🗑 Delete session" link at bottom right; on first tap shows inline confirmation with "Yes, delete" and "Cancel" buttons; `event.stopPropagation()` on all buttons prevents card toggle
- Edit session (in-place) parked as a future backlog item

### v2.8.2 — UX patch bundle (deployed)
- Viewport meta: added `maximum-scale=1.0` to prevent accidental pinch-to-zoom on iPhone
- Save Session button: removed `position:sticky` from `.save-bar` — button now sits naturally at the bottom of log content, requiring deliberate scroll to reach
- Drum scroll: added `overscroll-behavior: contain` to `.drum-scroll` — prevents scroll chaining to the page when a drum hits its end limit
- Weight log history: `buildWeightLogList` now shows 3 most recent entries by default with a "Show all X entries / Show less" toggle; `_wlExpanded` state variable and `toggleWLExpand()` function added

### v2.8.1 — Chart improvements + import mode bug fix (deployed)
- `_chartRange` variable ('1m' | '3m' | '6m' | 'all') + `setChartRange(r)` + `_filterChartData(data)` — filter applied before drawing
- Range selector (1M / 3M / 6M / All) rendered as seg-btn row above charts; calls `setChartRange()` then `renderProgress()`
- `_rollingAvg(data, window)` — computes N-day rolling average over a data array
- Dense mode (n > 30): raw line drawn faint (opacity 0.2, lineWidth 1); 7-day MA drawn bold (lineWidth 2.5) on top; dots suppressed
- Sparse mode (n ≤ 30): single bold line + dots as before
- Bug fix: import mode selector buttons used non-existent class `seg-btn-opt` — corrected to `seg-btn` inside `seg-row` container; `setImportMode()` updated accordingly

### v2.8.0 — VeSync CSV import (deployed)
- `_pendingImport[]` and `_importMode` ('weight' | 'both') state variables
- `parseVeSyncCSV(text)`: strips BOM, parses quoted `"DD/MM/YYYY, HH:MM"` first field, deduplicates to one entry per day (prefer entry with body fat data; on tie, keep earliest time), returns array sorted oldest-first
- `handleCSVImport(input)`: reads file via FileReader, calls parseVeSyncCSV, filters out dates already in weightLog, renders preview (total/new/skipped counts + import mode selector)
- `setImportMode(mode)`: toggles seg-btn active state between 'weight' and 'both'
- `confirmCSVImport()`: adds only new-date entries to weightLog (bodyFat set to '' if mode='weight'), sorts newest-first, calls debouncedSettingsSync(), calls renderProgress(), shows toast
- Progress screen: "Import from VeSync" card between Log Entry and Charts — explanatory text, file input labelled as button (accepts .csv), preview div rendered inline after file selection
- Body fat % caveat note shown in preview when BF data present ("bioelectrical impedance may be inaccurate")
- Import mode selector disabled if no new entries have BF data
- Existing entries never touched — import is strictly additive by date

### v2.7.0 — Help screen + onboarding (deployed)
- `#onboarding-overlay` div (z-index 400, centred, dark backdrop) with 5 slides covering each tab
- `ONBOARDING_SLIDES` array, `onboardingSlide` state, `showOnboarding()` / `dismissOnboarding()` / `onboardingNav(dir)` / `renderOnboarding()` functions
- First launch: shows automatically 400ms after load if `gymOnboarded` flag is absent; sets flag on dismiss
- `screen-help` div (no nav tab); `renderHelp()` renders comprehensive guide: Plan, Log, Timer, History, Progress, Settings, Tips sections
- `showScreen('help')` added; back button → settings
- Settings: "❓ How to use this app" card + "Show intro again" link

### v2.6.0 — Settings GitHub sync (deployed)
- `ghGetSettings()` / `ghPutSettings()` — fetch and write `settings.json` to gym-data repo
- `syncSettingsToGitHub()` — pushes local settings; `fetchSettingsFromGitHub()` — pulls remote and merges (github token always stays local)
- `debouncedSettingsSync()` — 2s debounce called from `writeSettings()`; `_pauseSettingsSync` flag prevents re-sync loop when writing from remote fetch
- Init: `fetchSettingsFromGitHub()` called at 1500ms after load (after sessions sync at 1000ms)
- Settings synced automatically across devices on every save; age, height, weightLog, measurements etc. all sync; token stays local

### v2.5.0 — Cardio logging (deployed)
- `logState.cardioActual: {duration, machine, avgHR}` — initialised empty, reset on `selectDay()`
- Log screen: "Actual" row appears inside session header when day has cardio — three inline inputs (duration/min, equipment text, avg HR/BPM); values saved to `logState.cardioActual` via `updateCardioActual(field, val)` without full re-render
- `doSaveSession()`: `cardioActual` saved with session and reset on cleanup
- History expanded session: cardio actual row shown below exercises if any field is present

### v2.4.0 — Progress screen (deployed)
- New `screen-progress` div — 6th screen, accessible via Settings → "📈 Body Composition & Measurements" card; no new nav tab
- `showScreen('progress')` toggles screen-progress active, hides all nav tabs, calls `renderProgress()`; back button returns to settings
- `gymSettings.weightLog`: array of `{date, weight, bodyFat}` entries, newest first
- `migrateWeightLog()` on init: seeds one entry from legacy static weightKg/bodyFatPct fields if weightLog is empty
- `getLatestBodyStats()`: scans weightLog for most recent weight and body fat values; falls back to legacy static fields; used by all calcs (BMI, BMR, TDEE)
- Settings Personal Stats: weight (kg) and body fat % inputs removed (now logged via Progress screen)
- Settings Body Measurements section removed — moved to Progress screen
- `renderProgress()`: header with back button, Log Entry form (date + weight + body fat), weight trend chart, body fat trend chart, log history with delete, body measurements section
- `drawBodyChart(canvasId, data, color, targetVal, targetLabel, unit)`: vanilla canvas chart with grid, area fill, line, dots, dashed target line (67kg / 25%), X-axis date labels; dark mode aware
- `addWeightEntry()` / `deleteWeightEntry()`: CRUD for weightLog; both call `renderProgress()` to redraw
- Planned version roadmap: 2.7.0 (all planned features done), 3.0.0 (if a new primary screen is added)

### v2.3.0 — RPE per set (deployed)
- rpe field (integer 1–10, nullable) added to all set objects throughout (selectDay, addSet, pickExercise, addCustomEx)
- updateRPE(ei, si, val) — validates 1–10 range, stores null if outside or empty
- Log screen: compact RPE input sits to the right of the note field on each set block — RPE label, number input (1–10), /10 suffix
- History: RPE shown as a small badge in the Note column of each set row — only renders if value present

### v2.2.0 — Block C: Injury flag, deload flag, progressive overload sparkline (deployed)
- `injuryModified` and `isDeload` booleans added to logState and saved with each session
- Session summary overlay (pre-save): two toggles — "⚠ Modified session" and "🔄 Deload week" — with explanatory sub-labels
- History session cards: amber "⚠ Modified" badge and grey "Deload" badge shown on flagged sessions
- `getExProgression(name, limit)`: scans all sessions, skips flagged (injuryModified/isDeload), returns last N max-weight data points per exercise
- `renderSparkline(vals, color)`: inline SVG sparkline (80×28px) with polyline + filled endpoint dot
- History expanded exercise blocks: sparkline row shows SVG + "Xkg last · peak Ykg ↗" + session count footnote; only renders when 2+ clean data points exist

### v2.1.0 — Block B: Exercise notes, key lifts, in-picker edit mode (deployed)
- Exercise library entries now store `note` (form cue text) and `keyLift` (boolean) fields
- `seedKeyLifts()` on first load: marks 5 default key lifts (Chest press machine, Lat pulldown, Seated cable row, Leg press, Cable tricep pushdown) — guarded by `gymExLibKL` flag
- Exercise picker list: ⭐ badge on key lifts, ✎ edit button on every row opens in-picker edit screen
- Edit screen: muscle group grid (visually selected), key lift toggle, form cue textarea — Cancel or Save
- Log screen: form cue note shown as italic hint under exercise name (pulled live from library)
- History screen: "⭐ Key lifts" filter chip — when active, expanded session shows only key lift exercises (or "No key lift exercises" message); ⭐ shown in exercise headers when key lifts filter is on

### v2.0.0 — Block A: Rest timer, session summary, session duration tracking (deployed)
- Rest timer: inline bar below rest-timer-bar div, 90s countdown with progress bar, Skip button, beep on completion; does NOT re-render log (preserves drum scroll positions)
- Session summary: bottom sheet before saving — exercises count, sets done/total, total volume, duration; Confirm & Save / Go Back buttons
- Session duration: startedAt set on first set tick OR on cardio timer start; savedAt = Date.now() at save; duration shown in History session cards as "⏱ Xmin"

### v1.9.0 — Timer fixes, complete screen, physio backlog (deployed)
- Timer complete screen: trophy bounce animation (tcTrophyIn), fade-up stats card (tcFadeUp), pulsing Done button (tcPulse)
- Critical fix: `if (!timerState.complete)` guard in `timerTick` prevents renderTimerRunning overwriting complete screen
- Font size for 4-digit timer strings (e.g. 13:30): reduced to 60px (was 80px), small GET READY phases use 64px
- Added 10 physio-recommended features to backlog (Items 1–10 in HIGH PRIORITY section)

### v1.8.0 — Exercise Library with searchable picker (implemented, not yet tested)
- gymExLib localStorage key: array of { name, muscleGroup } objects, sorted A–Z
- seedExLib() called on first app load: pre-populates library from all exercises in the current plan (respects existing gymExMeta muscle group assignments)
- gymExLibSeeded flag prevents re-seeding on subsequent loads
- Bottom-sheet picker overlay: #ex-picker-overlay, slides up from bottom, semi-transparent backdrop
- Picker has: search input (filters library by name), muscle group filter chips (horizontal scroll — All + 9 groups), exercise list with muscle group badges
- Typing a name not in the library shows a "+ Add X" button → prompts for muscle group → adds to library + gymExMeta + target
- "Add Exercise" button in Plan edit mode (replaces text input + button)
- "Add Exercise" button in Log screen (replaces text input + button)
- New exercises added via picker are stored in library immediately
- _epList global holds current filtered list so onclick by index avoids quote-escaping issues

### v1.7.0 — Timer fixes (deployed, confirmed working at gym)
- HR zone labels: fixed z.name → z.label (was undefined, now shows correct zone descriptions)
- Cardio tick on Log screen: tick-btn in cardio-box lets you mark cardio complete; changes colour to green when done; cardioDone saved with session
- Build-up rounds: optional build rounds before main work rounds; buildRounds + buildWork fields in timer config; build_ready → build → rest phases before get_ready → work; buildRound counter shown during build phases
- Start beep: 3 ascending tones (440/550/660 Hz) play on pressing Start — unlocks iOS audio context on the user gesture that starts the timer
- SVG progress ring: 260×260 viewBox, r=116, drains clockwise around the countdown number; colour matches current phase; smaller font (64px) for short GET READY phases

### v1.6.0 — Stage 4: Reorder, Copy Day, Muscle Badges, Export (deployed, not yet fully tested)
- Reorder exercises: ↑ ↓ arrows on each exercise row in Plan edit mode
- Reorder exercises: ↑ ↓ arrows on each exercise card in Log screen
- Copy day: "Copy from another day" dropdown in Plan edit mode per day — copies exercises, cardioType and cardioConfig
- Muscle group badges: global exercise meta store (gymExMeta in localStorage) keyed by exercise name
- Muscle group dropdown in Plan edit mode per exercise (Chest / Back / Shoulders / Arms / Quads / Hams / Glutes / Core / Full Body)
- Badges shown in Plan (tag view), Log (exercise card header), History (expanded exercise list)
- History export: Export CSV and Export JSON buttons at top of History screen
- CSV format: Date, Time, Day, Label, Exercise, Set, Weight (kg), Reps, Note
- JSON format: full raw sessions array — can be dropped into Claude for analysis

### v1.5.0 — Stage 3: Cardio interval timer (deployed, timer confirmed working)
- 5th nav tab: Timer (stopwatch icon, between Log and History)
- Pre-start screen: day selector chips (auto-selects today), editable config fields, HR zones reference card
- Intervals flow: Warm up → GET READY (3s) → Work/Rest rounds → Cool down → Complete
- Steady state flow: Warm up → Steady → Cool down → Complete
- Phase colours match day theme palette (green=Work, amber=Recover, grey=Warm Up/Cool Down, blue=Steady)
- Large countdown display (104px), phase label, round counter, BPM zone target
- Pause/Resume and Reset controls with large tap targets
- Mute toggle — uses Web Audio API (podcast-friendly, layers over audio without interrupting)
- Screen wake lock via navigator.wakeLock (releases on pause/reset/complete)
- Editable config does not overwrite saved plan — applies to that session only
- showScreen() updated to include 'timer'

### v1.4.0 — Stage 2: GitHub sync (deployed, confirmed syncing both directions)
- Full GitHub API sync using private repo `gym-data` (hitenshah-uk/gym-data)
- PAT stored in localStorage, entered once per device in Settings
- On session save: merges local + remote sessions, writes back to GitHub automatically
- On app load: fetches sessions.json from GitHub, merges with localStorage (1 second delay)
- Sync status pill in Settings: ● Synced / ● Sync error / ● Syncing… / ● Not configured
- Last synced timestamp displayed
- "Sync now" button for manual trigger (shown when token is present)
- Offline fallback: if no connection or token missing, uses localStorage silently
- Deduplication by session id (Date.now() timestamp)
- b64enc/b64dec using TextEncoder/TextDecoder (UTF-8 safe)
- updateSetting() triggers sync when githubToken is saved

### v1.3.0 — Stage 1: Settings page + structured cardio fields (deployed, confirmed working)
- 4th nav tab: Settings (gear icon)
- Settings sections: Personal Stats, Your Metrics, 1RM Calculator, Body Measurements, Training Preferences, GitHub Sync (placeholder)
- Your Metrics: BMI with classification, HR zones (5 zones with actual BPM ranges), BMR, TDEE with activity level selector
- 1RM Calculator: Epley formula (weight * (1 + reps/30))
- Body Measurements log: waist, chest, arms, thighs with date stamps
- Training start date with weeks trained counter
- Drum increment setting: 1kg / 2.5kg / 5kg segmented control
- WEIGHT_VALS now built dynamically from drumIncrement setting
- DEFAULT_PLAN restructured: cardio/cardioDetail strings replaced with cardioType + cardioConfig objects
- cardioType: 'intervals' | 'steady' | 'none'
- cardioConfig for intervals: { warmUp, work, rest, rounds, coolDown }
- cardioConfig for steady state: { warmUp, duration, coolDown }
- Migration logic in getPlan() converts old string format to new structure automatically
- getCardioLabel() generates display text programmatically from config (label always in sync)
- Plan edit mode: structured cardio fields with type selector and per-type inputs
- Log screen: HR BPM zone ranges shown when age is set in Settings

### v1.2.0 — Log screen colour & drum pickers
- Replaced number inputs with scroll drum pickers for weight (kg) and reps
- Drums pre-fill from last session values so minimal scrolling needed
- Exercise cards on Log screen now have coloured header bands per day type
- Set blocks have coloured left borders, labels, and drum headings
- Hint box uses day theme colour
- Last session values shown in coloured tinted box

### v1.1.0 — Colour coding & cardio detail
- Colour coding per day type: blue (Upper Push), green (Upper Pull), orange (Lower Body), purple (Full Body + Core), grey (Rest)
- Full cardio protocol detail added: intervals timing and steady state instructions
- Cardio detail shown on both Plan and Log screens
- Rest days and Training days separated into sections on Plan screen
- Bug fix: syntax error (`')'` in two places) that prevented JavaScript from running

### v1.0.0 — Initial release
- Three-screen app: Plan, Log, History
- Weekly plan pre-populated with full schedule
- Editable plan: add/remove exercises, toggle rest days, edit cardio notes
- Session logging with sets (weight, reps, note) and tick-to-complete
- Full session history in reverse chronological order
- Search and filter by day type in History
- Previous session values shown as hints during logging
- Dark mode support
- localStorage persistence
- Hosted on GitHub Pages: https://hitenshah-uk.github.io/gym-tracker/gym-tracker.html

---

## Backlog — Planned Features

### AWAITING REAL-WORLD TESTING (v1.4.0–v1.6.0)

The following were implemented but not yet tested in a real gym session. Issues to log here as they come up:

- GitHub sync — two-way confirmed on desktop/iPhone but not stress-tested over multiple sessions
- Timer — confirmed working once, not yet used in a full workout
- Exercise reorder — implemented, not tested at gym
- Copy day — implemented, not tested
- Muscle group badges — implemented, not tested
- History export — CSV and JSON confirmed downloading, not tested with multi-session data

---

## Backlog — Future Features

### HIGH PRIORITY (Physio-recommended, in order)

#### [FEATURE] #1 — Rest timer between sets
- After ticking a set done, auto-start a countdown inline below the completed set (not full-screen)
- Default 90 seconds, configurable per session
- User can dismiss early or let it run
- Soft beep when countdown reaches zero
- Implementation notes: countdown runs in logState, re-renders only the affected set block; use setInterval with cleanup on dismiss/done; beep via Web Audio API (same pattern as timer screen)

#### [DONE v2.4.0] #2 — Body composition trend chart
- Dated weight + body fat log in Progress screen; canvas chart per metric with dashed target lines (67kg, 25%); migrateWeightLog() seeds from legacy static fields on first load

#### [DONE v2.2.0] #3 — Progressive overload sparkline per exercise
- Inline SVG sparkline in History expanded exercise blocks; shows last 8 clean sessions (excl. modified/deload); displays last weight, peak, and trend arrow

#### [FEATURE] #4 — Session summary before saving
- When user taps "Save Session", show a summary card first: total volume (sets x reps x weight across all exercises), number of exercises completed, estimated duration (first set ticked to finish tap)
- Two buttons: Confirm & Save / Go Back
- Implementation notes: add logState.startedAt timestamp (set when first set is ticked done); compute volume and duration at save time; render summary in a modal-style overlay matching the exercise picker pattern

#### [FEATURE] #5 — Session duration tracking
- Record start timestamp when first set is ticked done; record end timestamp on save
- Store both in the session object (startedAt, savedAt)
- Display duration in History session cards
- Implementation notes: pairs with #4; startedAt set once (guard against overwrite on subsequent ticks); display as "Xh Ym" in History

#### [DONE v2.3.0] #6 — RPE field per set
- Compact input alongside note on each set; stored as integer 1–10 or null; shown as badge in History

#### [DONE v2.1.0] #7 — Exercise notes / form cues
- note field in gymExLib; editable via ✎ button in exercise picker; shown as italic hint under exercise name on Log screen

#### [DONE v2.2.0] #8 — Injury/limitation session flag
- Toggle in session summary overlay; amber ⚠ Modified badge in History; excluded from sparkline

#### [DONE v2.2.0] #9 — Deload week flag
- Toggle in session summary overlay; grey Deload badge in History; excluded from sparkline

#### [DONE v2.1.0] #10 — Key lift flag
- keyLift boolean in gymExLib; seeded for 5 defaults on first load; ⭐ shown in picker + History; "⭐ Key lifts" filter in History

---

### MEDIUM PRIORITY

#### [DONE v2.5.0] Cardio logging
- Duration, equipment, and avg HR fields in Log screen; stored with session; shown in History expanded view

### LOWER PRIORITY

#### [FEATURE] Edit saved session (in-place)
- Allow editing weights, reps, and notes on a previously saved session directly in History
- Would require pre-filling a log-style edit screen from saved session data and writing back to localStorage
- Parked after v2.9.0 delete implementation; delete + re-log on correct date is not viable (date would stamp as today)

#### [DONE v2.6.0] Settings sync to GitHub
- settings.json synced to gym-data repo; debounced 2s after each save; token always stays local

---

## Known Issues / Minor Fixes

- Version number in UI not always updating after GitHub Pages cache refresh — cosmetic only, clears on next upload

---

## Notes for Developer (Claude)

- Single file app: all HTML, CSS, JS in gym-tracker.html
- No external libraries or CDNs — pure vanilla JS only
- localStorage for all persistence (with in-memory fallback if localStorage blocked)
- GitHub Pages hosting: public repo `gym-tracker` at https://hitenshah-uk.github.io/gym-tracker/gym-tracker.html
- GitHub data sync: private repo `gym-data`, accessed via PAT stored in localStorage
- GitHub API base URL: https://api.github.com/repos/hitenshah-uk/gym-data/contents/
- Version number stored in JS variable APP_VERSION and displayed in UI
- When making changes: update APP_VERSION, add entry to Version History above
- Backlog is the single source of truth for decisions — update it when direction is confirmed

### Versioning Strategy (MAJOR.MINOR.PATCH)
- **PATCH** (x.y.z+1): Bug fixes, CSS tweaks, wording changes — nothing new added
- **MINOR** (x.y+1.0): One new user-visible feature, or a small group of tightly related features
- **MAJOR** (x+1.0.0): New primary screen added, or complete overhaul of a core section
- Rule of thumb: if you can point to it and say "that's new" → minor; if it fixes something broken → patch; if it fundamentally changes the shape of the app → major
- Planned version roadmap: 2.7.0 (all planned features done), 3.0.0 (if a new primary screen is added)
- Linux mount (bash) lags behind Windows file edits — always verify file state via Read tool, not bash tail/wc
