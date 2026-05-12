# ElevenLabs Eleven v3 (Alpha) — AI Prompt-Construction Reference Guide

**Purpose:** Machine-readable spec for an AI system that authors v3 prompts for short-film character/dialogue. Optimized for parsing, not human reading. Use as a constraint and pattern library.

## TL;DR

- v3 is a *performance* TTS model controlled via inline bracketed audio tags + punctuation + ALL-CAPS + voice choice + Stability mode (Creative / Natural / Robust); to elicit reliable acting, prompts MUST be >250 characters, voice must already match the desired register, and Stability should be set to **Creative** (max expressiveness) or **Natural** (balanced) — **never Robust** for emotional film work.
- Multi-character scenes use the **Text to Dialogue** API (`/v1/text-to-dialogue`): an ordered array of `{text, voice_id}` objects sharing context across speakers; max 10 unique voices, ≤2000 total chars per request, no limit on number of turns. Tags ARE per-line/inline; speaker labels in plain text (e.g., `Speaker 1:`) work only inside single-voice TTS, not as a v3 enum.
- Tags are free-form natural language inside `[brackets]`, case-insensitive, with no closing tag — effect ends at next contradicting tag, sentence end, or dialogue turn. ElevenLabs publishes only a small canonical set; the model accepts virtually any 1–3 word performance descriptor, but reliability degrades for tags far from the chosen voice's training timbre.

---

## 1. MODEL IDENTITY (constants)

```
model_id:              eleven_v3
TTS endpoint:          POST /v1/text-to-speech/{voice_id}
Dialogue endpoint:     POST /v1/text-to-dialogue
Languages:             70+ (full ISO list in §11)
Max chars (TTS):       5000 per request
Max chars (Dialogue):  2000 across all inputs[].text
Max speakers:          unlimited turns; ≤10 unique voice_ids per dialogue request
SSML <break>:          NOT supported in v3 (use ellipsis / em dash / [pause] instead)
SSML <phoneme>:        NOT supported in v3 (only in Flash v2 / English v1)
Determinism:           non-deterministic; pass integer seed (0 – 4_294_967_295) for partial repeatability
Free regenerations:    2 per dashboard generation if text + params unchanged
Status:                Alpha / research preview; PVCs not yet optimized — prefer IVC or library voice
```

---

## 2. PROMPT-CONSTRUCTION ALGORITHM (deterministic recipe for the AI)

Apply in order:

1. **Pick voice first.** Tag effectiveness is bounded by voice timbre. A calm/meditative voice will not shout convincingly; a hyped voice will not whisper. If script requires extreme range, designate one IVC/library voice per emotional register, not one voice for all.
2. **Set Stability:**
   - `Creative` → maximum tag responsiveness, character work, monologues with extreme range. Most prone to hallucination/voice drift.
   - `Natural` → default for dialogue scenes where voice must stay recognizable.
   - `Robust` → forbid for film/character work; only for narration that must not vary.
3. **Length-pad:** if generated text <250 characters, prepend or append in-character narrative context (not stage directions in brackets) to push above threshold. Short prompts hallucinate.
4. **Write text as a script** (natural speech, contractions, sentence fragments, beats), then **decorate**:
   - Lead each line/beat with an emotion or delivery tag.
   - Punctuate (§4) and capitalize (§5) for prosody.
   - Insert non-verbal reactions (`[sighs]`, `[laughs]`, `[gulps]`) at character-truthful moments — not decoratively.
   - Add ONE accent/character tag per persona, near the start of their first line, and re-anchor it every ~2–3 paragraphs because high-level tags fade.
5. **Sanity-check tag/voice compatibility** (§6) and tag/tag compatibility (§13).
6. **For multi-speaker scenes:** convert to Text-to-Dialogue array form (§7). Do NOT rely on `Speaker 1:` labels inside a single TTS call for true multi-voice output.
7. **Plan iteration.** v3 is non-deterministic; produce 3–5 generations and pick best. Lock with `seed` once a good take exists.

