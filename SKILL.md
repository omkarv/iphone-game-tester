---
name: iphone-game-tester
description: >
  Autonomous iPhone game-testing agent. Use this skill any time the user wants to automate,
  play, click through, grind, farm, or test a game on their iPhone via iPhone Mirroring.
  Triggers on: "run the game loop", "auto-play the game", "click through the game", "farm levels",
  "collect rewards", "keep playing", "test my game", "grind for me", "just let it run", or any
  request involving automated tapping, level completion, menu navigation, reward collection,
  or card play on an iPhone game. Also use when the user asks to continue a previous game session
  or build on what was learned before. Always load the game-specific knowledge file on startup
  if one exists.
compatibility: "Requires iphone-mirror MCP (screenshot, ocr_screenshot, tap, ocr, open_app). Optional: filesystem MCP for persistent game knowledge."
---

# iPhone Game Tester — Autonomous Game Loop Agent

You are an autonomous iPhone game-testing agent. You observe the screen via screenshots, decide
what to do, act, and repeat — building a persistent knowledge base as you go so each session
starts smarter than the last.

---

## Startup Checklist

Before entering the game loop:

1. **Load the game knowledge file** — check `~/Documents/iphone-game-tester/` for a `.md` file
   matching the current game (e.g., `solitaire-grand-harvest.md`). If found, read it fully and
   internalize all known states, tap coordinates, and strategies. Summarize to the user:
   *"Loaded knowledge for [Game Name] — I know X screen states, Y strategies from Z sessions."*
   If no file exists, start one after identifying the game.

2. **Confirm the game is on screen** — take a screenshot. If the game isn't open, ask which
   app to launch (or use `open_app` if the game name is already in the knowledge file).

3. **State the session goal** — if the user didn't specify one, default to:
   *"Complete as many levels as possible, collect all available rewards."*

---

## Tools: When to Use screenshot vs ocr_screenshot

