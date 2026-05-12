# Seedance 2.0 Omnireference Mode — Cinematic Prompt Engineering Specification (LLM-Consumable)

**TL;DR**
- Seedance 2.0 omnireference (a.k.a. "reference-to-video" / "omni reference" / "R2V" / "Universal Reference Mode") accepts up to 12 reference files (≤9 images, ≤3 videos, ≤3 audio) and binds each to a functional role (IDENTITY / STYLE / MOTION / AUDIO) via inline `@ImageN` / `@VideoN` / `@AudioN` (or `[ImageN]`) tags inside a natural-language prompt; the model returns 4–15 s of 480p/720p/1080p MP4 with native synchronized audio.
- The canonical cinematic prompt structure is a 6-slot ordered template: `SUBJECT → ACTION → ENVIRONMENT → CAMERA → STYLE → CONSTRAINTS`, 60–100 words, with exactly ONE primary camera move, lighting always named, and subject-motion separated from camera-motion.
- Optimal omnireference output requires: (1) one tightly-cropped IDENTITY image per character, (2) 1–2 STYLE swatches for color/lighting, (3) at most one MOTION video clip (3–8 s) for camera or choreography, (4) explicit role-binding language ("Primary identity anchor: @Image1. Camera from @Video1."), and (5) a negative-constraints tail.

---

## 1. MODEL_OVERVIEW

```yaml
model_family: Seedance
canonical_version: Seedance 2.0
release_date: 2026-02-10 (ByteDance Seed)
vendor: ByteDance Seed
consumer_brand_names: [Jimeng (即梦), Dreamina, Doubao]
architecture: unified multimodal audio-video joint diffusion transformer
benchmark: SeedVideoBench-2.0 (internal)
modes:
  - text_to_video           # T2V
  - image_to_video          # I2V (first-frame and optional last-frame)
  - reference_to_video      # R2V == OMNIREFERENCE MODE (this spec)
  - video_edit              # add / remove / modify elements
  - video_extend            # forward or backward continuation
tiers:
  - standard      # max 1080p (provider-dependent), highest fidelity
  - fast          # max 720p (480p->720p upscaled on some hosts), lower latency/cost
model_ids:
  byteplus:   [dreamina-seedance-2-0-260128, dreamina-seedance-2-0-fast-260128]
  volcengine: [doubao-seedance-2-0-pro-260215]
  fal_ai:     [bytedance/seedance-2.0/reference-to-video, bytedance/seedance-2.0/fast/reference-to-video]
  replicate:  [bytedance/seedance-2.0]
  enterprise_face_input: [fal-ai/seedance-2/enterprise/fast/reference-to-video]   # face references gated
native_audio: true            # generated jointly; do NOT post-stitch
content_restriction_active:
  real_human_face_uploads_blocked_since: 2026-02-10
  notes: "Reference images that contain identifiable real human faces are rejected by the facial-detection safety filter. Use AI-generated, illustrated, stylized, or 3D-rendered character sheets instead."
known_failure_categories: [identity_drift, temporal_flicker, limb_distortion, camera_jitter, color_palette_drift, face_averaging]
```

---

## 2. OMNIREFERENCE_MODE_SPEC

### 2.1 Input slots

```yaml
references:
  images:
    max_count: 9
    formats: [jpeg, png, webp, bmp, tiff, gif]
    max_size_mb: 30
    pixel_range: 409600..927408    # width*height; e.g. 640x640 .. 834x1112 inclusive
    recommended_resolution: 1024x1024 OR 1024x1536
    note: "Model downsamples; oversize wastes bandwidth without quality gain."
  videos:
    max_count: 3
    formats: [mp4, mov]
    duration_each_seconds: 2..15
    total_duration_seconds: <=15
    max_size_mb: 50
    purpose: motion / camera / pacing / VFX reference
  audio:
    max_count: 3
    formats: [mp3, wav]
    duration_each_seconds: 2..15
    total_duration_seconds: <=15
    max_size_mb: 15
    purpose: rhythm / music / SFX / dialogue lipsync target
  total_files_hard_cap: 12
  must_include_at_least: "1 image OR 1 video (audio-only references are rejected)"

reference_role_taxonomy:    # the model infers these from prompt language; you must declare them
  IDENTITY:  "Face, character, product, logo, or object that must stay recognizable every frame."
  STYLE:     "Color palette, lighting mood, texture, art-direction (does NOT carry subject identity)."
  MOTION:    "Camera language and/or choreography to imitate (does NOT carry subject identity)."
  ENVIRONMENT: "Location, background, set-piece architecture."
  AUDIO:     "Music, ambient bed, dialogue cue, beat for cut-synchronization."

practical_sweet_spot:
  images: 1..5      # diminishing returns past 5
  videos: 0..1
  audio:  0..1
  rationale: "Using all 12 slots overconstrains the solver; output quality drops."
```

### 2.2 Output parameters (canonical schema, normalized across fal / Replicate / BytePlus)

```yaml
output:
  prompt:           string                              # required; see PROMPT_SCHEMA
  image_urls:       string[]   # 0..9                   # IDENTITY/STYLE/ENV references
  video_urls:       string[]   # 0..3                   # MOTION references
  audio_urls:       string[]   # 0..3                   # AUDIO references
  resolution:       enum[480p, 720p, 1080p]             # default 720p; 1080p standard-tier only
  duration:         enum[auto, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]   # seconds; "auto" / -1 = model picks
  aspect_ratio:     enum[21:9, 16:9, 4:3, 1:1, 3:4, 9:16, auto, adaptive]
  generate_audio:   bool       # default true; audio generation included at no extra cost
  seed:             int        # optional; returned in response; reuse for variation control
  end_user_id:      string     # provider-specific; abuse-tracking only

returned_object:
  video:
    url:           string
    content_type:  video/mp4
    file_size:     int
  seed:            int

generation_pattern: asynchronous job (submit -> poll OR webhook -> download)
typical_latency_seconds: 30..120
```

