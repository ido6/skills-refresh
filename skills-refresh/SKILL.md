---
name: skills-refresh
description: Refresh my Claude Code setup - fix stale Claude model references, find dead skills, fix dangling references, and download or update Higgsfield skills. Pass "scheduled" for the unattended run.
disable-model-invocation: true
---

# Skills refresh

One pass that keeps `~/.claude` current. Every run hunts three kinds of problem and refreshes the Higgsfield skills:

- **stale**: names an old Claude model or uses an API pattern the current models reject.
- **dead**: a skill nobody needs any more.
- **dangling**: points at a skill or file that no longer exists.

## Mode

- **Interactive** (default, a person is chatting): ask before removing a skill, installing a new one, or deleting a standing rule.
- **Scheduled** (argument `scheduled`, or no person to answer): apply only the **safe changes** listed below, and put everything that needs a decision into the report as a proposal. Ask nothing; end with the report.

**Safe changes** (allowed in both modes, each file backed up first):

1. Rewriting stale model names.
2. Updating an already-installed `higgsfield-*` skill to Higgsfield's newer version.
3. Repointing a dangling reference to a file that still exists.

Everything else waits for a person: removing a skill, installing a new skill, and editing or deleting a rule in `CLAUDE.md` or `AGENTS.md`.

## Scope and backups

- **Scope:** `~/.claude/skills`, `~/.claude/agents`, `~/.claude/commands`, `~/.claude/rules`, `~/.claude/CLAUDE.md`, and `~/AGENTS.md` if present.
- **Plugin-managed copies stay read-only:** `~/.claude/plugins/` and `~/.claude/skills/synced/`. Their plugin overwrites them on sync, so list stale hits there in the report instead of editing them.
- **State folder:** `~/.claude/skills-refresh/` (create it if missing). It holds:
  - `state.json`: last run date, installed Higgsfield workflow versions, and proposals I declined, so they aren't proposed again.
  - `backups/<YYYY-MM-DD>/`: the original of every file you change, at its path relative to `~/.claude` (or `home/` for `~/AGENTS.md`).
  - `reports/<YYYY-MM-DD>.md`: each run's report.
- **Removed skills** move to `~/.claude/skills-removed-backup/<name>/`, never `rm`.

## Steps

### 1. Learn the current model lineup

Load the `claude-api` skill and read its **Current Models** table. That table is the source of truth, because the lineup changes between runs. Done when you hold every current family (Fable, Opus, Sonnet, Haiku) with its display name, model ID, and one-line positioning.

Fallback only if `claude-api` is unavailable (as of Oct 2026, most capable first):

| Model | ID | Positioning |
|---|---|---|
| Fable 5.1 | `claude-fable-5-1` | most capable, highest cost |
| Opus 5.5 | `claude-opus-5-5` | default, deep reasoning |
| Sonnet 5.5 | `claude-sonnet-5-5` | fast everyday coding |
| Haiku 5.5 | `claude-haiku-5-5` | cheapest, lightweight and bulk work |

### 2. Fix stale model references

Grep the scope, case-insensitive, for any Claude model name or ID older than the lineup from step 1. That covers `claude-3*`, `claude-instant`, `Opus|Sonnet|Haiku` followed by an older version number, a bare `Fable 5` or `Opus 5`, and dated IDs such as `claude-sonnet-4-20250514`. Skip codec hits like `opus_48000_*`. Done when every hit is either rewritten or listed as deliberately kept, with a reason.

How to rewrite each kind of hit:

- **Tier advice or model-selection tables:** rewrite for the current lineup. Include the top tier where tiers are listed.
- **Example IDs in code:** use the current ID of the same family.
- **Version-pinned headings:** drop the version.
- **Mentions meant to cover future versions:** write "Fable (5.1 or later)".
- **Translated copies** (for example `rules/**/zh/`): same change, in that language.

Keep these as they are:

- A third-party product's own list of supported models (for example ElevenLabs).
- Non-Claude models.
- Wording that is still true, such as "Opus 4.6+".