---

## 3. SYNTAX RULES (hard constraints)

```
TAG_FORMAT          = "[" descriptor "]"
descriptor          = 1–3 lowercase or UPPERCASE words; case-insensitive
PLACEMENT           = inline anywhere; effect begins immediately after tag
SCOPE_TERMINATION   = (a) next contradicting tag, (b) sentence end (. ! ?),
                      (c) end of dialogue turn / quoted line,
                      (d) gradual decay over a paragraph
NO_CLOSING_TAG      = true            # never write [/tag]
CHAINING            = "[tag1][tag2]" with no space is valid
COMMA_INSIDE_TAG    = allowed for compound directions, e.g. [doing deep voice, mocking]
TAGS_ARE_FREEFORM   = the model treats anything in [ ] as a performance direction;
                      unknown tags often still work but with lower reliability
NEVER_DO            = wrap dialogue itself in brackets; insert tags mid-word;
                      use SSML <break> with v3; rely on Robust mode for tag responsiveness
KNOWN_FAILURE_MODE  = model speaks the tag out loud (read-through). Mitigation:
                      use Creative/Natural stability, pick a more expressive voice,
                      lengthen surrounding text, regenerate (seed change),
                      remove ambiguous tags like [music] / [standing] (visual not auditory).
```

---

## 4. PUNCTUATION CONTRACT (v3-specific prosody)

```
.            full stop, full pause, intonation reset
,            short pause, natural breath
;            mid-length pause, semantic separation
…  / ...     trailing-off pause; signals hesitation, sadness, suspense (PREFERRED v3 pause)
-            short pause (less consistent)
—  (em dash) hard interruption / strong break; pairs with [interrupting]
?            rising inflection; question
!            volume + urgency lift (combine with ALL-CAPS for shout)
"…"          quoted line — scope of last tag often terminates at the closing quote
newline      treated like period; full pause
ELLIPSIS+CAPS = sad/dramatic; e.g. "It was a LONG day [sigh] … nobody listens anymore."
```

Do NOT use `<break time="x.xs" />` in v3 (officially unsupported and causes artifacts; reserved for v2/Flash). Replace with `…` or `[pause]` / `[short pause]` / `[long pause]`.

---

## 5. CAPITALIZATION CONTRACT

```
lowercase      neutral
Sentence case  neutral
ALL CAPS       loudness + emphasis on that word/clause
Mixed (one CAPS word in sentence) → that word is stressed
Combine: "I said [shouting] NOW!"  > "I said now."
```

Caps are additive to tags, not a substitute. Use sparingly; entire-paragraph caps degrade quality.

---

## 6. STABILITY MODES — FILM USE-CASE MAPPING

```
Creative  → monologue, breakdown scenes, comedy, screams, sobs, accents,
            character-voice gags. Tag fidelity highest. Voice drift risk highest.
Natural   → standard dialogue, two-handers, narration with mild emotion,
            most short-film work. RECOMMENDED DEFAULT.
Robust    → DO NOT USE for character/dialogue. Only for corporate/educational
            narration where tags must be muted.
```

If a generation reads tags aloud → drop one notch toward Creative AND/OR lengthen text. If voice cracks/morphs → drop toward Natural.

---

## 7. MULTI-SPEAKER FORMATS (two distinct paths)

### 7A. Text-to-Dialogue API (canonical for film scenes)

```json
{
  "model_id": "eleven_v3",
  "inputs": [
    {"text": "[curiously] Sam, did you hear that?", "voice_id": "VOICE_A"},
    {"text": "[whispers] Yeah… [gulps] stay behind me.", "voice_id": "VOICE_B"},
    {"text": "[panicked] We need to go—",              "voice_id": "VOICE_A"},
    {"text": "[interrupting][shouting] RUN!",          "voice_id": "VOICE_B"}
  ],
  "settings": { "stability": "creative" },
  "seed": 12345,
  "output_format": "mp3_44100_128"
}
```

