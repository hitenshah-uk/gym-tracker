# Gym Tracker — Version History & Backlog

## Current Version: 1.3.0

---

## Version History

### v1.3.0 — Settings page + structured cardio fields (Stage 1)
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
- GitHub Sync section in Settings shows "coming in v1.4.0" placeholder

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

### HIGH PRIORITY

#### [FEATURE] GitHub sync — cross-device data access
**Decision: Separate private repository (chosen approach)**

Architecture:
- `gym-tracker` repo (existing, public) — hosts the app via GitHub Pages
- `gym-data` repo (new, private) — stores all session data as JSON files

Why a private repo over a secret Gist:
- Secret Gists are unlisted but not truly private — anyone with the URL can read them
- A private GitHub repo is genuinely private — only accessible with your credentials
- Proper commit history: every session save becomes a timestamped Git commit
- Scalable: sessions, settings, and future data types stored as separate files
- Free on GitHub's free tier (private repos are unlimited; only Pages requires public)

Data files in `gym-data` repo:
- `sessions.json` — all logged sessions, appended on each save
- `settings.json` — user settings (age, height, weight etc.)

How it works:
- User creates a GitHub Personal Access Token (PAT) scoped to the `gym-data` repo only
- PAT entered once in the app's Settings page and stored in localStorage
- On session save: app calls GitHub API to read current `sessions.json`, appends new session, writes back — all automatic
- On app load: app fetches `sessions.json` from GitHub and uses it as the data source
- Offline fallback: if no connection or token missing, falls back to localStorage as before
- Optional "Sync now" button in Settings for manual trigger

Setup steps for user (one time only):
1. Create a new private repo called `gym-data` on GitHub
2. Go to GitHub → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens
3. Create a token with read/write access to the `gym-data` repo only
4. Paste token into the gym tracker Settings page
5. Done — all future sessions sync automatically

Security considerations:
- PAT is stored in localStorage on the iPhone — acceptable for a personal device
- Token scoped to one private repo only — limited blast radius if ever exposed
- User can revoke and regenerate the token at any time from GitHub settings

#### [DONE v1.3.0] Settings page — 4th nav tab
- Input fields: age, height (cm), weight (kg), body fat % (optional)
- Auto-calculate BMI with classification (Underweight / Normal / Overweight / Obese)
- Auto-calculate HR zones based on age (220 - age = max HR):
  - Zone 1: 50–60% — Warm up / recovery
  - Zone 2: 60–70% — Fat burn / steady state cardio
  - Zone 3: 70–80% — Aerobic / cardio improvement
  - Zone 4: 80–90% — Anaerobic / intervals
  - Zone 5: 90–100% — Max effort / peak
- Display zones as a visual colour-coded chart with actual BPM ranges
- BMR (Basal Metabolic Rate) and TDEE (Total Daily Energy Expenditure) with activity level selector
- 1RM calculator — enter weight and reps to estimate one rep max
- Body measurements log — waist, chest, arms, thighs, with date stamps (recomposition tracking)
- Training start date — displays weeks trained as a motivational counter
- Default weight drum increment — choose 1kg, 2.5kg, or 5kg steps
- GitHub sync section — PAT entry field, sync status indicator, "Sync now" button
- All settings persist to localStorage (and sync file via GitHub when configured)

#### [FEATURE] History export
**Decision: Export from desktop browser (simpler than iPhone)**

Export formats:
- CSV — Excel-compatible, one row per set
  - Columns: Date, Time, Day, Session Label, Exercise, Set Number, Weight (kg), Reps, Note
- JSON — full raw data, uploadable directly to Claude for analysis

How it works on desktop:
- Once GitHub sync is set up, desktop browser reads full history from GitHub automatically
- User clicks Export on History screen — file downloads directly in desktop browser
- No need to export from phone; desktop becomes the analysis hub

How it works on iPhone (fallback):
- iOS share sheet triggered via `navigator.share()` API
- User can save to Files, email to themselves, or share to any app

Use for Claude analysis:
1. Open gym tracker on desktop — full session history loads from GitHub
2. Click Export → download CSV or JSON file
3. In a Claude conversation, attach the file
4. Ask for analysis: progressive overload trends, volume by muscle group, consistency patterns, strength improvements over time etc.
- The longer the history, the richer the analysis

Export always available regardless of sync status — serves as manual backup too

#### [FEATURE] Reorder exercises — Plan and Log screens
- Up/down arrow buttons on each exercise row in edit mode (Plan screen)
- Same arrows available on Log screen to reorder before/during a session
- Drag-and-drop considered but arrow buttons more reliable on mobile with sweaty hands

### MEDIUM PRIORITY

