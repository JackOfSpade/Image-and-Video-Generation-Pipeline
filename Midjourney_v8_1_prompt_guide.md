# Midjourney v8.1 — AI Prompt Reference

Version: V8.1 (released to `midjourney.com` April 30, 2026; V8.1 Alpha shipped on `alpha.midjourney.com` April 14, 2026; later released to Discord). V8.1 is an evolution of V8.0 with V7-spirit aesthetics, native 2K HD by default, and restored image prompts. Several legacy parameters are temporarily limited in V8.1 — those are flagged inline.

---

## Parameters Go At The End

All `--flags` go at the end of the prompt text, after a single space. No commas or punctuation inside the parameter block. One double-hyphen `--`, not a single hyphen, not a spaced hyphen.

`vibrant California poppies --ar 2:3 --s 100 --v 8.1` ✅
`vibrant California poppies--ar 2:3` ❌ (no space)
`vibrant California --ar 2:3 poppies` ❌ (text after parameters)

---

## Version Selection

`--v 8.1` selects V8.1. `--v 8` selects V8.0 (decommissioning expected). `--v 7` selects V7 (legacy default). `--niji 7` selects anime model. Set default version in settings panel.

```
--v 8.1     # current, fastest, native 2K, V7-aesthetic
--v 8       # alpha, hyper-polished, less stable srefs
--v 7       # legacy flagship, atmospheric, painterly
--niji 7    # anime/manga, best linework
--niji 6    # legacy anime, has --style options
```

Personalization Global V7/V8 Profile must be unlocked before using V8.1.

---

## Resolution: HD vs SD in V8.1

V8.1 default is HD (native 2K, ~2048px on long edge). HD costs ~1.33 GPU minutes; SD < 1 GPU minute. HD is 3× faster + 3× cheaper than V8.0 HD. SD is 50% faster + 25% cheaper than V8.0 SD; SD at full quality runs as fast as V7 draft mode.

```
--hd        # force native 2K (default on V8.1)
--sd        # force standard resolution (~1K)
```

`Run as HD` web button reruns a seed-locked SD job in HD. It is **not** an upscaler; the seed may not perfectly preserve the SD image. To guarantee HD, generate with `--hd` from the start.

---

## --ar (Aspect Ratio)

`--ar W:H` or `--aspect W:H`. Default `1:1`. No decimals — use `139:100` not `1.39:1`. Extreme ratios beyond ~2:1 are experimental and unstable. Some upscalers/editor passes may slightly shift the ratio.

```
--ar 1:1    # square; social profile, icons, tiles
--ar 4:5    # portrait social feed (Instagram)
--ar 5:4    # near-square landscape; presentations
--ar 2:3    # vertical print, Pinterest, book covers
--ar 3:2    # classic photo print, landscape
--ar 16:9   # widescreen, video thumbnails, environments
--ar 9:16   # vertical video, Stories, mobile
--ar 21:9   # cinematic ultrawide, anamorphic
--ar 6:11   # tall portrait, phone wallpapers
--ar 7:4    # close to HD TV / smartphone
```

Wide ratios push environment/context. Tall ratios push subject focus and headroom.

---

## --stylize / --s

Range `0–1000`. Default `100`. Controls how much Midjourney applies its trained aesthetic.

```
--s 0–50      # literal, product, technical accuracy
--s 50–150    # default-band; most photoreal portraits
--s 150–300   # editorial, mood-driven
--s 300–600   # illustrative, concept art
--s 600–1000  # heavily stylized, surreal
```

V8/V8.1 caveat: extreme `--s` is less radical than V7. For visible style shift in V8.1 use `--s 400+`. For text legibility, drop `--s` to `25` or `0`.

---

## --chaos / --c

Range `0–100`. Default `0`. Controls variance across the four images in a grid.

```
--c 0      # tight, four near-identical
--c 10–25  # slight variance, controlled exploration
--c 25–50  # broader exploration
--c 50–75  # divergent compositions per cell
--c 75–100 # near-random; prompt adherence drops
```

`--c` combined with locked `--seed` produces structured divergence rather than randomness. Use `--c 50` + `--s 500` for diverse-but-coherent ideation.

---

## --weird / --w

Range `0–3000`. Default `0`. Adds unconventional / experimental aesthetic deviations. Experimental flag — behavior changes between versions. Not fully compatible with `--seed`. Returns of effect are non-linear; >500 yields diminishing strangeness, >1000 stabilizes into different bizarre archetypes.

```
--w 0          # standard
--w 100–500    # subtle quirks; recommended starting band
--w 500–1000   # noticeable strangeness
--w 1000–3000  # surreal; prompt fidelity collapses
```

`--weird` ≠ `--chaos`. `--chaos` spreads the four-grid; `--weird` bends individual aesthetics. Stack with `--s` of equal value (e.g., `--s 700 --w 700`) to preserve aesthetic coherence at high weirdness.

---

## --quality / --q

Values: `1` (default), `2`, `4`. `--q 3` auto-promotes to `--q 4`. Cost scales linearly with the value (2× for `--q 2`, 4× for `--q 4`). Affects only the initial 4-image grid, not variations/upscales/edits.

