# Suno v5.5 Prompt Construction Guide for AI/LLM Consumption

> Machine-readable reference for constructing Suno v5.5 prompts from pre-written lyrics. Optimized for parsing by an LLM that receives lyrics + creative direction from a user and must emit a valid Suno Custom Mode prompt (Style field + Lyrics field + Title + optional Exclude/sliders).

---

## TL;DR

- **Suno v5.5** (released 26 March 2026; codename succeeds v5/"chirp-crow") is a Custom-Mode-first generator that takes three independent inputs: a **Style** field (≤1000 chars; describes sound), a **Lyrics** field (≤5000 chars; contains the lyrics AND bracketed `[metatags]`), and a **Title** field (≤80 chars; cosmetic). Generation maxes at **~8 minutes per clip**, returns **2 variations per call**, and is non-deterministic.
- **Build every prompt as Style + Tagged-Lyrics + Exclude.** Style = comma-separated descriptors (genre → subgenre → tempo/BPM → key instruments → vocal direction → production → mood/era). Lyrics = user-provided lines with `[Section]` tags between blocks and optional inline parenthetical ad-libs. Negative direction goes in the Advanced Options **Exclude Styles** field (NOT inline) for reliability.
- **What v5.5 changed vs v5:** same base sonic quality, but adds **Voices** (verified voice cloning, replaces Persona button), **Custom Models** (up to 3 fine-tunes from ≥6 user tracks), and **My Taste** (passive preference learning, triggered by magic-wand icon next to Style). v5.5 is more responsive to nuanced descriptors ("close-mic breathiness", "detuned vintage keys") than v5 was. Detailed prompts always override My Taste defaults.

---

## USAGE INSTRUCTIONS FOR AI

You are a Suno prompt constructor. When the user supplies lyrics and creative direction, do the following in order. Do not generate, modify, or substitute the user's lyrics unless explicitly asked.

### Decision pipeline (run on every request)

1. **Parse user input.** Separate: (a) the literal lyrics block, (b) any creative direction (genre, mood, voice, references, BPM, era, instruments, exclusions, target length).
2. **Pick mode.**
   - If user provided lyrics → **Custom Mode**, Instrumental = OFF.
   - If user provided NO lyrics and asked for instrumental → **Custom Mode**, Instrumental = ON; leave Lyrics field empty (or only put bracketed section/instrument tags).
   - If the user just typed a vibe phrase and wants Suno to write everything → Simple Mode (out of scope for this guide; tell the user).