Rules:
- One array element = one continuous utterance by one speaker. Multiple sentences per element are fine; emotions can shift inside an element.
- Shared context: model sees the whole array, so prosody, emotional continuity, and interruption timing carry across speakers — this is the v3 multi-character advantage.
- Use `—` (em dash) at the END of one element and `[interrupting]` / `[cuts in]` at the START of the next to produce realistic interruption.
- Keep ALL `inputs[].text` summed ≤2000 chars per request. For longer scenes, chunk at scene-beats and concatenate audio externally.
- Up to 10 unique voice_ids. No turn count cap.

### 7B. Single-voice TTS with "Speaker N:" labels

Only valid when the SAME voice plays all parts (e.g., a single narrator acting all characters with tag-driven shifts). Format:

```
Speaker 1: [excitedly] Sam! Have you tried the new gear?
Speaker 2: [curiously] Just got it. [whispers] Listen to this—
Speaker 1: [impressed] No way!
```

Labels themselves are NOT spoken if surrounded by colon + line break; v3 treats them as paragraph delimiters. For real per-character voices, use 7A.

---

## 8. AUDIO TAG TAXONOMY (canonical + community-verified)

Tags are free-form. Below is the **anchor set** documented by ElevenLabs plus the broader categorized set repeatedly confirmed in independent sources (Artlist, audio-generation-plugin, Medium tag taxonomy, ElevenLabs help center, Webfuse, SpokenAAC, Jonathan Mast). Treat lists as *seed lists*, not enumerations — the model generalizes.

### 8.1 EMOTIONAL STATE / MOOD
```
[happy] [sad] [excited] [angry] [annoyed] [appalled] [thoughtful]
[surprised] [curious] [nervous] [frustrated] [tired] [calm]
[cheerfully] [flatly] [deadpan] [playfully] [flustered] [casual]
[mischievously] [sarcastic] [sorrowful] [resigned] [hopeful]
[melancholic] [panicked] [terrified] [confident] [hesitant]
[regretful] [embarrassed] [smug] [exasperated] [elated] [amazed]
[awe] [reflective] [wistful] [dramatic tone] [serious tone]
[matter-of-fact] [conversational tone] [sarcastic tone] [lighthearted]
[curious] [crying]
```

### 8.2 DELIVERY / PROSODY / TEMPO
```
[whispers] [whispering] [shouts] [shouting] [yelling] [screaming]
[loudly] [quietly] [softly] [gently] [murmuring] [muttering]
[slowly] [very fast] [rushed] [drawn out] [rapid-fire]
[timidly] [boldly] [emphasized] [stress on next word] [understated]
[singing] [sings] [singsong] [rhythmically] [chanting]
[pause] [short pause] [long pause] [continues after a beat]
[continues softly] [stammers] [hesitates] [pauses] [repeats]
[deliberate] [breathy] [hoarse] [raspy]
```

### 8.3 NON-VERBAL REACTIONS (vocal-tract sounds)
```
[laughs] [laughing] [laughs harder] [laughs softly] [laughs hard]
[starts laughing] [giggles] [chuckles] [light chuckle] [big laugh]
[hysterical laughing] [wheezing] [dying of laughter] [between laughter]
[sighs] [sighing] [sigh of relief] [exhales] [exhales sharply]
[inhales deeply] [breathes] [breath] [heavy breathing]
[gasps] [gasp] [happy gasp] [sharp inhale]
[crying] [sobbing] [sniff] [sniffles] [whimpers]
[clears throat] [coughs] [gulps] [swallows]
[snorts] [grunts] [groans] [moans] [yawns]
[hums] [tsks] [scoffs] [boos] [cheers] [woo] [hiccup]
```

