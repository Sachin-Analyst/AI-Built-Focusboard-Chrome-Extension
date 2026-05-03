# FocusBoard – Chrome Extension PRD
### Product Requirements Document · Version 1.0

---

## Overview

**Product Name:** FocusBoard  
**Type:** Chrome Browser Extension (New Tab Override)  
**Owner:** Sachin  
**Purpose:** Replace the default Chrome new tab page with a personal productivity dashboard that combines a daily focus prompt, task management with categories and timing, focus mode site blocking, motivational quotes, and a monthly calendar streak tracker.

---

## Guiding Principles

- Every feature should feel personal, not generic — it greets Sachin by name.
- Data persists locally using `chrome.storage.local` (no backend required).
- The UI should feel calm and intentional — full-screen wallpaper, minimal chrome.
- Each phase is independently shippable and testable.

---

## Tech Stack Recommendation

| Layer | Choice |
|---|---|
| Extension type | Chrome Manifest V3 |
| UI framework | Vanilla JS + HTML/CSS (or lightweight React if preferred) |
| Storage | `chrome.storage.local` |
| Site blocking | `chrome.declarativeNetRequest` + custom redirect page |
| Build tool | Vite or plain bundler |

---

## Phase 1 — Core New Tab Shell

**Goal:** Get the foundational new tab page running with wallpaper, greeting, live clock, and the daily focus prompt.

### 1.1 New Tab Override
- The extension overrides the new tab page (`newtab` override in `manifest.json`).
- On open, it renders a full-screen wallpaper behind all content.
- Default wallpaper: a high-quality bundled default image.

### 1.2 Wallpaper System
- Display a full-screen background image using `background-size: cover`.
- A semi-transparent overlay (dark or blur) ensures text readability on any image.
- On first install, use the bundled default wallpaper.
- User-uploaded wallpapers (Phase 3) override the default.

### 1.3 Greeting + Live Clock
- Top-left (or top-center): Display **"Good morning, Sachin"** / **"Good afternoon, Sachin"** / **"Good evening, Sachin"** based on system time.
- Directly below or alongside: Live digital clock updating every second (`HH:MM:SS` or `HH:MM AM/PM` — configurable).
- Font: Large, clean, white with subtle text-shadow.

### 1.4 Daily Focus Prompt
- Each morning (first new tab open after midnight), a modal or prominent card appears asking:  
  **"What is your main focus today, Sachin?"**
- A text input lets Sachin type a focus statement.
- On submit, the focus is saved to `chrome.storage.local` keyed by date (`YYYY-MM-DD`).
- The focus statement is shown on the dashboard for the rest of the day (replacing the prompt).
- If the prompt has already been answered today, it does not re-appear.

### 1.5 Storage Schema (Phase 1)
```json
{
  "settings": {
    "name": "Sachin",
    "wallpaper": null
  },
  "dailyFocus": {
    "2025-07-01": {
      "focus": "Finish SQL module",
      "category": null,
      "answered": true
    }
  }
}
```

### Acceptance Criteria
- [ ] New tab opens with full-screen wallpaper and readable overlay.
- [ ] Correct time-based greeting renders with Sachin's name.
- [ ] Live clock ticks every second.
- [ ] Focus prompt appears once per day, saves on submit, and shows the saved focus thereafter.

---

## Phase 2 — Task List with Categories

**Goal:** A complete task management system with color-coded categories, filters, and task status tracking.

### 2.1 Task Data Model
```json
{
  "tasks": [
    {
      "id": "uuid",
      "title": "Learn SQL JOINs",
      "category": "Learning",
      "status": "pending",
      "createdAt": "2025-07-01T09:00:00",
      "completedAt": null,
      "deleted": false
    }
  ]
}
```

**Status values:** `pending` | `completed` | `unfinished_excellence` | `deleted`

### 2.2 Default Categories
| Category | Color |
|---|---|
| Work | `#4A90D9` (Blue) |
| Personal | `#E67E22` (Orange) |
| Learning | `#27AE60` (Green) |

