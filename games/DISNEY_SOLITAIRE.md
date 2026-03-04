# Solitaire Grand Harvest — Game Knowledge File

**App name:** (check App Store — likely "Solitaire Grand Harvest" by Supertreat)
**Game type:** Golf Solitaire with a farming/harvest theme
**First observed:** 2026-03-01
**Sessions played:** 4

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

### Fan Layout (observed Level 32+)

Some levels use a **fan layout** instead of a standard pyramid:
- Cards are arranged in a fan/arc — only the **outer tip cards** on each arm of the fan are accessible
- Inner cards unlock **progressively** as outer ones are cleared (like peeling an onion from outside in)
- **Gold ✚ cards** appear as special cards in middle positions (e.g., 10♦, K♦) — clearing the outer tips eventually unlocks them
- Example (Level 32): gold ✚ 6♥ was the outer tip on the left — playing it revealed A♠ + 5♠ behind it

**Fan layout outer-tip algorithm:**
```
1. OCR all visible card positions
2. Sort by x% — leftmost and rightmost are the accessible outer tips
3. Play whichever outer tip is ±1 of current card
4. After each tip removal, re-sort to find new outer tip (next card inward on that arm)
5. Never reach inward past the revealed card — wait for the tip to be played first
```

**Fan layout strategy:** identify the two outer tip cards first (one per arm). Play whichever connects to current card. Each outer tip removal unlocks the next card inward.

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
2. If multiple equal options, prefer the card with the **deepest pile** behind it (most cards blocked by it)
3. Face-down / blocked cards cannot be played directly — lower cards must be cleared first to reveal them
4. Use WILD card to extend a streak, not just to play any card
5. Avoid drawing unless truly stuck — every draw resets your streak

### Arrow cards

Some cards have an **arrow indicator** that changes their value each turn:
- **↑ up arrow** — card value increases by 1 every turn (e.g., 5 → 6 → 7 …)
- **↓ down arrow** — card value decreases by 1 every turn (e.g., 8 → 7 → 6 …)

Factor the current turn's value (not the printed base value) when checking ±1 matches.
An arrow card approaching your current card's value can become playable in 1-2 turns — plan around it.

### When stuck
- Draw from pile at ~(0.38, 0.83)
- If still no plays, use WILD at ~(0.87, 0.79)
- If draw pile is exhausted and no plays remain, the level ends — tap the result button

### Coin management
- Levels cost coins to enter: ~1,200 for levels 31–32, ~1,500 for level 58+
- Confirmed: coins are deducted when PLAY is tapped (e.g., 5,483 → 3,983 = 1,500 deducted for Level 58)
- Do NOT proceed if coins are insufficient — report to user
- Collect all reward screens to replenish coins
- After END LEVEL → map shows a coin reward banner — always tap COLLECT before retrying

### End of level (draw pile exhausted)
- When draw pile runs out, "END LEVEL" button appears
- "+5 cards" option — **cost not confirmed** (may be coins or real money). Until confirmed, treat as paid/never use. If confirmed coins-only: apply the N≥7 rule from SKILL.md.
- WILD card: **free to use** from your hand (costs 0 coins); buying more costs 3,600 coins — don't buy
- **WILD timing rule**: use WILD when N ≤ 6 accessible cards remain AND draw pile is exhausted. See SKILL.md "End-of-Pile Decision" for full framework.

---

## Gotchas & Observations

- The coin cost increases with level number — check before auto-playing high levels
- "NEXT BONUS" on the map screen is a countdown timer, not a button — don't tap it
- The "LIMITED" badge on the map is a time-limited event — may require special attention
- The "21/65" (or similar) counter at the top is a scene progress or collectible counter — not hearts/lives
- Scene progress (e.g., 0/10) advances as levels are completed; completing a scene (10/10) unlocks the next scene
- If hearts/lives run out, wait 60 minutes for the coins bonus to respawn/regenerate — report to user and stop playing
- STREAK BONUS has 5 positions (●●●●● = full bonus); after a full streak it resets to ○●○○○ on the next play (not ○○○○○)
- WILD card costs 3,000–3,600 coins to buy (observed both values across sessions) — never buy it; only use the one granted for free
- **WILD usage rule**: Only use free WILD when ≤2–3 cards remain on the board AND draw pile is exhausted. With 6+ cards remaining, WILD saves only 1 card and rarely changes the outcome.
- Closing a "Try Again" dialog after a level can trigger auto-collection of a large pending reward (observed: coins jumped 3,983 → 33,733)
- OCR reliably detects: 2, 3, 4, 5, 6, 7, 8, A. OCR unreliable for: 9, J, Q, K (stylized fonts not recognized). Use visual screenshots to read those values.

---

## Session 4 Observations (Level 58 resumed + new level, 2026-03-04)

- Fan layout outer-tip rule confirmed: **sort accessible cards by x% — leftmost and rightmost are the accessible outer tips**. Tapping inner cards (not outer tips) silently fails.
- Playing 2♦ (outer right tip) revealed two new cards (4♦, 10♣) behind it — confirmed multi-card reveal after outer tip removal
- Level fail screen: shows "LEVEL ##" title + green "TRY AGAIN" button + red X close button (top-right of dialog) + cost shown at bottom
  - "Try Again" for Level 58 costs 1,500 coins (same as entry). Red X at ~(0.74, 0.19) closes dialog without spending coins.
- New level (5-stack fan layout): cards arranged as 5 vertical fan stacks each with one face-up center card; base card at bottom center. Observed: J♠, J♥, 2♠, 3♠, 6♥ as accessible center cards. **Level became unresponsive after loading** — possible game freeze or invisible intro overlay requiring manual dismiss.
- Draw pile position may differ between level layouts — confirm by visual inspection each level start

---

## Session 3 Observations (Level 58, 2026-03-03)

- Level 58 cost: 1,500 coins (confirmed: 5,483 → 3,983 on PLAY tap)
- Fan layout confirmed with purple frame / special slot in center
- STREAK BONUS resets to ○●○○○ (not ○○○○○) after a full 5-play streak
- The "21/65" style counter at top is a scene collectible/progress counter, not hearts

---

## TODO (things to learn in future sessions)

- [ ] Confirm whether Ace wraps (can A→K or only A→2)
- [x] What happens when scene is completed (0/10 → 10/10)? → Unlocks the next scene
- [x] Is there a way to play the Row 1 (top) cards directly? → No — face-down/blocked cards cannot be played; lower cards must be cleared first to reveal them
- [x] What happens when hearts/lives run out? → Wait 60 minutes for the coins bonus to respawn/regenerate
- [ ] Confirm exact coordinates for the level complete / reward screen buttons
