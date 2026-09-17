# DORA — Dynamic Overview & Retention Assistant

> **DORA** is a lean classroom tool that closes the real-time understanding gap:
> teachers know *in the moment* who has understood the lesson, and students can
> answer honestly and anonymously — without fear of being singled out.

This repository contains the **working interactive prototype** (single-file HTML),
the **pitch to the Canton of Bern & schools**, and the **engineering
specification / Definition of Done**.

## 🔗 Live prototype (always up to date)

**GitHub Pages:** https://architect-dna-ch.github.io/dora/

**Netlify (mirror):** https://silver-choux-d039bd.netlify.app/

The live version is served via GitHub Pages from the `main` branch. Every push
to `main` updates the live site automatically — so this is a **dynamic, ongoing
development** prototype, not a static snapshot.

> **Rollback:** the tag `pre-sync-feature` marks the last stable single-tab
> version before the two-tab sync + user-authored cards were added.
> `git checkout pre-sync-feature` to go back.

---

## What DORA does — in 30 seconds

1. **Launch Checkpoint (teacher):** mid-lesson, the teacher starts a short
   understanding question (e.g. *"What did we just cover?"*).
2. **Student Question Overlay (student):** every student answers **anonymously
   on their own device** — no fear of embarrassment.
3. **Live Class Progress (teacher):** the teacher sees **live, in real time** how
   many understood — and who is falling behind.

DORA is built on three evidence-based learning mechanisms:
**Active Recall**, **formative feedback**, and **spaced repetition**.

### Bonus: "My Personal Mode"
The prototype also includes a **private, self-directed learning space** for
students — active-recall cards, just-in-time **theory-gap micro-lessons**, and a
**spaced-repetition review queue**. This is where a student can learn on their
own terms, in short bursts, without a teacher watching or grades attached.

---

## Quick start

The prototype is a **single self-contained HTML file** — no build step, no
dependencies to install (Tailwind is loaded via CDN).

### Run it

Open [`index.html`](index.html) directly in any modern browser:

```bash
# macOS / Linux
open index.html
```

Or serve it locally:

```bash
# with Python 3
python3 -m http.server 8000
# then visit http://localhost:8000
```

### Try the demo

1. **Teacher view:** click **🚀 Launch Checkpoint** — a countdown starts and the
   student overlay opens. Watch the **Live Class Progress** fill as (simulated)
   students respond.
2. **Student overlay:** type an answer, optionally tick *"I feel confident"*,
   and submit.
3. **Personal Mode:** pick a topic (**Law OR/ZGB**, **Math**, or **Finance &
   Bookkeeping**), try to answer from memory, then **Reveal answer**. If you
   don't get it, DORA detects the missing prerequisite and serves a **2-minute
   micro-lesson** before you retry. Cards you master are scheduled into the
   **spaced-repetition review queue**.

### Two-tab live demo (teacher ↔ student, no backend)

DORA can run as **two tabs on the same machine** — one teacher, one student —
synced live via the browser `BroadcastChannel` API. No server, no account.

| Tab | URL | Behaviour |
|-----|-----|-----------|
| Teacher | `index.html?role=teacher` | Full control panel; launches/ends checkpoints |
| Student | `index.html?role=student` | Teacher panel hidden; overlay auto-opens and mirrors the session |

A **role banner** appears at the top of each tab with an **"Open other view"**
button that spawns the counterpart tab for you.

**Demo flow:**
1. Open `index.html?role=teacher` (or just `index.html` — teacher is the default).
2. Click **Open student view** in the banner → a second tab opens as the student.
3. In the teacher tab, click **🚀 Launch Checkpoint** → the student tab receives
   the question instantly and its countdown starts.
4. Submit an answer in the student tab → it appears live in the teacher's
   **Live Class Progress**.
5. End the session in the teacher tab → the student tab closes its overlay.

> `BroadcastChannel` only syncs tabs of the **same origin on the same device**.
> That is exactly right for a live demo on one laptop/projector. Cross-device
> sync would need a backend (out of scope for the prototype).

### Add your own cards (no code editing)

In **My Personal Mode → Choose what to master**, click **＋ Add your own card**.
Fill in a question, an answer, and (optionally) the missing prerequisite and a
short micro-lesson. You can target an existing topic or create a **new topic**.

Cards are saved to **`localStorage`** on that device only (`dora-user-cards-v1`)
— no server, no account, no code changes. They merge into the topic list on
every load.

---

## Project structure

```
dora/
├── index.html                    # Working interactive prototype (single file)
├── PITCH_DORA_Bern_DE.md         # Pitch to the Canton of Bern & schools (German)
├── PRESENTATION_Director_DE.md   # Mid-length demo script for a school director
├── SPEC.md                       # Definition of Done (DoD) engineering spec
├── MEMORY.md                     # Working notes / project memory
└── README.md                     # This file
```

---

## Documentation

- **Pitch:** [`PITCH_DORA_Bern_DE.md`](PITCH_DORA_Bern_DE.md) — the honest
  proposal for why we should finally address the frontal-teaching problem, and a
  concrete 1-semester pilot design for the Canton of Bern.
- **Engineering spec / Definition of Done:**
  [`SPEC.md`](SPEC.md) — the shared contract for what "done" means for every
  change (functional correctness, code quality, testing, docs, review, CI/CD,
  accessibility, security, release readiness).

---

## Status

**Prototype stage.** This is a working interactive demo (front-end only, mock
data, no backend). It is **ongoing development** — a proof of concept to
demonstrate the concept and gather feedback before any pilot.

---

## License & privacy

- **Open source:** the entire code is openly viewable — the Canton can verify
  itself what happens with data.
- **Privacy-first by design:** anonymous responses, no profiling of children, no
  data shared with third parties, aligned with the Swiss DSG. See the
  *Datenschutz & Ethik* section of the pitch for details.

---

*DORA — because understanding should not be guessed.*
