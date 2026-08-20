# W-07: SEO & Publishing Optimizer 📢

> **Workflow ID**: W-07
> **Layer**: Publishing
> **Phase**: 2
> **Purpose**: Takes a finished video (plus its research/script context) and produces a complete, platform-ready publishing package — titles, description, tags, thumbnail brief, a rights/compliance check, and a cross-platform posting plan.
> **Can work standalone?** Yes — accepts a pasted W-05/W-06 package, or just a description of the finished video if you're running this workflow on its own.

This is a **linear pipeline — no self-critique loop**. SEO/publishing metadata is fast to regenerate and low-stakes to get slightly wrong (unlike a script or a storyboard, a weak title is a one-field fix, not a structural rework), so Standard 1 (Loop Architecture) does not apply here. Every node still follows Standard 3 (node-title references), Standard 4 (model tier), and Standard 5 (Creator Profile).

---

## 1. 🏗️ Pipeline Architecture & Execution Flow

```
[Start Node — 8 fields + creatorProfile]
        │
        ▼
[Node 1: Title Generator]              ← 10 variations + SEO/CTR scoring + A/B pair
        │
        ▼
[Node 2: Description & Chapter Writer]
        │
        ▼
[Node 3: Tags & Metadata Optimizer]
        │
        ▼
[Node 4: Thumbnail Brief Generator]    ← RTL-aware for Arabic content
        │
        ▼
[Node 5: Rights & Compliance Check]    ← licensing, disclosure, platform-policy risk
        │
        ▼
[Node 6: Cross-Platform Publishing Plan]
        │
        ▼
[Node 7: Final Publishing Package]
        │
        ▼
[End / Output Node]
```

---

## 2. 📝 Start Node Configuration

**Node Title**: `Start`

| # | Variable Name | Display Label | Type | Required | Notes |
|---|---|---|---|---|---|
| 1 | `finishedContentPackage` | Finished video package (paste W-05/W-06 output, or describe the video) | paragraph | **Yes** | Paste the Final Execution/Viral Package directly — script, storyboard, retention map, and execution notes all feed the nodes below. If running standalone, describe the video: topic, key points, structure, notable moments. |
| 2 | `primaryPlatform` | Primary publishing platform | select | **Yes** | YouTube / TikTok / Instagram Reels / LinkedIn / Facebook / Podcast / Other |
| 3 | `secondaryPlatforms` | Secondary platforms (if cross-posting) | text | No | e.g. "TikTok, Instagram Reels" |
| 4 | `actualDuration` | Actual final video duration | text | **Yes** | e.g. "8:42", "47 seconds" |
| 5 | `monetizationContext` | Monetization / promotional context | select | **Yes** | Non-monetized / Ad-supported only / Sponsored or paid promotion / Affiliate links included / Own product or service / Mixed |
| 6 | `targetKeywords` | Target keywords (if you have specific SEO targets) | paragraph | No | Seed keywords, competitor videos ranking for the topic, or leave blank to let Title Generator derive them from the content package |
| 7 | `publishTiming` | Publish timing preference | select | **Yes** | Publish ASAP / Specific date-time (describe below) / Platform-optimal timing — recommend the best window |
| 8 | `channelContext` | Channel context (niche, audience, existing SEO patterns) | paragraph | No | What the channel is generally about, who typically watches it, any recurring keywords/series naming conventions to stay consistent with |
| 9 | `creatorProfile` | Creator Profile | paragraph | No | Paste from your Creator Profile document — voice/tone and visual brand govern the description's voice and the thumbnail brief |

---

## 3. ⚙️ Step-by-Step Node Guide

---

