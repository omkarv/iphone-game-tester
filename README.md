# iphone-mirror-skills

A collection of Claude Code skills for automated iPhone testing and game automation via [iPhone Mirroring](https://support.apple.com/en-us/120421).

## Requirements

- [Claude Code](https://claude.ai/claude-code) (CLI)
- [iphone-mirror-mcp](https://github.com/omkarv/iphone-mirror-mcp) — MCP server providing screenshot, OCR, tap, swipe, and app-launch tools
- iPhone Mirroring running on macOS (macOS Sequoia 15+, iPhone with iOS 18+)
- Optional: [filesystem MCP](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) for persistent game knowledge across sessions

## Skills

### `iphone-game-tester` ([SKILL.md](./SKILL.md))

An autonomous iPhone game-testing agent. Give it a game and a goal — it observes the screen, makes decisions, acts, and builds a persistent knowledge base that gets smarter each session.

**Capabilities:**
- Plays Golf Solitaire variants (e.g., Solitaire Grand Harvest) end-to-end
- Detects screen states via OCR (map, level start, in-level, reward screens, popups)
- Applies card-play strategy: prefer plays that unblock the most cards, manage streaks
- Handles ads, popups, and unexpected states gracefully
- Saves per-game knowledge to `~/Documents/iphone-game-tester/<game>.md` and loads it on startup

**Example prompts:**
```
Run the game loop for Solitaire Grand Harvest
Auto-play and collect all available rewards
Keep grinding until I run out of coins
```

### Game Knowledge Files (`games/`)

Persistent knowledge files captured from real sessions. Each file records known screen states, confirmed tap coordinates, strategies, and gotchas for a specific game.

| File | Game |
|---|---|
| [DISNEY_SOLITAIRE.md](./games/DISNEY_SOLITAIRE.md) | Solitaire Grand Harvest |

## Installation

1. Clone this repo somewhere on your machine:
   ```bash
   git clone https://github.com/omkarv/iphone-mirror-skills ~/Documents/iphone-mirror-skills
   ```

2. Add the skills directory to Claude Code's config (`~/.claude/settings.json` or via `/config`):
   ```json
   {
     "skillsDirectory": "~/Documents/iphone-mirror-skills"
   }
   ```

3. Install [iphone-mirror-mcp](https://github.com/omkarv/iphone-mirror-mcp) and add it to your MCP config.

4. Invoke a skill in Claude Code:
   ```
   /iphone-game-tester
   ```

## How It Works

Each `.md` file in the root is a **Claude Code skill** — a prompt template invoked with a slash command. Skills in this repo use the iphone-mirror MCP tools (`screenshot`, `tap`, `ocr`, `open_app`, `swipe`) to observe and interact with your iPhone screen in real time.

Game knowledge files (`games/*.md`) are separate from skills — they are read at session start and written at session end, acting as a memory layer so the agent improves over time.

## Contributing

Skills and knowledge files are plain markdown. To contribute a new game:
1. Run the `iphone-game-tester` skill on a new game
2. The agent will create a knowledge file under `~/Documents/iphone-game-tester/`
3. Copy that file into `games/` and open a PR