```
--q 1   # default
--q 2   # 2x compute, denser detail
--q 4   # 4x compute, max coherence; not compatible with --oref
```

`--hd` + `--q 4` together costs 16× a base SD job in V8.0; in V8.1 the multiplier is reduced (~1.5–2.5× for HD overall) but combined HD + `--q 4` is the most expensive combination and is excluded from Relax Mode.

---

## --seed

Whole integers `0` – `4294967295`. Same prompt + same seed produces near-identical results. V8.1 Alpha: seeds reproduce ~99% identical. Seeds affect only the initial grid, not variations or upscales.

```
--seed 12345
```

Use cases: lock a composition while iterating prompt text; produce series with identical lighting/composition; A/B test parameters against a fixed base.

---

## --sref (Style Reference)

Transfers visual style (palette, lighting, technique, mood) from a reference image or numeric code. Does not transfer subject, identity, or composition.

```
--sref https://image.url
--sref https://url1 https://url2          # blend multiple
--sref 1234567890                          # internal style code
--sref random                              # random style code (resolves to a fixed code)
--sref URL1::2 URL2::1 URL3::1             # weighted refs (Discord; V6/V7)
```

`--sref random` resolves to a fixed numeric code on submit, allowing reuse. Permutations + `--sref random` produce a different code per permutation.

Acceptable input formats: `.png .gif .webp .jpg .jpeg`. Must be public URL.

### --sw (Style Weight)

Range `0–1000`. Default `100`. Higher = stronger style adherence. `--sw` is more impactful with numeric style codes than with reference images. `--sw` is **not** compatible with Moodboards.

```
--sw 0–50      # subtle hint
--sw 50–150    # balanced
--sw 150–300   # strong style match
--sw 300–1000  # dominant style
```

### --sv (Style Reference Version)

Selects the underlying SREF algorithm.

```
--sv 4   # old V7 sref engine (pre June 16, 2025)
--sv 6   # default for V7
--sv 7   # default for V8 / V8.1 (4× faster + 4× cheaper than --sv 6; supports --hd, --p, --stylize, --exp)
```

`--sref random` and numeric `--sref` codes are compatible only with `--sv 4` and `--sv 6`.

---

## --cref (Character Reference) — V6 / Niji 6 Only

Not supported in V7, V8, V8.1. Use `--oref` instead in V7/V8/V8.1.

```
--cref https://image.url --cw 100   # V6 / Niji 6
```

`--cw 0–100`. Default `100` = face + hair + clothing. `--cw 0` = face only (good for outfit/hair changes). Multiple URLs allowed: `--cref URL1 URL2`. Best with Midjourney-generated character images, not real photos.

---

## --oref (Omni Reference) — V7 / V8.1

Single image reference for subject identity (person, object, creature). Replaces `--cref` for V7+. **In early V8.1 Alpha `--oref` was temporarily unavailable; it is being restored ahead of the V8 editor / inpaint upgrades — verify availability per release.**

```
--oref https://image.url --ow 100
```

### --ow (Omni Weight)

Range `1–1000` (treat as `0–1000`; some clients accept `0`). Default `100`. One reference image per prompt only.

```
--ow 25–50     # style transfer (e.g., photo → anime); over-specify subject in text
--ow 50–100    # loose resemblance
--ow 100–400   # default-band, recommended ceiling for normal use
--ow 400–1000  # only when fighting high --stylize / --exp; preserves logos, exact face, exact clothing
```

When `--stylize` or `--exp` is high, raise `--ow` proportionally or the reference will be overwhelmed. Don't exceed `--ow 400` unless competing with high stylization.

### --oref Constraints

- One image only
- 2× GPU cost vs base V7 image
- Not compatible with: `--q 4`, Vary Region, Pan, Zoom Out, Draft Mode, Conversational Mode, current Editor inpainting/outpainting (uses V6.1 underneath)
- Multiple subjects: put two characters in a single reference image, name both in text prompt
- For style-shift away from reference, lower `--ow` AND repeat physical features in the text

---

## --p (Personalization / Profiles / Moodboards)

Applies a Personalization profile or Moodboard. V7 profiles carry over to V8.1.

```
--p                    # apply default profile(s)
--p pID                # specific Personalization profile by ID
--p mID                # specific Moodboard by ID
--p code               # specific snapshot code from a previous prompt
--p code1 code2        # blend multiple profile/moodboard codes
--profile <id>         # alternate alias for --p
```

Combine codes by listing them space-separated after `--p`. Moodboard codes do not support per-code weights; SREF codes do (`--sref code1::2 code2::1`).

### Profile Stability Tiers

```
40 ratings      # minimum to start
200 ratings     # stable, reliable
2000+ ratings   # max refinement
```

Multiple named profiles allowed; multiple can be active simultaneously (blended). Rating method since Feb 26, 2026: scrolling-grid selection (replaces 1v1 pairs).

`--sref` and `--p` can be combined in the same prompt for compound aesthetic control.

---

