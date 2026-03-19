# Claude Skills

A collection of custom skills for [Claude Code](https://claude.ai/claude-code) — drop-in workflows that give Claude specialized capabilities beyond its defaults.

## What are Claude Skills?

Skills are instruction packages that teach Claude a complete, repeatable workflow. When you install a skill, Claude automatically uses it whenever your request matches the skill's purpose — no manual prompting needed.

## How to install a skill

1. Download the `.skill` file for the skill you want
2. Drag it into the Claude Code window (or your Claude interface)
3. Claude will confirm the skill is installed
4. From then on, Claude will use it automatically when relevant

---

## Skills in this repo

### 📊 [portfolio-cio](./portfolio-cio/)

**CIO-grade portfolio analysis** — Give Claude any position statement (PDF, screenshot, or pasted list of holdings) and it produces a full investment committee briefing:

- ETF look-through across all holdings (e.g. NVDA exposure across QQQ + SMH + SOXX)
- Macro risk sensitivity matrix with quantitative $ estimates per scenario
- Polymarket implied probabilities alongside each macro scenario
- Stress tests across 10+ named scenarios
- Benchmarked against Goldman Sachs, JPMorgan, BlackRock public views
- Reddit / Seeking Alpha sentiment + FT/WSJ headlines
- Specific buy/sell/trim recommendations with share counts
- Output: self-contained dark-theme HTML dashboard, one dated section per upload

**Install:** Download [`portfolio-cio.skill`](./portfolio-cio/portfolio-cio.skill) and drag into Claude Code.

---

## License

MIT — free to use, modify, and share.