### 2.3 Custom Categories
- Users can create custom categories with a name and a color (color picker).
- Custom categories are stored in `chrome.storage.local` under `settings.categories`.
- Max 10 total categories (default 3 + 7 custom).

### 2.4 Add Task Flow
- A text input with a **+** button at the top of the task list.
- A category dropdown next to the input (defaults to last-used category).
- On submit: task is added to the list with its category color shown as a left border or color dot.

### 2.5 Task Status Actions
Each task row has an action menu or inline buttons:
- ✅ **Mark Completed** — strikethrough text, green check, moves to bottom.
- ⭐ **Mark Unfinished Excellence** — amber indicator, kept visible (acknowledges good effort even if incomplete).
- 🗑️ **Delete** — removes from list (soft-delete, hidden from view but kept in storage for streak calculation).

### 2.6 Category Filter Chips
- A row of filter chips at the top of the task list: **All | Work | Personal | Learning | [custom…]**
- Clicking a chip filters the list to show only tasks of that category.
- Active chip is highlighted with the category color.
- "All" is selected by default.

### 2.7 UI Layout
```
[ + Add a task...      ] [Category ▼] [Add]
[ All ] [ Work ] [ Personal ] [ Learning ] [ Custom ]

○  Learn SQL JOINs                    [Learning]  ···
○  Prepare Q3 report                  [Work]      ···
✅ Call dentist                        [Personal]  ···
```

### Acceptance Criteria
- [ ] Tasks can be added with a category.
- [ ] Each category renders with its assigned color.
- [ ] Tasks can be marked Completed, Unfinished Excellence, or Deleted.
- [ ] Filter chips correctly show/hide tasks by category.
- [ ] Custom categories can be created with a name and color.
- [ ] All pre-existing tasks continue to work after adding new categories.

---

## Phase 3 — Focus Timing + Calendar Streak

**Goal:** Track time spent on each focus task per day. Display a monthly calendar with focus streaks per category.

### 3.1 Focus Timing

Each task can have a time log entry attached:
```json
{
  "timeLogs": [
    {
      "id": "uuid",
      "taskId": "task-uuid",
      "category": "Learning",
      "label": "SQL JOINs study",
      "durationMinutes": 120,
      "date": "2025-07-01"
    }
  ]
}
```

**Log a Focus Session UI:**
- Next to each task, a **"Log Time"** button or clock icon.
- A small form appears: pre-filled task name, category (auto from task), duration input (hours + minutes), and a date picker (defaults to today).
- On save, the entry is recorded.

**Daily Summary:**  
A collapsed section at the bottom of the task list shows today's logged time, e.g.:
```
Today's Focus:  Work — 1h 30m  |  Learning — 2h  |  Total: 3h 30m
```

### 3.2 Monthly Calendar Streak

A calendar panel (bottom or side panel) showing the current month:

- Each day cell shows colored dots or small category chips for categories that had logged time that day.
- Days with any logged focus are considered "focus days."
- The current consecutive-day streak is shown above the calendar:  
  **"🔥 7-day focus streak"**

**Streak Rules:**
- A day counts if at least one time log entry exists for that date.
- Streak resets if a day with no logs is found going backwards from today.
- Streak is calculated at render time from `timeLogs`.

**Calendar Interaction:**
- Clicking a past day shows a tooltip/popover: that day's focus statement + time logs.

### Acceptance Criteria
- [ ] Time can be logged against any task with a label, duration, and date.
- [ ] Today's total focus time is shown, broken down by category.
- [ ] Monthly calendar renders with per-day category indicators.
- [ ] Consecutive streak count is correctly calculated and displayed.
- [ ] Clicking a past day shows focus + time log data for that day.

---

## Phase 4 — Motivational Quotes

**Goal:** Display a daily rotating motivational quote from a curated list of 31 quotes.

### 4.1 Quote Display
- Position: Right side of the dashboard, vertically centered or near the bottom-right.
- Style: Italic, white text, semi-transparent background card, or just bare text with text-shadow.
- One quote per day — determined by `dayOfYear % 31` (so it cycles through all 31 quotes annually).

### 4.2 Quote List (Complete — do not alter wording)

