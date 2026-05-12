# Pipeline Recommendations (May 2026)

## Current Pipeline Recap

```
Midjourney → (Text/Numbers?) → ChatGPT Images  OR  Nano Banana Pro → 2K image → Seedance
```

## Category Leaders (full versions only — lite/flash/mini excluded)

| Category | Leader(s) |
|---|---|
| Prompt adherence | **GPT Image 2** (thinking latent space, ~95% spatial accuracy), Reve Image, Nano Banana Pro |
| Visual quality (aesthetic / cinematic) | **Midjourney V8.1** (king of aesthetics, painterly detail, cinematic lighting) |
| Visual quality (photoreal) | **Imagen 4 Ultra**, **Nano Banana Pro** |
| Composition & structure | **GPT Image 2**, **Nano Banana Pro** (flawless grids, "left/right/30° tilt" spatial reasoning) |
| Style & creative control | **Midjourney V8.1** (style raw, --oref, Omni Reference), GPT Image 2 (style transfer) |
| Text rendering | **GPT Image 2** (Latin + CJK + Hindi + Bengali, multi-step typography), Nano Banana Pro (intricate typography), Ideogram v3 (specialist) |
| Image-to-image editing | **Nano Banana Pro** (14 refs, 5-character consistency), **GPT Image 2** (16 refs, structural fidelity, multi-step edits) |

## Per-Step Analysis

The pipeline must be evaluated as a *chain*, not as independent slots — each node inherits the previous node's output as a reference image.

### Step 1 — Midjourney (text-to-image, no reference)
This slot needs the strongest pure text-to-image aesthetic. **Midjourney V8.1** (default since 30 Apr 2026) retains the lead for cinematic/painterly aesthetics, now outputs HD 2K natively, is ~5× faster, and has materially better prompt adherence, hands, and text-in-quotes rendering. No leader in pure text-to-image aesthetics has overtaken it. **KEEP.**

### Step 2a — ChatGPT Images branch (text/numbers present)
This slot receives Midjourney's output and must add legible text/numbers while preserving composition. The original "ChatGPT Images" entry referred to the gpt-image-1 generation. **GPT Image 2** (released 21 Apr 2026) is a genuine capability jump, not just a version bump:

- Adds a "thinking" latent space that plans composition, checks its work, and iterates
- Native output up to 3840×2160 (4K), vs prior ~1024
- Multi-script text rendering (Latin, Chinese, Japanese, Korean, Hindi, Bengali)
- Up to 16 reference images on the edit endpoint with pixel-stable masking
- In head-to-head testing against Nano Banana 2 Pro and FLUX.2 [max], GPT Image 2 won 4 of 6 categories — including image editing, style transfer, text rendering, and infographics

**UPDATE: ChatGPT Images → GPT Image 2.**

### Step 2b — Nano Banana Pro branch (no text/numbers)
This slot takes Midjourney's output and refines / extends / upscales it. **Nano Banana Pro** (Gemini 3 Pro Image) remains the leader for image-to-image refinement, character consistency (up to 5 people across scenes, 14 reference images), spatial reasoning, and native 2K → 4K upscale with a 16-bit color pipeline. Nano Banana 2 exists but is the faster/lighter sibling — excluded per the "no lite/flash/mini" rule. No challenger has overtaken Nano Banana Pro in this exact slot. **KEEP.**

### Step 3 — Seedance (video, terminal node)
Not altered per instructions. Note for downstream awareness: Seedance 2.0 enforces a strict face policy and rejects realistic human faces on the image-to-video endpoint — relevant if Step 2 outputs photoreal portraits.

## Pipeline-Level Decisions