### 3. Fix stale API patterns

In Claude API code and docs, grep for these patterns:

- `budget_tokens`
- `thinking: {type: "enabled"`
- assistant prefill
- forced `tool_choice` `any` or `tool`
- `web_search_20250305`
- `web_fetch_20250910`
- top-level `output_format` on `messages.create`

The `claude-api` skill gives the current replacement for each. A parameter with the same name in another vendor's API (for example ElevenLabs `output_format`) is not a hit. Done when every hit is fixed or listed with a reason.

These fixes are not on the safe list: in scheduled mode, propose them in the report.

### 4. Find dead skills

Candidates:

- skills marked DEPRECATED or superseded;
- empty folders;
- folders with no `SKILL.md` and nothing pointing at them;
- stub skills that only call another skill;
- local duplicates of a skill a plugin already provides.

Before calling a folder dead, grep `commands/`, `hooks/` and `settings.json` for its path. An empty folder can still be a command's save target (`/learn` writes to `skills/learned/`). Skip anything `state.json` records as declined.

In interactive mode, show the candidates, recommended ones first with a one-line reason each, and move the ones I pick to the removed-skills backup. In scheduled mode, list them in the report.

Done when every folder in `~/.claude/skills` has been judged.

### 5. Fix dangling references

Grep the whole scope for the name of every skill in `~/.claude/skills-removed-backup/`, and for skill names or `references/` paths that resolve to nothing. Handle each hit:

- **Path to a file that now lives elsewhere** (inside a kept skill or the backup): copy the file into the kept skill's `references/` folder and repoint the path.
- **Standing rule** ("always load skill X") in `CLAUDE.md` or `AGENTS.md`: ask me in interactive mode; propose it in scheduled mode.
- **Any other mention:** replace it or remove it, keeping the edit minimal.

Done when the grep comes back empty outside plugin-managed folders, or every remaining hit is a listed proposal.

### 6. Refresh Higgsfield skills

Needs the Higgsfield connector, whose tools include `get_workflow_instructions` and `get_workflow_bundle_file`. If it isn't connected, say so in the report and skip this step.

1. Call `get_workflow_instructions` with no argument to get the catalog: each workflow's name, description and version.
2. **Installed** `higgsfield-<name>` skills: compare the catalog version and content with the local copy.
   - Newer or different: back up the old copy, then overwrite.
   - Overwriting means: load the workflow with `get_workflow_instructions`, save its SKILL.md as `~/.claude/skills/higgsfield-<name>/SKILL.md`, and save every listed bundle file (fetched with `get_workflow_bundle_file`) at the same relative path.
   - Keep valid skill frontmatter. Add one line at the top recording the source version and today's date.
   - Record the version in `state.json`.
3. **New** workflows (in the catalog, not installed, not declined): in interactive mode, list them with a one-line description each and install the ones I pick. In scheduled mode, propose them in the report.
4. **Acting skill:** Higgsfield's catalog had no acting (character performance) skill as of Oct 2026. Check whether one has appeared. Outside the catalog, install only a skill clearly published by Higgsfield itself. Third-party registries and random repos are report-only, since an installed skill runs as instructions in every session.
5. Never call `sync_agents`. It uploads my skills to Higgsfield; it doesn't download theirs.

Done when every catalog workflow is installed and current, proposed, or recorded as declined.

### 7. Verify

Re-run the greps from steps 2, 3 and 5. Confirm that every `higgsfield-*` folder and every skill you edited still has a `SKILL.md` with valid frontmatter (`name`, `description`). Done when the greps return only listed exceptions and every frontmatter check passes.

### 8. Report

Write `reports/<YYYY-MM-DD>.md` and update `state.json`. Then reply with a short summary in plain language:

1. **Changed:** each file and what changed, with the backup location.
2. **Removed or installed.**
3. **Waiting for your decision:** proposals, each with a one-line reason.
4. **Left alone on purpose:** each item and why.

In scheduled mode, start the summary with how many proposals are waiting, so I know whether to open the report.