## --no (Negative Prompt)

```
--no item1, item2, item3
```

Single `--no` per prompt. Comma-separated. Equivalent to weighting each term `::-0.5`. Each term is parsed independently — `--no modern clothing` parses as `no modern` + `no clothing` (can trip moderation).

**⚠ V8.1 INCOMPATIBLE (verified on public build):** `--no` returns the hard error `"--no is not compatible with --version 8.1"` and the job fails. This was reported as "limited in early V8.1 Alpha and being restored" in earlier docs, but on the current public V8.1 release `--no` is **rejected outright**. Do not use `--no` with `--v 8.1`. Verify per future build before assuming it has been re-enabled.

### V8.1 Workarounds

Since `--no` is unavailable on V8.1, fall back to one of:
- **Multi-prompt negative weighting** (Discord only, V8.1 website strips it): `still life:: cartoon::-0.5`
- **Stronger positive prompting**: describe what you DO want with concrete physical/lighting/camera detail. V8.1 responds better to positive specifics than to negation in any case.
- **Switch to `--v 7`** for the single generation, which still supports `--no`, then bring the seed/composition forward.
- **Generate, then Vary Region in the Editor** to repaint unwanted elements (Editor uses V6.1 underneath, which supports `--no`).

### Effective Patterns (V7 / V6 / Niji — NOT V8.1)

```
--no anime, cartoon, illustration, painting, drawing, sketch, 3D render   # photoreal lock
--no text, watermark, signature, frame, border                            # clean output
--no blur, depth of field                                                 # flat graphic / vector
--no smile, makeup                                                        # neutral portrait
--no oversaturated, HDR, artificial                                       # natural color
--no busy, cluttered, crowded                                             # minimal composition
--no motion blur, chromatic aberration, jpeg artifacts                    # specific artifacts only
```

### Anti-Patterns

- Using `--no` at all with `--v 8.1` — the job will fail with a hard error.
- `--no ugly, bad, deformed, low quality` — model has no stable concept of these; ineffective.
- Negating qualities inherent to subject (`--no wet` on a rainy scene) — produces incoherence.
- Long lists (>5 items) — diminishing returns.
- First term in `--no` carries the strongest weight; put the most important exclusion first.

If `--no` seems ignored on V7/V6/Niji, lower `--s` below 750 — high stylize overrides exclusions.

---

## --tile

Generates a single seamless-tileable image. Add `--tile`. No value. Use external pattern checker to preview the repeat.

```
seamless watercolor leaves --tile --ar 1:1
```

Upscaling tile images breaks the seam — do not upscale.

**V8.1 known issue:** faint border on some sides of `--tile` outputs that breaks the repeat — fix in progress. For production patterns, generate at base resolution and tile externally.

---

## --raw / --style raw

Disables Midjourney's default aesthetic processing. Output adheres more literally to prompt. Compatible with V5.1+, including V8.1.

```
--raw
--style raw
```

In V8.0, `--raw` was strongly recommended to defeat the over-polished default. In V8.1 the default returned to V7-spirit aesthetics; `--raw` is now optional and used for maximum prompt literality, photoreal accuracy, or to disable interpretive flourishes. Combine with `--s 0–100` for maximum photographic accuracy.

---

## --niji 7 (Anime Model)

```
--niji 7
```

Cleaner linework, sharper eyes/reflections, more literal prompts. Best `--sref` performance of any current model. **`--cref` not supported on Niji 7.** Personalization (`--p`) and Moodboards supported on Niji 7 (since Feb 26, 2026).

`--niji 6` legacy supports style presets:

```
--niji 6 --style expressive   # dynamic, stylized
--niji 6 --style cute         # kawaii
--niji 6 --style scenic       # background-focused
--niji 6 --style original     # classic Niji
```

V7/V8.1 do not support `--style expressive/cute/scenic`.

---

## --draft (Draft Mode)

10× faster, ½ GPU cost. Lower detail. For exploration only, not finals.

```
--draft
```

Draft outputs can be Enhanced (improves base detail without changing composition) or upscaled to a final-quality version. SD V8.1 at full quality already runs as fast as V7 draft mode — `--draft` is most useful in V7.

---

## Conversational Mode

Activated via UI button (chat-bubble icon), not a `--flag`. LLM rewrites/extends your prompt or responds to natural-language follow-ups ("now make it winter"). Voice mode requires Draft Mode active. Conversational Mode is supported in V8.1. Refer to prior images in chat as `image 1`, `image 2`, etc.

---

## Generation Speed Modes

Mode selection is in settings, not a per-prompt parameter (except where noted).

```
--fast    # override: single job in Fast Mode
--relax   # override: single job in Relax Mode
--turbo   # legacy turbo mode (V5/V5.1/V5.2 era)
```

V8.1 supports Fast, Turbo, and Relax (Standard/Pro/Mega plans). Combined `--hd` + `--q 4` is excluded from Relax. `--repeat`, permutations, and Omni Reference are not available in Relax.

---

## --repeat / --r

