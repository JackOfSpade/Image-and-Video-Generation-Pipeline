# Midjourney V8.1 — AI-Consumption Prompt Reference Guide

> **File type:** Reference document for LLM ingestion. Optimized for machine parsing, not human reading. Use as system/context input when generating Midjourney V8.1 prompts for advanced/pro users.

---

## TL;DR

- **Midjourney V8.1 went stable on April 30, 2026** on `midjourney.com` and Discord (alpha launched April 14, 2026 on `alpha.midjourney.com`). It is Midjourney's fastest model: HD/2K is default, ~4–5× faster than earlier versions, and the V8.0 aesthetic has been rolled back to "the spirit of V7." V8.0 is being decommissioned.
- **Use this guide as ground truth for prompt construction.** Section `<SYNTAX>` defines exact parameter grammar; `<PARAMS>` is the canonical compatibility/range table; `<TEMPLATES>` provides parameterized prompt skeletons; `<RULES>` enumerates hard constraints; `<FAILURE_MODES>` lists known V8.1 weaknesses to avoid.
- **Major V8.1-specific differences vs. V7/V8.0:** HD (2048-px native) is default; `--iw` image weights are back; new auto Prompt Shortener; updated `Describe`; seeds are ~99% reproducible; moodboards/`--sref` are super-stable; `--no` negative prompts are **NOT compatible** with V8.1 (the API rejects jobs containing `--no` with `"--no is not compatible with --version 8.1"`) — use positive description or moodboards instead; `--oref`, `--cref`, and the V8 upscaler/editor are NOT YET available in V8.1 (Editor/Pan/Zoom still fall back to V6.1); a Global V7/V8 Personalization Profile MUST be unlocked before V8.1 will run.

---

## Key Findings

```yaml
model_id: midjourney_v8_1
status: stable_production
release_date_alpha: 2026-04-14            # alpha.midjourney.com
release_date_stable: 2026-04-30           # midjourney.com + Discord
predecessor: v8_0_alpha (2026-03-17)      # being decommissioned
ranking_data_url: midjourney.com/rank-v8-1
next_version_in_training: v8_2
default_model_on_main_site: V7 was default until V8.1 stable; V8.1 is now the recommended version, but V7 remains the documented "default" in some surfaces during transition
access_surfaces: [midjourney.com (web), Discord, alpha.midjourney.com]
parameter_flag: "--v 8.1"  # also selectable in Settings → Version
hard_prereq: "Global V7/V8 Personalization Profile must be unlocked"
native_resolution_hd: 2048x2048           # --hd default
native_resolution_sd: 1024x1024           # --sd toggle
gpu_cost_sd: "<1.0 GPU-minute / 4-image job"
gpu_cost_hd: "~1.33 GPU-minutes / 4-image job"
speed_vs_v7: "~4-5x faster on standard jobs; SD-quality V8.1 ≈ V7 draft mode speed"
seed_reproducibility: "~99% identical given same prompt+seed"
aesthetic_direction: "Consistent and familiar aesthetic in the spirit of V7"
default_sref_version: --sv 7   # introduced March 21, 2026, default in V8 series
```

---

## `<SYNTAX>` — Canonical Prompt Grammar

```
/imagine [IMAGE_URLS...] [TEXT_PROMPT] [--PARAMETERS...]
```

**Order rules (enforced):**
1. Image-prompt URL(s) (if any) come FIRST, space-separated, BEFORE any text.
2. Text prompt comes next. Use natural-language sentences, NOT keyword stacks.
3. ALL `--parameters` come LAST, after the text. Order among parameters does not matter.
4. Parameter names are lowercase, prefixed by two ASCII hyphens `--`.
5. Parameter values are separated from the name by ONE space (e.g. `--ar 16:9`, NOT `--ar=16:9`).
6. Aspect-ratio values must be integers separated by `:` — decimals are rejected (use `139:100`, not `1.39:1`).
7. Quoted literal text inside the prompt uses straight double quotes `"OPEN"` to instruct in-image typography.

**Minimal valid prompt:**
```
elderly fisherman on weathered dock at dawn --v 8.1 --ar 3:2
```

**Maximal pro prompt skeleton:**
```
<IMG_URL_1> <IMG_URL_2> [SUBJECT][SUBJECT DETAILS], [CONTEXT/ENV], [STYLE/MOOD], [CAMERA/LENS/LIGHTING], "[QUOTED_TEXT_IF_ANY]" --v 8.1 --ar 16:9 --s 250 --c 15 --w 0 --iw 1.25 --sref <URL_OR_CODE> --sw 120 --p <profileID> --hd --seed 12345
# NOTE: --no is NOT compatible with --v 8.1. Express exclusions via positive description, moodboard, or post-process.
```

---

## `<PARAMS>` — V8.1 Parameter Table (Authoritative)