| # | Quote |
|---|---|
| 1 | There are decades where nothing happens and days where decades happen |
| 2 | Everything is a win when the goal is experience |
| 3 | What you get doesn't make you happy, what you become gives you happy |
| 4 | It's the possibility of having a dream come true that makes life interesting |
| 5 | He who has a why to live can bear almost any how |
| 6 | Don't let anyone rob you of your imagination or your curiosity. It's your place in the world, it is your life. |
| 7 | Yes we need to stay busy in our life, productivity to increase chance to build a great life — still we need to have the ability to notice people's efforts. |
| 8 | Life is made being lost and finding ourselves again |
| 9 | The uncertainty of my future isn't gone, yet each day I woke up excited to discover life was present to me — and don't rush it, Sachin |
| 10 | If everything doesn't happen quite the way you'd like, it doesn't make too much difference because you can fix it |
| 11 | You are what you read |
| 12 | All roads lead to something you were always predestined to do |
| 13 | Don't wait until you know who you are to get started |
| 14 | Don't overanalyze. Do the work. |
| 15 | On the surface of the ocean there is stillness. What is happening on the surface doesn't matter — it is what's going below that will kill you |
| 16 | Not all beauty is so immediately beautiful |
| 17 | The fire you seek is already burning within; your only job is to stop pouring water on it |
| 18 | Growth is often quiet and invisible. Do not mistake silence for a lack of progress |
| 19 | Energy follows intention. Focus on the 'why,' and the 'how' will fuel itself |
| 20 | Your potential is a reservoir, not a cup. The more you draw from it, the more it refills |
| 21 | Resilience is not the absence of fatigue, but the decision to honor the purpose over the pain |
| 22 | Every small action is a vote for the person you are becoming |
| 23 | Internal power is built in the moments when no one is watching and nothing is guaranteed |
| 24 | Do not wait for the storm to pass; learn to generate your own heat from within |
| 25 | The process is the teacher. If you skip the struggle, you lose the lesson |
| 26 | Your mind is a garden. Discipline is the fence that keeps the distractions out and the energy in |
| 27 | Great things are not done by impulse, but by a series of small things brought together |
| 28 | The depth of your struggle determines the height of your success. Keep digging |
| 29 | Consistency transforms average ability into extraordinary results |
| 30 | True strength is the ability to maintain your light even when the world feels dark |
| 31 | You are not behind; you are in preparation. Trust the timing of your own evolution |

### Acceptance Criteria
- [ ] One quote renders per day, cycling through all 31 quotes.
- [ ] Quote is displayed on the right side of the new tab page.
- [ ] Quote styling is readable over any wallpaper.

---

## Phase 5 — Focus Mode (Site Blocker)

**Goal:** A toggle that blocks distracting websites and shows a custom motivational redirect page instead of an error.

### 5.1 Focus Mode Toggle
- A prominent toggle button on the dashboard: **"Focus Mode: OFF / ON"**
- When toggled ON, blocking rules are activated via `chrome.declarativeNetRequest`.
- When toggled OFF, all rules are removed and blocked sites load normally.
- Focus Mode state persists in `chrome.storage.local` — survives browser restart.

### 5.2 Default Block List
The following domains are blocked by default (and subdomains):
- `instagram.com`
- `youtube.com`
- `linkedin.com`
- `web.whatsapp.com`
- `twitter.com` / `x.com`
- `facebook.com`
- `reddit.com`
- `tiktok.com`
- `netflix.com`

### 5.3 Custom Block List (managed in Settings — Phase 6)
- Users can add or remove domains from the block list.
- Stored under `settings.blockList` in `chrome.storage.local`.

### 5.4 Redirect Page
When a blocked site is visited, the user is redirected to a bundled extension page (`blocked.html`) instead of seeing a Chrome error.

**Blocked page content:**
- Full-screen page matching the FocusBoard aesthetic (dark background, same font).
- Large centered message:  
  > **"Let's get back to the task at hand. You're doing great."**
- Shows the current day's focus statement (pulled from storage).
- A **"Back"** button (goes to previous page or new tab).
- Small text showing which site was blocked.