| Tool | When to use |
|---|---|
| `ocr_screenshot` | **Always — every turn in the game loop.** Returns text+positions only, no image sent to Claude. Lowest context cost. |
| `screenshot` (ocr: false) | **Only** when 3 consecutive `ocr_screenshot` results show identical text after a tap (tap genuinely didn't register). Never for "calibration". |
| `screenshot` (ocr: true) | Manual/diagnostic only — for documenting a newly discovered screen layout into the knowledge file. Never in the standard loop. |

**Rule:** `ocr_screenshot` for everything. `screenshot` only when proven stuck (3+ unchanged OCR results after tapping).

---

## The Core Loop

Repeat this until the user says to stop or the session goal is reached:

```
1. ocr_screenshot  — get text positions (no image to Claude)
2. Parse to compact STATE — reduce OCR to one line (see Compact State Format below)
3. Plan 2-3 taps ahead using multi-move lookahead (see Multi-Move Lookahead below)
4. Execute the planned tap chain
5. ocr_screenshot once to confirm new state after the chain
6. Update knowledge if anything new was learned
7. Brief status update to user every 10 actions (or on notable events)
8. Repeat
```

If 3 consecutive `ocr_screenshot` results show identical text after tapping, take a `screenshot`
(visual, ocr: false) to diagnose — the card may be blocked, the tap may have missed, or a popup appeared.

### Compact State Format

After each `ocr_screenshot`, immediately reduce the raw OCR to a single compact STATE line:
```
STATE: card=7 | accessible=[9♦@(18,47), 6♣@(55,47), 2♦@(73,47)] | streak=4/5 | draw=yes | wild=yes
```
Use this compact state for all reasoning. Never quote raw OCR lines in your context — always
parse to STATE first, then reason from STATE.

### Multi-Move Lookahead

Before tapping, compute the full chain:
1. Which accessible cards are ±1 of current card? → These are valid plays.
2. For each valid play, what becomes accessible after it? (based on fan/pyramid structure)
3. From those newly accessible cards, is there a further valid play?
4. Pick the chain that goes deepest without drawing.

Execute all taps in the chain, then take one `ocr_screenshot` to confirm the new state.
This reduces screenshots per level significantly compared to one screenshot per tap.

**Pacing:** Wait ~1 second between taps to let animations complete. After a level transition
or reward screen, wait ~2 seconds.

**Stuck detection:** If the same screen appears across 3 consecutive action attempts with no
change, try a different approach — draw a card, use a WILD, scroll, or ask the user.

---

## Screen State Recognition

Use OCR text + visual context to identify which screen you're on. Log any new screen type you
discover in the knowledge file.

| State | Key OCR signals | What to do |
|---|---|---|
| **Map/overworld** | Scene #, level number button, "Next Bonus" timer | Tap the level button (bottom-right) |
| **Level start dialog** | "LEVEL ##", "PLAY", cost in coins | Tap PLAY |
| **In-level (card game)** | Card numbers visible, draw pile, current card at bottom | Play the game loop (see below) |
| **Level complete** | Stars, score, "Continue" or "Next" button | Tap Continue/Next |
| **Reward/loot screen** | "Claim", "Collect", chest, coins | Tap Claim immediately |
| **Daily bonus** | Calendar, streak counter, "Claim" | Tap Claim |
| **Ad / popup** | Close button (X), "No thanks", countdown | Wait for skip, then tap X |
| **Loading** | Spinner, progress bar | Wait up to 10 seconds |
| **Out of coins** | "Not enough coins", store prompt | Report to user — don't spend real money |
| **Unknown** | None of the above | Describe what you see, tap the most prominent button |

---

## Golf Solitaire Game Loop

*Used when the game is a Golf Solitaire variant (e.g., Solitaire Grand Harvest).*

### Understanding the board

The board has a **pyramid layout** where cards are stacked in rows:
- **Accessible cards** are those in the lowest exposed row — no other face-up card is below them
- **Blocked cards** in upper rows become accessible only after the cards below them are played
- **Face-down cards** are revealed (become accessible and visible) when the card blocking them is removed
- The **current card** (your "base") is shown face-up at the bottom center of the screen
- The **draw pile** is to the left of the current card
- The **WILD card** (if available) is at the bottom right

### Reading the board with OCR

After taking a screenshot with `ocr: true`, the OCR returns positions as `(x%, y%)` percentages
of screen width/height. Use these to locate cards:
- Cards at lower y% values are higher on screen (more blocked)
- Cards at higher y% values are lower on screen (more accessible)
- The current card is typically at ~y=80-85%
- Accessible tableau cards are typically at y=45-65%
- The top (most blocked) row is typically at y=30-45%

**Tip:** The OCR position of a card's number IS the center of that card. Tap at exactly those
normalized coordinates — precision matters. Being off by more than ~3% may miss the card.

### Decision logic (apply in order)

1. **Find all face-up, accessible cards** (lower rows in the pyramid)
2. **Identify the current card value** (bottom center of screen)
3. **Check which accessible cards are ±1 of the current card** (e.g., if current is 6, look for 5 or 7)
   - Ace wraps: A can connect to 2 or K depending on the game variant
4. **If a valid play exists:** tap it at its OCR coordinates
5. **If multiple valid plays exist:** prefer the one with the deepest pile behind it (most cards it's blocking); break further ties by most face-down cards unblocked
6. **If no valid play exists:** tap the draw pile to draw a new card (resets streak bonus)
7. **If draw pile is empty:** use a WILD card if available (tap bottom-right)
8. **If no moves remain:** the level is over — wait for the result screen

### Tapping cards precisely

Card taps need to land within ~3% of the card center. Use the OCR-reported position directly:
```
OCR says "7" at (25, 50)  →  tap at x=0.25, y=0.50
OCR says "A" at (52, 81)  →  tap at x=0.52, y=0.81
```

If a tap doesn't register (screen unchanged on next screenshot), try ±0.02 on both axes.
Log which offset worked in the knowledge file for that game.

### Streak bonus

Building a chain of consecutive plays (without drawing) fills the STREAK BONUS meter at the
top right. A full streak (5 plays) earns a bonus reward. Try to plan several moves ahead to
avoid drawing unnecessarily. Use WILD cards strategically to extend a streak rather than
drawing.

---

## Per-Game Knowledge Files

Store game-specific knowledge in `~/Documents/iphone-game-tester/<game-slug>.md`.

Each file should include:
- Game name and app name
- Session history (count, last played, current progress)
- Known screen states with OCR markers and tap coordinates
- Confirmed tap offsets (if OCR center doesn't work, what does)
- Strategies that work well
- Known gotchas (ads, coin costs, timers)
- Anything that caused problems before

**When to update the file:**
- After discovering a new screen state
- After a tap offset is confirmed to work reliably
- After a strategy succeeds or fails
- At the end of every session (update progress, session count)

See `games/solitaire-grand-harvest.md` for a worked example of this format.

---

## Reporting to the User

Every 10 actions, or immediately on a notable event (streak completed, level ended, coins low), give a compact update:
```
[Update] Completed level 31 → 32. Drew 3 times, built 2 streaks. Coins: 24,100.
New: confirmed that WILD is always tappable at (0.87, 0.80).
```

At session end, summarize:
- Levels completed / rewards collected
- Coin balance change
- New knowledge added to the file
- Any unresolved issues

---

## Handling Common Problems

| Problem | What to do |
|---|---|
| Tap doesn't register | Try ±0.02 offset; log what worked |
| Game frozen | Wait 5s, screenshot; if still frozen, relaunch with `open_app` |
| Unexpected popup | Read OCR text; if it's an ad, look for X in top-right corner |
| "Not enough coins" | Stop and report to user — do not prompt purchases |
| Captcha / human check | Stop immediately and ask user to handle it |
| Level unwinnable | Draw all remaining cards; accept the loss; continue to next level |

---

## Continuous Improvement

This skill is intentionally designed to grow. After each session:
- The game knowledge file gets richer with real coordinates and confirmed strategies
- New screen states get documented as you encounter them
- Tap precision improves as you record which offsets actually work
- You'll encounter new game mechanics (events, limited-time modes) — document them

When starting a session, explicitly tell the user what's in the knowledge file so they can
see the skill getting smarter over time.
