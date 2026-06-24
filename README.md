# ⚔ Battle of the Hash
### A Live CySA+ CS0-003 Review Game — Students vs. The Boss

![Firebase](https://img.shields.io/badge/Firebase-Realtime%20DB-orange?logo=firebase)
![Deployment](https://img.shields.io/badge/Deploy-GitHub%20Pages-blue?logo=github)
![No Dependencies](https://img.shields.io/badge/Dependencies-Zero-green)
![Single File](https://img.shields.io/badge/Architecture-Single%20HTML-cyan)
![Built During](https://img.shields.io/badge/Built%20During-ADHD%20Hyperfocus-purple)

---

## The Origin Story

*(Skip this if you have no sense of humor. Actually, if you have no sense of humor, you probably shouldn't be teaching cybersecurity.)*

My lead instructor and I were talking after class one night about ways to make review sessions more engaging for students. She casually mentioned maybe using a little JavaScript to build something basic — "like a pong-type thing," she said.

She didn't know what she was doing.

My brain, operating in its natural state of barely-contained chaos, heard "pong" and immediately started asking *"I wonder if..."* — and that was it. I was gone. What followed was approximately six hours of ADHD hyperfocus at its most productive and least responsible, during which I built a Firebase-backed real-time multiplayer boss battle game instead of sleeping.

The result is this: a fully functional, single-file HTML game where 30 students collectively fight their instructor as a final boss, answering CySA+ CS0-003 exam questions to drain his HP while he fires AI-generated taunts at them in real time.

She wanted pong.

I built a raid boss.

To her credit, she loved it. The cohort loved it more.

---

## What This Actually Is

**Battle of the Hash** is a real-time multiplayer review game for CompTIA CySA+ (CS0-003) certification prep. It runs on a single HTML file with no build step, no npm, no server, and no recurring cost.

It uses Firebase Realtime Database to sync game state across 30+ simultaneous players, an Anthropic AI integration for live villain taunts, and Web Audio API for browser-native sound effects — all without a single external file beyond the Firebase SDK CDN.

The game is designed around the graduation cohort experience: students battle their instructor as a final boss across 25 questions that ramp from simple recall to a PBQ-style incident response scenario.

---

## Features

- **Three roles, one file** — Host (lead instructor), Hasher (villain boss), and Student, all URL-routed within a single HTML file
- **Real-time sync** — Firebase Realtime Database pushes game state to all connected clients simultaneously. Votes appear live. HP drains in real time. Taunts fire instantly.
- **25 CySA+ CS0-003 questions** mapped to exam objectives, difficulty-scaled from simple recall through PBQ-style scenario analysis
- **Dynamic damage mechanic** — class must break 60% correct on each question to deal damage. Q25 is a kill shot that drains remaining HP regardless of score.
- **Randomized answer positions** — correct answer shuffles position per question using a seeded algorithm, consistent across all devices
- **AI-generated taunts** — The Hasher panel generates villain taunts via Anthropic claude-sonnet-4-6, written to Firebase and pushed to every screen in real time
- **10 preset taunts + custom input** — for when you want to go off script
- **Web Audio API sound engine** — 13 distinct sound events, zero external files, mutable per session
- **Q25 log view** — The final PBQ scenario renders as color-coded SIEM log entries (Event IDs, NetFlow, timestamps) instead of prose
- **Session persistence** — students auto-rejoin on refresh via sessionStorage
- **500 HP each side** — built for 25 questions at 15 HP flat per miss, kill shot on Q25

---

## Architecture

```
battle-of-the-hash/
└── index.html              ← The entire game. One file.

Firebase Realtime Database
└── rooms/
    └── HASH2026/
        ├── status           lobby | question | reveal | gameover
        ├── currentQuestion  0–24
        ├── classHP          0–500
        ├── hasherHP         0–500
        ├── currentQ/        shuffled options written by host, read by all clients
        │   ├── text
        │   ├── A / B / C / D
        │   ├── correct       shuffled correct letter (single source of truth)
        │   └── timeLimit
        ├── players/
        │   └── {uid}/
        │       ├── name
        │       ├── answer
        │       └── hasAnswered
        ├── hasherTaunt       string, written by Hasher, read by all
        ├── revealData/
        │   ├── correct
        │   ├── votes
        │   ├── classWon
        │   ├── correctPct
        │   └── explanation
        └── gameResult        null | class | hasher
```

**Answer shuffle:** The host computes a deterministic shuffle of A/B/C/D options when starting each question (seeded by question index) and writes the result to Firebase. All clients read from Firebase — no local shuffle computation, no cross-device inconsistency.

**AI taunts:** The Hasher panel calls the Anthropic API directly from the browser using the `anthropic-dangerous-direct-browser-access` header. The API key is entered at runtime via a password field on the Hasher panel — it is never stored in the file or the repository.

---

## Role Guide

| Role | URL | Who | What They Do |
|---|---|---|---|
| Host | `?role=host` | Lead Instructor | Shared on Zoom. Controls question flow, sees live votes, triggers reveals. |
| Hasher | `?role=hasher` | The Boss (Instructor) | Private tab. Sees live vote intel. Fires taunts. Watches the class struggle. |
| Student | `?role=player` | Every student | Opens on their own device. Joins by name. Submits answers. |
| Landing | (no param) | Anyone | Role selector with room code display. |

---

## Damage Mechanic

| Scenario | Result |
|---|---|
| > 60% answer correctly (Q1–24) | Hasher takes exactly 15 HP |
| ≤ 60% answer correctly (Q1–24) | Class takes exactly 15 HP |
| Q25: > 60% correct | Hasher HP → 0. Class wins. |
| Q25: ≤ 60% correct | Class HP → 0. Hasher wins. |

The 60% threshold means split rooms don't randomly determine outcomes — the class has to actually know the material.

---

## Build It Yourself

### Prerequisites

- A Google account (for Firebase)
- A GitHub account (for Pages hosting)
- An Anthropic API account with credits (for AI taunts — optional but recommended)

Everything else is free. No paid tiers, no servers, no build tools.

---

### Step 1 — Firebase Project

1. Go to [firebase.google.com](https://firebase.google.com) → **Create a project**
2. Give it a name (e.g., `battle-of-the-hash`). Disable Google Analytics — you don't need it.
3. **Build → Realtime Database → Create database**
   - Region: United States (us-central1)
   - Start in **test mode**
4. Once the database is created, go to the **Rules** tab and replace the contents with:

```json
{
  "rules": {
    "rooms": {
      "HASH2026": {
        ".read": true,
        ".write": true
      }
    }
  }
}
```

Click **Publish**.

5. **Project Settings → Your apps → Add app → Web (`</>`)**
   - Nickname: `battle-of-the-hash`
   - Leave Firebase Hosting unchecked (you're using GitHub Pages)
   - Click **Register app**
6. Copy the `firebaseConfig` object shown on the next screen. You'll need these 7 values.

---

### Step 2 — Configure the HTML File

Open `index.html` in any text editor. Find the `FB_CFG` block near the top of the script section and paste your values:

```javascript
const FB_CFG = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT.firebaseapp.com",
  databaseURL:       "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
  projectId:         "YOUR_PROJECT",
  storageBucket:     "YOUR_PROJECT.firebasestorage.app",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

The `ANT_KEY` line directly below it should be left as an empty string:

```javascript
let _antKey = sessionStorage.getItem('bh_ant') || '';
```

The Anthropic key is entered at runtime on the Hasher panel — it never goes in the file. This is intentional. It means the repo can be public without exposing API credentials.

If you want to change the room code from `HASH2026`, update this line:

```javascript
const ROOM = "HASH2026";
```

---

### Step 3 — Deploy to GitHub Pages

1. Create a repository on GitHub (public or private — both work, but public is free for Pages)
2. Upload `index.html` as `index.html` in the root of the repo (not the original filename)
3. **Settings → Pages → Branch: main → Save**
4. Wait ~2 minutes for the build. Your game will be live at:

```
https://yourusername.github.io/your-repo-name/
```

---

### Step 4 — Get an Anthropic API Key (for AI Taunts)

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. **API Keys → Create key** — name it whatever you want
3. Copy the key (starts with `sk-ant-...`)
4. On game day, open `?role=hasher`, paste it into the orange key field, and click Save. It persists for the browser session only.

> **Cost note:** AI taunt generation calls `claude-sonnet-4-6` with a 150-token max output. A full 25-question game with 10 taunts costs literal cents. Add $5 in credits and you'll run this game dozens of times before you need to think about billing.

---

### Step 5 — Test Before Game Day

Open three browser tabs and run through the full loop:

1. `?role=host` → Click **Initialize Arena** → confirm the battle stage loads
2. `?role=player` → Enter a name → confirm you appear in the student list on the host screen
3. Back on host → Click **Start — Question 1** → confirm the question appears on the player tab
4. Answer on the player tab → confirm the vote count ticks up on the host screen
5. Click **Reveal** → confirm HP bars update and the correct answer highlights
6. `?role=hasher` → enter your Anthropic key → fire a preset taunt → confirm it appears on the host screen

If all six steps work, you're good.

---

### Game Day Checklist

**Before class:**
- [ ] Firebase Rules published (open read/write on the room node)
- [ ] `index.html` deployed to GitHub Pages and live URL confirmed
- [ ] Full test loop completed (see Step 5)

**At game time:**
- [ ] Open `?role=hasher` in a private tab → paste Anthropic key → Save
- [ ] Lead instructor opens `?role=host` → shares screen on Zoom
- [ ] Post the base URL in Zoom chat for students
- [ ] Host clicks **Initialize Arena**
- [ ] Students join → host clicks **Start — Question 1**

---

## Customizing the Question Bank

Questions live in the `QUESTIONS` array in `index.html`. Each question follows this format:

```javascript
{
  text: "Your question text here.",
  A: "First option",
  B: "Second option",
  C: "Third option",
  D: "Fourth option",
  correct: "B",          // original correct letter before shuffle
  damage: 15,            // HP damage value (overridden by kill shot on final Q)
  category: "CATEGORY NAME",
  explanation: "Explanation shown after reveal."
}
```

The answer shuffle happens at runtime — define your correct answer as the original letter (A/B/C/D) and the engine handles randomizing the display positions.

For the final question (last in the array), set `damage: 999`. This triggers the kill shot mechanic regardless of the actual damage value.

**Optional — log-style evidence rendering (for PBQ questions):**

Add a `logLines` array to any question to render evidence as color-coded SIEM log entries instead of prose:

```javascript
{
  text: "Your stem question here.",
  logLines: [
    { cls: "ll-alert", txt: "[02:47:03]  ⚠  SIEM P1 ALERT — [alert text]" },
    { cls: "ll-event", txt: "[02:47:03]  EVT 4624  [log details]" },
    { cls: "ll-net",   txt: "[02:47:11]  NETFLOW   [connection details]" }
  ],
  A: "...", B: "...", C: "...", D: "...",
  correct: "D", damage: 999,
  ...
}
```

CSS classes: `ll-alert` (red), `ll-event` (green), `ll-net` (yellow).

---

## Tech Stack

| Layer | What | Why |
|---|---|---|
| Hosting | GitHub Pages | Free, static, always on |
| Real-time sync | Firebase Realtime Database (Spark tier) | Free, handles 100 concurrent connections, sub-100ms sync |
| AI | Anthropic claude-sonnet-4-6 | Villain taunts. Contextual. Sharp. |
| Audio | Web Audio API | Zero external files. Deterministic. Works offline. |
| Frontend | Vanilla HTML/CSS/JS | No build step. One file. Deployable anywhere. |
| SDK | Firebase 9.23.0 compat (CDN) | Script tag. Done. |

---

## Security Notes

- The Firebase config values (apiKey, projectId, etc.) are safe to expose publicly — they are not secret keys, they are identifiers. Access is controlled by Firebase Security Rules.
- The Anthropic API key is **never stored in the file**. It is entered at runtime in the Hasher panel and stored in browser sessionStorage only. It disappears when the tab closes.
- For long-term protection after your game runs, update your Firebase Rules to deny all access:

```json
{ "rules": { ".read": false, ".write": false } }
```

---

## Credits

Built by **Shane McKenney**, cybersecurity instructor and CompTIA CySA+ subject matter expert.

Accidentally inspired by a lead instructor who said "pong" and meant it literally.

Built with Claude (Anthropic) during a single ADHD hyperfocus sprint that will not be repeated. Probably.

---

*For questions about the game, the question bank, or what ADHD hyperfocus actually feels like at 2AM when you're supposed to be prepping lesson plans — find me on LinkedIn or GitHub.*