### 5.5 Implementation Notes
- Use `chrome.declarativeNetRequest` with dynamic rules (not static).
- Redirect all matched URLs to `chrome-extension://<id>/blocked.html?site=<blocked-domain>`.
- Rules are added/removed programmatically when toggle changes.

### Acceptance Criteria
- [ ] Toggle ON activates blocking; toggle OFF deactivates it.
- [ ] Visiting a blocked site redirects to `blocked.html`, not a browser error.
- [ ] Blocked page shows the focus message and today's focus statement.
- [ ] Focus Mode state persists across new tabs and browser restarts.

---

## Phase 6 — Settings Panel

**Goal:** A settings screen accessible via a gear icon where Sachin can manage all personal preferences.

### 6.1 Access
- A **⚙️ Settings** icon in the top-right corner of the new tab page.
- Clicking opens the Settings panel (full overlay or slide-in panel).
- A **"Return to Focus Board"** button closes the panel.

### 6.2 Settings Sections

#### Section A: Personal
- **Your Name:** Text input, defaults to "Sachin". Used in greeting and quotes.

#### Section B: Wallpaper
- **Upload Wallpaper:** File input accepting JPG/PNG/WebP.
- Uploaded image is stored as a base64 data URL in `chrome.storage.local` under `settings.wallpaper`.
- **Remove Wallpaper:** Resets to the bundled default.
- Preview thumbnail shown after upload.

#### Section C: Categories
- List of all current categories (default + custom) with their color swatches.
- **Add Category:** Name input + color picker + "Add" button.
- **Delete:** Remove a custom category (disabled for default 3; asks for confirmation if tasks exist in that category).
- Color change: Click the swatch to reopen the color picker.

#### Section D: Block List
- List of currently blocked domains with a remove (×) button per entry.
- **Add Domain:** Text input + "Add" button. Strips `https://`, `www.`, trailing slashes automatically.
- Preset domains are shown but can also be removed.

#### Section E: Clock Format
- Toggle: **12-hour (AM/PM)** vs **24-hour** clock display.

### 6.3 Auto-Save
- All settings save immediately on change (no "Save" button needed).
- Changes to the block list take effect immediately if Focus Mode is ON.

### Acceptance Criteria
- [ ] Settings panel opens and closes cleanly.
- [ ] Name change reflects on the greeting immediately on return.
- [ ] Wallpaper upload changes the background immediately.
- [ ] Categories can be added and deleted; changes reflect in task list.
- [ ] Block list additions/removals take effect without restart.

---

## Full Storage Schema (Final)

```json
{
  "settings": {
    "name": "Sachin",
    "wallpaper": null,
    "clockFormat": "12h",
    "focusModeEnabled": false,
    "blockList": [
      "instagram.com",
      "youtube.com",
      "linkedin.com",
      "web.whatsapp.com",
      "twitter.com",
      "x.com",
      "facebook.com",
      "reddit.com",
      "tiktok.com",
      "netflix.com"
    ],
    "categories": [
      { "id": "work",     "name": "Work",     "color": "#4A90D9", "isDefault": true },
      { "id": "personal", "name": "Personal", "color": "#E67E22", "isDefault": true },
      { "id": "learning", "name": "Learning", "color": "#27AE60", "isDefault": true }
    ]
  },
  "dailyFocus": {
    "2025-07-01": {
      "focus": "Finish SQL module",
      "answered": true
    }
  },
  "tasks": [
    {
      "id": "uuid",
      "title": "Task title",
      "category": "learning",
      "status": "pending",
      "createdAt": "ISO8601",
      "completedAt": null,
      "deleted": false
    }
  ],
  "timeLogs": [
    {
      "id": "uuid",
      "taskId": "task-uuid",
      "category": "learning",
      "label": "SQL JOINs study",
      "durationMinutes": 120,
      "date": "2025-07-01"
    }
  ]
}
```

---

## UI Layout Reference