Runs the same prompt multiple times in one submission. Fast/Turbo only — not Relax. Each run consumes Fast GPU time. Stripped from the saved prompt.

```
--r 4    # Basic: 2–4
--r 10   # Standard: 2–10
--r 40   # Pro / Mega: 2–40
```

Discord requires confirmation click before `--repeat` runs.

---

## Permutation Prompts `{a, b, c}`

Curly braces with comma-separated options expand into multiple jobs. Fast Mode only.

```
A {red, green, yellow} bird                              # 3 jobs
A {red, green} bird in the {jungle, desert}              # 4 jobs (cartesian)
A small house --stylize {100, 500, 1000}                 # 3 jobs varying parameter
A landscape --ar {1:1, 16:9, 9:16}                       # 3 jobs varying ratio
A cat --v {7, 8.1} --niji {7}                            # nest as needed
```

Job caps per submission: Basic 4, Standard 10, Pro/Mega 40 (Turbo: 40). Each permutation consumes Fast GPU time independently.

Use `--sref random` inside a permutation prompt to get a different style code per permutation.

---

## Multi-Prompt `::` Syntax + Weights

`::` is a hard concept break. `space ship` = single concept; `space:: ship` = blended concepts.

```
space:: ship                  # equal blend
space::2 ship                 # space twice as important as ship
still life painting:: fruit::-0.5    # de-emphasize fruit
```

Rules:
- No space before `::`; one space after
- Decimal weights allowed in V4+, Niji 4+, V5+, V6+
- Default weight is `1` if omitted
- Sum of weights must be positive (`fruit::-2` alone fails; `painting:: fruit::-0.5` works)
- `--no x` is equivalent to `x::-0.5`
- Parameters still go at the very end, after multi-prompt fragments
- Officially listed compatibility: V1–V6.1 / Niji 4–6.1; in V7/V8/V8.1 multi-prompts work in Discord but the website strips them — use Discord when needing weighted multi-prompts

---

## Image Prompts (URLs at Front)

```
https://image.url subject description --ar 16:9 --v 8.1
https://url1 https://url2 description --iw 1.5
https://url1 https://url2                      # blend (no text = like /blend)
```

- URL must be public, ending in `.png .jpg .jpeg .gif .webp`
- Image URLs go at the **start** of the prompt
- Multiple URLs allowed; separated by spaces
- Need either ≥2 image URLs or ≥1 image URL + text prompt — a single image URL with no text is invalid
- Removed in V8.0 Alpha; **restored in V8.1**

### --iw (Image Weight)

Range `0–2` in V5/V6/V7/V8.1. Default `1`. (V3 historically supported wider range.) Decimal values valid (`--iw 0.25`, `--iw 1.5`).

```
--iw 0–0.5     # prompt dominant
--iw 0.5–1     # balanced
--iw 1–1.5     # image dominant
--iw 1.5–2     # near-replica style transfer
```

In Discord V6 and earlier, individual image URLs can also be weighted: `URL1::2 URL2::1`.

---

## Reference System Distinctions

| System | Transfers | Use For | V8.1 Status |
|---|---|---|---|
| Image Prompt (URL + `--iw`) | composition, content, color | broad inspiration | Supported |
| `--sref` | style, palette, lighting, mood | aesthetic match | Supported (`--sv 7` default) |
| `--cref` | character identity (face/hair/clothes) | character consistency | V6 / Niji 6 only |
| `--oref` | subject identity (person/object/creature) | subject consistency | V7 / V8.1 (verify alpha availability) |
| Moodboard (`--p mID`) | aggregated personal aesthetic from rated set | persistent style across projects | Supported |
| Personalization Profile (`--p pID`) | user taste from ratings | persistent personal bias | Supported |

Combine: `--oref` (subject) + `--sref` (style) + `--p` (personal aesthetic) is valid. Adjust weights down (`--ow 100`, `--sw 100`) when stacking, otherwise references compete and one wins.

---

## --exp (Experimental Aesthetics)

Range `0–100`. Default `0`. Adds tone-mapping, HDR-like punch, surface detail.

```
--exp 0       # off
--exp 5–10    # subtle, safe with other params
--exp 10–25   # recommended ceiling for combined use
--exp 25–50   # strong, may override --stylize
--exp 50–100  # dominant; overrides --p and high --s
```

`--exp 25+` overrules personalization and high stylize; raise `--ow`/`--sw` to maintain reference fidelity when stacking.

---

## --motion (Video)

Image-to-video. Generates 5-second clips, extendable to 21 seconds (4 extensions × 4s). Web-only. Costs ~8× regular image GPU. Standard, Pro, Mega plans can generate HD video (Fast Mode only). Pro/Mega only for Relax video (SD only).

Video-only parameters (incompatible with most image params):

```
--video                  # animate the prompt's source image
--motion low             # subtle, ambient (default)
--motion high            # dynamic camera + subject motion
--raw                    # reduce default video stylization
--loop                   # first frame = last frame
--end <image_url>        # define ending keyframe
--bs 1 | --bs 2 | --bs 4 # batch size (default 4)
```

URL goes at the start of the prompt, `--video` at the end:

