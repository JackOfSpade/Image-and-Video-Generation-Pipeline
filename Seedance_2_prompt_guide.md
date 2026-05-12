Reference syntax
- Up to 9 images + 3 videos (15s total) + 3 audio per generation
- Auto-labeled `@Image1…@Image9`, `@Video1…@Video3`, `@Audio1…@Audio3`
- `@Image` = visual anchor (Face ID, wardrobe, scene composition, color script)
- `@Video` = motion anchor (camera path, choreography, pacing, transitions)
- `@Audio` = rhythm anchor (lip-sync, beat matching, BGM mood)
- Example: *"Use the girl from @Image1 as protagonist. Replicate camera movement from @Video2. Sync dialogue to @Audio1."*

---

Reference roles
- Only ONE reference defines each of: identity, lighting direction, color palette, camera behavior
- Hero identity image: front three-quarter portrait, even light, neutral expression, plain background
- Style references: 3–5 small patches (lighting swatch, palette block, texture sample)
- Motion reference: 5–10s, gentle luminance, trimmed to the move segment
- Audio reference: ≤15s, clean non-overlapping sound, no heavy reverb
- Reference strength: visual ID 0.80–0.85, mood boards 0.75–0.80, motion 0.50–0.60, audio 0.40–0.50

---

Before multi-shot, run 4-second test: `@Image1` + minimal action + locked camera. If face drifts, re-source reference.

---

For human face references: generate portrait via Nano Banana Pro / Seedream / Midjourney (front view, neutral expression, studio lighting, plain background) and upload as `@Image1`.

---

Prompt stack
- Default order: Subject → Action → Camera → Style → Constraints
- Animation override: lead first line with aesthetic/style (Style moves to front)
- 60–100 words
- Front-load priority directives
- For multi-shot, wrap inside shot block

---

Shot blocks
```
Montage, multi-shot, don't use one camera angle or single cut, [style anchors]
Shot 1 [0–4s]: [camera + subject + action + lighting]
Shot 2 [4–9s]: [camera + subject + action + lighting]
Shot 3 [9–15s]: [camera + subject + action + lighting]
```
- Open with "Montage, multi-shot, don't use one camera angle or single cut"
- Transition language: "hard cut to", "seamless morph into", "whip pan to", "match cut on [shape]"

---

Use `[00:00-00:05]` or `(0-2s)` for cuts. Nest inside shot blocks.

---

For action / performance / fight: `Single continuous shot 15s: [camera position] [subject action beat by beat] [VFX inline] [escalation arc]`

---

Match-cut: cut between two shots sharing visual similarity (shape, color, motion, composition).

---

Camera movement
- Compound moves allowed; sequence them temporally — *"start: slow dolly-in, then: gentle pan right for the final 2 seconds"*
- Pair each movement with speed modifier (slow/medium/fast) and distance (1–2 ft)
- Reliable: dolly in/out, tracking shot, follow shot, pan left/right, tilt up/down, crane up/down, orbit / 360°, handheld (pair with "subtle sway"), steadicam, gimbal, Hitchcock zoom / dolly-zoom / vertigo effect, whip pan, rack focus, fisheye, low angle, high angle, Dutch angle, eye level, POV / first-person
- Skip f-stops, ISO numbers, exact mm focal lengths

---

Only ONE element fast at a time. Slow everything else.

---

Camera models: "Sony Venice", "Sony A7S3", "ARRI Alexa", "anamorphic lens".

---

For Hitchcock zoom / compound moves / signature camera: upload 5–10s reference video, tag `@Video1`, prompt *"Replicate camera movement from @Video1."* Trim to the move segment.

---

Lighting
- Always include a lighting description
- Phrasings: "golden hour backlight", "three-point lighting with warm key", "chiaroscuro contrast", "neon-lit interior", "Roger Deakins lighting", "Euphoria color grade", "color temperature: warm/cool"

---

Style anchors
- Max one director + one camera + one film-stock reference
- Anchors: "Wes Anderson symmetry", "Blade Runner color grading", "Akira Kurosawa cinematography", "Roger Deakins lighting", "Studio Ghibli–inspired", "35mm Kodak film", "Euphoria color grade", "Hans Zimmer sound design"

---

Label speaker emotion before quoted line: *she softly whispers "Just looking at you."*

---