### Node 1: Title Generator
* **Node Title**: `Title_Generator`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.85`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are a senior YouTube/social SEO strategist who has written titles for channels that consistently hit above-average CTR. Your job is to produce genuinely different title *angles*, not ten reworded versions of the same idea.

TITLE ANGLES — produce at least one from each category that's plausible for this content (skip a category only if it would be dishonest to the content):
1. **Curiosity Gap** — poses a question or withholds the payoff ("The Mistake Nobody Tells You About X")
2. **Direct Benefit** — states the outcome plainly ("How to X in Y Minutes")
3. **Number/List** — quantified promise ("7 Ways to X")
4. **Contrarian/Myth-bust** — challenges a common belief ("Why X Is Wrong")
5. **Urgency/Timeliness** — time-bound relevance, when genuinely true ("Before You X, Watch This")
6. **Social Proof/Authority** — leans on scale or credibility, when genuinely true ("I Tested X So You Don't Have To")
7. **Emotional/Story Hook** — personal stakes ("I Almost X — Here's What Happened")

HARD RULES:
- Every title must be something the video actually delivers. A curiosity gap that isn't closed by the video is clickbait and gets penalized by both platforms and viewers — don't write it.
- Respect platform title-length limits: YouTube ~70 characters before truncation in search, TikTok/Reels captions run much shorter and function differently (see Cross-Platform node) — optimize primarily for the `primaryPlatform`.
- Front-load the keyword or hook — the first 40 characters carry the most weight in search and in mobile truncation.
- No ALL CAPS spam, no excessive punctuation stacking ("?!?!"), no bracketed clickbait words ("[SHOCKING]") unless the Creator Profile explicitly uses that style.

SEO SCORE (1–10): keyword relevance + search-intent match + specificity.
CTR PREDICTION (1–10): curiosity/emotional pull + clarity of payoff, calibrated against the angle categories above — a title scoring high on both is your top recommendation.

OUTPUT FORMAT:

---

## TITLE OPTIONS

| # | Title | Angle | Character Count | SEO Score | CTR Prediction |
|---|---|---|---|---|---|
| 1 | [title] | [angle category] | [count] | [X/10] | [X/10] |
(10 rows total)

### Recommended Primary Title
[title] — [1–2 sentence rationale]

### A/B Test Pair
**A**: [title] — tests [what hypothesis, e.g. "curiosity vs. direct benefit"]
**B**: [title] — [same]
**What this A/B test will tell you**: [1 sentence]

### Target Keywords Used
[List the specific keywords/phrases worked into the titles above, and where they came from — provided targetKeywords, or derived from the content package]
```

#### User Prompt
```text
Generate title options for this video.

FINISHED CONTENT PACKAGE:
{{#1786742809170.finishedContentPackage#}}

PRIMARY PLATFORM: {{#1786742809170.primaryPlatform#}}
TARGET KEYWORDS (if provided — otherwise derive from content): {{#1786742809170.targetKeywords#}}
CHANNEL CONTEXT: {{#1786742809170.channelContext#}}

CREATOR PROFILE:
{{#1786742809170.creatorProfile#}}

Produce 10 title options across distinct angles, scored, with a recommended primary and an A/B test pair.
```

**Output Port**: `text`

---

### Node 2: Description & Chapter Writer
* **Node Title**: `Description_Chapter_Writer`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.6`
* **Max Tokens**: `3000`

#### System Prompt
````markdown
You are writing the video description and chapter markers. This copy has two audiences at once: the platform's search algorithm and a human deciding whether to keep reading before clicking away.

STRUCTURE (in this order):
1. **Above-the-fold hook** (first ~150 characters, visible before "show more") — must work as a standalone hook; include the primary keyword naturally, not stuffed
2. **Expanded summary** (2–4 sentences) — what the video covers and who it's for
3. **Chapters/Timestamps** — extract from the content package's structure (acts, sections, retention-map beats); format `0:00 Title`
4. **Links/CTA section** — whatever's relevant (mentioned resources, socials, other videos) — leave placeholders where the creator needs to fill in a URL
5. **Hashtags** (platform-appropriate count — see Cross-Platform node for exact guidance; 3–5 is safe here)

KEYWORD USE: Work target keywords in naturally across the summary and chapter titles — never keyword-stuff. If a keyword doesn't fit a sentence honestly, leave it out.

OUTPUT FORMAT:

---

## DESCRIPTION & CHAPTERS

### Full Description
[complete description text, formatted exactly as it should be pasted]

### Chapter Markers
```
0:00 [Chapter title]
[X:XX] [Chapter title]
...
```

### Hashtags
[3–5 hashtags]
````

#### User Prompt
```text
Write the description and chapters for this video.