```
https://image.url subject description --video --motion high --ar 16:9
```

Image-prompt parameters are stripped automatically when animating. Video moderation is independent — blocked jobs cost zero.

---

## Editor Tools (V8.1 image → V6.1 editor)

V8.1 images can be loaded into the Editor, but **Editor functionality (Pan, Zoom Out, Vary Region) currently runs on V6.1**. Editor results inherit V6.1 aesthetics. Native V8 Edit / inpaint / outpaint model is on the published roadmap.

```
Vary Region   # erase + regenerate selected area (inpainting)
Pan           # extend canvas in one direction; uses V6.1
Zoom Out      # 1.5× / 2× / Custom (1.0–2.0); uses V6.1
Remix         # edit prompt while varying / panning / zooming
```

Custom Zoom: append a `--zoom 1.0–2.0` value via the Custom Zoom dialog.

Omni Reference images must have `--oref` and `--ow` removed before submitting to Editor.

---

## Describe (Image → Text)

Right-click any image → Describe, or drag image to prompt bar → drop on Describe. Returns 4 prompt variations describing the image. Updated in V8.1 to produce longer, more detailed prompts in V8 prompting style. Click one to load, or `Run all prompts` to generate from all four.

For closer match to source, attach the source image as `--sref` alongside the Describe-generated prompt.

---

## Prompt Shortener

Auto-engages in V8.1 when prompt exceeds the length limit (~1300 characters). Long prompt indicator appears; the system condenses overruns rather than truncating. Output may show increased variability when the shortener fires. To retain control, keep prompts inside the limit manually.

---

## Prompt Structure

V7+/V8.1 reads natural language. Keyword-soup ("8k, masterpiece, beautiful, stunning, ultra-detailed") degrades V8.1 results — strip it.

### Recommended Order (front-loaded)

```
[SUBJECT + identifying details]
[ACTION / POSE]
[ENVIRONMENT / SETTING]
[LIGHTING]
[COMPOSITION / FRAMING / CAMERA]
[STYLE / MEDIUM / ARTIST OR DIRECTOR REFERENCE]
[MOOD / ATMOSPHERE]
[--parameters]
```

Words at the front have stronger influence than words at the end. The first concept anchors the generation.

### Length Targets

```
10–30 tokens    open interpretation; abstract, experimental
30–80 tokens    balanced; most prompts
80–150 tokens   detailed control; specific scenes
150+ tokens     diminishing returns; conflicting signals; auto-shortener may fire
```

### What V8.1 Reads Well

- Full sentences with concrete nouns and verbs
- Specific lighting language (see Lighting block)
- Specific lens / camera body terms
- Single-source artist / director / photographer references
- Quoted text for in-image typography: `sign reading "OPEN"`
- Material descriptors: `oxidized copper with verdigris`, `brushed aluminum`, `worn leather`

### What V8.1 Reads Poorly

- Generic intensifiers (`amazing`, `incredible`, `masterpiece`)
- Quality-claim spam (`8k`, `ultra-detailed`, `award-winning`)
- Contradictions (`dark, bright, moody, cheerful`)
- Long blocks of in-image text (multi-word body copy)
- Specific named real people (use physical description instead)
- Surreal / non-representational requests (V8 tends to "fix" these toward legibility)
- Specific freckles, exact tattoos, exact logos on character refs

---

## Camera & Lens Vocabulary (Effective in V8.1)

```
Camera bodies:
  ARRI ALEXA, ARRI ALEXA Mini, ARRI ALEXA 65
  RED Komodo, RED V-Raptor, RED Helium 8K
  Sony Venice, Sony A7R IV
  Hasselblad H6D, Hasselblad X2D
  Phase One IQ4
  Leica M11, Leica SL3
  Canon EOS R5, Nikon Z8

Lenses / focal lengths:
  24mm f/1.4   wide environmental
  35mm f/2.0   documentary, street
  50mm f/1.4   neutral, classic
  85mm f/1.8   portrait, shallow DOF
  105mm f/2.0  compressed close-up
  135mm f/2.0  tight portrait
  Anamorphic   2.39:1 horizontal flares
  Macro        1:1, extreme close-up
  Tilt-shift   miniature effect

Film stocks / palettes:
  Kodak Portra 400, Kodak Ektar, Kodak Vision3
  Fuji Velvia, Fuji Pro 400H
  CineStill 800T

Shot types:
  extreme wide / wide / medium-wide / medium / medium close-up / close-up / extreme close-up
  low angle / high angle / eye level / Dutch angle / overhead / dolly / tracking / aerial / drone
```

---

## Lighting Vocabulary (Effective in V8.1)

```
Hard / Soft:
  hard directional sunlight, hard key light, no fill
  soft diffused light, softbox, overcast daylight

Patterns:
  Rembrandt, butterfly / paramount, split, loop, broad, short, rim, edge

Time / source:
  golden hour, blue hour, civil twilight
  noon overhead, harsh midday
  moonlight, starlight, bioluminescent glow
  candlelight, firelight, sodium vapor street light, neon, fluorescent
  tungsten warm 3200K, daylight 5600K

Effects:
  volumetric light, god rays, light shafts through fog
  long shadows, hard shadows, no shadows
  practical lighting, motivated lighting
  chiaroscuro, low key, high key
  three-point lighting (key, fill, rim)
  catchlight in eyes
  bounced fill from camera left
```