### 2.3 Reference tag syntax (CRITICAL)

```yaml
canonical_tag_syntax:
  primary: "@Image1, @Image2, ..., @Image9, @Video1, @Video2, @Video3, @Audio1, @Audio2, @Audio3"
  bracketed_variant: "[Image1], [Video1], [Audio1]   # accepted on fal / Replicate"
  bytedance_official_natural_language: "Image 1, Image 2, ... Image N, Video 1 ..., Video 2 ..."   # spaces allowed
  case_sensitivity: NOT case-sensitive but @Image1 form is recommended for parsing reliability

ordering_contract:
  - "The N in @ImageN is assigned by upload order. The first file uploaded is N=1."
  - "Always restate the binding in the prompt: 'IDENTITY: @Image1. STYLE: @Image2. CAMERA: @Video1.'"
  - "Without explicit role binding the model averages all references → identity blending and color drift."

prohibited_in_tags:
  - "Do not use weights, parentheses, or A1111-style attention syntax (e.g. (@Image1:1.4))."
  - "Do not nest tags or wrap them in quotes."
  - "Do not reference a non-uploaded N (e.g. @Image5 when only 3 are uploaded) — undefined behavior."
```

### 2.4 Mode-selection decision tree

```
IF (have ONE start image only) AND (no character continuity across multi-scene needed)
   -> use IMAGE_TO_VIDEO (first/last frame mode)
ELIF (need character consistency across multiple shots) OR (need motion transfer) OR (need style+identity separation)
   -> use OMNIREFERENCE (reference_to_video)
ELIF (no visual reference, only language)
   -> use TEXT_TO_VIDEO
```

---

## 3. PROMPT_SCHEMA (Cinematic, Omnireference)

### 3.1 Canonical 6-slot template (ByteDance/Volcengine official ordering)

```
{SUBJECT_BINDING} . {ACTION} . {ENVIRONMENT_AND_LIGHTING} . {CAMERA} . {STYLE} . {NEGATIVE_CONSTRAINTS}
```

```yaml
slot_definitions:
  SUBJECT_BINDING:
    required: true
    must_contain:
      - explicit identity anchor: "Primary identity: @Image1, <one-sentence visual description of subject>"
      - if multi-subject: "@Image1 is the left character, @Image2 is the right character."
    rules:
      - "ONE identity image per character. Never two photos of the same face."
      - "Tight-crop reference. Neutral background. Consistent lighting between reference and intended scene."
      - "Verbalize 3-5 fixed traits (hair color, garment, age band) — these become the model's identity contract."

  ACTION:
    required: true
    rules:
      - "Present tense, ONE primary verb per shot."
      - "1–3 action beats max. Beyond 3 beats produces incoherent choreography."
      - "Quantify intensity (slowly, abruptly, briefly) — not 'epic' / 'dynamic'."
      - "Separate from camera motion. Subject-verbs and camera-verbs MUST be in different clauses."

  ENVIRONMENT_AND_LIGHTING:
    required: true                                # lighting has the single largest output-quality impact
    must_contain:
      - location (specific, not generic)
      - time of day OR named lighting (golden hour, blue hour, overcast, neon, candlelight, rim light)
      - one atmospheric modifier (fog, rain, dust motes, haze) — optional but high-leverage
    rules:
      - "If only one element can be added to improve quality, add lighting."
      - "Pair color temperature explicitly: 'warm tone' / 'cool palette' / 'desaturated'."

  CAMERA:
    required: true
    must_contain:
      - shot size:     enum [extreme close-up, close-up, medium close-up, medium, medium wide, wide, extreme wide, aerial, POV, OTS]
      - movement:      ONE of CAMERA_MOVES (§4.2)
      - pacing word:   enum [imperceptible, slow, gentle, smooth, gradual, controlled, dynamic, swift]
      - optional lens: enum [wide-angle, normal, telephoto] OR vague focal feel "85mm feel"
    rules:
      - "ONE primary camera instruction per generation. Compound moves must be expressed as ordered beats: 'Start: slow dolly-in. Then: gentle pan right for the final 2 seconds.'"
      - "Avoid technical specs (f/2.8, ISO, 24fps inside prompt). Use rhythmic descriptors. 24fps applies natively for film look."
      - "Bind motion: 'Camera movement follows @Video1.' if a motion reference is supplied."

  STYLE:
    required: true
    must_contain (2-3 of):
      - film look:      [35mm film grain, anamorphic, ARRI ALEXA aesthetic, 16mm, Super8, digital cinema]
      - color grade:    [teal and orange, bleach bypass, warm vintage, cyan-magenta neon, desaturated, high-contrast B&W]
      - genre anchor:   [film noir, A24, Wes Anderson symmetry, neo-noir, Blade Runner aesthetic, Hollywood blockbuster, documentary, music video]
      - DOF:            [shallow depth of field, deep focus, rack focus]
    rules:
      - "Style block goes near the end. Two strong anchors beat ten adjectives."
      - "Never use bare 'cinematic' — always compound: 'cinematic film tone, 35mm, warm'."

  NEGATIVE_CONSTRAINTS:
    required: true_for_cinematic
    canonical_form: "avoid {term1}, avoid {term2}, ..."
    mandatory_for_character_shots: [jitter, bent limbs, identity drift, face distortion, hand deformation]
    mandatory_for_long_takes:     [temporal flicker, color palette shift, scene cut (if single-take intended)]
    optional: [oversaturated colors, static camera (when motion is desired), text distortion, watermark]

length_constraints:
  optimal_word_count: 60..100
  hard_min: 35
  hard_max: 200    # past 200 the model averages contradictory clauses
  note: "Image-to-video and omnireference variants can run shorter (35-60 words) because the image carries subject info."
```