3. **Build the Style field.** Use the schema in §2.1. Hard cap: 1000 characters. Optimal target: **120–300 characters**. Front-load load-bearing tags (genre, subgenre) in the first 20–30 words; Suno truncates silently at the cap and weights early tokens more heavily.
4. **Tag the lyrics.** Insert `[Section]` tags on their own lines before each lyrical block. Preserve every line break the user wrote. Add inline vocal-direction tags `[Whispered]`, `[Belted]`, etc. ONLY where (a) the user requested them, or (b) the lyric line itself unambiguously calls for it (e.g., parenthetical stage direction inside the user's text). Never invent new lyric words.
5. **Build the Exclude string.** Comma-separated list of unwanted elements (e.g., `autotune, brass, choir, electric guitar`). Use Advanced Options → Exclude Styles. Keep ≤ ~6 items.
6. **Choose sliders.**
   - Weirdness: 0.20 for commercial/safe, 0.50 default, 0.65–0.80 for experimental/genre-blend.
   - Style Influence: 0.65–0.80 when Style field is precise; 0.40–0.55 when you want creative latitude.
   - Audio Influence: only relevant for Covers / Add Vocals / Add Instrumentals.
7. **Set Title.** Short, descriptive, ≤80 chars. Has negligible effect on audio.
8. **Validate.** Run the checklist in §10 before emitting.
9. **Emit output.** Use the exact schema in §3.

### Hard rules

- DO put `[Section]` tags on their own line, in square brackets, e.g. `[Verse 1]`. Tags are case-insensitive; canonicalize to Title Case for readability.
- DO NOT put production/genre/mood descriptors in the Lyrics field (they belong in Style). They will either be ignored or sung.
- DO NOT include real artist names in Style ("sounds like Adele") — unreliable and often filtered. Translate to descriptors ("powerful female alto, piano-driven pop ballad").
- DO NOT add "[Intro]" if you want a strong opening — it is notoriously unreliable; instead describe the intro inline as `[Instrumental: 8-bar atmospheric pad intro]` or put it in the Style field.
- DO NOT exceed 4–7 primary descriptors in Style; descriptors compete and over-stuffed prompts produce muddy averages.
- DO NOT stack 3+ genres; use at most 2 (one dominant first, one modifier second).
- DO NOT put inline negatives like "no drums" in the Style field — they are unreliable. Use the Exclude field.
- DO NOT request exact BPM lock; BPM is treated as approximate guidance. State it anyway (it helps), but expect ±5–10 BPM drift.
- DO emit `[End]` as the final tag if the user wants a hard stop; otherwise emit `[Outro]` followed by 1–4 closing lines.
- DO drop explicit gender vocal tags ("male vocals", "female vocals") if the user is using a cloned Voice or a Persona — they waste characters and may conflict.

---

## 1. MODEL & PLATFORM FACTS (v5.5)

| Property | Value | Confidence |
|---|---|---|
| Model name (UI) | v5.5 | Official |
| Release date | 26 March 2026 | Official (help.suno.com, suno.com/blog) |
| Predecessor | v5 (internal codename "chirp-crow", Sept 2025) | Official |
| Max generation length per call | ~8 minutes | Official |
| Variations returned per generation | 2 | Official |
| Required tier for v5.5 access | Pro ($10/mo) or Premier ($30/mo) | Official |
| Free tier model | v4.5-All | Official |
| Custom Mode available | Yes (Style / Lyrics / Title separate fields) | Official |
| Instrumental toggle | Yes | Official |
| Exclude Styles (Advanced Options) | Yes — Pro/Premier | Official |
| Creative sliders | Weirdness, Style Influence, Audio Influence | Official |
| Voices (voice cloning) | New in v5.5; verification required; Pro/Premier | Official |
| Custom Models (fine-tune) | New in v5.5; ≥6 uploaded songs/model; up to 3 per user; Pro/Premier | Official |
| My Taste (passive personalization) | New in v5.5; all tiers; magic-wand icon next to Style | Official |
| Stem export | 2-stem (all tiers), 12-stem (Pro/Premier) | Official |
| Studio DAW | Premier tier only | Official |
| Time signature in prompt | NOT model-conditioning in v5.5; only Studio grid honors it | Official (Studio 1.2 release notes) |
| Commercial rights | Pro/Premier own outputs | Official |

### Field character limits (verified against community API mirrors and HookGenius/MusicSmith analyses)

| Field | v3.5 / v4 | v4.5 / v4.5+ / v5 / **v5.5** | Notes |
|---|---|---|---|
| Style | 200 | **1000** | Silent truncation past cap; front-load |
| Lyrics (Custom Mode) | 3000 | **5000** | Long lyrics cause Suno to rush/skip; **practical optimum 1500–3500** |
| Title | 80 | **80** | Cosmetic only |
| Simple-Mode prompt | 500 | **500** | Single field, model writes lyrics |

### Practical optima (community-tested, multiple sources)

- Style field: **100–300 chars** is the sweet spot for adherence. 1000 is a hard ceiling, not a target.
- Lyrics field: **1500–3500 chars** for best audio quality across an 8-min generation.
- Number of style descriptors: **4–7**.
- Number of distinct instruments named: **2–4**.
- Number of genre tags: **1–2** (max 2; second one modifies the first).
- Number of mood/energy descriptors: **1–2**.
- Number of exclusions: **2–6**.

---

## 2. PROMPT ARCHITECTURE

Three independent inputs control generation:

```
1. STYLE FIELD            → defines sonic "world" (genre, instruments, mood, production)
2. LYRICS FIELD           → defines what is sung + local section structure via [tags]
3. EXCLUDE STYLES FIELD   → defines what to forbid (Advanced Options)
+ Creative Sliders (Weirdness, Style Influence, Audio Influence)
+ Optional: Voice / Custom Model / Persona dropdown selection
```

### 2.1 Style-field schema

Comma-separated descriptors, in this order of importance:

```
{genre}[, {subgenre}], {tempo descriptor and/or BPM}, {2-4 key instruments}, {vocal direction}, {production/texture}, {mood/emotion}[, {era}]
```

Concrete template:

```
{Subgenre}, {tempo word} {BPM}, {instrument1}, {instrument2}, {instrument3}, {vocal gender + texture}, {production aesthetic}, {mood}[, {era}]
```

Examples (each <250 chars):

```
Indie folk, mid-tempo 92 BPM, fingerpicked acoustic guitar, upright bass, brushed snare, warm female alto with breathy delivery, lo-fi analog warmth, nostalgic and wistful
```

```
Melodic trap, dark and atmospheric, 145 BPM, deep sub 808s, glitchy hi-hat rolls, reverb-drenched pads, male vocals with pitch-shifted ad-libs, modern hard-hitting mix, minor key
```

```
80s synth-pop, upbeat 118 BPM, gated-reverb drums, Moog bass, shimmering arpeggios, bright polished female vocal, radio-ready mix, nostalgic neon energy
```

### 2.2 What works vs what does NOT work in Style

| Works | Does not work / unreliable |
|---|---|
| Genre + subgenre (`indie rock`, `liquid drum and bass`) | Specific artist names (`sounds like Radiohead`) — filtered or unreliable |
| Tempo word + BPM number (`fast, 174 BPM`) | Exact metronome lock (BPM is approximate ±5–10) |
| Instrument names (2–4 of them) | Mixing/engineering jargon (`sidechain compression`, `−12 LUFS`) |
| Vocal gender + texture (`raspy male baritone`) | Negative phrases in Style (`no drums`) — use Exclude instead |
| Production aesthetics (`lo-fi tape`, `polished radio mix`) | Vague compliments (`amazing`, `professional`, `epic`) |
| Mood adjectives (1–2, internally consistent) | Contradictory moods (`dark` + `euphoric` averages to muddy) |
| Era markers (`1985`, `90s grunge`, `2020s`) | Stacked 3+ genres (averages to generic) |
| Key (`minor key`) — loose | Specific keys (`F# minor`) — loose, often ignored |

### 2.3 Lyrics-field schema

Section tags live on their own lines; user lyrics are placed between them verbatim:

```
[Intro: optional descriptor]            ← optional, often unreliable; prefer omitting
[Verse 1]
<user line>
<user line>
<user line>
<user line>

[Pre-Chorus]                            ← optional
<user line>
<user line>

[Chorus]
<user line>
<user line>
<user line>
<user line>

[Verse 2]
<user line>
...

[Chorus]                                ← repeating same tag encourages melodic repetition
<same chorus lyrics verbatim>

[Bridge: optional parameterized descriptor]
<user line>
...

[Chorus]
<same chorus lyrics>

[Outro]
<user line>
<user line>

[End]                                   ← optional, prevents trailing audio
```

### 2.4 Parameterized section tags (v4.5+/v5/v5.5)

Per-section modifiers attach with a colon. Treated as local style overrides for that block only:

```
[Verse 1: whispered vocals, acoustic guitar only]
[Chorus: full band, soaring vocals, gang harmonies]
[Bridge: piano only, stripped down, vulnerable male vocal]
[Outro: fade out, ambient reprise]
[Instrumental Break: distorted guitar solo, 8 bars]
```

### 2.5 Inline vocal cues and ad-libs

- **Inline performance tags** (one tag per line, before the line it modifies):
  ```
  [Whispered]
  In the silence of the night
  [Belted]
  AND I CAN'T LET GO
  ```
- **Parenthetical ad-libs** (sung as quick vocal interjections — kept inline, NOT in brackets):
  ```
  We're rolling out tonight (oh yeah)
  Lights down low (hey!)
  ```
- **All-caps to suggest emphasis/shouting on a line** is commonly used by the community and works in v5+.

### 2.6 Indicating repeats

- To repeat a chorus: paste the same `[Chorus]` tag and the same lyrics again. v5.5 will repeat melody/arrangement more reliably than v4.x.
- To repeat a line as a hook: write it 2–4 times under a `[Hook]` tag (Suno may compress shorter than what's written).
- `(x2)` / `(repeat)` annotations are inconsistently honored; prefer explicit duplication.

### 2.7 Non-lyrical sections inside Lyrics field

- Pure instrumental sections inside a vocal track:
  ```
  [Instrumental Break]
  (8 bars, soaring guitar solo)
  ```
  The parenthetical descriptor functions as a duration/character hint, not literal sung text.
- Sound effects appear as their own bracketed tags on their own line:
  ```
  [Rain]
  [Thunder]
  ```
  Reliability is moderate; SFX tags work better at section boundaries than mid-line.

### 2.8 Length-of-section control

- Suno infers section length from the **count of lyric lines** beneath each section tag. 4 lines ≈ shorter verse; 8 lines ≈ longer verse.
- For instrumental sections, a parenthetical bar count (`(8-bar solo)`, `(16-bar break)`) acts as a soft duration hint, not a lock.

---

## 3. OUTPUT SCHEMA THE AI MUST EMIT

Always emit exactly this fenced structure when returning a Suno prompt to the user:

````markdown
**MODE:** Custom Mode  (v5.5)
**INSTRUMENTAL:** OFF  ← or ON if no lyrics

**STYLE FIELD** (≤1000 chars; current: NNN)
```
<comma-separated style descriptors>
```

**LYRICS FIELD** (≤5000 chars; current: NNNN)
```
[Section Tag]
<lyric line 1>
<lyric line 2>
...

[Next Section Tag]
...

[End]
```

**TITLE** (≤80 chars)
```
<short title>
```

**EXCLUDE STYLES** (Advanced Options)
```
<comma-separated exclusions, or "(none)">
```

**CREATIVE SLIDERS**
- Weirdness: 0.XX
- Style Influence: 0.XX
- Audio Influence: 0.XX (only if audio reference is used)

**OPTIONAL**
- Voice: <persona name | cloned voice name | none>
- Custom Model: <model name | none>
````

---

## 4. EXHAUSTIVE TAG DICTIONARY

All tags use `[Square Brackets]`. They are **case-insensitive**; the AI should canonicalize to Title Case. Treat anything NOT in these tables as "creative/experimental" — Suno may ignore or literally sing them.

### 4.1 Structural section tags (high reliability)

```
[Intro]                  [Verse]                [Verse 1] ... [Verse N]
[Pre-Chorus]             [Chorus]               [Post-Chorus]
[Hook]                   [Bridge]               [Refrain]
[Interlude]              [Breakdown]            [Break]
[Build]                  [Build-Up]             [Drop]
[Instrumental]           [Instrumental Intro]   [Instrumental Break]
[Outro]                  [End]
```

Notes:
- `[Intro]` is unreliable; prefer `[Instrumental Intro]` with a parenthetical descriptor, or omit and let the model open naturally.
- `[End]` is the hard-stop tag; use it as the final tag to prevent trailing audio.
- Number repeated sections (`[Verse 1]`, `[Verse 2]`) to differentiate; reuse `[Chorus]` verbatim to enforce repetition.

### 4.2 Instrument-feature tags

```
[Guitar Solo]            [Piano Solo]           [Bass Solo]
[Drum Solo]              [Saxophone Solo]       [Synth Solo]
[Strings Rise]           [Brass Stabs]          [Percussion Break]
[String Quartet]         [Choir Vocals]
```

### 4.3 Vocal direction tags (placed inline in Lyrics)

Gender / character:
```
[Male Vocal]     [Female Vocal]    [Male Vocalist]   [Female Vocalist]
[Duet]           [Choir]           [Boy]             [Girl]
[Man]            [Woman]
```

Vocal style:
```
[Whispered]      [Whisper]         [Spoken Word]     [Rap]
[Harmonies]      [Harmonized]      [Falsetto]        [Belted] / [Belting]
[Growl]          [Crooning]        [Operatic]        [Scat]
[Humming]        [Ad-lib]          [Backing Vocals]  [Vocal Layering]
```

Vocal processing/effect:
```
[Reverb]         [Delay]           [AutoTune]        [No AutoTune]
[Distorted Vocals] [Filtered Vocals] [Vocoder]       [Telephone Effect]
[Dry Vocals]     [Lo-fi Vocals]
```

Vocal emotion:
```
[Vulnerable]     [Powerful]        [Soft]            [Aggressive]
[Melancholic]    [Joyful]          [Sultry]          [Defiant]
[Intimate]       [Pained]
```

Note: Drop `[Male Vocal]` / `[Female Vocal]` when a Voice (clone) or Persona is selected from the dropdown — the selected Voice supplies vocal identity.

### 4.4 Dynamics & production tags

```
[Fade In]        [Fade Out]        [Silence]
[Crescendo]      [Decrescendo]
[Tempo: slow]    [Tempo: 128 BPM]  [Key Change]
[Loop-Friendly]                                   ← v5+, ends section so it loops cleanly
[Callback: keep chorus vibe]                      ← v5 Studio extend/replace only
```

### 4.5 Instrument vocabulary (put in Style field, not Lyrics)

Keyboards: `piano, electric piano, Rhodes, Wurlitzer, organ, Hammond organ, synth, analog synth, Moog synth, synth pad, harpsichord, clavinet`

Strings (fretted/bowed): `acoustic guitar, electric guitar, distorted guitar, bass guitar, slap bass, upright bass, violin, strings, string quartet, cello, harp, ukulele, banjo, mandolin, sitar`

Drums/percussion: `drums, acoustic drums, electronic drums, 808s, 808 bass, drum machine, TR-909, breakbeat, brush drums, percussion, taiko drums, congas, bongos, tambourine, handclaps`

Winds/brass: `saxophone, tenor sax, alto sax, trumpet, trombone, French horn, brass section, flute, clarinet, harmonica, accordion`

Electronic: `synth bass, arpeggiated synth, lead synth, synth stabs, pad, pluck synth, acid bass, supersaw, wobbly bass, glitch`

Orchestral: `orchestra, full orchestra, chamber orchestra, orchestral strings, brass stabs, timpani, choir vocals, cinematic percussion`

### 4.6 Mood / emotion / energy descriptors (Style field)

Mood: `uplifting, melancholic, haunting, dark, joyful, nostalgic, somber, romantic, intense, dreamy, peaceful, anxious, euphoric, mysterious, aggressive, playful, epic, intimate, bittersweet, triumphant, brooding, defiant, reflective, confessional`

Energy: `high energy, medium energy, low energy, chill, driving, explosive, building, relaxed, frantic, steady, anthemic, hypnotic, propulsive`

Texture/production: `lo-fi, gritty, clean, raw, lush, sparse, tape-saturated, vinyl hiss, atmospheric, punchy, warm, bright, muddy, polished, vintage, modern, dry, reverb-heavy, wide stereo, vocal-forward, compressed`

### 4.7 Genre vocabulary (Style field)

High-reliability primary genres:
`pop, indie pop, synth-pop, dream pop, bedroom pop, art pop, electropop, dance pop, K-pop, J-pop, city pop, rock, indie rock, alt rock, classic rock, punk rock, post-punk, hard rock, garage rock, blues rock, shoegaze, emo, hip-hop, rap, boom bap, trap, lo-fi hip-hop, drill, UK drill, R&B, neo-soul, soul, funk, contemporary R&B, EDM, house, deep house, tech house, techno, trance, dubstep, drum and bass, liquid drum and bass, ambient, synthwave, retrowave, chillwave, future bass, electro, industrial, IDM, downtempo, hardstyle, country, modern country, outlaw country, bluegrass, folk, indie folk, Americana, singer-songwriter, jazz, smooth jazz, bebop, cool jazz, jazz fusion, bossa nova, blues, delta blues, Chicago blues, swing, reggae, dancehall, latin, salsa, bachata, reggaeton, flamenco, afrobeats, afrobeat, classical, baroque, romantic, orchestral, chamber music, cinematic, epic, minimalist, neoclassical, opera, gospel, Christmas, new age, acoustic, ballad`

Useful fusion pairs (community-tested with strong v5.5 success rates): `pop + EDM, gospel + trap, jazz + hip-hop, indie folk + bedroom pop, lo-fi hip-hop + jazz, country + rock, synthwave + hip-hop, progressive house, melodic techno`.

### 4.8 Era markers (Style field)

`1950s, 1960s, 1970s, 1980s, 1990s, 2000s, 2010s, 2020s, vintage, modern, retro, futuristic, mid-century, golden-age`. Combine with genre: `1985 synth-pop`, `90s boom bap`, `70s disco`, `2000s emo`.

### 4.9 Sound-effect tags (Lyrics field, own line)

Environmental: `[Rain] [Thunder] [Wind] [Ocean Waves] [Birds Chirping] [City Ambience] [Forest] [Fire Crackling]`