| Parameter | Alias | Range / Values | Default | V8.1 Support | Notes |
|---|---|---|---|---|---|
| `--v` | `--version` | `8.1`, `8`, `7`, `6.1`, `6`, `5.2`, `5.1`, `5`, `4`, `3`, `2`, `1` | site-dependent | ✅ Use `--v 8.1` | Setting also available in Version section of Settings panel. |
| `--niji` | — | `7`, `6`, `5`, `4` | n/a | ❌ Mutually exclusive with `--v` | Use `--niji 7` for anime/manga; do NOT combine with `--v`. |
| `--ar` | `--aspect` | `W:H` integers; common: `1:1`, `2:3`, `3:2`, `4:5`, `5:4`, `6:11`, `9:16`, `16:9`, `21:9` | `1:1` | ✅ Full | No decimals. Extremely wide/tall ratios are experimental. Aspect change mid-iteration usually invalidates composition — lock early. |
| `--stylize` | `--s` | `0`–`1000` integer | `100` | ✅ Full | V8.1 sweet spot: `100`–`400`. Above ~500 returns less radical change than in V7 (V8-family stylize range is compressed). Combine `--s 25–75` with `--style raw` for max photoreal. |
| `--chaos` | `--c` | `0`–`100` integer | `0` | ✅ Full | Also exposed as "Variety" slider in web UI. Use `50–75` for exploration, `0–25` for finals. |
| `--weird` | `--w` | `0`–`3000` integer | `0` | ✅ Full | Not fully compatible with `--seed`. Not compatible with moodboard jobs (auto-stripped). |
| `--quality` | `--q` | `1`, `2`, `4` (V7); `1`, `2`, `4` historically valid in V8.0 (`--q 4` = extra coherence at 4× cost) | `1` | ⚠️ Use sparingly | In V8.1 the `--hd` default supersedes most reasons to use `--q 4`. `--q 4` + `--hd` combined was the only Relax-incompatible combo in V8.0. |
| `--hd` | — | flag | **ON by default in V8.1** | ✅ V8.1-specific | Native 2K (2048-px). 3× faster + 3× cheaper than V8.0 HD. Cost ~1.33 GPU-min per 4-image job. |
| `--sd` | — | flag | off (HD default) | ✅ V8.1-specific | Forces Standard Definition (1024-px). 50% faster + 25% cheaper than V8.0 SD. SD-quality jobs match V7 draft-mode speed. Set as default during May 2026 server transition. |
| `--raw` / `--style raw` | — | flag | off | ✅ Full | Removes default Midjourney "house" styling. Recommended for product, editorial, documentary, and any literal-rendering job. Less needed in V8.1 than in V8.0 because default aesthetic is already calmer. |
| `--seed` | — | integer `0`–`4294967295` | random | ✅ Full | V8.1 seeds are ~99% reproducible (a major V8-series improvement). Use for A/B prompt iteration. |
| `--stop` | — | `10`–`100` | `100` | ✅ Full | Stops generation early to get less-finished/sketchier outputs. |
| `--tile` | — | flag | off | ⚠️ Known issue | Generates seamless repeating texture. V8.1 has a documented faint-border edge bug; Midjourney is working on a fix. Use V6.1/V7 if production-critical until patched. |
| `--no` | — | comma-separated tokens (e.g. `--no text, frame, watermark`) | none | ❌ **NOT compatible with V8.1** | The V8.1 backend rejects any job containing `--no` with the error `"--no is not compatible with --version 8.1"`. Strip the parameter and express exclusions via (a) positive description ("documentary photography, naturalistic skin" instead of `--no anime, cartoon`), (b) a moodboard/profile tuned to exclude unwanted features, or (c) post-process in the Editor. Still supported on V7, V6.1, and Niji 7. |
| `--iw` (image weight) | — | `0`–`2` (default `1`); some V-versions extend to `0`–`3` | `1` | ✅ Restored in V8.1 | Image prompts and image weights are back in V8.1 after being broken in V8.0. Increment by 0.25 when tuning. |
| `--sref` | — | image URL(s) OR numeric style code(s) OR `random` | none | ✅ Full + super-stable in V8.1 | Multi-image: separate URLs with spaces. Codes work via `--sv 4` (pre-Jun 2025 V7 model), `--sv 6` (V6.1 codes), `--sv 7` (V8-series default, 4× faster/cheaper). Sref random is allowed only with `--sv 4` and `--sv 6`. |
| `--sw` (style weight) | — | `0`–`1000` | `100` | ✅ Full | V8.1 sweet spot 50–150 for subtle, 150–300 for strong, 300+ for dominant. |
| `--sv` (style-reference version) | — | `4`, `6`, `7` | `7` in V8 series | ✅ Full | `--sv 7` is the new default introduced March 21, 2026 — 4× faster/cheaper and compatible with `--hd`, `--p`, `--stylize`, `--exp`. |
| `--p` | `--profile` | flag (use defaults) OR profile/moodboard ID | off unless toggled | ✅ Full + required to access V8.1 | `--p` alone applies the user's default profile(s). `--p <pID>` applies a specific ranked profile. `--p <mID>` applies a specific moodboard. Stack: `--p code1 code2`. **HARD REQUIREMENT: a Global V7/V8 Personalization Profile must be unlocked or V8.1 refuses to render.** Stylize controls intensity (0=off, 1000=max). |
| `--oref` (Omni Reference) | — | image URL | n/a | ❌ NOT YET IN V8.1 | Confirmed not yet available in V8.1; planned to return before the V8 editor model. Use V7 if you need it now. Editor/gallery jobs that contain `--oref` cannot be re-opened in the V8.1 Editor — strip `--oref` and `--ow` first. |
| `--ow` (Omni weight) | — | `0`–`1000` (default `100`) | `100` | ❌ Tied to `--oref` — unavailable in V8.1 | Auto-stripped by the web UI when `--oref` is absent. |
| `--cref` (Character Reference) | — | image URL(s) | n/a | ❌ Not in V8.1 | V6/V6.1/Niji 6 only. V7 replaced it with `--oref`. V8.1 currently has neither — character consistency workflows should stay on V7 (or future V8 character system, hinted by the Niji team). |
| `--cw` (Character weight) | — | `0`–`100` | `100` | ❌ Tied to `--cref` | Auto-stripped by web UI when `--cref` absent. |
| `--exp` (experimental aesthetics) | — | `0`–`100` | `0` | ✅ Full | V8.1 sweet spot `5`–`25`. At ≥50 overrides `--stylize` and `--p`. Compatible with `--sv 7`. |
| `--repeat` | `--r` | integer (plan-dependent) | n/a | ✅ Full | Runs the prompt N times; consumes GPU per run. |
| `--draft` | — | flag | off | ⚠️ V7 feature | Per official docs, Draft Mode is V7-compatible. In V8.1, SD-quality is already at V7-draft speed, so the workflow equivalent is `--sd` instead of `--draft`. Conversational Mode IS supported in V8.1. |
| `--motion` (video) | — | `low`, `high` | `low` | ✅ via Animate, but V8.1 still uses V1 Video Model | Animate any V8.1 image via web. HD video available on Standard/Pro/Mega (Fast only); Relax video on Pro/Mega only. |
| `--bs` (video batch size) | — | `1`, `2`, `4` | `4` | ✅ | Lowers default 4-clip output for cost savings. |
| `--profile` | (alias of `--p`) | profile/moodboard ID | n/a | ✅ Full | Identical to `--p code`; both forms accepted. Stack with `--sref` codes for blended aesthetics. |