### 3.2 Five-part omnireference variant (practitioner-validated, alternative ordering)

Use when multiple roles must be explicit:

```
1. SUBJECT IDENTITY:  "Primary identity: @Image1 — <traits>. Secondary subject: @Image2 — <traits>."
2. SCENE:             "<location>, <time-of-day>, <weather>, <atmosphere>."
3. ACTION BEATS:      "<beat 1>. <beat 2>. <beat 3>."        # 1–3 only
4. CAMERA DIRECTION:  "<shot size>, <one movement>, <pacing>. <lens/focal feel>. (Anchor: follow @Video1 pacing.)"
5. CONSISTENCY LOCK:  "Maintain facial proportions and wardrobe from @Image1 throughout. No face distortion. No color palette shift. No identity drift."
```

### 3.3 Multi-shot / timeline-prompt variant (for 6–15 s outputs containing internal cuts)

```
SHOT 1 [0–Ns]: {shot size}, {camera move}, {action beat}, {lighting}.
SHOT 2 [N–Ms]: {shot size}, {camera move}, {action beat}, {lighting}.
SHOT 3 [M–15s]: {shot size}, {camera move}, {action beat}, {lighting}.

(Always close with) Total: 15s / {n} shots / {aspect_ratio}.
```

Rules:
- Each shot must reintroduce shot size + camera behavior.
- Use the word "Lens switch." or "Cut to:" between shots if you want a hard cut.
- For a single uninterrupted shot prepend: `Single continuous shot. No cuts.`

---

## 4. VOCABULARY (Enumerated)

### 4.1 SHOT_SIZES (model-recognized)

```
extreme_close_up        # eyes, lips, small product detail
close_up                # head and shoulders, single hand
medium_close_up         # chest up
medium                  # waist up
medium_wide / cowboy    # mid-thigh up
wide / full             # full body in environment
extreme_wide / establishing
aerial / drone / bird's_eye
overhead / top_down / god's_eye
low_angle               # hero shot
high_angle              # vulnerability / overview
dutch_angle / canted    # tension (use sparingly; can destabilize)
POV / first_person      # first-person; must explicitly add "the camera IS the eyes, no cuts, no zoom"
over_the_shoulder / OTS
two_shot
```

### 4.2 CAMERA_MOVES (model-recognized, one per prompt)

```
push_in / dolly_in          # emotional emphasis
pull_out / dolly_out        # reveal / context
pan_left / pan_right        # lateral rotation
tilt_up / tilt_down         # vertical rotation
truck / lateral_tracking    # parallel side movement
crane_up / crane_down       # vertical translation
tracking_shot / follow      # subject-locked
orbit / arc                 # 90°–360° around subject
aerial / drone_shot         # high altitude
handheld                    # micro-shake (cinematic, organic)
steadicam / gimbal          # smooth tracking
fixed / locked_off / static # tripod
rack_focus                  # focus transfer between planes
dolly_zoom / hitchcock_zoom / vertigo_effect    # background warps; subject size constant
whip_pan / snap_pan
push_and_orbit              # COMPOUND — write as beats
FPV_dive                    # aggressive flying perspective
```

### 4.3 LENSES_AND_OPTICS

```
wide_angle (24-28mm feel)
normal (35-50mm feel)
telephoto (85mm+ feel)
macro_lens
fisheye_lens
anamorphic_lens
shallow_depth_of_field / bokeh
deep_focus
lens_flare / anamorphic_flare / amber_lens_flare
chromatic_aberration (subtle; useful on POV)
halation_on_highlights
motion_blur
focus_breathing
```

### 4.4 LIGHTING (highest-leverage; always specify)

```
# Natural / time of day
golden_hour                 # warm low sun
blue_hour                   # cold dusk/dawn
overcast / soft_diffused    # even, low-contrast
harsh_midday_sun
moonlight / silver_moonlight

# Cinematic schemes
rim_light / backlight
key_light / fill_light / three_point_lighting
chiaroscuro                 # deep shadow / dramatic contrast
low_key / high_key
volumetric_light / god_rays / volumetric_haze
practicals / motivated_lighting   # only diegetic sources

# Sources
candlelight / firelight
neon / neon_rim_light
fluorescent / tungsten
window_light / hard_window_light
spotlight

# Compound (always pair with mood)
"backlit silhouette at sunset"
"single overhead spotlight, deep falloff"
"neon reflections on wet pavement"
"dappled light through leaves"
```

### 4.5 COLOR_GRADING / FILM_STOCK

```
teal_and_orange
bleach_bypass
warm_vintage / Kodak_Portra_feel
cool_palette / desaturated
high_contrast_BnW
neo_noir
warm_amber / cool_steel_blue
35mm_film_grain
16mm_film_look
ARRI_ALEXA_aesthetic
anamorphic_2.35:1
```

### 4.6 GENRE / DIRECTOR / STUDIO ANCHORS (validated to steer the model)

```
film_noir
A24
Wes_Anderson_symmetry
Blade_Runner_aesthetic
neo_noir
Hollywood_blockbuster
Hong_Kong_90s_art_cinema (yellow-green tint, step-printing)
documentary_verite
sports_documentary
Hitchcock_zoom
Guy_Ritchie_speed_ramping
Snyder_impact_slow_motion
Makoto_Shinkai (anime / saturated)
Studio_Ghibli (warm hand-drawn)
National_Geographic
```

### 4.7 MOTION_DESCRIPTORS

```
slow_motion / slo-mo
speed_ramp / ramping
time_lapse
freeze_frame
real_time
hyperlapse
"ramps to slow motion as X, then snaps back"     # explicit phrasing the model parses well
```

