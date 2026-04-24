# Word Games Project — Claude Reference

## Git & GitHub Setup
- **Repo 1 (Word Game 1):** https://github.com/vishnupriya-vg/Word-Game-1
- **Repo 2 (Word Game 2):** https://github.com/vishnupriya-vg/Word-Game-2
- **GitHub Pages (live):** https://vishnupriya-vg.github.io/Word-Game-1/
- Branch: `master`
- Remotes: `origin` → Word-Game-1, `wg2` → Word-Game-2
- Always commit with clean, descriptive messages and push to both remotes when relevant.

---

## Game 1 — Bomb Tap: Odd One Out

### Overview
A timed vocabulary game for grades 3–10 where players identify the word that does NOT belong in a group. The other three words share a relationship (synonyms or same category); the odd one out is an antonym or unrelated word. A bomb countdown creates urgency.

### File
- `index.html` — entire game (HTML + CSS + JS inline)
- `questions.js` — question bank (160 questions, 20 per grade G3–G10)

### Gameplay
- Player selects a grade (G3–G10) on the start screen.
- Each round: 4 words are shown; player taps the odd one out.
- Correct: bomb diffuses, celebration sound, score +1.
- Incorrect: bomb explodes, blast sound, game over.
- Timer counts down (scales with grade: 30s at G3, 10s at G10).
- Score tracked across rounds within a session.

### Question Format (`questions.js`)
```js
{ words: ['SYN1','SYN2','SYN3','ANTONYM'], odd: 3, category: '...', explanation: '...' }
```
- 160 questions total (20 per grade)
- `odd` is the zero-based index of the odd-one-out word
- Words shuffled on display so position of odd word varies

### Key Features
- **Sound effects** (Web Audio API — no external files):
  - `playTick(urgent)` — ticking sound; faster/higher when urgent
  - `playBlast()` — explosion sound on wrong answer
  - `playCelebration()` — ascending arpeggio on correct answer
- **Alarm lights**: Two `.alarm-light` divs flank the bomb emoji, blink red when ≤5s remain
- `setAlarm(on)` toggles `.active` class on alarm lights
- Timer scales: `ROUND_TIME = Math.max(10, 30 - (grade - 3) * 3.33)`

### Visual Design
- Dark bomb-defuse aesthetic
- Bomb emoji as centrepiece with blinking alarm lights
- Red/green feedback flash on answer

---

## Game 2 — Parcel Post: Word Sorting Game

### Overview
A vocabulary sorting game for grades 3–10. Each round a target word is shown; players sort 6 arriving parcels (2 synonyms, 2 antonyms, 2 neither words) into three baskets. Three rounds per session (one target word per round). Warm, pleasant postal theme.

### Files
- `parcel-post.html` — entire game (HTML + CSS + JS inline)
- `parcel-questions.js` — question bank (3 questions per grade G3–G10)

### Gameplay Flow
1. **Grade Select screen**: Stamp-styled grade buttons (G3–G10).
2. **Game screen** (3 rounds):
   - Header: grade badge | round indicator | score + exit button
   - Timer bar: 60 seconds per round
   - Target board displays the round's target word
   - Conveyor belt delivers parcels one at a time
   - Player clicks a basket to sort the current parcel:
     - **Synonym basket** (green) — words that mean the same
     - **Antonym basket** (red) — words that mean the opposite
     - **Neither basket** (tan) — same part of speech but clearly different meaning
   - Correct sort: parcel flies into basket with animation; 10 pts scored
   - Wrong sort: parcel bounces back with red tint + shake; word requeued; 0 pts for that word
   - Basket seals when both its words are correctly placed (ascending arpeggio + glow animation)
   - Round ends when all 3 baskets are sealed or timer hits 0
3. **Scorecard screen**: per-round word chips (✓ or strikethrough), score per round (max 60), total (max 180)

### Question Format (`parcel-questions.js`)
```js
const PARCEL_Q = {
  3: [
    { target:'HAPPY', syn:['JOYFUL','GLAD'], ant:['SAD','MISERABLE'], nei:['CALM','CURIOUS'] },
    ...
  ],
  ...
}
```
- 3 questions per grade (G3–G10), 24 total
- `nei` (neither) words: same part of speech as target, adjacent domain, unambiguously NOT synonyms or antonyms
- Content rule: neither words must not be interpretable as synonyms or antonyms in any common usage

### Graded Word List
| Grade | Target Words |
|-------|-------------|
| G3 | HAPPY, BIG, FAST |
| G4 | BRAVE, ANGRY, CLEAN |
| G5 | ABUNDANT, COURAGEOUS, CHEERFUL |
| G6 | INTELLIGENT, GENEROUS, CAUTIOUS |
| G7 | DILIGENT, FRUGAL, CANDID |
| G8 | EPHEMERAL, TENACIOUS, BENEVOLENT |
| G9 | LOQUACIOUS, INTREPID, MAGNANIMOUS |
| G10 | EQUIVOCAL, PERFIDIOUS, SANGUINE |

### Key Features
- **Parcel flight animation**: `getBoundingClientRect()` + `position:fixed` clone technique. Clone is created at exact belt coordinates, then CSS-transitioned to basket coordinates.
- **Wrong sort bounce-back**: Clone moves 45% toward chosen basket (momentum), then returns with red tint, shakes (translateX oscillation), fades out.
- **Feedback bar**: Fixed-height div (not floating toast) between belt and baskets — no overlap with any other element. Cleared automatically when next parcel appears (`clearFeedback()` in `showNextParcel()`).
- **Sound effects** (Web Audio API):
  - Uh-oh buzz on wrong sort
  - Ascending arpeggio [523, 659, 784, 1047 Hz] on basket seal
- **Scoring**: 10 pts per correct first-attempt sort; 0 for any word that needed a retry. Max 60/round, 180/session.
- **Timer**: 60 seconds per round; fill bar animates green → orange → red.
- **Exit button**: `⬅ Exit` in header returns to grade select; game state resets.

### Visual Design
- Background: `linear-gradient(160deg, #fef9f0, #faeedd)` — warm cream
- Header: `linear-gradient(135deg, #c84a18, #e06020)` — postal red
- Synonym basket: `#f0faf0` / `#66bb6a` — green
- Antonym basket: `#fff5f5` / `#ef9a9a` — soft red
- Neither basket: `#faf7f2` / `#bcaaa4` — tan
- Conveyor belt: animated rolling dashes via `beltRoll` keyframe

### State Object
```js
const st = {
  grade, rounds,       // selected grade; shuffled array of 3 question objects
  ri,                  // current round index (0–2)
  queue,               // remaining parcels to sort this round
  sorted,              // { syn:[], ant:[], nei:[] } — correctly placed words
  failed,              // set of words that needed a retry (excluded from score)
  score,               // running total score
  timer, timerID       // seconds remaining; setInterval handle
}
```

---

## Tech Stack
- Vanilla HTML/CSS/JS — no frameworks, no build tools
- Web Audio API for all sounds (no external audio files)
- CSS keyframe animations: `enterRight`, `sealPulse`, `wordDrop`, `beltRoll`
- Fisher-Yates shuffle for randomising words and round order
- `getBoundingClientRect()` + `position:fixed` for flight animations

---

## Other Files
- `mockup-games.html` — playable one-round visual demo of Parcel Post and Bloom Garden (used for design approval, not a production game)
- `notes.md` — miscellaneous design notes
- `Vocab games- WORD LIST.xlsx` — source word list spreadsheet
