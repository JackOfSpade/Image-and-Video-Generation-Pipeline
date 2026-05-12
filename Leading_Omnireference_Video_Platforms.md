# Leading Omnireference Video Generation Platforms (2026)

"Omnireference" here means a single video-generation model that accepts **reference inputs across all three modalities — image, video, and audio — in one generation call**, not just text + image. Native audio *output* alone (e.g., Veo 3.1, Sora 2) does not qualify; the model must let the user *upload* audio (and video) clips as conditioning references.

Only the **latest full-tier model** of each platform is considered. Lite, Flash, Mini, Turbo, Fast, and budget variants are excluded even when their version number is numerically higher.

---

## Tier 1 — True Omnireference (Image + Video + Audio reference inputs)

### 1. Alibaba **HappyHorse 1.0** — *Reference-to-Video flagship from Alibaba ATH*
- **Reference inputs:** Text, image, video, and audio — up to **12 multimodal inputs** combined.
- **Architecture:** Unified 40-layer self-attention Transformer, 15B params; all modalities flow through a single shared token sequence (no cross-attention branches). Audio and video are produced in a single forward pass.
- **Output:** 720p/1080p, 4–10s, 16:9 / 9:16 / 1:1, native embedded audio.
- **Quality:** **#1 on the Artificial Analysis Video Arena** (Elo ~1,361 T2V / ~1,398 I2V) — currently the highest-rated reference-to-video model in public benchmarks.
- **Why #1:** Best benchmark scores in its tier *and* full omnireference modality coverage with a genuinely unified joint-generation architecture.

### 2. ByteDance **Seedance 2.0 (Pro)** — *coined the "Omni Reference" workflow*
- **Reference inputs:** Up to **9 reference images + 3 reference videos (2–15 s each) + 3 reference audio clips (2–15 s each)** — 12 files total, each addressable by an `@mention` role (face, camera move, BGM, etc.).
- **Architecture:** Unified multimodal audio-video joint generation; audio and video produced together (lip-sync, ambience, music) in one pass.
- **Output:** 4–15 s, native synchronized audio, strong physical/motion realism.
- **Why #2:** The most *granular* role-assignment for individual reference files among current shipping models and a very mature creator workflow, but trails HappyHorse on Video Arena Elo.

### 3. Kuaishou **Kling 3.0 (Omni)** — *flagship "unified multimodal" tier*
- **Reference inputs:** Text, image, video, and audio in one generation. Image used as start frame / identity / style; video for motion + scene reference; audio for voice / music / ambience reference. Element-library system encodes subjects from ~4 angles for 3D-consistent identity.
- **Output:** Up to ~15 s, multi-shot storyboarding, multi-character coreference, **native 4K @ 60 fps** with synchronized native audio and frame-accurate lip-sync in 5+ languages.
- **Why #3:** The strongest *cinematic* delivery (4K/60, multi-shot directing) among omnireference models, and the only Tier-1 model with full 4K/60 native audio in production.

### 4. Alibaba **Wan 2.6** — *leading open-weights omnireference model*
- **Reference inputs:** T2V, I2V, and **R2V (reference-to-video)** endpoints. Accepts up to **3 reference videos** (3–8 s each) from which it extracts both appearance *and* voice, plus reference images and reference audio for music/SFX/voice driving. Up to 150 reference frames for identity+audio consistency.
- **Output:** 5–15 s, 1080p, multi-shot sequences, native audio with phoneme-level lip-sync.
- **Why #4:** The best open-source omnireference option — full image/video/audio referencing with strong character+voice preservation — but lower fidelity ceiling than HappyHorse / Seedance / Kling in head-to-head Arena comparisons.

---

## Tier 2 — Omnireference-capable but narrower in one axis

### 5. ShengShu **Vidu Q3 (Pro)** — *Reference-to-Video specialist*
- Reference inputs cover subjects, environments, costumes, props, styles, plus start/end-frame control; native 16-second audio-video joint output in 1080p.
- **#1 on the SuperCLUE Reference-to-Video leaderboard** and **#2 on Artificial Analysis Video Arena**, just behind Sora 2 — but its *audio-reference upload* path is less explicit/flexible than Tier 1; primarily image-reference + text-driven audio.

### 6. Tencent **HunyuanCustom** — *open-source research-grade*
- A multimodal-driven architecture explicitly supporting **image, video, audio, and text conditions** (LLaVA-fused text-image, ID enhancement, AudioNet spatial cross-attention, video-driven injection).
- Outperforms VACE, Pika, Vidu, Kling, Hailuo in published ID-consistency and realism benchmarks, but is a research/customization framework rather than a turnkey commercial platform — placed lower for that reason.

---

## Tier 3 — Strong models that are *not* omnireference (for contrast)

These are excluded from the ranking because they lack at least one of the three reference modalities, but they're listed so the boundary is clear:

| Model | Image ref | Video ref | Audio ref | Notes |
|---|---|---|---|---|
| Google **Veo 3.1** | ✅ (up to 3, "Ingredients") | ❌ | ❌ | Native audio *output* only — no audio upload, no video reference |
| OpenAI **Sora 2 Pro** | ✅ (`input_reference`) | ❌ (generated-video → video only) | ❌ | Native audio output, but no user audio/video references |
| Runway **Gen-4.5** | ✅ (character + style refs) | Limited | ❌ | Industry-leading visual quality (Elo ~1,247 T2V) but silent video and no audio-reference input |
| MiniMax **Hailuo 2.3** | ✅ | ❌ in core model | ❌ in core model | Media Agent orchestrates multi-modal assets externally; core model is T2V/I2V without native audio |