### 8.4 CHARACTER / ARCHETYPE VOICES
```
[deep voice] [doing deep voice, mocking] [high voice] [childlike tone]
[robotic voice] [robotic tone] [pirate voice] [evil scientist voice]
[fantasy narrator] [sci-fi AI voice] [classic film noir]
[old man voice] [old woman voice] [drill sergeant] [auctioneer]
[sports commentator] [news anchor] [villainous] [heroic]
[seductive] [menacing] [grandfatherly]
```

### 8.5 ACCENTS (use "[strong X accent]" for higher reliability)
```
[British accent] [strong British accent] [American accent]
[Southern US accent] [New York accent] [Australian accent]
[Irish accent] [Scottish accent] [Welsh accent]
[French accent] [strong French accent] [German accent]
[Spanish accent] [Italian accent] [Russian accent]
[Indian English accent] [Japanese accent] [Cockney accent]
```
Note: Accent tags are voice-dependent; pick a base voice without a conflicting accent.

### 8.6 SOUND EFFECTS / AUDIO EVENTS (inline, voice-rendered SFX — quality varies)
```
[applause] [clapping] [cheering]
[gunshot] [explosion] [thunder] [knocking] [door creaks]
[footsteps] [gentle footsteps] [leaves rustling] [wind howling]
[bird chirping] [phone ringing] [glass breaking] [crowd murmuring]
[binary beeping] [keyboard typing]
```
Use SFX tags sparingly and verify per voice; layer real SFX in post if reliability matters.

### 8.7 OVERALL DIRECTION / SCENE TYPE (set once, near top)
```
[football] [wrestling match] [horror movie scene] [romantic drama]
[interrogation scene] [bedtime story] [audiobook narrator]
[radio play] [video game cutscene] [trailer voice]
```

### 8.8 MULTI-CHARACTER CONVERSATIONAL CONTROL
```
[interrupting] [cuts in] [overlapping] [jumping in]
[talking over] [trailing off] [finishing each other's sentences]
```
These work best inside Text-to-Dialogue (§7A), placed at the START of the responding speaker's element, with the previous element ending on `—`.

### 8.9 COMPOUND / NESTED DIRECTIONS (alpha — confirmed working)
```
[between laughter]              # speech delivered mid-laugh
[doing deep voice, mocking]     # comma-separated nested intent
[nervous laugh]                 # adjective + non-verbal
[keep screaming]                # carry the previous shout
[inhale and pauses]             # combined action
[talking in a sad way]
[quietly almost whispering]
[breath again]                  # re-anchor
```

---

## 9. TAG COMBINATION MATRIX

### 9.1 Combinations that reliably reinforce each other
```
[whispering] + […] (ellipses)         → suspense / secrecy
[shouting]   + ALL-CAPS + !!!         → max volume + emphasis
[crying]     + [sniff] + […]           → grief beat
[laughing]   + [between laughter]      → laughing-while-speaking
[sarcastic]  + [deadpan]               → dry irony
[nervous]    + [stammers] + [gulps]    → fear stack
[British accent] + [exasperated]       → character + emotion (one persona)
[Southern US accent] + [calmly]
[dramatic tone] + [pause] + [awe]      → reveal beat
[excited] + [rushed]                   → manic delivery
[interrupting] + [shouting]            → angry cut-in
```

### 9.2 Combinations that conflict / cancel / read-out-loud
```
[whispering]  + [shouting]             # mutually exclusive
[robotic]     + [crying]               # timbre fight; one wins
[deadpan]     + [excited]              # cancels both, often flat
[sings]       + [whispering]           # rare voice can do; usually fails
Voice trained calm + [shouting]        # voice timbre overrides tag
Voice trained shouty + [whispering]    # same problem inverted
Many SFX tags ([gunshot] x 4 in <50w)  # destabilizes voice; layer in post
Visual-only tags: [standing] [grinning] [pacing] [music]   # NOT auditory →
   model may read them aloud. Forbidden per ElevenLabs Enhance system prompt.
```