```
┌────────────────────────────────────────────────────────────────┐
│ [Wallpaper full screen]                               [⚙️]     │
│                                                                  │
│  Good morning, Sachin          "Quote of the day              │
│  10:45 AM                       appears here on the           │
│                                  right side of screen"         │
│  Today's Focus: Finish SQL module                               │
│                                                                  │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ + Add a task...                     [Category▼] [Add]│      │
│  │ [All] [Work] [Personal] [Learning] [+Custom]         │      │
│  │                                                       │      │
│  │ ● Learn SQL JOINs              [Learning] [⏱] [···] │      │
│  │ ● Prepare Q3 Report            [Work]     [⏱] [···] │      │
│  │ ✅ Call dentist                 [Personal]            │      │
│  └──────────────────────────────────────────────────────┘      │
│                                                                  │
│  ┌──── May 2025 ──────────┐   🔥 7-day streak                  │
│  │ Mo Tu We Th Fr Sa Su   │   [Focus Mode: OFF ○──]            │
│  │  1  2  3  4  5  6  7   │                                     │
│  │ ●  ●     ●  ●  ●  ●    │                                     │
│  └────────────────────────┘                                     │
└────────────────────────────────────────────────────────────────┘
```

---

## Build Phases Summary

| Phase | Feature | Complexity |
|---|---|---|
| 1 | Shell: wallpaper, greeting, clock, daily focus prompt | Low |
| 2 | Task list with categories, filters, status actions | Medium |
| 3 | Focus timing, time logs, calendar streak | Medium-High |
| 4 | Motivational quotes (daily rotation) | Low |
| 5 | Focus Mode toggle + site blocking + redirect page | High |
| 6 | Settings panel (name, wallpaper, categories, block list) | Medium |

> **Recommended build order:** 1 → 4 → 2 → 3 → 5 → 6  
> Phase 4 is quick and gives visual richness early. Phase 5 requires careful handling of Chrome's `declarativeNetRequest` API — tackle it after core features are stable.

---

## File Structure (Recommended)

```
focusboard-extension/
├── manifest.json
├── newtab.html
├── blocked.html
├── src/
│   ├── newtab.js
│   ├── blocked.js
│   ├── background.js          ← service worker (focus mode rules)
│   ├── storage.js             ← storage helpers
│   ├── quotes.js              ← quotes array + getQuoteForToday()
│   └── components/
│       ├── clock.js
│       ├── taskList.js
│       ├── calendar.js
│       ├── focusMode.js
│       └── settings.js
├── styles/
│   ├── newtab.css
│   └── blocked.css
└── assets/
    └── default-wallpaper.jpg
```

---

## manifest.json Permissions Required

```json
{
  "manifest_version": 3,
  "permissions": [
    "storage",
    "declarativeNetRequest",
    "declarativeNetRequestWithHostAccess"
  ],
  "host_permissions": ["<all_urls>"],
  "chrome_url_overrides": {
    "newtab": "newtab.html"
  },
  "background": {
    "service_worker": "src/background.js"
  },
  "web_accessible_resources": [
    {
      "resources": ["blocked.html"],
      "matches": ["<all_urls>"]
    }
  ]
}
```

---

## Edge Cases & Notes for the AI Coder

- **Storage size limit:** `chrome.storage.local` has a 10MB limit. Wallpaper images stored as base64 can be large — consider warning if image > 2MB and compressing to canvas before storing.
- **Focus prompt timing:** "Morning" prompt should show on the first new tab open after midnight, not literally only between 6–12. Use date string comparison (today's date !== lastPromptDate).
- **Streak calculation:** Must be computed client-side from `timeLogs` at render time. Do not store streak as a separate value — it must always be derived to stay accurate.
- **Task deletion:** Use soft-delete (`deleted: true`) so time logs tied to deleted tasks still count for streak.
- **Focus Mode on restart:** On service worker startup, check `settings.focusModeEnabled` and re-apply `declarativeNetRequest` rules if true.
- **Blocked page redirect:** The `blocked.html` page must be listed in `web_accessible_resources` to be redirectable via `declarativeNetRequest`.
- **Quote index 9** references "don't rush it, Sachin" — this is intentional and personal. Preserve exact wording.

---

*End of PRD — FocusBoard Chrome Extension v1.0*