### Multi-Prompt Weighting `::`

```
SEGMENT_A ::W1 SEGMENT_B ::W2 SEGMENT_C ::W3
```
- `::` is a hard divider; Midjourney processes each segment as an independent concept then composites.
- Weight `W` is appended directly after `::` with no space. Default `W=1`. Decimals OK in V4+ family. Negative weights allowed (equivalent to the legacy `--no` on pre-V8.1 versions) **only if the sum of all weights remains positive**.
- **Caveat for V8.1:** Midjourney's own docs list multi-prompt compatibility as "versions 1, 2, 3, 4, Niji 4, 5, Niji 5, 6, Niji 6, 6.1." Multi-prompt `::` behavior in V7 and V8.x is undocumented and unreliable. Because `--no` is also incompatible with V8.1, the recommended V8.1 exclusion strategy is **positive description** ("documentary photography, real-world physics" in place of `--no anime, cartoon`) plus moodboard tuning — not `::` and not `--no`.

---

## `<RESOLUTION_AND_COST>` — Decision Logic

```yaml
default_mode: HD
HD:
  resolution: 2048x2048 (1:1) or equivalent in other aspect ratios
  cost: ~1.33 GPU-min per 4-image job
  speed_vs_v8_0_HD: 3x faster, 3x cheaper
  use_when: ["final delivery", "print", "client deliverable", "needs >1k detail"]
SD:
  resolution: ~1024x1024 (1:1)
  cost: <1.0 GPU-min per 4-image job
  speed: equivalent to V7 draft mode at full quality
  use_when: ["exploration", "iteration", "concept hunting", "many variations"]
run_as_HD_button:
  behavior: reruns SD job at HD using same seed (NOT a true upscaler — minor variations expected)
  warning: "Do NOT use Run-as-HD to preserve a specific SD image. Run HD from the start if a specific composition matters."
upscaler_for_v8_1: NOT YET RELEASED  # roadmap: V8 upscalers next, then V8 edit/inpaint/outpaint
editor_for_v8_1_images:
  status: works, but Editor/Pan/Zoom/Vary-Region still use V6.1 model under the hood
```

---

## `<MODE_MATRIX>` — Speed Modes × Subscription

