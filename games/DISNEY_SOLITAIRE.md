# Solitaire Grand Harvest — Game Knowledge File

**App name:** (check App Store — likely "Solitaire Grand Harvest" by Supertreat)
**Game type:** Golf Solitaire with a farming/harvest theme
**First observed:** 2026-03-01
**Sessions played:** 1

---

## Game Overview

A Golf Solitaire card game set in a harvest-themed world. Each "scene" contains 10 levels.
Each level costs coins to play (observed: 1,200 coins for Level 31).
The goal is to clear as many cards from the pyramid tableau as possible using a Golf Solitaire
rule: tap cards that are ±1 of the current "base" card.

---

## Map / Overworld Screen

**OCR markers:** Scene number, level number, "NEXT BONUS" countdown, coin balance

**Key UI elements (approximate normalized coordinates):**
- Coin balance: top-left ~(0.13, 0.17)
- Scene/progress indicator: bottom-left ~(0.13, 0.86)
- Level button (green, "LEVEL ##"): bottom-right ~(0.87, 0.86) ← **tap this to start a level**
- Next Bonus timer: bottom-center ~(0.48, 0.88)
- Limited-time event badge: left side ~(0.06, 0.28)
- Settings gear: top-right ~(0.91, 0.17)

**Known behavior:** Scene 3 has 10 levels (0/10 counter). Tapping the LEVEL button opens a
level-start dialog.

---

## Level Start Dialog

**OCR markers:** "LEVEL ##", "PLAY", "cost", coin amount

**Key UI elements:**
- Level title (blue panel, center): ~(0.50, 0.39)
- PLAY button (green): ~(0.46, 0.69) ← **tap this**
- Cost indicator (bottom): ~(0.48, 0.87) — observed 1,200 coins for level 31
- Close button (red X, top-right of dialog): ~(0.72, 0.26)

---

## In-Level (Card Game) Screen

**OCR markers:** Card numbers (2–10, J, Q, K, A), STREAK BONUS, coin balance

### Board Layout

The tableau uses a pyramid structure. Rows (from bottom/accessible to top/blocked):

```
Row 1 (most blocked, top of screen): ~y=35%  — 5 cards (e.g., 9 2 Q 3 10)
Row 2 (middle):                       ~y=50%  — 3-5 cards, mix of face-up/face-down
Row 3 (most accessible, bottom):      ~y=60%  — 3 accessible cards + face-downs
```

- **Face-up cards** can be tapped if accessible (no card below them in their column)
- **Face-down cards** become face-up when the card blocking them (below/in front) is removed
- Row 1 cards (top) are most blocked — only accessible late in the level

### Bottom UI (always visible)

- **Draw pile** (face-down stack): bottom-left ~(0.37, 0.83) ← tap to draw
- **Current card** (face-up, center-bottom): ~(0.52, 0.82) — read its value via OCR
- **Undo button** (green, with count): ~(0.62, 0.83) — 4 undos observed
- **WILD card** (yellow/orange, bottom-right): ~(0.87, 0.79) — 1 available observed

### Streak Bonus

- Top-right area: 5 dots that fill with each consecutive play (no drawing)
- Full streak (5 dots) = bonus reward
- **Drawing resets the streak to 0**
- OCR shows streak as filled/empty circles, e.g., "●●●●●" or "○○○○●"

### Confirmed Tap Mechanics

From live testing (2026-03-01):
- **Use exact OCR coordinates** — the `(x%, y%)` values from OCR output map directly to
  normalized tap coordinates `(x/100, y/100)`
- Tap precision must be within ~3% of card center to register
- If a tap doesn't work, try ±0.02 on y-axis first (cards are taller than wide)

**Confirmed working taps:**
| Action | OCR position | Tap coords | Notes |
|---|---|---|---|
| Play 5 (center) | ~(50, 65) | (0.50, 0.50) | Worked first try — center of screen |
| Play 6 (revealed) | ~(44, 59) | (0.44, 0.59) | Worked |
| Play 7 (left) | (25, 50) | (0.25, 0.50) | Worked at exact OCR position |
| Draw from pile | ~(37, 83) | (0.38, 0.83) | Worked, drew Ace |
| Open level | ~(87, 86) | (0.88, 0.87) | Opened level dialog |
| Tap PLAY | ~(46, 69) | (0.46, 0.69) | Started level, cost 1,200 coins |

**Taps that missed:**
| Action | Tap coords tried | Reason missed |
|---|---|---|
| Play 7 | (0.23, 0.60) | Too low — missed the card |
| Play 7 | (0.19, 0.55) | Slightly too left |
| Play 8 | (0.71, 0.45) and (0.70, 0.47) | Card may have been partially blocked by adjacent face-down |

**Lesson:** Always use the OCR-reported (x%, y%) position directly. Don't estimate visually.

---

## Strategy Notes

### Card play priority
1. Play the card that unblocks the most face-down cards (prefer mid-pyramid positions)
2. If multiple equal options, prefer the card furthest from the draw pile (saves it for later)
3. Use WILD card to extend a streak, not just to play any card
4. Avoid drawing unless truly stuck — every draw resets your streak

### When stuck
- Draw from pile at ~(0.38, 0.83)
- If still no plays, use WILD at ~(0.87, 0.79)
- If draw pile is exhausted and no plays remain, the level ends — tap the result button

### Coin management
- Levels cost coins to enter (~1,200 for level 31)
- Do NOT proceed if coins are insufficient — report to user
- Collect all reward screens to replenish coins

---

## Gotchas & Observations

- The coin cost increases with level number — check before auto-playing high levels
- "NEXT BONUS" on the map screen is a countdown timer, not a button — don't tap it
- The "LIMITED" badge on the map is a time-limited event — may require special attention
- Hearts/lives indicator visible at top (7/55 observed) — may limit plays
- Scene progress (e.g., 0/10) advances as levels are completed in that scene

---

## TODO (things to learn in future sessions)

- [ ] Confirm whether Ace wraps (can A→K or only A→2)
- [ ] What happens when scene is completed (0/10 → 10/10)?
- [ ] Is there a way to play the Row 1 (top) cards directly, or must lower rows be cleared first?
- [ ] What does the "7/55" counter at top mean?
- [ ] What happens when hearts run out?
- [ ] Confirm exact coordinates for the level complete / reward screen buttons