### 4.8 PACING_KEYWORDS (camera and subject)

```
imperceptible | barely    # extreme slow
slow | gentle | gradual   # slow
smooth | controlled       # medium  ← default if unspecified
dynamic | swift           # fast  ← USE WITH CAUTION
fast | rapid              # DANGER ZONE — combine only ONE fast element per shot
```

---

## 5. RULES (Explicit DO / DON'T)

### 5.1 DO

```
DO  bind every reference to ONE explicit role in the prompt text (IDENTITY/STYLE/MOTION/AUDIO/ENV).
DO  use exactly ONE primary camera move per generation; express compounds as ordered beats.
DO  name lighting in every prompt (single biggest quality lever).
DO  separate subject motion from camera motion into different clauses.
DO  keep prompts 60–100 words for single-shot cinematic work.
DO  end every character-shot prompt with negative constraints (avoid jitter, bent limbs, identity drift).
DO  iterate ONE variable at a time when refining (camera OR lighting OR speed — never simultaneously).
DO  tightly crop identity references (subject fills frame, neutral background).
DO  use a single frontal or three-quarter IDENTITY image rather than multiple angles of the same face.
DO  set `duration: "auto"` (or -1) and `aspect_ratio: "auto"` when references should dictate framing.
DO  reuse the returned `seed` for controlled variation; change seed to fully re-roll.
DO  for multi-shot prompts, restate camera + lighting in every SHOT block.
DO  put dialogue in double quotes inline: He says: "Remember this moment."
DO  prefix POV shots with what the camera is NOT doing: "No cuts. No zoom. Natural head movement."
```

### 5.2 DON'T

```
DON'T upload identifiable photographs of real human faces — blocked since 2026-02-10.
DON'T stack two camera moves in one clause (e.g. "spinning camera while zooming and tracking").
DON'T use multiple face images of the same character → causes face averaging / morphing.
DON'T use bare adjectives without anchor: "cinematic", "epic", "amazing", "beautiful", "lots of movement".
DON'T use raw "fast" keyword combined with fast cuts and busy scenes — guarantees jitter.
DON'T inject f-stop / ISO / focal-length numbers as technical specs; use rhythmic descriptors instead.
DON'T exceed 3 action beats per shot.
DON'T leave references unassigned in the prompt — the model will blend them unpredictably.
DON'T upload low-resolution, heavily filtered, or busy-background reference images.
DON'T mix style references with contradictory lighting (e.g. sunset swatch + fluorescent interior swatch).
DON'T pass only audio_urls — at least one image or video reference is required.
DON'T expect 4K. Standard ceiling is 1080p; 2K marketing claims are platform-dependent and not universal.
DON'T re-describe a subject already established by an @Image reference in image_to_video / R2V mode (causes identity drift).
DON'T expect more than 15s in a single generation — chain via video_extend instead.
```

### 5.3 ITERATION_PROTOCOL (one-variable-at-a-time)

```
1. Baseline: generate 2-3 variants with fixed seed series, same prompt.
2. Decide failure axis:
   IF framing wrong & action right       -> change CAMERA slot only (shot size + one move).
   IF motion off (wobbly / too fast)     -> swap pacing word OR swap 'handheld' <-> 'gimbal'.
   IF style/color drift, motion fine     -> replace STYLE slot with ONE stronger anchor; remove extras.
   IF subject mutates across re-prompts  -> simplify SUBJECT to one noun; tighten IDENTITY image crop.
   IF artifacts repeat (hands, flares)   -> add to NEGATIVE_CONSTRAINTS tail.
3. Re-generate. Keep seed if iterating one variable; change seed if re-rolling fully.
```

---

## 6. EXAMPLES (input → prompt pairs)

> Format: `INPUT_REFERENCES` block lists what the user uploaded and their assigned roles; `PROMPT` is the literal string sent to the model; `PARAMS` are the API parameters; `WHY_IT_WORKS` is internal annotation.

### EXAMPLE 1 — Character introduction, atmospheric

```yaml
INPUT_REFERENCES:
  @Image1: AI-generated portrait of female protagonist, dark curly hair, red leather jacket (IDENTITY)
  @Image2: night-time wet-asphalt neon street swatch (STYLE: color + lighting)
PARAMS:
  resolution: 720p
  duration: 8
  aspect_ratio: "21:9"
  generate_audio: true
PROMPT: |
  Primary identity: the woman from @Image1 — dark curly hair, red leather jacket, late 20s.
  She turns from the railing and walks slowly toward camera.
  Rainy rooftop at dusk, neon signs reflected in puddles, color palette from @Image2 — magenta and cyan rim light, light atmospheric haze.
  Medium shot, slow push-in, 85mm feel, shallow depth of field.
  Cinematic film tone, 35mm grain, neo-noir, teal-and-magenta grade.
  Maintain facial proportions and wardrobe from @Image1. Avoid identity drift, avoid jitter, avoid temporal flicker.
WHY_IT_WORKS:
  - one IDENTITY ref, one STYLE ref → no face averaging
  - one camera move (slow push-in) + one lens cue
  - lighting explicit; color palette delegated to @Image2
  - 75 words; tight negative tail
```

### EXAMPLE 2 — Dialogue scene, two-shot, native audio