### 9.3 Layering order convention (best results)
```
[ACCENT/CHARACTER] [EMOTION] [DELIVERY] [NON-VERBAL]  text…
e.g.  [French accent][exasperated][quietly] Zat is impossible. [sighs]
```

---

## 10. RELIABILITY HEURISTICS FOR DIFFICULT EFFECTS

### 10.1 Whisper
- Tag: `[whispers]` (verb form often beats noun) or `[whispering]`
- Voice: pick a voice with breathy/quiet samples in training.
- Pair with: lowercase, short clauses, `…`, no `!`.
- Stability: Natural or Creative.

### 10.2 Shout / scream
- Tag: `[shouting]`, then re-anchor with `[keep screaming]` on the next sentence.
- Pair with: ALL-CAPS, `!!!`, exclamation, very short sentences.
- Voice: pick a voice with energetic/loud samples. Soft voices fail.
- Avoid in Robust mode.

### 10.3 Crying / sobbing
- Stack: `[sobbing]` lead, then mid-line `[sniff] [sniff]`, then `[crying]`, then `[sigh]` to release.
- Use `…` heavily; broken sentences (fragments) outperform full sentences.
- Add narrative context BEFORE the line ("She tried to hold it in, but—") to prime emotion.

### 10.4 Laughter
- Pure laugh: `[laughs]`. Sustained while speaking: `[laughing]` or `[between laughter]`.
- Build a ladder: `[chuckles] → [giggles] → [laughs] → [laughs harder] → [hysterical laughing]`.
- Combine with `[wheezing]` or `[dying of laughter]` for peak.

### 10.5 Sarcasm
- Tag: `[sarcastic]` or `[sarcastic tone]`. Add italic-style emphasis with ALL-CAPS on the ironic word.
- Example: `[sarcastic] Oh, GREAT idea. Really brilliant.`
- Often pairs with `[deadpan]`.