### Additions — None
- **Reve Image** has best-in-class prompt adherence and editing, but overlaps GPT Image 2's strengths in the text branch and Nano Banana Pro's strengths in the no-text branch. No deficiency is solved by inserting it.
- **FLUX.2 [max]** is strong for brand/luxury reference work but lost 0/6 in the head-to-head and struggled with faces, grids, and text — adding it would not remove a current deficiency.
- **Imagen 4 Ultra** leads pure photorealism, but its slot is text-to-image (same as Midjourney's). Midjourney V8.1 still wins aesthetic/cinematic, and replacing the creative base with a pure-photoreal model would lose stylistic range upstream of both branches.
- **Ideogram v3** is a typography specialist, but GPT Image 2 now matches or exceeds it on accuracy while also handling composition and editing — no orthogonal gain.

### Deletions / Simplifications — None
- Collapsing both branches into Nano Banana Pro: rejected — GPT Image 2 is meaningfully better at in-image text and multi-step typographic edits, which is exactly why the text branch exists.
- Collapsing both branches into GPT Image 2: rejected — Nano Banana Pro is meaningfully better at multi-character consistency and 14-reference compositing, which is exactly why the no-text branch exists.
- Dropping Midjourney: rejected — neither branch model matches its aesthetic ceiling as a creative starting point, and feeding a less-aesthetic base into Step 2 degrades the entire chain.

### Updates — One
- **ChatGPT Images → GPT Image 2** (capability uplift, not a version bump).
- Midjourney is implicitly already on V8.1 as the platform default; no pipeline-text change required, but workflows should rely on its native 2K output and quote-based text syntax.
- Nano Banana Pro is already the latest full version.

## Recommended Pipeline

```
Midjourney
   │
   ▼
Text / Numbers?
   │
   ├── Yes ──▶ GPT Image 2
   │              │
   └── No  ──▶ Nano Banana Pro
                  │
                  ▼
        Output: 2K resolution image
                  │
                  ▼
              Seedance
```

**Net change:** One model updated (ChatGPT Images → GPT Image 2). Structure unchanged. No additions, no deletions.

## Sources

- [Atlas Cloud — Best AI Image Editing Models in 2026 (GPT Image 2, Flux 2 Pro, Nano Banana 2, Seedream)](https://www.atlascloud.ai/blog/guides/best-ai-image-editing-models-2026)
- [Overchat — GPT Image 1.5 vs Nano Banana Pro vs FLUX.2 [max]](https://overchat.ai/ai-hub/ultimate-ai-image-generator-showdown)
- [WaveSpeed — What Is Midjourney V8? Features, Pricing, Speed 2026](https://wavespeed.ai/blog/posts/what-is-midjourney-v8-features-pricing-how-to-use-2026/)
- [FelloAI — Midjourney V8.1 Review: HD by Default, 5× Faster](https://felloai.com/midjourney-v8-1-review/)
- [Google — Nano Banana Pro: Gemini 3 Pro Image](https://blog.google/innovation-and-ai/products/nano-banana-pro/)
- [WaveSpeed — Google Nano Banana Pro Complete Guide 2026](https://wavespeed.ai/blog/posts/google-nano-banana-pro-complete-guide-2026/)
- [Evolink — GPT Image 2 vs GPT Image 1.5 (2026)](https://evolink.ai/blog/gpt-image-2-vs-gpt-image-1-5-2026)
- [fal.ai — GPT Image 2 vs GPT Image 1.5: What's the Difference?](https://fal.ai/learn/tools/gpt-image-2-vs-gpt-image-1-5)
- [LaoZhang — Nano Banana Pro Face Consistency Guide 2026](https://blog.laozhang.ai/en/posts/nano-banana-pro-face-consistency-guide)
- [LM Arena Image Editing Leaderboard](https://arena.ai/leaderboard/image-edit)
- [Artificial Analysis — Image Editing Leaderboard](https://artificialanalysis.ai/image/leaderboard/editing)
- [ZSky AI — AI Image Benchmark: 10,000 Images Tested 2026](https://zsky.ai/blog/ai-image-generator-benchmark-study-2026)
- [Atlas Cloud — Seedance 2.0 Complete Guide 2026](https://www.atlascloud.ai/blog/guides/seedance-2.0-complete-guide)
- [MindStudio — MidJourney V8 vs V7 Comparison](https://www.mindstudio.ai/blog/midjourney-v8-vs-v7-comparison)