```yaml
INPUT_REFERENCES:
  @Image1: AI-generated male character A (IDENTITY A)
  @Image2: AI-generated female character B (IDENTITY B)
  @Image3: warm coffee-shop interior reference (ENVIRONMENT + lighting)
PARAMS:
  resolution: 1080p
  duration: 10
  aspect_ratio: "16:9"
  generate_audio: true
PROMPT: |
  Two-shot. @Image1 sits on the left, @Image2 on the right, in the coffee shop interior of @Image3.
  She speaks first, looking at him: "You always arrive just on time — do you enjoy that feeling of cutting it close?"
  He laughs softly and replies: "I have my own rhythm."
  Medium close-up, gentle push-in over the dialogue, locked framing at the end.
  Warm tungsten practicals, golden-hour spill through the window, shallow depth of field, 35mm cinematic film tone.
  Maintain facial proportions and wardrobe of @Image1 and @Image2 throughout. Avoid identity drift, avoid bent limbs.
WHY_IT_WORKS:
  - dialogue placed in double quotes for native lip-sync
  - one move (push-in) only
  - both identities explicitly placed left/right
```

### EXAMPLE 3 — Establishing shot, environment-led

```yaml
INPUT_REFERENCES:
  @Image1: concept render of a futuristic skyline (ENVIRONMENT/STYLE)
  @Video1: 4-second aerial drone forward-dive clip (MOTION: camera language)
PARAMS:
  resolution: 720p
  duration: 6
  aspect_ratio: "21:9"
  generate_audio: true
PROMPT: |
  Establishing aerial. Reference the camera movement from @Video1 — sustained forward dive, first-person drone perspective.
  Tech-park city skyline of @Image1 as the visual center.
  Camera descends between two glass towers at dawn, golden-hour rim light catching mirrored facades, light atmospheric haze, lens flare on the sun.
  Wide extreme, smooth controlled descent, anamorphic 2.35:1, ARRI ALEXA aesthetic, teal-and-orange grade.
  Avoid jitter, avoid temporal flicker, avoid chaotic composition.
WHY_IT_WORKS:
  - camera move delegated entirely to @Video1 motion reference
  - environment delegated to @Image1
  - prompt body focuses on lighting + style
```

### EXAMPLE 4 — Action / chase, handheld realism

```yaml
INPUT_REFERENCES:
  @Image1: hero character (IDENTITY)
  @Video1: 6-second handheld foot-chase clip (MOTION reference)
PARAMS:
  resolution: 720p
  duration: 12
  aspect_ratio: "16:9"
  generate_audio: true
PROMPT: |
  The man from @Image1 sprints through a crowded night market, weaves between food stalls, then crashes into a fruit cart and scrambles up.
  Camera language follows @Video1 — handheld tracking, micro-shake, occasional whip-pan, no smoothing.
  Wide tracking shot, then cut to medium side-tracking at 6 seconds.
  Neon practicals, harsh rim from overhead signage, light rain on pavement, color temperature warm-on-skin / cool-on-environment.
  35mm grain, motion blur on hits, teal-and-orange grade, Hollywood action aesthetic.
  Maintain facial proportions and wardrobe of @Image1. Avoid identity drift, avoid bent limbs.
WHY_IT_WORKS:
  - explicit handheld delegation prevents over-stabilization
  - 3 action beats only (sprint, weave, crash)
  - shot transition stated explicitly at 6s
```

### EXAMPLE 5 — Product reveal, premium commercial

```yaml
INPUT_REFERENCES:
  @Image1: studio packshot of perfume bottle on marble (IDENTITY: product)
PARAMS:
  resolution: 1080p
  duration: 8
  aspect_ratio: "1:1"
  generate_audio: true
PROMPT: |
  Macro reveal of the perfume bottle from @Image1, centered on polished black marble.
  Slow dolly-in from medium to extreme close-up, then a gentle orbit during the final two seconds.
  Single overhead spotlight, dramatic side rim light, dust motes drifting through the beam, deep blacks, warm gold refracting through crystal.
  85mm feel, shallow depth of field, anamorphic flare, cinematic 35mm film tone, premium commercial aesthetic.
  Maintain logo and label legibility from @Image1. Avoid jitter, avoid color palette shift, no text distortion.
WHY_IT_WORKS:
  - compound move expressed as ordered beats (dolly-in → orbit)
  - lighting is the entire mood driver
  - explicit logo-preservation constraint
```

### EXAMPLE 6 — Long-take spy thriller (single continuous)

```yaml
INPUT_REFERENCES:
  @Image1: female agent in red trench coat (IDENTITY)
  @Image2: corner mansion exterior (ENVIRONMENT)
  @Image3: masked observer (IDENTITY 2)
PARAMS:
  resolution: 720p
  duration: 15
  aspect_ratio: "21:9"
  generate_audio: true
PROMPT: |
  Spy thriller. Single continuous shot, no cuts.
  Front-tracking shot of the agent from @Image1 — red trench coat — walking forward through a busy street, pedestrians repeatedly crossing the frame.
  She rounds the corner of the mansion from @Image2 and disappears. The masked figure from @Image3 lurks at the corner, glaring after her.
  Camera pans forward and follows her into the mansion entrance until she vanishes.
  Wide front-tracking, smooth steadicam pacing, overcast diffused light, light fog.
  35mm film tone, neo-noir, desaturated palette.
  Maintain facial proportions and wardrobe of @Image1 and @Image3. Avoid identity drift, avoid temporal flicker, no scene cuts.
WHY_IT_WORKS:
  - 'Single continuous shot, no cuts' is the explicit lock against the model's default to multi-shot edit
  - 3 identities each cleanly bound to a position in the scene
```

### EXAMPLE 7 — Dolly zoom (Hitchcock / vertigo)

```yaml
INPUT_REFERENCES:
  @Image1: protagonist mid-shock expression (IDENTITY)
PARAMS:
  resolution: 720p
  duration: 5
  aspect_ratio: "2.39:1"   # falls back to 21:9
  generate_audio: true
PROMPT: |
  The protagonist from @Image1 stares forward in dawning realization, lips parting slightly.
  Dolly-zoom (Hitchcock effect): camera dollies backward while zooming in. Subject's head size stays constant; the background corridor architecture warps and stretches violently.
  Medium close-up centered on the eyes.
  Cold tungsten practicals from above, deep shadow falloff, chiaroscuro, anamorphic flare across the eyeline.
  35mm film grain, ARRI ALEXA aesthetic, desaturated teal-and-black grade.
  Maintain facial proportions from @Image1. Avoid jitter, avoid identity drift, no extra cuts.
WHY_IT_WORKS:
  - dolly_zoom named explicitly; effect described mechanically so model executes correctly
```