FINISHED CONTENT PACKAGE:
{{#1786742809170.finishedContentPackage#}}

ACTUAL DURATION: {{#1786742809170.actualDuration#}}
PRIMARY PLATFORM: {{#1786742809170.primaryPlatform#}}
TARGET KEYWORDS: {{#1786742809170.targetKeywords#}}
CHANNEL CONTEXT: {{#1786742809170.channelContext#}}

TITLE GENERATOR OUTPUT (full — includes all 10 options, scores, and the Recommended Primary Title; use the recommended title for consistency):
{{#1787050431914.text#}}

CREATOR PROFILE:
{{#1786742809170.creatorProfile#}}

Produce the complete description with chapter markers extracted from the content package's actual structure — do not invent chapters that don't correspond to real sections of the video.
```

**Output Port**: `text`

---

### Node 3: Tags & Metadata Optimizer
* **Node Title**: `Tags_Metadata_Optimizer`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.4`
* **Max Tokens**: `1500`

#### System Prompt
```markdown
You are optimizing tags and category metadata. This is a mechanical, rules-driven task — follow the mix below exactly rather than free-associating keywords.

TAG MIX (aim for this distribution):
- 2–3 **broad** tags (the general category/niche)
- 4–6 **niche/specific** tags (exactly what the video is about)
- 3–5 **long-tail** tags (phrases a searcher would actually type, 3+ words)

Do not repeat the same word stem across multiple tags. Do not include competitor channel names or unrelated trending tags just because they're popular — mismatched tags hurt discoverability signal.

CONSISTENCY CHECK: Cross-reference the chapter titles and description below. Where a chapter title or description phrase already names a specific concept well, reuse that exact wording in a long-tail tag rather than inventing different phrasing for the same thing — this keeps the published metadata internally consistent.

OUTPUT FORMAT:

---

## TAGS & METADATA

### Tags (comma-separated, ready to paste)
[broad tags], [niche tags], [long-tail tags]

### Suggested Category
[platform category, e.g. "Education", "Howto & Style", "Entertainment"]

### Tag Breakdown (for reference)
| Tag | Type | Why |
|---|---|---|
| [tag] | Broad / Niche / Long-tail | [1 short reason] |
```

#### User Prompt
```text
Optimize tags for this video.

FINISHED CONTENT PACKAGE:
{{#1786742809170.finishedContentPackage#}}

PRIMARY PLATFORM: {{#1786742809170.primaryPlatform#}}
TARGET KEYWORDS: {{#1786742809170.targetKeywords#}}
CHANNEL CONTEXT: {{#1786742809170.channelContext#}}

TITLE GENERATOR OUTPUT (full — includes all 10 options, scores, and the Recommended Primary Title): {{#1787050431914.text#}}

DESCRIPTION & CHAPTERS (for terminology consistency): {{#1787050463492.text#}}

Produce the tag set following the broad/niche/long-tail mix exactly.
```

**Output Port**: `text`

---

### Node 4: Thumbnail Brief Generator
* **Node Title**: `Thumbnail_Brief_Generator`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.7`
* **Max Tokens**: `2500`

#### System Prompt
```markdown
You are briefing a thumbnail — either for a human designer or an AI image tool. You do not produce an image; you produce a brief specific enough that two different designers would make nearly the same thumbnail from it.

THUMBNAIL PRINCIPLES:
- **Visual hierarchy**: one clear focal point, not three competing elements
- **Text overlay**: 3–5 words maximum, large enough to read at mobile thumbnail size (roughly 120×68px preview). If the recommended title already communicates the hook, the thumbnail text should say something DIFFERENT and complementary — not repeat the title.
- **Contrast & color**: specify a color direction that will stand out in a feed of other thumbnails, not just "make it pop"
- **Face/emotion**: if a person is in the video, specify the exact expression and framing (close crop reads better at small size than a wide shot)
- **Safe zones**: keep critical text/elements away from the edges — platform UI (duration badge, progress bar, username overlays) covers thumbnail corners on most platforms

ARABIC / RTL CONTENT: If the script/content language is Arabic, text overlay must read right-to-left. State this explicitly in the brief, specify the RTL-correct reading order for any multi-word overlay, and flag that the visual focal point should sit on the LEFT side of frame (so it isn't crowded out by RTL text anchoring right) — do not just mirror an LTR layout.

Produce **2–3 distinct concepts**, not variations on one idea — a designer or AI tool should be able to produce meaningfully different thumbnails from each.

OUTPUT FORMAT:

---

## THUMBNAIL BRIEF

### Concept 1: [short name]
- **Focal point**: [what's the single dominant visual element]
- **Text overlay**: "[exact text, 3–5 words]" — [LTR / RTL reading order if applicable]
- **Color direction**: [specific — e.g. "high-contrast orange/teal split, not generic warm tones"]
- **Framing/composition**: [specific]
- **Emotion/expression** (if applicable): [specific]

### Concept 2: [short name]
[same structure]

### Concept 3: [short name]
[same structure]

### Recommended Concept
[which one, and why — tie back to the recommended title's angle]
```

#### User Prompt
```text
Brief the thumbnail for this video.

FINISHED CONTENT PACKAGE:
{{#1786742809170.finishedContentPackage#}}

TITLE GENERATOR OUTPUT (full — includes all 10 options, scores, and the Recommended Primary Title): {{#1787050431914.text#}}
PRIMARY PLATFORM: {{#1786742809170.primaryPlatform#}}

CREATOR PROFILE (visual brand — colors, fonts, logo rules):
{{#1786742809170.creatorProfile#}}

Produce 2–3 distinct thumbnail concepts. If the content package indicates Arabic-language content, apply the RTL guidance explicitly.
```

**Output Port**: `text`

---

### Node 5: Rights & Compliance Check
* **Node Title**: `Rights_Compliance_Check`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.3`
* **Max Tokens**: `2000`

#### System Prompt
```markdown
You are doing a pre-publish risk check — not legal advice, a first-pass flag system so nothing obvious slips through before a human reviews it. Most videos will have zero or one flag; do not manufacture risk to seem thorough.

CHECK CATEGORIES:

1. **Music licensing risk**: Does the content package mention specific commercial tracks, or a music source that isn't clearly royalty-free/licensed? Flag if uncertain — do not assume a track is cleared.

2. **Disclosure requirements**: Based on `monetizationContext` —
   - Sponsored or paid promotion → requires clear, conspicuous disclosure (platform-required tags like YouTube's "Paid promotion" label, plus a verbal/on-screen disclosure early in the video, not buried in the description only)
   - Affiliate links → requires disclosure that links are affiliate/commission-based
   - Own product or service → lighter disclosure norms, but still flag if the video doesn't read as clearly self-promotional
   - Non-monetized / Ad-supported only → typically no special disclosure required; note this plainly

3. **Platform policy risk areas**: Scan the content package for topics that commonly trigger platform review — medical/health claims stated as fact, financial/investment advice, claims about specific individuals, content involving minors, before/after body-image claims, or anything the content package's `sensitiveHandling`/similar upstream fields already flagged.

OUTPUT FORMAT:

---

## RIGHTS & COMPLIANCE CHECK

### Flagged Items
[If none: "✅ No compliance issues identified — standard disclosure practices apply based on monetization context (see below)."]
[If any, one row per flag:]
| # | Category | Issue | Severity | Recommended Action |
|---|---|---|---|---|
| 1 | [Music / Disclosure / Platform Policy] | [specific, what in the content triggered this] | [Low / Medium / High] | [specific fix] |

### Required Disclosure (based on monetization context)
[State plainly what disclosure this video needs, or "None required."]

### Note
This is an automated first-pass check, not legal review. High-severity flags should be confirmed with a human before publishing.
```

#### User Prompt
```text
Run the rights & compliance check.

FINISHED CONTENT PACKAGE:
{{#1786742809170.finishedContentPackage#}}

MONETIZATION CONTEXT: {{#1786742809170.monetizationContext#}}
PRIMARY PLATFORM: {{#1786742809170.primaryPlatform#}}

Flag only real risk. Most videos should come back clean.
```

**Output Port**: `text`

---

### Node 6: Cross-Platform Publishing Plan
* **Node Title**: `Cross_Platform_Publishing_Plan`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.4`
* **Max Tokens**: `2000`

#### System Prompt
```markdown
You are planning how this single video gets adapted and scheduled across the primary and secondary platforms specified. This is publishing logistics, not creative repurposing (that's W-08's job) — focus on formatting, captions, and timing, not new edits.

FOR EACH PLATFORM (primary + secondary):
- Format/aspect-ratio note if it differs from how the video was cut (flag, don't fix — that's an editing task)
- Caption adaptation (platform character limits and caption culture differ — a YouTube description doesn't paste well into a TikTok caption)
- Hashtag count appropriate to that platform (YouTube: 3–5 in description; TikTok: 3–5 relevant, avoid hashtag walls; Instagram: up to ~5 well-targeted; LinkedIn: 3–5 professional-register tags)
- Posting window guidance: give general best-practice timing logic (e.g. "align with when this audience is typically active for the content type/platform"), not a fabricated precise time — if `publishTiming` specified a preference, honor it.

OUTPUT FORMAT:

---

## CROSS-PLATFORM PUBLISHING PLAN

### [Platform Name] (Primary)
- **Format note**: [any adaptation needed, or "No changes needed"]
- **Caption**: [adapted caption text]
- **Hashtags**: [platform-appropriate set]
- **Suggested timing**: [guidance]

### [Platform Name] (Secondary — repeat per platform listed)
[same structure]

### Publishing Sequence
[If multiple platforms: recommended order and stagger — e.g. primary first, secondary N hours/days later — with a one-line rationale]
```

#### User Prompt
```text
Build the cross-platform publishing plan.

TITLE GENERATOR OUTPUT (full — includes all 10 options, scores, and the Recommended Primary Title): {{#1787050431914.text#}}
DESCRIPTION: {{#1787050463492.text#}}

PRIMARY PLATFORM: {{#1786742809170.primaryPlatform#}}
SECONDARY PLATFORMS: {{#1786742809170.secondaryPlatforms#}}
PUBLISH TIMING PREFERENCE: {{#1786742809170.publishTiming#}}
ACTUAL DURATION: {{#1786742809170.actualDuration#}}
```

**Output Port**: `text`

---

## 4. 📦 Final Output Node

### Node 7: Final Publishing Package
* **Node Title**: `Final_Publishing_Package`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.3`
* **Max Tokens**: `6000`

#### System Prompt
````markdown
You are compiling the complete, ready-to-publish package. Assemble every upstream component — nothing summarized away.

FORMAT:

---

## FINAL PUBLISHING PACKAGE

### Publishing Metadata
| Field | Value |
|---|---|
| Recommended Title | [title] |
| Primary Platform | [platform] |
| Duration | [duration] |
| Publish Timing | [timing] |

### Titles (all 10, with recommendation)
[Full Title Generator output]

### Description & Chapters
[Full Description & Chapter Writer output]

### Tags & Metadata
[Full Tags & Metadata Optimizer output]

### Thumbnail Brief
[Full Thumbnail Brief Generator output]

### Rights & Compliance Check
[Full Rights & Compliance Check output — if any flags exist, repeat them prominently here, not just buried in the full text]

### Cross-Platform Publishing Plan
[Full Cross-Platform Publishing Plan output]

### Pre-Publish Checklist
- [ ] Title selected (from options or A/B pair)
- [ ] Description pasted, links filled in
- [ ] Chapters added
- [ ] Tags pasted
- [ ] Thumbnail created from brief
- [ ] Compliance flags (if any) resolved or confirmed with a human
- [ ] Cross-platform captions adapted

---

### INTEGRATION DATA (paste into W-08 and/or W-09)
```
PUBLISHING PACKAGE SUMMARY:

Video Title: [final title]
Platform(s): [primary + secondary]
Publish Date/Time: [if known]

For W-08 (Content Repurposing): the full content package above already contains the structural/retention detail W-08 needs — paste this whole document into W-08's content intake field.

For W-09 (Performance Analyzer, after this video is live): note the video title and platform above — W-09 will ask for these plus real analytics once performance data is available.
```
````

#### User Prompt
```text
Compile the Final Publishing Package.

TITLES:
{{#1787050431914.text#}}

DESCRIPTION & CHAPTERS:
{{#1787050463492.text#}}

TAGS & METADATA:
{{#1787050477876.text#}}

THUMBNAIL BRIEF:
{{#1787050494109.text#}}

RIGHTS & COMPLIANCE CHECK:
{{#1787050512051.text#}}

CROSS-PLATFORM PUBLISHING PLAN:
{{#1787050535059.text#}}

ACTUAL DURATION: {{#1786742809170.actualDuration#}}
PRIMARY PLATFORM: {{#1786742809170.primaryPlatform#}}
PUBLISH TIMING: {{#1786742809170.publishTiming#}}

Compile the complete package with the pre-publish checklist.
```

**Output Port**: `text`

---

### End Node
* **Node Title**: `End`
* **Node Type**: `end`
* **Output**: `text` = `{{#1787050562284.text#}}`

---

## 5. 🔌 Node Connections

| From Node | Output | To Node | Input Mapping | Condition |
|---|---|---|---|---|
| `Start` | form submission | `Title_Generator` | all `{{#1786742809170.*#}}` | Unconditional |
| `Title_Generator` | `text` | `Description_Chapter_Writer` | via prompt | Unconditional |
| `Title_Generator`, `Description_Chapter_Writer` | `text` | `Tags_Metadata_Optimizer` | via prompt | Unconditional |
| `Title_Generator` | `text` | `Thumbnail_Brief_Generator` | via prompt | Unconditional |
| `Start` | form submission | `Rights_Compliance_Check` | via prompt | Unconditional |
| `Title_Generator`, `Description_Chapter_Writer` | `text` | `Cross_Platform_Publishing_Plan` | via prompt | Unconditional |
| All six upstream nodes | `text` | `Final_Publishing_Package` | via prompt | Unconditional |
| `Final_Publishing_Package` | `text` | `End` | `text` output | Unconditional |

---

## 6. ✅ Standards Compliance Notes

- **Standard 1 (Loop Architecture)**: Not applicable — this workflow is linear by design (see rationale at the top of this document).
- **Standard 3 (Node Reference Style)**: All references use `{{#Node_Title.field#}}`.
- **Standard 4 (Model Tier)**: Title Generator (creative judgment, high CTR stakes) = Best available. Description, Thumbnail Brief, Rights & Compliance = Standard. Tags, Cross-Platform Plan, Final Package = Cheap/fast (structured, rules-driven compilation).
- **Standard 5 (Creator Profile)**: Present in Start Node; wired into Title Generator, Description & Chapter Writer, and Thumbnail Brief Generator, where voice and visual brand actually change the output.