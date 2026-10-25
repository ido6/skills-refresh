---
name: higgsfield-skill-finder
description: Find and load the skills a generation prompt needs - acting, camera, character sheets, UGC, product shots, presets - from the Higgsfield and idoGen connectors and local skills. Use before writing or fixing an AI image or video prompt, or when asked which skills fit a prompt.
---

# Higgsfield skill finder

Turns one prompt into the short list of skills it needs, then loads them, so the prompt gets written with the right expertise. The output is a **shortlist**: each entry names a skill, its source, why it fits, and the exact call that loads it.

This skill only searches and reads. Never generate anything, and never call `execute_preset`, `apps_invoke`, or `sync_agents` from here. Loading instructions is free; generating spends credits.

## 1. Break the prompt into needs

Read the prompt (and any attached media) and list every **need** it carries. A prompt usually has two to four. Done when every subject, action, shot and deliverable in the prompt maps to at least one need below, or is noted as needing no skill.

| Need | Signals in the prompt | Usual skill (source) |
|---|---|---|
| **acting** | a person or character reacts, speaks, shows emotion, ensemble, lip-sync, talking head | `idogen-acting` (idoGen) |
| **cinematic video** | any video shot, camera move, lens, lighting, multi-shot geography | `idogen-video-brain` (idoGen) |
| **image craft** | stills, references, plates, cross-shot consistency | `idogen-image-brain` (idoGen) |
| **multishot omni reference** | several @Image refs across shots, Seedance 2.5 sequence | `seedance-25` (local) |
| **previz / blocking** | Blender, blockout, camera plan before paid video, multi-cam dialogue | `blender-to-seedance` (local), `idogen-auteur-previz` (idoGen) |
| **character sheet** | turnaround, model sheet, expression sheet, consistent character | `character-sheet` (Higgsfield), `idogen-character` (idoGen) |
| **UGC** | creator review, unboxing, try-on, tutorial, website tour | `ugc-video` (Higgsfield) |
| **product stills** | packshot, lifestyle, hero banner, catalog | `product-photoshoot` (Higgsfield), `idogen-product-studio` (idoGen) |
| **ads** | ad creatives, Meta ads, ad variants | `ads-studio`, `ad-multiplier` (Higgsfield) |
| **thumbnail** | YouTube/Instagram cover | `thumbnail-generation` (Higgsfield) |
| **narration / faceless** | narrator, explainer, story channel | `narration`, `faceless-video` (Higgsfield) |
| **edit / montage** | cut existing clips, captions, soundtrack, motion graphics | `video-montage`, `video-editing` (Higgsfield) |
| **remix / effect** | a named effect, a reference edit to copy, Halloween-style transformations | Higgsfield presets and Katana (step 2) |
| **whole video** | "make me a video", multi-shot commercial or narrative | `idogen-video-producer` (idoGen) |

The table is a starting map, not the truth: the live catalogs in step 2 win, and they may hold a better match the table doesn't know.

## 2. Search the live catalogs

Search every source that is connected. If a connector is missing, note it in the shortlist and keep going. Done when every need has been checked against every connected source.

- **Higgsfield workflows:** `get_workflow_instructions` with no argument returns name, description and version for each workflow. Match descriptions to the needs.
- **Higgsfield recipes and commands:** `get_preset_instructions` with no `preset` lists recipes (`/hero-shot`, `/reel-cover`, ...), production commands, and Katana titles.
- **Higgsfield preset galleries:** `get_presets` with `query` set to one or two words of the need (for example "dialogue", "crying", "product spin"). Search one need per call; omit `source` to cover Viral and Marketing Studio, and use `source: "genjutsu"` or `"soul"` when the need is motion transfer or an image style. Browsing only; don't open widgets or pass `initial_inputs`.
- **Higgsfield marketplace apps:** `apps_search` with a short query per need.
- **idoGen workflows:** `list_workflows`. This is where acting lives: `idogen-acting`, `idogen-video-brain` and `idogen-image-brain` are adapted from a production studio's ACTING, CINEDANCE and LIRA systems, which have no Higgsfield catalog entry.
- **Local skills:** the skills already listed in your context, including any `higgsfield-*` local copies. A local `higgsfield-<name>` copy and the live Higgsfield workflow of the same name are one skill; prefer the live one.

## 3. Build the shortlist

Rank by how much each skill changes the prompt's result. Keep it tight: one skill per need, two only when they cover different layers (for example `idogen-video-brain` for camera plus `idogen-acting` for performance). Done when every need has a pick or an explicit "no skill needed", and nothing on the list duplicates another entry.

Write it as a table:

| Need | Skill | Source | Why | Load with |
|---|---|---|---|---|
| acting | `idogen-acting` | idoGen | two people argue on camera | `get_workflow_instructions({workflow: "idogen-acting"})` |

Add one line on any need with no good match, and on any connector that wasn't available.

## 4. Load and hand back

Load every shortlisted skill now, in shortlist order:

- **Local skill:** the Skill tool.
- **Connector workflow:** that connector's `get_workflow_instructions`, plus any bundle files it says to read.
- **Preset or recipe:** `get_preset_instructions` with its id. This reads its inputs only; it doesn't run it.

Then continue the original task with what they say. Done when every pick is loaded, or its load failure is reported, and the prompt work has resumed.

If the user asked only "which skills fit", show the shortlist and stop before loading.