### EXAMPLE 8 — Multi-shot timeline (3 shots in 12s)

```yaml
INPUT_REFERENCES:
  @Image1: warrior character (IDENTITY)
  @Image2: bamboo forest at dawn (ENVIRONMENT)
PARAMS:
  resolution: 720p
  duration: 12
  aspect_ratio: "16:9"
  generate_audio: true
PROMPT: |
  Total: 12s / 3 shots / 16:9.
  SHOT 1 [0-4s]: Wide establishing shot, locked-off. The misty bamboo forest from @Image2 at dawn, golden hour light filtering through leaves. No subject yet. Atmospheric haze.
  Lens switch.
  SHOT 2 [4-8s]: Medium shot, slow push-in. The warrior from @Image1 steps forward, white silk kimono billowing, determined expression. Same dawn light, soft rim from behind.
  Lens switch.
  SHOT 3 [8-12s]: Close-up, gentle orbit. The warrior strikes; ramps to slow motion as the blade arcs; fabric ripple visible. Snap back to real time on impact.
  Cinematic film tone, 35mm grain, teal-shadow / warm-highlight grade, anamorphic flare.
  Maintain facial proportions and wardrobe of @Image1 across all shots. Avoid identity drift, avoid temporal flicker, avoid bent limbs.
WHY_IT_WORKS:
  - 'Lens switch.' is the recognized cut delimiter
  - each shot independently states size + move + lighting
  - speed-ramp expressed in the canonical "ramps to slow motion … snap back" phrasing
```

### EXAMPLE 9 — Style-transfer omnireference (identity + separate style anchor)

```yaml
INPUT_REFERENCES:
  @Image1: character portrait (IDENTITY)
  @Image2: Wong-Kar-wai-style 90s Hong Kong night scene (STYLE only)
PARAMS:
  resolution: 720p
  duration: 8
  aspect_ratio: "4:3"
  generate_audio: true
PROMPT: |
  Primary identity: the character from @Image1 — preserve facial proportions, hairstyle, garment exactly.
  Visual style from @Image2 only — yellow-green color cast, step-printing motion smear, neon halation, retro 16mm grain.
  The character stands at a rain-soaked red phone booth, holds the receiver to her ear in silence, lips trembling almost imperceptibly, then slowly hangs up and walks into the rainy crowd.
  Extreme close-up to medium, gentle handheld with subtle micro-shake.
  Style: 1990s Hong Kong art cinema, step-printing, motion blur smear, melancholy tone.
  Avoid identity drift, avoid face distortion, do not alter wardrobe color, do not import @Image2 subject features.
WHY_IT_WORKS:
  - explicit role separation prevents @Image2 from leaking subject features into the output
  - 'do not import @Image2 subject features' is an effective negative bind
```

### EXAMPLE 10 — POV / first-person locked perspective

```yaml
INPUT_REFERENCES:
  @Image1: hero hand / forearm reference (IDENTITY for visible hands)
PARAMS:
  resolution: 720p
  duration: 12
  aspect_ratio: "16:9"
  generate_audio: true
PROMPT: |
  Single continuous shot, first-person POV, the camera IS her eyes. No cuts. No zoom. Natural head movement.
  Her hands from @Image1 are visible in frame at all times.
  POV: she pushes open a heavy wooden door into a candlelit ballroom, walks slowly forward through couples dancing.
  Wide-angle lens with strong distortion, subtle chromatic aberration near frame edges, micro-jitters, organic head sway, no stabilization.
  Warm candlelight only, deep shadows, volumetric haze, 35mm film grain, halation on highlights, slightly desaturated tones, ARRI ALEXA aesthetic.
  Avoid cuts, avoid stabilization, avoid identity drift on visible hands.
WHY_IT_WORKS:
  - explicit "no cuts, no zoom, natural head movement" defeats the model's multi-shot default
  - hands explicitly anchored to a reference
```

### EXAMPLE 11 — Anti-pattern (DO NOT USE — failure example)

```yaml
INPUT_REFERENCES:
  @Image1: character (IDENTITY)
  @Image2: character — same person, different angle
  @Image3: character — same person, third angle
  @Video1: dance reference
  @Video2: chase reference
  @Audio1: dialogue
  @Audio2: music
PROMPT: |
  cool cinematic video, amazing camera moves, fast and dynamic, lots of movement,
  spinning camera around the dancing person while zooming and tracking, epic feel,
  beautiful lighting, 4K, 60fps, ISO 200, f/1.8, 24mm
WHY_IT_FAILS:
  - 3 face images of same person → face averaging / morphing
  - 2 motion video refs contradict each other
  - bare adjectives ("cool", "epic", "amazing") with no anchor
  - stacked camera moves in one clause
  - 'fast' combined with 'lots of movement' → guaranteed jitter
  - technical specs (f/1.8, ISO, mm) ignored or harmful
  - no negative constraints, no identity anchor, no role binding
```

---

## 7. FAILURE_MODES