Specificity beats adjectives: `single overhead key light, no fill, hard shadows` beats `dramatic lighting`.

---

## Color Vocabulary

```
Palettes:
  monochromatic, complementary, analogous, triadic, split-complementary
  pastel, muted, desaturated, vibrant, saturated, neon
  warm-cool contrast, teal-and-orange
  black and white, sepia, duotone

Specifics:
  burnt sienna, prussian blue, payne's grey, cadmium red
  verdigris, ochre, vermilion
  cyan-magenta cyberpunk, purple-orange synthwave, amber-blue Spielberg
```

---

## Style / Movement / Artist References

V8.1 recognizes art movements, named photographers, named directors, and many fine artists. Single named reference > stacked names.

```
Photographers:    Annie Leibovitz, Helmut Newton, Saul Leiter, Vivian Maier, Sebastião Salgado, Gregory Crewdson
Directors:        Roger Deakins, Ridley Scott, Denis Villeneuve, David Fincher, Alfonso Cuarón, Wes Anderson, Christopher Nolan, Terrence Malick
Painters:         Van Gogh, Vermeer, Caravaggio, Mucha, Rembrandt, Beksinski, Hokusai
Illustrators:     Craig Mullins, Greg Rutkowski, Moebius, Syd Mead, Ralph McQuarrie
Movements:        Art Deco, Art Nouveau, Bauhaus, Brutalism, Vienna Secession, Ukiyo-e, Bauhaus, Memphis design, Abstract Expressionism
```

Avoid named real-person likenesses (actors, public figures) for subjects — produces uncanny-valley results and may be moderation-blocked. Describe the person physically instead.

---

## Text Rendering

V8.1 handles short in-image text well when wrapped in straight quotes:

```
neon sign reading "OPEN"
storefront with "BAKERY" in serif gold lettering
poster with "JAZZ NIGHT" in art deco type
```

Lower `--s` to `0–25` for maximum text legibility. Single words and 2–4 word phrases work; long sentences and stylized fonts remain unreliable. For precise typography, generate without text and overlay in post.

---

## V8.1 Known Limitations / Quirks

- Hands and complex body positions still imperfect; better than V8.0 but not solved.
- Small distant faces clearer in HD mode; use `--hd` for crowd / wide-shot legibility.
- Occasional residual blur or pixelation in some outputs (acknowledged, fix in progress).
- `--tile` outputs may show a faint border on some edges that breaks seamless repetition.
- `--no` parameter is **rejected** on V8.1 (`"--no is not compatible with --version 8.1"`); was reported "limited in early V8.1 Alpha and being restored" but remains incompatible on the current public release. `--oref` is supported but routes through V6.1 for Editor operations.
- `--oref` and Editor still route through V6.1 underneath until the V8 edit / inpaint / outpaint models ship.
- Limited abstraction — V8 tends to "fix" surreal prompts into something more legible. For surreal work consider V7 or Niji 7.
- Age drift carryover from V8.0 — subjects sometimes render older than specified; restate age explicitly in the subject phrase.
- `--weird` not fully reproducible with `--seed`.
- `--q 4` incompatible with `--oref`.
- Multi-prompt `::` weights may be stripped on the website Imagine bar; use Discord for reliable weighted multi-prompts in V7/V8.1.
- Personalization Global Profile must be unlocked (rate ≥40 images) before V8.1 will generate.
- V8.1 does not yet have its own native upscaler — `Run as HD` is a re-render, not an upscale.

---

## Compatibility Matrix

| Parameter | V8.1 | V7 | Niji 7 | Niji 6 |
|---|---|---|---|---|
| `--ar` | ✅ | ✅ | ✅ | ✅ |
| `--s 0–1000` | ✅ | ✅ | ✅ | ✅ |
| `--c 0–100` | ✅ | ✅ | ✅ | ✅ |
| `--w 0–3000` | ✅ | ✅ | ✅ | ✅ |
| `--q 1/2/4` | ✅ | ✅ | ✅ | ✅ |
| `--seed` | ✅ ~99% | ✅ | ✅ | ✅ |
| `--raw` / `--style raw` | ✅ | ✅ | partial | partial |
| `--sref` URL | ✅ (`--sv 7`) | ✅ (`--sv 6`) | ✅ best | ✅ |
| `--sref` numeric code | ✅ (`--sv 4`/`6`) | ✅ | ✅ | ✅ |
| `--sw 0–1000` | ✅ | ✅ | ✅ | ✅ |
| `--cref` / `--cw` | ❌ | ❌ | ❌ | ✅ |
| `--oref` / `--ow` | ✅* | ✅ | ❌ | ❌ |
| `--p` / `--profile` | ✅ | ✅ | ✅ (Feb 2026+) | optional |
| `--no` | ❌ rejected | ✅ | ✅ | ✅ |
| `--iw 0–2` | ✅ | ✅ | ✅ | ✅ |
| Image prompts | ✅ | ✅ | ✅ | ✅ |
| `--tile` | ⚠ border bug | ✅ | ❌ | ❌ |
| `--hd` / `--sd` | ✅ | ❌ | ❌ | ❌ |
| `--exp 0–100` | ✅ | ✅ | ✅ | partial |
| `--draft` | ✅ | ✅ | ✅ | ✅ |
| `--repeat` / `--r` | ✅ Fast/Turbo only | ✅ | ✅ | ✅ |
| Permutations `{a,b}` | ✅ Fast only | ✅ | ✅ | ✅ |
| Multi-prompt `::` | Discord only | Discord only | ✅ | ✅ |
| `--video` / `--motion` | image-to-video web | ✅ | ❌ | ❌ |
| `--style expressive/cute/scenic` | ❌ | ❌ | ❌ | ✅ |
| Conversational Mode | ✅ | ✅ | — | — |
| Editor (Pan/Zoom/Vary) | ✅ but uses V6.1 underneath | ✅ V6.1 | — | ✅ |