| Mode | Basic | Standard | Pro | Mega | V8.1 Compatible |
|---|---|---|---|---|---|
| Fast | ✅ | ✅ | ✅ | ✅ | ✅ |
| Relax | ❌ | ✅ | ✅ | ✅ | ✅ (added Mar 21 2026; all commands except `--hd` + `--q 4` combined) |
| Turbo | ✅ | ✅ | ✅ | ✅ | Available where surfaced |
| Stealth | — | — | ✅ | ✅ | — |

---

## `<REFERENCE_SYSTEMS>` — Style / Personalization / Image

### Style Reference (`--sref`)
- Two value types: image URL(s) OR numeric style code(s) (from Midjourney's internal library).
- Multiple values: space-separated (`--sref URL1 URL2` or `--sref 123456 789012`).
- Style versions selectable via `--sv 4|6|7`. V8.1 default is `--sv 7`.
- `--sref random` requires `--sv 4` or `--sv 6` (V8.1 default `--sv 7` is incompatible with `random`).
- Weight via `--sw 0–1000`. Default `100`.
- V8.1 makes srefs "super stable" — the V8.0 sref drift bug is fixed.
- Cannot create a custom numeric code from an uploaded image (use moodboards instead for that).

### Moodboards / Personalization (`--p` / `--profile`)
- HARD REQUIREMENT: Global V7/V8 Personalization Profile must be unlocked (rate images at `midjourney.com/personalize` until the progress bar fills).
- V7 personalization profiles are fully backwards-compatible with V8.1.
- `--p` → applies user's default profile(s).
- `--p <ID>` or `--profile <ID>` → applies a specific Ranked Profile, Moodboard, or Standard Profile.
- Stack multiple: `--p code1 code2 code3` (space-separated). Active profile selection now allows multiple simultaneously.
- Stylization profiles increase in stability at 40 / 200 / 2000 ratings (minimum / stable / max).
- Moodboards can be blended with `--sref` codes/URLs in a single prompt (e.g. `--sref 142710498 --profile drgmjoi 2jrqbw6`).
- `--stylize` controls personalization intensity: 0 disables, 1000 maximizes. Default 100.
- Moodboards are NOT compatible with `--sv`, `--sw`, or `--weird` (web UI auto-strips the weird parameter from moodboard jobs).

### Image Prompts (`--iw`)
- Restored in V8.1 (was unavailable in V8.0).
- Drag-and-drop or paste image URL(s) BEFORE the text portion.
- Multi-image without text → behaves like Discord `/blend`.
- Per-image weight via multi-prompt `::` (e.g. `URL1::2 URL2::1`) plus overall `--iw`.
- File extensions: `.png`, `.gif`, `.webp`, `.jpg`, `.jpeg`.
- Best practice: crop reference to match `--ar` of target output.
- Prompts with images-only (no text) are INCOMPATIBLE with `--stylize` and `--weird`.

### Omni / Character / Object Reference
- **Not yet supported in V8.1.** Stay on V7 with `--oref` if subject-consistency across images is required.

---

## `<TEMPLATES>` — Production-Ready Prompt Skeletons

### T1. Cinematic photoreal (recommended V8.1 default)
```
[Shot type] [framing] of [subject phys. description], [action/pose], [costume], [setting/time-of-day], shot on [camera body] with [lens, aperture], [lighting style], [mood/atmosphere], documentary photography, naturalistic color, real-world physics
--v 8.1 --ar 16:9 --s 150 --style raw --p
# V8.1 rejects --no. Bake exclusions into the positive description ("documentary photography, naturalistic" repels anime/illustration; "real-world physics" repels CGI/3D-render look).
```

### T2. Editorial portrait
```
[Subject phys. description, age, ethnicity, hair, eyes], [expression], [pose], [lighting pattern: Rembrandt | butterfly | split | rim | loop] lighting, shallow depth of field, [background], shot on [medium-format camera] with [85–110mm lens]
--v 8.1 --ar 4:5 --s 100 --style raw
```

### T3. Product / commercial (literal)
```
[Product] on [surface], unpopulated empty scene, [background style], [lighting setup], commercial product photography, sharp detail, [brand aesthetic adjective], clean unbranded surfaces, no signage
--v 8.1 --ar 1:1 --s 50 --style raw
# V8.1 rejects --no. Inline "unpopulated empty scene" and "clean unbranded surfaces, no signage" replace the legacy `--no people, hands, text, watermark`.
```

### T4. Concept art / fantasy
```
[Character/scene], [worldbuilding details], [magical/SF element], [lighting], [medium: concept art | painterly illustration | matte painting], influenced by [Artist A] and [Artist B]
--v 8.1 --ar 21:9 --s 500 --w 150 --c 25
```

### T5. Anime / manga (route through Niji 7, not V8.1)
```
[Character description], [pose/action], [expression], [setting], [color palette]
--niji 7 --ar 3:4 --sref <URL_or_code> --sw 150
```

### T6. Typography / signage (V8.1 strength)
```
[Scene with sign/poster], the words "QUOTED TEXT" in [typography style], [material/medium], [lighting]
--v 8.1 --ar 16:9 --s 100 --style raw
# Keep quoted text ≤ 4 words for best legibility.
```

### T7. Image-prompted variation
```
<IMG_URL> [optional text describing the desired transformation]
--v 8.1 --iw 1.25 --ar 4:5 --s 150
# --iw < 1 favors text; --iw > 1 favors source image.
```

### T8. Style transfer from sref code(s)
```
[Scene description, neutral, no style words]
--v 8.1 --sref <code1> <code2> --sw 150 --ar 3:2 --s 100
# Avoid style words in the text — let --sref carry the look.
```

### T9. Moodboard-driven brand image
```
[Subject and scene], [practical lighting/context]
--v 8.1 --profile <moodboardID> --ar 4:5 --s 200
# Pair with --sref for hybrid control.
```

### T10. Tileable texture
```
seamless [material] texture, [surface qualities], top-down macro detail
--v 8.1 --tile --ar 1:1 --s 100 --style raw
# Known V8.1 edge-border bug — verify in a pattern checker before production use.
```

### T11. Positive-exclusion refinement (V8.1 replacement for negative-prompt-heavy workflows)
```
[Subject], [scene], [lighting], documentary photography, real-world physics, naturalistic color, clean unbranded surfaces, no text or signage in frame, full-bleed composition with no border or frame, sharp natural focus
--v 8.1 --ar 3:2 --s 100 --style raw
# V8.1 rejects --no. The positive-clause version above achieves the same artifact-stripping effect via description.
# Anti-anime/cartoon: "documentary photography, real-world physics, naturalistic color"
# Anti-text/watermark: "clean unbranded surfaces, no text or signage in frame"
# Anti-frame/border: "full-bleed composition, edge-to-edge"
# Anti-oversaturation/HDR: "naturalistic color, neutral grade"
```

### T12. Multi-prompt weighted composite (LEGACY — use sparingly in V8.1)
```
[concept_A] ::2 [concept_B] ::1 [unwanted_concept] ::-0.5
--v 8.1 --ar 16:9
# Multi-prompt :: is officially documented only through 6.1; behavior in V8.1 is inconsistent. --no is also rejected by V8.1. Prefer natural-language positive description in V8.1.
```

---

## `<RULES>` — Hard Constraints for V8.1 Prompt Generation

```xml
<rules>
  <r1>ALWAYS include `--v 8.1` (or rely on user's default model setting) unless the task is anime/manga (use `--niji 7`).</r1>
  <r2>NEVER mix `--v` and `--niji` in the same prompt.</r2>
  <r3>NEVER include `--oref`, `--ow`, `--cref`, or `--cw` in a V8.1 prompt — they are not supported. If subject consistency is required, switch to `--v 7` + `--oref`.</r3>
  <r4>NEVER use `--draft` in V8.1 prompts. Use `--sd` for fast exploration instead.</r4>
  <r5>NEVER include decimals in `--ar`. Convert (e.g., 1.85:1 → 37:20).</r5>
  <r6>NEVER name living actors, named celebrities, or trademarked characters as the subject. Describe people physically (age, ethnicity, build, hair, eyes, expression, costume).</r6>
  <r7>NEVER stack quality keywords ("8k, ultradetailed, masterpiece, beautiful, stunning"). V7+ ignores or actively degrades on keyword spam. Write full clauses describing what is in the frame.</r7>
  <r8>ALWAYS use STRAIGHT double quotes `"..."` to mark in-image text. Keep quoted text ≤ 4 tokens for high success rate.</r8>
  <r9>ALWAYS lock aspect ratio early — V8.1 composes differently per ratio; switching mid-iteration usually invalidates the seed.</r9>
  <r10>**`--no` is NOT compatible with V8.1.** The backend rejects jobs containing `--no` with `"--no is not compatible with --version 8.1"`. Express exclusions through positive description (e.g., "documentary photography, naturalistic" in place of `--no anime, cartoon`) or via a moodboard tuned to suppress unwanted features. The legacy guidance about two-word `--no` phrases parsing as independent tokens still applies on V7, V6.1, and Niji 7 — not V8.1.</r10>
  <r11>If using ONLY image prompts (no text), do NOT add `--stylize` or `--weird` — they are silently incompatible.</r11>
  <r12>When using a moodboard (`--p <mID>` or `--profile <mID>`), do NOT add `--sv`, `--sw`, or `--weird` — auto-stripped or unsupported.</r12>
  <r13>If `--exp ≥ 50`, do NOT also rely on `--stylize` or `--p` for look control — they will be overridden.</r13>
  <r14>For maximum photorealism: `--s 25–100` + `--style raw` + specific camera/lens/lighting clauses.</r14>
  <r15>For maximum stylization: `--s 300–700` + named artistic-medium clause + optional `--sref`.</r15>
  <r16>Seed for reproducibility: a single integer; V8.1 reproduces to ~99% identity. Use the same seed when A/B testing prompt-token deltas.</r16>
  <r17>Run-as-HD is NOT an upscaler — it reruns from seed at HD. To guarantee a specific composition in HD, render in HD from the start.</r17>
  <r18>V8.1 rewards LONGER, MORE SPECIFIC prompts than V7 did. Aim for 40–120 tokens of meaningful scene direction. Prompts beyond the length cap auto-trigger Prompt Shortener, which may introduce variability — for maximum control, keep within limit yourself.</r18>
  <r19>The user MUST have a Global V7/V8 Personalization Profile unlocked, or V8.1 returns an error. If generating prompts for first-time users, advise them to complete profile setup first.</r19>
  <r20>HD is the default — only add `--sd` when explicitly wanting standard resolution. Do not add `--hd` redundantly (it is the active default during normal operation; during temporary server-transition periods Midjourney may swap the default to SD, in which case `--hd` re-enables HD).</r20>
</rules>
```

---

## `<PROMPT_HIERARCHY>` — Token-Order Weighting (V8.1)

V8.1 weights early tokens most heavily. Construct prompts in this order:

1. **Subject** (concrete noun phrase): "elderly fisherman"
2. **Subject details** (physical, costume, expression): "weathered face, silver beard, kind eyes, navy wool sweater"
3. **Action / pose**: "mending nets, hands visible"
4. **Context / environment / time-of-day**: "on a wooden dock at dawn, mist over the harbor"
5. **Style / mood**: "documentary photography, contemplative"
6. **Technical (camera / lens / lighting)**: "shot on Leica M11 with 50mm f/1.4, soft morning side-light"
7. **In-image text (quoted)** if any: `"OPEN"`
8. **Parameters**: `--v 8.1 --ar 3:2 --s 150 --style raw --p`  *(do NOT add `--no` — incompatible with V8.1; bake exclusions like "documentary photography, naturalistic" into the positive description instead)*

---

## `<FAILURE_MODES>` — Known V8.1 Weaknesses

| Failure mode | Mitigation |
|---|---|
| Faint border edges on `--tile` outputs breaking the pattern repeat | Verify in a seamless-pattern checker; fall back to V6.1/V7 for production tiles until fixed. |
| `--oref` / `--cref` not available — character consistency across images fails | Use V7 + `--oref` for character work; revert to V8.1 once V8 character ref ships. |
| Editor / Pan / Zoom / Vary Region still rendering at V6.1 quality on V8.1 images | Plan final composition in the prompt; avoid heavy post-edits until V8 Editor ships. |
| Occasional blurriness / pixelation in a minority of V8.1 outputs | Use feedback buttons in the lightbox; regenerate; or run as HD explicitly. |
| Hands and bodies still occasionally distorted (improved vs V7 but not perfect) | `--no hands` is unavailable in V8.1 — instead crop hands out of frame in the composition ("torso-up framing, hands out of view"), describe hand position precisely ("hands resting at sides, fingers relaxed"), or fix in the V6.1-backed Editor post-render. |
| Text rendering worse than V8.0 (Midjourney explicitly traded text quality for V7-aesthetic restoration) | Keep quoted text ≤ 4 words; do final typography in post; consider V8.0 if available specifically for text-heavy work. |
| Long prompts auto-shortened invisibly, increasing variability | Keep prompts within the length limit (~limit not formally documented; community reports historical ~300 chars but V8.1 raises it). When the long-prompt icon appears, manually edit instead of relying on the Shortener. |
| Multi-prompt `::` weighting unreliable in V8.1 | Both `::` and `--no` are unreliable/rejected on V8.1. Prefer pure natural-language emphasis and positive description; rely on moodboards/`--sref` for repeatable style control. |
| Personalization required to even use V8.1 | Always advise unlocking Global V7/V8 Profile first. |
| Aspect-ratio composition not transferable | Set `--ar` BEFORE iterating; treat ratio change as a new experiment. |
| Stylize ≥ 500 returns smaller delta than expected (V8-family stylize range compression) | Anchor stylize in 100–400 band; reach beyond by adding `--exp 10–25` rather than pushing `--stylize`. |

---

## `<BEST_PRACTICES_V8_1>` — Patterns That Work

```yaml
prompting_voice:
  - Write full clauses and sentences, not keyword lists.
  - "trend towards longer, more specific prompting" (Midjourney's own V8-series guidance).
  - Be literal about medium: "35mm film photograph, Kodak Portra 400 palette, visible grain" beats "film look".
  - Name lighting precisely: "single overhead key light, no fill, hard shadows" beats "dramatic lighting".
  - Reference photographers/cinematographers/directors for style anchoring (e.g., "Roger Deakins cinematography", "Annie Leibovitz portraiture", "Hélène Binet architectural photography").

token_economy:
  - 30–80 tokens: balanced control (most general prompts).
  - 80–150 tokens: detailed scene direction (V8.1 sweet spot for pro work).
  - 150+ tokens: diminishing returns; Prompt Shortener may engage.

stylize_recipes:
  photorealistic: --s 25–100 --style raw
  editorial: --s 100–250
  illustrative: --s 300–500
  abstract/experimental: --s 600–1000 + --w 500–2000

exploration_to_production_loop:
  1: "Explore with --sd (V8.1 SD ≈ V7 draft speed). High --chaos 50–75."
  2: "Identify direction. Lock seed. Lower --chaos to 0–25."
  3: "Refine prompt tokens. Same seed, small edits."
  4: "Render final in HD (default). Or 'Run as HD' to upgrade the selected seed."

sref_workflow:
  - Use --sv 7 default for V8.1; --sv 6 if migrating legacy V6.1 codes.
  - Multi-sref blending: --sref URL1 URL2 (averages styles).
  - Combine moodboard + sref: --profile <mID> --sref <code> for project-consistent yet image-anchored output.

positive_exclusion_recipes:
  # V8.1 rejects --no. Replace each legacy --no list with a positive-clause equivalent baked into the prompt body.
  remove_photo_realism_artifacts:
    legacy_v7:   "--no anime, cartoon, illustration, painting, 3d render, CGI"
    v8_1_clause: "documentary photography, naturalistic color, real-world physics, photographic skin texture, no stylization"
  clean_image:
    legacy_v7:   "--no text, watermark, signature, frame, border, logo"
    v8_1_clause: "clean unbranded surfaces, no text or signage in frame, full-bleed composition with no border or frame"
  natural_skin:
    legacy_v7:   "--no makeup, filters, oversaturation, HDR"
    v8_1_clause: "bare unfiltered skin with visible pores and natural texture, neutral color grade, no makeup"
  simple_composition:
    legacy_v7:   "--no busy, cluttered, crowded"
    v8_1_clause: "minimalist composition, empty negative space, single subject in frame"
  flat_graphic_look:
    legacy_v7:   "--no blur, depth of field"
    v8_1_clause: "deep focus, hyperfocal sharpness from edge to edge, flat graphic rendering"
```

---

## `<DESCRIBE_AND_SHORTENER>` — V8.1 Helper Tools

```yaml
describe:
  trigger: right-click image → "Describe", or drag image to top of prompt bar → Describe
  output: 4 candidate prompts written in V8 prompting style (longer, more detailed than previous Describe)
  use_for: reverse-engineering an admired image's prompt; modify before running
  prompts_clear_on: page refresh

prompt_shortener:
  trigger: automatic when input exceeds prompt length limit
  ui_signal: "long prompt" icon appears with note that prompts of this length will be adjusted (may increase variability)
  behavior: input is shown in full but model receives a condensed version
  best_practice: stay within length limit manually to retain full control; rely on Shortener only as a safety net

conversational_mode:
  status: supported in V8.1
  use_for: natural-language prompt drafting; voice input optional
  reference_images_by: "image 1", "image 2", etc. in conversation
```

---

## `<COMPATIBILITY_MATRIX>` — V7 vs V8.0 vs V8.1 vs Niji 7

| Capability | V7 | V8.0 Alpha | V8.1 | Niji 7 |
|---|---|---|---|---|
| Default model on midjourney.com | Until Apr 30 2026 | (alpha-only) | After Apr 30 2026 | n/a |
| Native 2K | No | `--hd` (4× cost) | **Default** | No |
| Speed vs V7 baseline | 1× | ~5× | ~4–5× | ~ |
| Text rendering with quotes | Good | Best | Slightly worse than V8.0 | Good |
| `--no` (negative prompt) | ✅ | ✅ | ❌ **rejected by backend** | ✅ |
| `--oref` | ✅ | ✅ | ❌ (not yet) | ❌ |
| `--cref` | ❌ | ❌ | ❌ | ❌ |
| `--sref` stability | ✅ | unstable | ✅ super-stable | best |
| Moodboards (`--p`) | ✅ | unstable | ✅ super-stable | ✅ (since Feb 26 2026) |
| Image prompts + `--iw` | ✅ | ❌ broken | ✅ restored | ✅ |
| Draft Mode `--draft` | ✅ | ❌ | ❌ (use `--sd`) | ❌ |
| Editor / Pan / Zoom on its own outputs | V6.1 backend | V6.1 backend | V6.1 backend | n/a |
| Upscalers | ✅ Subtle/Creative | ❌ | ❌ (roadmap) | ✅ |
| Video (V1 model) | ✅ | ✅ | ✅ | ✅ |
| Aesthetic default | reference | over-processed | "spirit of V7" | clean anime |
| Stylize useful range | 0–1000 (full) | 100–400 compressed | 100–400 most useful | 0–1000 |

---

## Recommendations

### When to Generate Prompts Targeting V8.1
- **Default to V8.1** for photoreal product, editorial, cinematic, architectural, and typography work. It is the fastest, sharpest, and most economically priced V8-series model.
- **Use V7 instead** when the workflow requires (a) `--oref` character consistency across images, (b) Draft Mode, (c) heavy use of the Editor (Pan/Zoom/Vary Region) where V6.1 backend artifacts would be visible, or (d) Vary Region inpainting.
- **Use Niji 7 instead** for any anime/manga, illustrated character, or Eastern-aesthetic illustration work — and Niji 7's `--sref` is still the platform's strongest style-transfer engine.

### Staged Prompt-Generation Workflow for LLMs
1. **Classify intent** → photoreal / illustration / anime / typography / abstract.
2. **Select model flag** → `--v 8.1` (default), `--v 7` (only for `--oref`/Editor work), or `--niji 7` (anime).
3. **Pick template** (T1–T12 above) matching the intent.
4. **Fill SUBJECT-first hierarchy** (see `<PROMPT_HIERARCHY>`).
5. **Apply parameter recipe** from `<BEST_PRACTICES_V8_1>` (`--s`, `--style raw`, etc.). Do NOT add `--no` for V8.1 — express exclusions through positive-clause description (see `positive_exclusion_recipes`).
6. **Add reference layer** if user supplied images/codes/moodboards: `--sref`, `--p`, image URL + `--iw`.
7. **Audit against `<RULES>`** — strip forbidden parameters, fix `--ar` decimals, ensure subject is described physically not by celebrity name.
8. **Validate** prompt length is within typical limits; if longer, warn the user that Prompt Shortener will engage.

### Benchmarks that should change these recommendations
- **V8.2 release** (currently training; rating data being collected at `midjourney.com/rank-v8-1`): re-evaluate aesthetic defaults and any new parameters.
- **V8 upscalers ship** (next on Midjourney's roadmap): drop the "Run-as-HD-is-not-an-upscaler" caveat; revise the Editor matrix.
- **V8 edit / inpaint / outpaint model ships**: Editor and Vary Region recommendations should swap from V6.1-backend caveat to V8 native.
- **`--oref` returns to V8.1**: drop the "use V7 for character consistency" branch.
- **V8.0 decommission** (any week now): drop all V8.0 references from this guide.
- **Multi-prompt `::` officially documented for V8.x**: re-promote `::` from legacy fallback to first-class technique.

---

## Caveats

- **Midjourney releases are fast and undocumented.** Some behaviors (e.g., exact `--q` value support in V8.1, exact character cap that triggers Prompt Shortener, exact `--iw` upper bound) are not formally published in V8.1's release notes; values in this guide reflect the most authoritative public sources (Midjourney's `updates.midjourney.com` post, `docs.midjourney.com` Version article, and multiple May 2026 reviews). Treat numeric ranges marked "historically" or "community-reported" as approximate.
- **The `docs.midjourney.com` Multi-Prompts page lists compatibility as "versions 1–6.1," omitting V7/V8.x.** It is unclear whether `::` is unsupported in V8.1 or merely undocumented. Empirical testing required for production reliance.
- **Editor / Pan / Zoom Out / Vary Region currently use V6.1** even on V8.1-generated images. Quality and prompt fidelity inside those tools will not match V8.1 native render quality until the V8 edit/inpaint/outpaint model ships.
- **`--draft` Draft Mode is documented as V7-compatible.** Reviewers report V8.1 SD-quality already matches V7 draft speed, so practical Draft-Mode workflows transfer to `--sd` in V8.1.
- **A Global V7/V8 Personalization Profile is required to use V8.1.** New users must complete the personalization image-selection flow first; this is not optional.
- **`--oref` and `--cref` are unavailable in V8.1.** This is a regression compared to V7; Midjourney has signaled `--oref` will return before the V8 edit model, but no firm date.
- **V8.0 is being decommissioned a few weeks after V8.1 stable.** Workflows that relied on V8.0-specific behavior (notably its stronger text rendering) should test in V8.1 immediately.
- **Speed and cost claims (1.33 GPU-min HD, <1 GPU-min SD, "3× faster/cheaper") are from Midjourney's own announcement** (`updates.midjourney.com/v8-1-alpha/` and `/v8-1-updates/`) and may be subject to revision during the documented "server transition" period in which SD was temporarily made default to conserve compute.
- **During temporary server transitions**, Midjourney may force SD as default; verify HD is active in Settings or append `--hd` explicitly.
- **The 50-style "Style Creator" feature** referenced in some third-party May 2026 articles is a UI / curated-styles tool, not a per-prompt parameter; do not fabricate a `--style <name>` flag beyond the documented `--style raw` (V7) / `raw` toggle.
- **This guide reflects the state of Midjourney V8.1 as of approximately May 12, 2026.** Re-validate against `docs.midjourney.com/hc/en-us/articles/32199405667853-Version` and `updates.midjourney.com` before any production deployment.