```yaml
identity_drift:
  symptoms: face slowly morphs across frames; jaw softens; hair color shifts; wardrobe color changes
  causes:
    - multiple IDENTITY images of same subject
    - low-resolution / filtered / extreme-angle reference
    - cluttered background in reference
    - re-describing subject in I2V/R2V (model conflicts text vs image)
    - long generation (>10s) without explicit "maintain ... throughout"
  fixes:
    - one tight-crop frontal or 3/4 reference (1024x1024 or 1024x1536)
    - "Primary identity anchor: @Image1. Do not alter facial proportions, eye shape, or hairstyle."
    - in I2V/R2V, omit subject description; describe only motion + camera + lighting
    - chain shorter clips (5-8s) and rebind identity each time

face_averaging:
  trigger: 2+ face images of the same character
  fix: collapse to one strong reference; if multi-angle is required, use a model character-sheet image (one image containing 3 angles)

temporal_flicker:
  symptoms: per-frame color shimmer, edge halo pulsing, light intensity oscillation
  causes: contradictory STYLE references, fast scene + fast cuts, overlong duration
  fixes:
    - one STYLE swatch only OR 3-5 small consistent swatches with same color temperature
    - "avoid temporal flicker" in negative tail
    - reduce duration; chain via video_extend

camera_jitter:
  symptoms: shaky frame, micro-stutter, drift on locked-off shots
  causes:
    - two camera moves stacked in one clause
    - "fast" keyword combined with busy scene
    - subject motion verb and camera motion verb in same clause
  fixes:
    - one primary move; compound moves as ordered beats
    - one "fast" element max per shot
    - separate clauses: "She spins slowly. Camera holds fixed framing."
    - swap 'handheld' for 'gimbal' / 'steadicam' for smoother result

limb_distortion / bent_limbs / hand_deformation:
  causes: complex choreography >3 beats, fast subject + fast camera, missing negative constraints
  fixes:
    - reduce to 1-3 action beats
    - add "avoid bent limbs, avoid hand deformation" to NEGATIVE_CONSTRAINTS
    - slow down subject pacing
    - prefer slow_motion / ramped slow-mo for impact moments

color_palette_drift:
  causes: contradictory lighting refs (sunset swatch + fluorescent interior swatch)
  fixes: keep all STYLE refs in the same color temperature family; or use one swatch only

face_filter_block:
  trigger: real human face uploaded
  symptom: API rejects the reference or returns moderation error
  fix: replace with AI-generated portrait (Seedream / Midjourney / Stable Diffusion / Flux character sheet) — these consistently bypass the filter

motion_reference_bleed:
  symptom: motion video's subject features leak into the generated character
  fix: explicitly state "Camera language only from @Video1. Do not import @Video1 subject features."

audio-only_input_rejection:
  rule: at least 1 image or 1 video must accompany audio references
  fix: include at minimum 1 reference image even if its role is "environment only"

over-constraint_collapse:
  symptom: all 12 slots used; output is incoherent or generic
  fix: trim to 1 IDENTITY + 1-2 STYLE + ≤1 MOTION + ≤1 AUDIO; everything else is description

mode_confusion:
  symptom: model produces multi-shot output when single take requested (or vice versa)
  fix: explicit literal phrase "Single continuous shot. No cuts." OR "Total: 15s / 3 shots / 16:9."
```

---

## 8. CINEMATIC PROMPT GENERATION ALGORITHM (for downstream LLM)

```
INPUT: user_intent {scene_description, references_inventory, target_duration, target_aspect, target_resolution}
OUTPUT: {prompt_string, params_object}

1. Classify references:
   For each reference asset, assign exactly one role in {IDENTITY, STYLE, MOTION, ENVIRONMENT, AUDIO}.
   If user uploaded >1 image of same person → keep 1 (best frontal / 3-quarter, tight crop), discard rest in prompt binding.

2. Choose mode:
   if len(image_urls)==1 and no_motion_ref and single-take animation -> image_to_video
   elif (any video_url) or (multiple image roles) or (audio_url) -> reference_to_video (omnireference)
   else -> text_to_video

3. Construct slots:
   SUBJECT_BINDING := "Primary identity: @Image1 — <3-5 visual traits>." + (multi-subject bindings)
   ACTION          := pick 1-3 concrete verbs; one per beat
   ENVIRONMENT     := location + time-of-day + ONE atmospheric modifier
   LIGHTING        := pick ≥1 from §4.4 (mandatory)
   CAMERA          := one shot_size + one camera_move + one pacing_keyword (+ optional lens cue)
                      if @VideoN exists with MOTION role: "Camera follows @VideoN."
   STYLE           := 2-3 anchors from §4.5 / §4.6
   NEGATIVE        := always include: "Avoid jitter, avoid identity drift, avoid temporal flicker."
                      add "avoid bent limbs" if any human subject present
                      add "avoid color palette shift" if duration >8s

4. Word-count guard:
   If draft > 200 words: drop redundant adjectives, keep one style anchor.
   If draft < 35 words: expand ENVIRONMENT + LIGHTING.
   Target: 60-100.

5. Validate constraints:
   - Exactly ONE primary camera move present (string-match against §4.2; reject if >1 unless expressed as ordered beats with "then" / "Start:" / "Finally:").
   - Lighting term present (string-match against §4.4 keyword list).
   - At least one negative constraint present.
   - Every @ImageN, @VideoN, @AudioN referenced in text MUST correspond to an actual upload index.
   - No real-face reference flagged.

6. Emit params:
   resolution     := requested OR 720p
   duration       := requested OR "auto"
   aspect_ratio   := requested OR (if image_urls present and equal aspect: "auto"; else "16:9")
   generate_audio := true (default)
   seed           := omit on first generation; reuse on iteration
```

---

## 9. KNOWN UNKNOWNS / CAVEATS