#### [FEATURE] Cardio interval timer
**Decision: Dedicated Timer tab in nav bar (Plan, Log, Timer, History)**

**Cardio settings — stored in Plan per day, not global Settings**
- Each training day with cardio stores structured fields, not free text
- Cardio type toggle: Intervals or Steady State
- If Intervals, fields: warm up (min), work (sec), rest (sec), rounds, cool down (min)
- If Steady State, fields: warm up (min), duration (min), cool down (min)
- The cardio label on Plan and Log screens auto-generates from these values — no manual text entry, no risk of label and timer getting out of sync
- Values are editable per day independently — Tuesday and Thursday can have different settings
- Current defaults (Tuesday / Thursday): warm up 5 min, work 40 sec, rest 90 sec, 10 rounds, cool down 5 min
- Current defaults (Wednesday / Sunday): warm up 5 min, duration 30 min, cool down 5 min

**Timer tab — full session structure**

Intervals session flow:
1. Warm up — countdown, Zone 1–2 BPM range displayed
2. Rounds — alternating work / rest phases:
   - Work phase: green display, countdown, "WORK" label, Zone 4 BPM range
   - 3-second "GET READY" warning before each work phase
   - Rest phase: amber display, countdown, "RECOVER" label, Zone 1–2 BPM range
   - Round counter shown throughout (e.g. Round 4 of 10)
3. Cool down — countdown, Zone 1 BPM range displayed
4. Completion screen

Steady state session flow:
1. Warm up — countdown, Zone 1–2 BPM range
2. Steady state — full countdown, Zone 2–3 BPM range shown persistently, cardio note shown (bike or cross trainer preferred)
3. Cool down — countdown, Zone 1 BPM range
4. Completion screen

**Flexibility — on-the-day adjustments**
- Timer pre-loads with the saved plan values for the day being logged
- All values editable before starting (work, rest, rounds, warm up, cool down)
- In-session adjustments do not overwrite the saved plan — they apply to that session only
- Timer reads from logState.selectedDay to know which day's settings to load

**HR zones**
- BPM ranges calculated from age stored in Settings (220 - age = max HR)
- Each phase shows zone name + actual BPM range (e.g. "Zone 4 — 136–153 BPM")
- Falls back to percentage display if age not yet set in Settings

**Audio — podcast friendly**
- Uses Web Audio API in ambient mode — does not interrupt podcasts playing through earphones
- Short tones only: single beep = work phase starting, double beep = rest phase starting, triple beep = session complete
- Mute toggle available on timer screen

**Screen lock**
- Uses navigator.wakeLock to keep screen on during timer session
- Fully supported on iPhone 14 Pro running iOS 16.4+ (user confirmed iOS 26.3.1)
- Releases automatically when timer is paused, completed, or user navigates away

**Start/pause/reset controls**
- Large tap target for pause/resume (easy mid-workout)
- Reset returns to pre-start edit screen with saved plan values reloaded

#### [FEATURE] Copy day in Plan
- In each day's edit panel, a "Copy from day..." dropdown
- Select any other training day to copy its exercise list and cardio notes
- Useful when schedule changes (e.g. Tuesday plan moved to Monday)
- Prompts to confirm before overwriting

### LOWER PRIORITY

#### [FEATURE] Exercise muscle group badges
- Replaces emoji icon idea — more informative and consistent
- Small coloured badge per exercise showing muscle group: CHEST / BACK / QUADS / SHOULDERS / ARMS / CORE / FULL BODY
- User assigns muscle group to each exercise in plan edit mode
- Badge shown next to exercise name in Plan, Log, and History screens
- Colour coded per group for quick visual scanning
- Review after implementation to decide if further icon work warranted

#### [IMPROVEMENT] Cardio logging
- Currently cardio details are displayed but not logged
- Option to log actual cardio: duration, machine used, average HR
- Stored with session data and visible in History
- Feeds into Claude export analysis

---

## Known Issues / Minor Fixes

- None currently logged

---

## Notes for Developer (Claude)

- Single file app: all HTML, CSS, JS in gym-tracker.html
- No external libraries or CDNs — pure vanilla JS only
- localStorage for all persistence (with in-memory fallback if localStorage blocked)
- GitHub Pages hosting: public repo `gym-tracker` at https://hitenshah-uk.github.io/gym-tracker/gym-tracker.html
- GitHub data sync: private repo `gym-data` (to be created), accessed via PAT stored in localStorage
- GitHub API base URL: https://api.github.com/repos/hitenshah-uk/gym-data/contents/
- Version number stored in JS variable APP_VERSION and displayed in UI
- When making changes: update APP_VERSION, add entry to Version History above
- Backlog is the single source of truth for decisions — update it when direction is confirmed