*\* `--oref` was limited in early V8.1 Alpha; supported on current build. `--no` is rejected on V8.1 with a hard error — use V7 or describe what you want positively.*

---

## Cost / Mode Compatibility (V8.1)

```
Default                  ~1× (HD baseline; ~1.33 GPU min)
--sd                     <1× (~<1 GPU min)
--q 2                    2× compute
--q 4                    4× compute (incompatible with --oref)
--hd + --q 4             highest cost; excluded from Relax
--oref                   2× compute; not Relax/Draft/Conversational/--q 4
--sref / Moodboard       cheaper on --sv 7 (4× cheaper than --sv 6)
--sv 6 with sref         4× cost in V8 Alpha; verify on V8.1
--repeat                 N× per repeat; Fast/Turbo only
Permutations             N× per permutation; Fast only
Video                    ~8× image cost
HD video                 ~3.2× SD video cost; Pro/Mega only
```

---

## Aspect Ratio Use Map

```
1:1     square          social profile, icons, tile bases, balanced product
4:5     vertical-soft   Instagram feed portraits, mobile-friendly editorial
5:4     near-landscape  desktop wallpaper, presentation slides
2:3     vertical        photography prints, magazine covers, Pinterest
3:2     classic-photo   landscape print, photojournalism
16:9    widescreen      YouTube, video thumbnails, environmental establishing shots
9:16    vertical video  Stories / Reels / TikTok / phone wallpaper
21:9    cinematic-wide  film stills, anamorphic environments, panoramas
6:11    tall portrait   phone wallpapers, vertical posters
7:4     near-HD         smartphone, HD TV
1.91:1  social-link     LinkedIn / Facebook share cards
```

Wide ratios force environmental composition; tall ratios force subject-centric framing. Choose ratio before generating — changing later via Pan/Zoom uses V6.1 and shifts aesthetic.

---

## Decision Heuristics

```
Photoreal portrait:        --v 8.1 --ar 4:5 --s 50–150 --raw
Photoreal product:         --v 8.1 --ar 1:1 --s 0–50 --raw --hd --q 2
Cinematic environment:     --v 8.1 --ar 21:9 --s 150–300
Concept art / illustration:--v 8.1 --ar 16:9 --s 400–700
Surreal / experimental:    --v 7 --w 500–1500 --c 25–50 (V8 fixes surrealism)
Anime / manga:             --niji 7 --ar 3:4 --s 100
Character consistency:     --v 8.1 --oref URL --ow 100–300 (single subject)
Style consistency:         --v 8.1 --sref URL --sw 100–200 OR --p mID
Logo / vector:             --v 8.1 --raw --s 0–50 (or --v 7 --raw --s 0–50 --no blur, depth of field)
Text-heavy poster:         --v 8.1 --raw --s 0–25 ("quoted text")
Tile / pattern:             --v 8.1 --tile --ar 1:1 (verify border, do not upscale)
Story / extended canvas:   generate base → Editor → Pan/Zoom Out (V6.1)
Video:                     image-to-video → --motion low | high --raw
Rapid ideation:            SD V8.1 (already as fast as V7 draft) OR V7 --draft
```

---

## Workflow Templates

### Photoreal portrait
```
Close-up portrait of a [age] [identity] with [physical features], [expression], [pose], [environment], [lighting pattern] lighting, shot on [camera] with [lens], [mood] atmosphere
--ar 4:5 --s 100 --raw --v 8.1
```
(For `--no anime, cartoon, illustration` exclusions, use `--v 7` instead — `--no` is rejected on V8.1.)

### Cinematic wide
```
Wide cinematic shot of [subject] in [environment], [time of day], [weather/atmosphere], in the style of [director] cinematography, captured on [camera] with [lens], [color palette]
--ar 21:9 --s 200 --v 8.1
```

