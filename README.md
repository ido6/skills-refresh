# skills-refresh

Two Claude Code skills:

- **`skills-refresh`** keeps your `~/.claude` setup current.
- **`higgsfield-skill-finder`** finds the best skills for a generation prompt (acting, camera, character sheets, UGC, product shots, presets) and uses them to write, improve, fix or change the prompt.

## skills-refresh

Each run:

- **Stale references:** rewrites old Claude model names and old Claude API patterns in your skills, agents, rules and `CLAUDE.md`. It reads the current model lineup from the bundled `claude-api` skill on every run.
- **Dead skills:** finds deprecated, empty, orphaned, stub and duplicate skills, and moves the ones you pick to a backup folder.
- **Dangling references:** fixes mentions of skills or files that no longer exist.
- **Higgsfield skills:** downloads Higgsfield's skill workflows through the Higgsfield connector, and keeps the installed ones up to date.

Every changed file is backed up first, and each run writes a report.

## Install

Copy the skill folder into your Claude Code skills directory:

```bash
git clone https://github.com/ido6/skills-refresh.git
mkdir -p ~/.claude/skills
cp -r skills-refresh/skills-refresh skills-refresh/higgsfield-skill-finder ~/.claude/skills/
```

## Use

- `/skills-refresh` runs it interactively. It asks before removing a skill, installing a skill, or changing a rule in `CLAUDE.md` or `AGENTS.md`.
- `/skills-refresh scheduled` runs it unattended. It makes only these safe changes:
  - replaces old model names;
  - updates Higgsfield skills you already installed;
  - repoints references to files that still exist.

  Everything else goes into the report as a proposal.

To run it every two weeks in the Claude desktop app, create a scheduled task with the cron expression `0 10 1,15 * *` (10:00 on the 1st and 15th of each month) and this prompt: `Read ~/.claude/skills/skills-refresh/SKILL.md and follow it in scheduled mode.` The skill can only be loaded by a typed `/skills-refresh`, so a scheduled run has to read the file directly.

## Files it writes

- `~/.claude/skills-refresh/state.json`: last run, installed Higgsfield versions, proposals you declined
- `~/.claude/skills-refresh/backups/<date>/`: originals of every changed file
- `~/.claude/skills-refresh/reports/<date>.md`: the report for each run
- `~/.claude/skills-removed-backup/<name>/`: skills you chose to remove

## higgsfield-skill-finder

Claude loads it on its own whenever an AI image or video prompt is written or edited. You can also ask "which skills fit this prompt?".

1. It splits the prompt into needs, such as acting, camera work, character sheet, UGC or product stills.
2. It searches every connected source that could hold a match:
   - Higgsfield workflows, recipes, preset galleries and marketplace apps;
   - idoGen workflows;
   - your local skills.
3. It picks the best skill for each need and reads it, including the reference files that apply.
4. It uses those skills to write, improve, fix or change the prompt.

It returns the final prompt, ready to paste, plus the generation settings (model, duration, audio, negative prompt), which skills it used, and what changed. It never generates anything, so it spends no credits.

Install:

```bash
cp -r higgsfield-skill-finder ~/.claude/skills/
```

## License

MIT
