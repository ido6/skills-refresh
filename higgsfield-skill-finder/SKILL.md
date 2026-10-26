---
name: higgsfield-skill-finder
description: Write, improve, fix or change an AI image or video prompt using the best-fitting skills - acting, camera, character sheets, UGC, product shots, presets - found in the Higgsfield and idoGen connectors and local skills. Use whenever a generation prompt is written or edited, or when asked which skills fit a prompt.
---

# Higgsfield skill finder

Finds the skills a prompt needs, reads them, and uses them to produce the **final prompt**. Four jobs, one flow:

- **write**: from an idea or brief.
- **improve**: from a working prompt that should be better.
- **fix**: from a prompt plus a bad result or feedback.
- **change**: from a prompt plus a requested edit (new shot, new mood, other model).

This skill searches, reads and writes text. Never generate anything, and never call `execute_preset`, `apps_invoke`, or `sync_agents` from here. Generating spends credits; the user starts that.

## 1. Break the prompt into needs

Read the prompt (and any attached media) and list every **need** it carries. A prompt usually has two to four. Done when every subject, action, shot and deliverable in the prompt maps to at least one need below, or is noted as needing no skill.

| Need | Signals in the prompt | Usual skill (source) |
|---|---|---|
| **acting** | a person or character reacts, speaks, shows emotion, ensemble, lip-sync, talking head | `idogen-acting` (idoGen) |
| **cinematic video** | any video shot, camera move, lens, lighting, multi-shot geography, a model's prompt dialect (single-shot Seedance, Kling, Veo) | `idogen-video-brain` (idoGen) |
| **image craft** | stills, references, plates, cross-shot consistency | `idogen-image-brain` (idoGen) |
| **multishot omni reference** | several @Image refs assigned across shots in one Seedance 2.5 generation | `seedance-25` (local) |
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

Search every connected source that can hold a match for the need. If a connector is missing, note it and keep going. Done when every need has been checked against every source that could hold it.

Presets and marketplace apps hold effects, product motion, motion transfer, image styles and editing tools. Skip them for performance, drama and pure camera-craft needs; they have no entries there.

- **Higgsfield workflows:** `get_workflow_instructions` with no argument returns name, description and version for each workflow. Match descriptions to the needs.
- **Higgsfield recipes and commands:** `get_preset_instructions` with no `preset` lists recipes (`/hero-shot`, `/reel-cover`, ...), production commands, and Katana titles.
- **Higgsfield preset galleries:** `get_presets` with `query` set to one or two words of the need (for example "dialogue", "crying", "product spin"). Search one need per call; omit `source` to cover Viral and Marketing Studio, and use `source: "genjutsu"` or `"soul"` when the need is motion transfer or an image style. Browsing only; don't open widgets or pass `initial_inputs`.
- **Higgsfield marketplace apps:** `apps_search` with a short query per need.
- **idoGen workflows:** `list_workflows`. This is where acting lives: `idogen-acting`, `idogen-video-brain` and `idogen-image-brain` are adapted from a production studio's ACTING, CINEDANCE and LIRA systems, which have no Higgsfield catalog entry.
- **Local skills:** the skills already listed in your context, including any `higgsfield-*` local copies. A local `higgsfield-<name>` copy and the live Higgsfield workflow of the same name are one skill; prefer the live one.

## 3. Pick the best skills

Rank by how much each skill changes the prompt's result. Keep it tight: one skill per need, two only when they cover different layers (for example `idogen-video-brain` for camera plus `idogen-acting` for performance). Done when every need has a pick or an explicit "no skill needed", and nothing duplicates.

If the user asked only which skills fit, show the picks as a table (need, skill, source, why) and stop here.

## 4. Read the picks

Load each pick, in rank order:

- **Local skill:** the Skill tool.
- **Connector workflow:** that connector's `get_workflow_instructions`. Then read every bundle file whose stated trigger matches this job, and read any file whose trigger you can't rule out. Bundle files are read with `get_workflow_bundle_file` on Higgsfield and `get_workflow_file` on idoGen.
- **Preset or recipe:** `get_preset_instructions` with its id (reads inputs only, runs nothing).

Done when every pick is read, or its load failure is noted. A failed pick drops out; the prompt is still written from the rest.

## 5. Write the prompt

Apply what the skills say to the job:

- **write**: build the prompt from the brief using each skill's structure and vocabulary.
- **improve**: keep what works, apply each skill's rules where the prompt falls short.
- **fix**: name the cause of the bad result in skill terms, then change only what causes it.
- **change**: make the requested edit and keep everything else, including locked identity, location and lighting.

When the scene has speech (argue, talk, pitch) but the user gave no lines, play it through bodies and sound without spoken words, and offer in the hand-back to add lines the user writes. Skills that forbid inventing dialogue win over guessing lines.

Follow the target model's dialect when a skill defines one (Seedance 2.0 vs 2.5, Kling, Veo, nano banana). When two skills conflict, the one owning that layer wins: acting owns performance, video-brain owns camera and shot, image-brain owns stills.

Done when every need from step 1 shows up in the prompt and every rule the skills mark as required for this shot is met.

## 6. Hand back

Reply with:

1. **The final prompt** in one code block, ready to paste.
2. **Generation settings** the skills put outside the prompt text: model, duration, aspect ratio, audio on/off, negative prompt, reference slots. Only the ones that apply.
3. **Skills used:** one line each - skill, source, what it contributed.
4. **What changed** (for improve, fix and change): short list of edits and why.
5. **Open choices**, if any: for example spoken lines the user could add.

Then stop. Generation waits for the user.