### Product
```
[Product] on [surface], [background], [lighting setup], commercial photography, high detail, [brand aesthetic]
--ar 1:1 --s 25 --raw --hd --q 2 --v 8.1
```
(For `--no clutter, hands, text` exclusions, swap to `--v 7` — `--no` is rejected on V8.1.)

### Anime character
```
[Character description with hair color, eye color, outfit details], [pose], [expression], [background], [color palette]
--niji 7 --ar 3:4
```

### Character consistency series
```
[Scene / action] --oref [URL of base character] --ow 100 --v 8.1 --ar 4:5
[Different scene] --oref [same URL] --ow 100 --v 8.1 --ar 16:9
```

### Style-locked series
```
[Subject A] --sref [code] --sw 150 --v 8.1
[Subject B] --sref [code] --sw 150 --v 8.1
[Subject C] --sref [code] --sw 150 --v 8.1
```

### Subject + Style combined
```
[Description] --oref [subject URL] --ow 150 --sref [style URL] --sw 100 --v 8.1
```

---

## Anti-Patterns (Do Not Use)

```
beautiful, stunning, masterpiece, 8k, ultra-detailed, intricate, trending on artstation
```

These degrade V7/V8/V8.1 outputs. Replace with concrete physical / lighting / camera detail.

```
--no ugly, bad, deformed, low quality
```

Ineffective — model has no stable concept of these. Use specific artifacts (`--no motion blur, jpeg compression`) instead.

```
single image URL with no text prompt
```

Invalid. Either add text or add a second image URL.

```
--cref URL --v 8.1
--cref URL --niji 7
```

`--cref` is V6 / Niji 6 only. Use `--oref` for V7 / V8.1; for Niji 7 there is currently no character-reference equivalent.

```
--oref URL --q 4
```

Incompatible — `--q 4` with `--oref` will fail or ignore one parameter.

```
--sw 200 --p mID
```

`--sw` is not compatible with Moodboards. Apply weight via `--sref` codes instead, or use Personalization profile alone.

```
multi --no commands
```

Single `--no` per prompt with comma-separated terms. `--no x --no y` is invalid.

```
--no anything --v 8.1
```

Rejected on V8.1 with `"--no is not compatible with --version 8.1"`. Drop the `--no` (use positive description instead) or switch to `--v 7`.

```
1.5:1
```

Decimals not allowed in `--ar`. Use `3:2` or `150:100`.

---

## Quick Parameter Reference Card

```
MODEL
  --v 8.1 / --v 8 / --v 7 / --niji 7 / --niji 6
  --hd / --sd                       (V8.1 resolution toggle)
  --raw / --style raw

GEOMETRY
  --ar W:H                          0 decimals; default 1:1
  --tile                            seamless single tile

AESTHETIC
  --s 0–1000      default 100
  --c 0–100       default 0
  --w 0–3000      default 0; weak vs --seed
  --exp 0–100     default 0
  --q 1/2/4       default 1; no q3; q4 ≠ --oref

REFERENCES
  image URL (front of prompt)
  --iw 0–2        default 1
  --sref URL | code | random
  --sw 0–1000     default 100; ≠ Moodboards
  --sv 4 | 6 | 7  sref engine version
  --oref URL      single image; V7/V8.1
  --ow 0–1000     default 100
  --cref URL      V6 / Niji 6 only
  --cw 0–100      default 100
  --p / --profile [pID|mID|code ...]

CONTROL
  --seed 0–4294967295
  --no item1, item2, item3        (single --no per prompt; ❌ NOT V8.1 — V7/V6/Niji only)
  --repeat / --r 2–40             Fast/Turbo only
  {a,b,c} permutations            Fast only

VIDEO
  https://startURL ... --video --motion low|high --raw --loop --end <url> --bs 1|2|4

WEIGHTS (Discord)
  concept::N                      hard break + weight
  URL::N                          per-image weight (image prompts / sref)
```

---

## Final Directives for Prompt Construction

1. Write subject first. Front-load the most important concept.
2. Use natural language sentences, not keyword lists.
3. One named artist/director/photographer reference is enough; stacking dilutes.
4. Pick `--ar` before composition, not after.
5. Lower `--s` for literal output; raise for artistic interpretation.
6. Quote in-image text. Lower `--s` for legibility.
7. Use `--raw` when default V8.1 aesthetic is too polished.
8. Stack references conservatively: `--ow 100` + `--sw 100` + `--p` simultaneously, then adjust.
9. Use `--seed` for series consistency, `--c` for controlled variance, `--w` only for exploration.
10. Do not use `--no` with `--v 8.1` — it is rejected with a hard error. For exclusions on V8.1, rely on positive prompting; if `--no` is essential, switch to `--v 7`.
11. Generate with `--hd` from start when HD output matters — `Run as HD` is a re-render not an upscale.
12. For surreal / abstract work, prefer `--v 7` over `--v 8.1`.
13. For anime / manga, prefer `--niji 7` over `--v 8.1`.
14. For character consistency on Niji 7, no native `--cref` / `--oref` — use detailed text + `--sref` of a base character image.
15. Never use real-person names as subjects; describe physically instead.