### 10.6 Accent
- Use `[strong X accent]` over `[X accent]` for stronger render.
- Place at START of first line and re-anchor every ~150 words.
- Voice base must not contradict (don't ask a thick-Texan voice for `[strong French accent]`).

### 10.7 Singing
- `[sings]` or `[singing]` or `[singsong]`. Highly experimental; results vary widely. For short films, generate 5+ takes.

---

## 11. LANGUAGE & ACCENT CONTROL

### 11.1 Supported language codes (ISO 639-3, 70+)
```
afr ara hye asm aze bel ben bos bul cat ceb nya hrv ces dan nld eng
est fil fin fra glg kat deu ell guj hau heb hin hun isl ind gle ita
jpn jav kan kaz kir kor lav lin lit ltz mkd msa mal cmn mar nep nor
pus fas pol por pan ron rus srp snd slk slv som spa swa swe tam tel
tha tur ukr urd vie cym
```

### 11.2 Cross-language behavior
- Tags work across languages: `[sad]` or `[whispering]` modulate the target-language output equivalently.
- Tags themselves remain English-bracketed even in non-English text.
- A single text block can switch languages mid-stream; v3 will follow.
- For accent within a language: choose base voice trained in that locale first; then add `[strong X accent]` only as fallback.

---

## 12. CHARACTER VOICE CONSISTENCY (across multi-turn dialogue)

```
RULE-1  One voice_id per character. Never multiplex.
RULE-2  In Text-to-Dialogue (§7A), all that character's turns must reuse the
        same voice_id verbatim.
RULE-3  Re-anchor accent/character tag every 2–3 turns; broad tags decay.
RULE-4  Keep stability uniform across a scene (don't switch Creative↔Natural mid-scene).
RULE-5  Pin emotional baseline at start of each turn even if continuing the same mood.
RULE-6  Lock the seed once a take is approved; reuse seed for re-renders of the
        same text. Seed only stabilizes when text + voice + settings are identical.
RULE-7  Avoid having a character do extreme range (whisper → scream) in ONE turn;
        split into two turns to preserve timbre.
RULE-8  For long scenes, chunk at scene beats (≤2000 chars), but pass the same
        voice_ids and seed across chunks for continuity.
```

---

## 13. KNOWN PITFALLS & MITIGATIONS

```
PITFALL  Tags spoken aloud (read-through)
FIX      Voice mismatch or Robust mode. Switch to Natural/Creative, pick more
         expressive voice, lengthen surrounding text >250 chars, regenerate.

PITFALL  Voice drift / timbre morph mid-paragraph
FIX      Lower expressiveness (Natural instead of Creative), shorten paragraph,
         remove conflicting tags, use seed.

PITFALL  Inconsistent results between generations
FIX      Generate 3–5 takes, lock the best with seed. v3 is non-deterministic
         by design.

PITFALL  Professional Voice Clone sounds poor
FIX      PVCs are not optimized for v3. Use Instant Voice Clone or library voice.

PITFALL  Numbers / dates / currency mispronounced
FIX      Normalize before sending: "$1,000" → "one thousand dollars", "2024-01-01"
         → "January first, twenty twenty-four". v3 has built-in normalization but
         it is imperfect; pre-normalize complex strings.

PITFALL  SSML <break> instability (artifacts, sped speech)
FIX      Do NOT use <break> with v3. Use ellipsis, em dash, or [pause] tags.

PITFALL  Excess SFX tags in short window destabilize voice
FIX      ≤1 SFX tag per ~50 words. Layer SFX in post for reliability.

PITFALL  Accent tag ignored
FIX      Voice timbre overrides. Pick a base voice closer to target locale; use
         "[strong X accent]"; re-anchor every paragraph.

PITFALL  Speed drift (model too fast/slow)
FIX      Use the speed setting (0.7 – 1.2; default 1.0). Or insert [slowly] /
         [rushed]. Don't go below 0.7 or above 1.2 (artifacts).

PITFALL  Visual stage directions get read aloud
FIX      Forbidden tags in brackets: [standing] [grinning] [pacing] [music]
         [staring]. Convert to auditory equivalents or describe in narrative text.

PITFALL  Short prompt (<250 chars) → inconsistent / abrupt output
FIX      Always pad to >250 chars with in-character narrative or stage-setting
         prose. This is per official ElevenLabs guidance.
```

---

## 14. CANONICAL PROMPT TEMPLATES FOR SHORT-FILM SCENES

### 14.1 Single-character emotional monologue (Creative stability)

```
[reflective] You know how I've been stuck on that ending for weeks? Just… staring
at the page like it owed me money. [frustrated sigh] I almost trashed it last
night. [pause] But then I sat down again, this time without trying so hard —
and the whole thing just [happy gasp] CLICKED. [laughs softly] It feels…
[whispers] it feels alive now. Like it finally has a soul.
```

### 14.2 Tense two-hander with interruption (Text-to-Dialogue, Natural stability)

```json
{
  "model_id": "eleven_v3",
  "inputs": [
    {"text": "[curiously] You're back early.",                 "voice_id": "VOICE_A"},
    {"text": "[flatly] Yeah.",                                   "voice_id": "VOICE_B"},
    {"text": "[concerned] …did something happen at—",            "voice_id": "VOICE_A"},
    {"text": "[interrupting][cold] Don't. [pause] Just don't.",  "voice_id": "VOICE_B"},
    {"text": "[hurt][whispers] Okay.",                            "voice_id": "VOICE_A"}
  ],
  "settings": {"stability": "natural"},
  "seed": 271828
}
```

### 14.3 Comedic banter (Creative stability)

```json
{
  "inputs": [
    {"text": "[excitedly] Knock knock!",                           "voice_id": "MARK"},
    {"text": "[chuckles][exasperated] Oh god, no. Not again.",     "voice_id": "CHRIS"},
    {"text": "[laughing] Come on, PLEASE — this one's GOOD.",      "voice_id": "MARK"},
    {"text": "[deadpan] The last ten weren't funny. The next ten won't be funny. You're not funny. And you NEVER will be.", "voice_id": "CHRIS"},
    {"text": "[hysterical laughing] How many engineers does it take to—", "voice_id": "MARK"},
    {"text": "[interrupting][shouting] OH MY GOD I'm going home!", "voice_id": "CHRIS"}
  ],
  "settings": {"stability": "creative"}
}
```

### 14.4 Breakdown / crying scene (Creative stability, IVC with emotional samples)

```
[sobbing] I don't know why I'm [sniff] [sniff] crying this hard… [crying] it
just feels like [sigh] a lot right now. [clears throat] I know I'll be okay,
I just need a minute to [sigh] let it out. [whispers] Just a minute.
```

### 14.5 Drill-sergeant / shout scene (Creative)

```
[shouting] LOOK ME IN THE EYES AND TELL ME I'M WRONG!!! [keep screaming]
Tell me you're NOT the rat who's been talking behind our backs!
[quietly almost whispering] Everyone in this room kept their mouth shut…
[inhale and pauses] except one person. So why does it smell like it's you?
[talking in a sad way] If you ARE the rat — [breath again] [shouting] you
better confess now! [quiet again] Before the silence in this room turns
into something you won't walk away from.
```

### 14.6 Accented character (Creative, voice base must be neutral)

```
[strong French accent][playfully] Zat's life, my friend — you can't control
everysing. [laughs softly] But you can pour anozer glass of wine, no?
```

### 14.7 Narration with embedded character lines (single voice)

```
[audiobook narrator][dramatic tone] In the ancient land of Eldoria, where
skies shimmered and forests whispered secrets to the wind, lived a dragon
named Zephyros. [sarcastically] Not the "burn it all down" kind…
[giggles] but he was gentle, wise, with eyes like old stars. [whispers]
Even the birds fell silent when he passed.
```

### 14.8 Multilingual switch within one line

```
[calm] The meeting starts at nine. [strong French accent] Ne sois pas en
retard, d'accord? [English][playfully] Translation: don't be late.
```

---

## 15. ENHANCE-MODE OPERATING CONTRACT (the official Enhance LLM prompt, abridged & encoded)

When the AI is asked to *enhance* (auto-tag) plain dialogue for v3, follow this contract derived from ElevenLabs' internal Enhance prompt:

```
DO
  - Insert audio tags that describe AUDITORY phenomena only.
  - Place tags immediately before (preferred) or immediately after the dialogue
    segment they modify.
  - Diversify emotional palette across the script.
  - Add emphasis via ALL-CAPS on chosen words, add or strengthen ! ? and ellipses
    where contextually justified.
DO NOT
  - Alter, add, or remove any words from the original dialogue.
  - Convert narrative description into tags (don't turn "he laughed loudly" into
    "[laughing loudly] he laughed").
  - Use non-auditory tags: [standing] [grinning] [pacing] [music] [staring].
  - Invent new dialogue lines.
  - Pick tags that contradict the line's intent.
  - Introduce profanity, political, religious, NSFW, or other sensitive material
    that wasn't already there.
WORKFLOW
  1. Read each line, infer mood/context.
  2. Choose 1–3 tags from the taxonomy in §8 (or contextually appropriate synonyms).
  3. Place strategically (before phrase = sets tone; after phrase = punctuates).
  4. Apply emphasis (CAPS / ! / ? / …).
  5. Verify naturalness; remove tags that don't add value.
OUTPUT
  - Enhanced dialogue text ONLY, preserving the conversational layout.
  - All tags inside [square brackets], lowercase preferred.
```

---

## 16. DECISION TREE FOR THE AI WHEN BUILDING A PROMPT

```
INPUT: script line(s), character list, emotional intent

1. Is there >1 distinct character with distinct voices?
   YES → use Text-to-Dialogue array (§7A); one inputs[] entry per turn.
   NO  → use TTS single call.

2. Is the desired emotion extreme (shouting / sobbing / hysterics)?
   YES → set stability = "creative"; pick a voice trained in that register.
   NO  → set stability = "natural".

3. Is the line <250 characters AND non-conversational context?
   YES → pad with in-character lead-in narrative until >250 chars.

4. For each beat in the line:
   - prepend [emotion] tag.
   - if non-verbal reaction is in the script ("she sighed"), replace with [sighs]
     placed at the moment of the action; remove the descriptor word.
   - if interruption is needed, end previous turn with "—" and start next turn
     with [interrupting] or [cuts in].
   - convert "!!!" intent to ALL-CAPS + [shouting] when applicable.
   - convert pauses to "…" or [pause].

5. Validate against §9.2 (conflicts) and §13 (pitfalls). Strip conflicts.

6. Re-anchor accent/character tags every ~150 words or every 2–3 turns.

7. Append seed for reproducibility if a "good take" exists.

8. Emit final prompt.
```

---

## 17. RECOMMENDED DEFAULTS (for the AI's first attempt)

```
model_id           = "eleven_v3"
stability          = "natural"     # upgrade to "creative" if emotion is extreme
similarity_boost   = 0.75          # if exposed by the surface
style              = 0.4           # if exposed; raise toward 0.7 for big performances
use_speaker_boost  = true
speed              = 1.0           # only adjust within 0.7–1.2
output_format      = "mp3_44100_128"
seed               = <random int> then locked once approved
```

---

## 18. CAVEATS (must be surfaced to the calling system)

- **Alpha status.** Behavior, tag responsiveness, and supported features can change without notice. Re-test scripts after model updates.
- **Non-determinism.** Even with `seed`, subtle differences may occur. Plan for multiple takes; budget 3–5× the final-runtime token cost.
- **Tag enumeration is open-ended.** The lists in §8 are the union of (a) official ElevenLabs documentation (best-practices page + Text-to-Dialogue page + Enhance system prompt) and (b) widely-reproduced community taxonomies (Medium tag taxonomy, audio-generation-plugin's curated library of ~1,800 tags, Artlist examples). Treat them as seed patterns; the model accepts free-form auditory descriptors.
- **PVC voices underperform v3 in alpha.** Steer clients to IVC or library voices for film work.
- **SFX tags are unreliable as a class.** For shippable short films, layer real SFX in post; use SFX tags only for scratch tracks or comedic effect.
- **Reading-out-loud failure** still occurs in ~5–15% of generations depending on voice; build a "tag-read detection" review step (auto-transcribe the output with STT and reject takes whose transcript contains bracketed words or known tag stems).
- **Real-time use not supported.** v3 is not designed for live conversational agents (use Flash v2.5 for that).
- **Some sources (e.g., elevenlabsmagazine.com, 11labs-ai.com) are SEO-driven third-party blogs**; their dates ("March 2026", "April 2026") and pricing claims (80% discount through "June 2026") have not been confirmed against ElevenLabs' primary changelog and should be treated as unverified context, not normative. Defer to elevenlabs.io/docs for any pricing / availability assertion.

---

## 19. APPENDIX — MINIMUM-VIABLE PROMPT SKELETON FOR THE AI TO FILL

```
[<scene-overall tag, optional>][<character/accent tag>][<emotion tag>] <line, with
natural punctuation, ALL-CAPS on the stress word, and "…" where appropriate>.
[<non-verbal reaction tag, optional>] <continuation if needed>.
```

Multi-character version (Text-to-Dialogue):

```
inputs[i] = {
  text:  "[<character/accent>][<emotion>][<delivery>] <line with punctuation>.
          <next sentence>. [<non-verbal>] <closing fragment>—",
  voice_id: "<stable voice_id for this character>"
}
```

End of reference.