---

## Final Ranking (full omnireference platforms only)

| Rank | Platform (full model) | Image | Video | Audio | Joint A/V output | Public quality signal |
|---|---|---|---|---|---|---|
| **1** | **HappyHorse 1.0** (Alibaba) | ✅ ≤9 | ✅ | ✅ | ✅ single-pass | #1 Artificial Analysis Video Arena |
| **2** | **Seedance 2.0 Pro** (ByteDance) | ✅ ≤9 | ✅ ≤3 | ✅ ≤3 | ✅ single-pass | Industry-leading "Omni Reference" workflow |
| **3** | **Kling 3.0 (Omni)** (Kuaishou) | ✅ | ✅ | ✅ | ✅ 4K/60 native audio | Top cinematic delivery, strong Elo |
| **4** | **Wan 2.6** (Alibaba, open) | ✅ | ✅ ≤3 | ✅ | ✅ phoneme-level lip-sync | Best open-weights omnireference |
| 5 | Vidu Q3 Pro (ShengShu) | ✅ | partial | partial | ✅ 16s native | #2 Video Arena overall, narrower ref-audio path |
| 6 | HunyuanCustom (Tencent) | ✅ | ✅ | ✅ | ✅ | Research framework, not a turnkey product |

**Bottom line:** As of mid-2026, the four platforms that genuinely deliver an omnireference workflow at production quality — image *and* video *and* audio as user-uploaded references, with joint audio-video synthesis — are, in order: **HappyHorse 1.0 → Seedance 2.0 Pro → Kling 3.0 Omni → Wan 2.6**. Veo 3.1 and Sora 2 Pro, despite leading certain quality benchmarks, are *not* omnireference systems because neither accepts user-supplied audio or video reference clips.

---

## Sources
- [Alibaba HappyHorse 1.0 — fal.ai](https://fal.ai/happyhorse-1.0)
- [HappyHorse 1.0 capabilities — AI/ML API Blog](https://aimlapi.com/blog/happy-horse-by-alibaba-cloud-full-model-overview-capabilities-pricing-and-use-cases)
- [HappyHorse joint audio-video architecture — Readability](https://www.readability.com/how-to-use-happyhorse-ai-for-native-joint-audio-video-generation-complete-guide)
- [Alibaba revealed as creator of HappyHorse-1.0 — CNBC](https://www.cnbc.com/2026/04/10/alibaba-happyhorse-ai-video-model-benchmark-reveal.html)
- [Seedance 2.0 — seed.bytedance.com](https://seed.bytedance.com/en/seedance2_0)
- [Seedance 2.0 Omni Reference guide — Vicsee](https://vicsee.com/blog/seedance-2-0-omni-reference)
- [Seedance 2.0 vs Kling 3.0 vs Sora 2 vs Veo 3.1 — WaveSpeed](https://wavespeed.ai/blog/posts/seedance-2-0-vs-kling-3-0-sora-2-veo-3-1-video-generation-comparison-2026/)
- [Kling 3.0 Omni Guide — Vidguru](https://www.vidguru.ai/blog/kling-3.0-omni-guide.html)
- [Kling 3 Native 4K 60fps Multi-Modal — Alici AI](https://alici.ai/kling-3)
- [Wan 2.6 Multi-Shot & Reference Video — Imagine.art](https://www.imagine.art/features/wan-2-6)
- [Wan 2.6 Developer Guide — fal.ai](https://fal.ai/learn/devs/wan-26-developer-guide-mastering-next-generation-video-generation)
- [Vidu Q3 Reference-to-Video launch — PRNewswire](https://www.prnewswire.com/news-releases/shengshu-launches-vidu-q3-reference-to-video-with-expanded-visual-and-audio-capabilities-302740489.html)
- [Vidu Q3 16s native audio — Vidu AI](https://www.vidu.com/vidu-q3)
- [HunyuanCustom — arXiv 2505.04512](https://arxiv.org/html/2505.04512v1)
- [HunyuanCustom — Tencent GitHub](https://github.com/Tencent-Hunyuan/HunyuanCustom)
- [Veo 3.1 Ingredients to Video — Google Blog](https://blog.google/innovation-and-ai/technology/ai/veo-3-1-ingredients-to-video/)
- [Veo 3.1 Gemini API docs — Google AI](https://ai.google.dev/gemini-api/docs/video)
- [Sora 2 — OpenAI](https://openai.com/index/sora-2/)
- [Sora 2 video generation guide — OpenAI Developers](https://developers.openai.com/api/docs/guides/video-generation)
- [Runway Gen-4.5 — Runway Research](https://runwayml.com/research/introducing-runway-gen-4.5)
- [MiniMax Hailuo 2.3 — MiniMax News](https://www.minimax.io/news/minimax-hailuo-23)
- [Artificial Analysis Video Arena](https://artificialanalysis.ai/video/arena)