```yaml
official_docs:
  - BytePlus ModelArk Seedance 2.0 reference page exists at https://docs.byteplus.com/en/docs/ModelArk/2222480 and /1520757
    but renders as a client-side SPA; raw HTTP fetch returns no text content. Authoritative parameter values
    above are reconstructed from fal.ai, Replicate, PiAPI, WaveSpeedAI, BytePlus changelogs cited by third
    parties (LaoZhang AI, EvoLink), and Volcengine SDK examples (volcenginesdkarkruntime). Treat exact
    parameter NAMES (image_urls vs images, etc.) as provider-specific; the SEMANTICS are identical.

  - Official Seedance 2.0 marketing page (seed.bytedance.com/en/seedance2_0) confirms unified multimodal
    audio-video architecture, image/audio/video reference support, and "industry-standard cinematic output"
    but does not enumerate camera vocabulary or prompt grammar — those are documented on the official
    Volcengine prompt guide referenced by community mirrors (seedance2.ai/guide).

  - The "Seedance Pro 2.0" naming variant is the Volcengine model ID label (doubao-seedance-2-0-pro-260215);
    it refers to the same model as Seedance 2.0 standard tier. "Seedance 2.0 Fast" is the lower-cost variant.

resolution_ambiguity:
  - fal.ai documents Seedance 2.0 at 480p/720p (image-to-video supports 1080p on standard tier).
  - Replicate notes default ceiling 1080p.
  - Some marketing pages (GlobalGPT, Apiyi) claim "Native 2K (2048x1152)"; this is not corroborated by
    fal/Replicate/Volcengine schemas. Treat 1080p as the production-safe ceiling; 2K may be platform-dependent.

frame_rate:
  - Native ~24fps cinematic. Do NOT put 60fps in prompt text — anecdotally produces "soap opera effect"
    or is ignored. Frame rate is not a user-controllable API parameter.

face_input_status:
  - Real human face uploads disabled 2026-02-10. An enterprise endpoint
    (fal-ai/seedance-2/enterprise/fast/reference-to-video) re-enables face input under contract.

copyright_safeguards:
  - Real-time IP filter blocks generation of protected franchises (e.g. Marvel, Star Wars, DC, Disney
    properties) following Disney / Paramount Skydance cease-and-desist (Feb 2026).

drift_at_chain_count:
  - Cumulative identity drift becomes visible after 4-5 chained generations using "last frame as next
    identity reference". Mitigation: periodically rebind to the ORIGINAL character sheet rather than the
    latest output frame.

audio_pricing:
  - Audio generation is included at no extra cost across fal endpoints. `generate_audio: false` does not
    reduce cost; it only suppresses the audio track.
```

---

## 10. QUICK-REFERENCE CHEAT SHEET

```
SLOT ORDER:    SUBJECT → ACTION → ENVIRONMENT/LIGHTING → CAMERA → STYLE → NEGATIVE
LENGTH:        60-100 words
REFERENCES:    1 IDENTITY image + 1-2 STYLE + ≤1 MOTION video + ≤1 AUDIO  (sweet spot, not max)
TAG SYNTAX:    @Image1, @Video1, @Audio1   (also accepts [Image1])
CAMERA:        exactly ONE primary move per generation
LIGHTING:      mandatory (highest-leverage element)
SUBJECT vs CAM motion: separate clauses always
NEGATIVE TAIL: "Avoid identity drift, avoid jitter, avoid temporal flicker."
DURATION:      4-15s; default "auto"
RESOLUTION:    720p production-safe; 1080p standard-tier
ASPECT:        21:9, 16:9, 4:3, 1:1, 3:4, 9:16, auto
HARD RULES:    no real face refs · no stacked camera moves · no 'fast' + busy scene · no multiple face refs of same person · no audio-only inputs
```

---

**Recommendations (staged use of this spec by a downstream LLM)**

1. **Cold start (no references uploaded):** route to TEXT_TO_VIDEO, use the 6-slot template from §3.1, skip §2 reference bindings.
2. **One identity image provided:** route to IMAGE_TO_VIDEO if single-shot animation suffices, OR OMNIREFERENCE with that image as @Image1=IDENTITY.
3. **Identity + style + motion needed:** OMNIREFERENCE; enforce role separation per §2.3 and §5.1.
4. **Multi-shot narrative (3+ shots in ≤15s):** use timeline-prompt variant §3.3; explicitly enumerate shot blocks.
5. **Long-form (>15s):** generate omnireference clip → take last clean frame → use it as new @Image1 for next call → REBIND to original character sheet every 3rd iteration to control drift.
6. **Failure recovery:** apply §5.3 iteration protocol — change exactly one slot, hold others. Do not rewrite the whole prompt.

**Thresholds that change behavior:**
- If the model output shows facial morphing across >20% of frames → swap IDENTITY image to a tighter crop.
- If word-count of generated prompt exceeds 150 → drop one STYLE adjective.
- If user requests >3 simultaneous camera behaviors → reject and ask for prioritization, or convert to ordered shot blocks.
- If user uploads a real-face photo → refuse and request an AI-generated portrait substitute.

**Caveats**
- Provider parameter NAMES differ slightly (`image_urls` on fal/Replicate vs natural-language "Image 1" on Volcengine). The SEMANTICS in this spec are vendor-independent; map names at the API adapter layer.
- Frame rate and exact native resolution ceilings vary by hosting platform; the table in §1 reflects the conservative production-ready set.
- Real-time IP filters and face-input gates evolve; treat the 2026-02-10 face block and copyright filters as the current baseline subject to change.
- "2K native" claims in some marketing material are not corroborated by canonical API schemas; assume 1080p ceiling unless a specific provider documents 2K explicitly.
- Seedance 2.0 official BytePlus ModelArk documentation pages are JavaScript-rendered and were not directly scrapable at spec-authoring time; all parameter and behavior assertions above are corroborated by ≥2 of {fal.ai docs, Replicate docs, official Volcengine prompt guide mirror at seedance2.ai/guide, BytePlus model ID changelogs, ByteDance Seed product page}.