Lip-sync
- Paste exact transcript of spoken words into prompt
- Medium close-up framing
- Locked camera
- Front-facing or three-quarter face angle only
- 5–10 words per line
- Reference recorded at ~80% natural speaking speed
- No head-movement instructions
- Supported languages: Mandarin, English, Japanese, Korean, Spanish, French, German, Portuguese

---

Audio under 15s. Clean, non-overlapping, no heavy reverb.

---

Audio mix
- Narrative: *"Dialogue clean and prominent, music low, ambient subtle."*
- Music video: *"Music leads, ambient secondary, no dialogue."*
- Prevent abrupt cutoff: *"Music: low piano note enters at 3s, resolves on last frame. Silence holds final 0.5s."*

---

Beat sync
- Anchor events to beats: *"Scene change at 3s mark (first beat), character gesture at 6s mark (second beat), transition at 9s mark (third beat). @Audio1 provides rhythm and pacing."*
- BPM 90–140
- Multi-outfit / multi-shot music video: chain `@Image1` through `@Image7` cutting to keyframes and rhythm of `@Video1`

---

For I2V: prompt motion only + "maintain character consistency".

---

FLF2V
- Upload start AND end image
- Match aspect ratios across both frames AND output
- Keep both frames at similar shot scale
- Consistent lighting and color grade between frames
- Use asymmetric compositions

---

Extension: set generation length to extension duration only, not total final length. Re-run drifted segments before chaining.

---

Video editing
- Upload video + reference image
- Pattern: *"Keep the original motion and camera work from @Video1. Change the character's hair to long red. Add the shark from @Image1 slowly rising in the background."*

---

Animation
- Lead first line with aesthetic: *"Cinematic stylized 3D animation, photorealistic environment, stylized characters"* / *"Studio Ghibli–inspired watercolor, visible brush strokes"*
- Use clean keyframe image as both first frame and style reference
- Describe physics with same precision as character action (particle simulation, dust, energy VFX, cloth dynamics)
- 2D anime: shot type FIRST, atmospheric detail in every prompt, use word "cut" for hard transitions vs morphs
- Realistic monster/creature: add *"no 3D, no cartoon, no VFX"*
- Inline VFX bracket notation: `[VFX: branching electric circuits pulsing with white-blue current]`

---

Constraints
- Append at END of every prompt
- Targeted terms tied to actual failure; the full default block below also works as a baseline
- Default: `4K, Ultra HD, rich details, sharp clarity, cinematic texture, natural colors, soft lighting, no blur, no ghosting, no flickering, stable picture.`
- Jitter: *"Avoid jitter. Stable picture."*
- Face drift: *"Face stable, no deformation."*
- Bent limbs: *"Avoid bent limbs. Natural smooth movements."*
- Mirrored features: *"no mirrored features, no missing piercings"*
- Plastic CG skin: *"no 3D, no cartoon, no VFX"*

---

Lock seed integer when iterating. Change ONE variable per generation. Lock for character block; allow drift on scene variations.

---

Iterate on Fast tier. Re-render winners on Standard.

---

Pipeline
1. Seedance Fast — prototyping
2. Seedance Standard — winners
3. Topaz Video AI Starlight or Magnific Video Upscaler — 4K upscale
4. After Effects — Topaz Enhancement + Motion Deblur plugins on timeline
5. DaVinci Resolve Studio — color grade
6. ElevenLabs — dialogue post-overdub if needed

---

Failure modes

| Failure | Fix |
|---|---|
| Flicker / temporal artifacts | Reduce visual complexity in low-contrast areas; reduce reference count; lock seed |
| Hand/finger distortion | Reframe wider, fewer thin lines, slower motion |
| Identity drift across shots | Reduce to 2 strong references; add immutable anchors ("beanie, nostril ring") in every shot description |
| Mirrored features | Add "no mirrored features, no missing piercings" |
| Stylization shift mid-clip | Lock seed for character block |
| Face warping during dialogue | Remove head-movement cues; locked MCU framing |
| Garbled text / logos | Composite text in post |
| Plastic skin on creatures | Add "no 3D, no cartoon, no VFX" |
| Generation flagged / blocked | Use AI-generated portraits; rephrase brand names; switch to text-only mode |
| Audio cuts off abruptly | Add "silence holds final 0.5s" |
| Shaky despite "stable" | Slow ALL but one element |

---

Composite all text, signs, logos, and subtitles in post.

---

When prompts are blocked or output is weak: write scene description in Chinese, keep dialogue and on-screen text in English.
