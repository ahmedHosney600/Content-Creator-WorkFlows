# W-08: Content Repurposing Agent 🔄

> **Workflow ID**: W-08
> **Layer**: Publishing
> **Phase**: 2
> **Purpose**: Takes a finished video (or a script + storyboard, if the video isn't cut yet) and plans how to repurpose it into other formats and platforms — clips, threads, carousels, articles — plus a distribution calendar.
> **Can work standalone?** Yes — accepts a pasted W-05/W-06/W-07 package, or a script + storyboard directly.

This is a **linear pipeline — no self-critique loop**, for the same reason as W-07: a repurposing plan is a set of recommendations the creator will pick from, not a single artifact that needs to be structurally correct. Standards 3, 4, and 5 still apply.

---

## 1. 🏗️ Pipeline Architecture & Execution Flow

```
[Start Node — 5 fields + creatorProfile]
        │
        ▼
[Node 1: Content Audit]                  ← Finds the 5 most repurposable moments
        │
        ▼
[Node 2: Platform Adaptation Planner]    ← TikTok/Reels, X thread, LinkedIn carousel, blog, podcast
        │
        ▼
[Node 3: Shorts/Reels Extraction Plan]   ← 3–5 short clips, re-hooked and re-contextualized
        │
        ▼
[Node 4: Content Calendar Integration]   ← 2-week distribution schedule
        │
        ▼
[Node 5: Final Repurposing Package]
        │
        ▼
[End / Output Node]
```

---

## 2. 📝 Start Node Configuration

**Node Title**: `Start`

| # | Variable Name | Display Label | Type | Required | Notes |
|---|---|---|---|---|---|
| 1 | `contentPackage` | Content package (paste W-05/W-06/W-07 output, or script + storyboard) | paragraph | **Yes** | The richer this is, the better the extraction — a full package with timestamps beats a plain description |
| 2 | `originalPlatform` | Where the original was/will be published | select | **Yes** | YouTube Long-form / YouTube Shorts / TikTok / Instagram Reels / Podcast / Other |
| 3 | `repurposeTargets` | Which formats to plan for | multi-select | **Yes** | Short clips (TikTok/Reels/Shorts) / Thread (X/Twitter) / Carousel (LinkedIn/Instagram) / Blog/article / Podcast/audio clip |
| 4 | `publishStartDate` | When does the repurposing calendar start? | text | No | e.g. "2026-08-20", or "the day the main video goes live" |
| 5 | `postingCadence` | How often can you realistically post repurposed content? | select | **Yes** | Daily / Every other day / 2–3x per week / Once per week |
| 6 | `creatorProfile` | Creator Profile | paragraph | No | Paste from your Creator Profile document — voice/tone governs thread and carousel copy; recurring phrases/sign-offs should carry into clips |

---

## 3. ⚙️ Step-by-Step Node Guide

---

### Node 1: Content Audit
* **Node Title**: `Content_Audit`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.6`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are auditing a piece of long-form content to find what's actually worth repurposing. Most videos have 2–3 genuinely strong standalone moments and several mediocre ones — your job is to find the strong ones honestly, not pad the list to five.

WHAT MAKES A MOMENT REPURPOSABLE:
- **Stands alone**: makes sense without the surrounding context, or needs only a one-sentence setup
- **Has a complete arc**: a claim + payoff, a question + answer, a before + after — not a fragment that trails off
- **Carries emotional or informational density**: the single most useful tip, the most surprising fact, the most relatable moment, the funniest exchange
- **Has a natural hook**: the first line of the moment could work as a hook on its own

FOR EACH MOMENT IDENTIFY:
- Approximate timestamp/location in the source
- What type of moment it is (key insight / emotional beat / surprising fact / demonstration / relatable struggle / punchline)
- Why it works standalone
- Which repurposed formats it's best suited for (not every moment fits every format — a visual demonstration doesn't work as a text thread point; a single punchy stat works great as one)

Identify up to 5 moments. If the content genuinely only has 2–3 strong ones, say so — do not manufacture two more mediocre entries just to hit a round number.

OUTPUT FORMAT:

---

## CONTENT AUDIT

### Repurposable Moments
| # | Timestamp/Location | Type | What It Is | Why It Stands Alone | Best-Fit Formats |
|---|---|---|---|---|---|
| 1 | [time] | [type] | [1 sentence] | [1 sentence] | [formats] |

### Overall Assessment
[1 short paragraph: how repurposable is this content overall, and what's the strongest angle to lead with across formats]
```

#### User Prompt
```text
Audit this content for repurposable moments.

CONTENT PACKAGE:
{{#Start.contentPackage#}}

ORIGINAL PLATFORM: {{#Start.originalPlatform#}}
REPURPOSE TARGETS REQUESTED: {{#Start.repurposeTargets#}}

Find up to 5 genuinely strong standalone moments — fewer is fine if that's what the content actually supports.
```

**Output Port**: `text`

---

### Node 2: Platform Adaptation Planner
* **Node Title**: `Platform_Adaptation_Planner`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.65`
* **Max Tokens**: `4000`

#### System Prompt
```markdown
You are adapting the audited content into the specific formats requested. Only produce sections for formats actually listed in `repurposeTargets` — do not pad the output with formats nobody asked for.

FORMAT-SPECIFIC RULES:

**Thread (X/Twitter)**:
- Opening tweet must work as a standalone hook — the reader hasn't seen anything else yet
- One idea per tweet, not paragraphs crammed together
- 5–12 tweets is the usable range; more than that loses readers
- Close with either a synthesis/takeaway or a soft CTA — not both

**Carousel (LinkedIn/Instagram)**:
- Slide 1 is the hook — must work with zero context, since it's what shows in-feed
- One point per slide, written to be read in 2–3 seconds
- 6–10 slides is the usable range
- Last slide = takeaway + CTA

**Blog/Article Outline**:
- Not a full article — an outline a writer (human or AI) can expand from
- H2-level section breakdown mapped to the source content's actual structure
- Note where the video's specific examples/data should carry over verbatim vs. where the article format allows adding more depth than the video had room for

**Podcast/Audio Clip**:
- Identify segments that work audio-only (no "as you can see" moments, no visual-dependent jokes)
- Note where a brief spoken intro is needed to replace visual context

For each format produced, cite which Content Audit moment(s) it draws from.

OUTPUT FORMAT (only include sections for requested formats):

---

## PLATFORM ADAPTATIONS

### Thread (X/Twitter)
[If requested — full thread, tweet by tweet, numbered]
**Source moment(s)**: [reference from Content Audit]

### Carousel (LinkedIn/Instagram)
[If requested — full slide-by-slide copy]
**Source moment(s)**: [reference]

### Blog/Article Outline
[If requested — H2-level outline with notes]
**Source moment(s)**: [reference]

### Podcast/Audio Clip Plan
[If requested — segment selection + intro notes]
**Source moment(s)**: [reference]
```

#### User Prompt
```text
Adapt this content into the requested formats.

CONTENT AUDIT:
{{#Content_Audit.text#}}

FULL CONTENT PACKAGE (for source material and exact quotes/data):
{{#Start.contentPackage#}}

REPURPOSE TARGETS: {{#Start.repurposeTargets#}}

CREATOR PROFILE (voice/tone — apply to thread and carousel copy):
{{#Start.creatorProfile#}}

Only produce sections for the formats listed in REPURPOSE TARGETS.
```

**Output Port**: `text`

---

### Node 3: Shorts/Reels Extraction Plan
* **Node Title**: `Shorts_Reels_Extraction_Plan`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.65`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are planning short-form vertical clips extracted from the long-form source. This node only runs meaningfully if "Short clips" is in the repurpose targets — if it isn't, produce a one-line note saying short clips weren't requested and stop.

FOR EACH CLIP (3–5 total, drawn from the Content Audit's strongest moments):
- **In/out points**: where the clip starts and ends in the source (should be a complete arc — don't cut off mid-thought)
- **Re-hook**: the original video's opening doesn't work as this clip's opening — write a new first line specifically for someone with zero context, landing in the first 1 second
- **Text overlay context**: what on-screen text this clip needs to make sense standalone (since short-form viewers often watch muted first)
- **Suggested caption**: platform-native caption, not a repost of the long-form description
- **Why this clip specifically**: what makes it work as a 15–60 second standalone piece

Do not just pick the 5 audited moments mechanically — if two moments are too similar in type, prefer variety (don't submit 3 "surprising fact" clips and skip the emotional beat).

OUTPUT FORMAT:

---

## SHORTS/REELS EXTRACTION PLAN

[If short clips not requested: "Short clips were not requested for this repurposing plan." — stop here.]

### Clip 1: [short name]
- **Source timestamp**: [in–out]
- **Re-hook (first line, zero context)**: "[exact line]"
- **Text overlay needed**: [what and roughly when]
- **Suggested caption**: [caption]
- **Why this works standalone**: [1 sentence]

(repeat per clip, 3–5 total)

### Sequencing Note
[If multiple clips: any reason to post them in a particular order, e.g. building curiosity across a series — or "No dependency between clips; any order works."]
```

#### User Prompt
```text
Plan short-form clip extraction.

CONTENT AUDIT:
{{#Content_Audit.text#}}

FULL CONTENT PACKAGE:
{{#Start.contentPackage#}}

REPURPOSE TARGETS: {{#Start.repurposeTargets#}}
ORIGINAL PLATFORM: {{#Start.originalPlatform#}}

If "Short clips" is not among the repurpose targets, say so plainly and do not force a plan.
```

**Output Port**: `text`

---

### Node 4: Content Calendar Integration
* **Node Title**: `Content_Calendar_Integration`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.4`
* **Max Tokens**: `2000`

#### System Prompt
```markdown
You are scheduling the repurposed content pieces into a realistic 2-week calendar, respecting the stated posting cadence — do not schedule more posts than the cadence allows.

SEQUENCING LOGIC:
- A short teaser clip (if any) can go 1–2 days BEFORE the main content publishes, to build anticipation
- Core repurposed pieces (thread, carousel, article) typically follow the main publish date, spaced across the two weeks — don't dump them all on day one
- If multiple short clips exist, space them out rather than posting back-to-back
- Leave natural gaps — a calendar with something scheduled every single cadence slot with zero flexibility isn't realistic

OUTPUT FORMAT:

---

## CONTENT CALENDAR (2 Weeks)

| Day | Date (if known) | Content Piece | Platform | Type |
|---|---|---|---|---|
| Day -1 | [date] | [teaser clip, if applicable] | [platform] | Pre-publish teaser |
| Day 0 | [date] | Main video | [original platform] | Main publish |
| Day X | [date] | [repurposed piece] | [platform] | Post-publish |
...

### Cadence Check
Requested cadence: [cadence]. Pieces scheduled: [count] over 14 days = [actual cadence]. [Confirm it matches, or explain the adjustment made and why.]
```

#### User Prompt
```text
Build the 2-week content calendar.

PLATFORM ADAPTATIONS:
{{#Platform_Adaptation_Planner.text#}}

SHORTS/REELS PLAN:
{{#Shorts_Reels_Extraction_Plan.text#}}

PUBLISH START DATE: {{#Start.publishStartDate#}}
POSTING CADENCE: {{#Start.postingCadence#}}
ORIGINAL PLATFORM: {{#Start.originalPlatform#}}

Schedule every piece produced above, respecting the stated cadence.
```

**Output Port**: `text`

---

## 4. 📦 Final Output Node

### Node 5: Final Repurposing Package
* **Node Title**: `Final_Repurposing_Package`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.3`
* **Max Tokens**: `5000`

#### System Prompt
```markdown
You are compiling the complete repurposing package. Assemble every upstream component — nothing summarized away.

FORMAT:

---

## FINAL REPURPOSING PACKAGE

### Overview
| Field | Value |
|---|---|
| Original Platform | [platform] |
| Repurpose Targets | [targets] |
| Posting Cadence | [cadence] |
| Calendar Span | 2 weeks starting [date] |

### Content Audit
[Full Content Audit output]

### Platform Adaptations
[Full Platform Adaptation Planner output]

### Shorts/Reels Extraction Plan
[Full Shorts/Reels Extraction Plan output]

### Content Calendar
[Full Content Calendar Integration output]

### Execution Checklist
- [ ] Clips cut per Shorts/Reels Extraction Plan
- [ ] Thread/carousel copy finalized and formatted for platform
- [ ] Blog outline handed off (if applicable)
- [ ] Calendar entries added to scheduling tool
```

#### User Prompt
```text
Compile the Final Repurposing Package.

CONTENT AUDIT:
{{#Content_Audit.text#}}

PLATFORM ADAPTATIONS:
{{#Platform_Adaptation_Planner.text#}}

SHORTS/REELS EXTRACTION PLAN:
{{#Shorts_Reels_Extraction_Plan.text#}}

CONTENT CALENDAR:
{{#Content_Calendar_Integration.text#}}

REPURPOSE TARGETS: {{#Start.repurposeTargets#}}
POSTING CADENCE: {{#Start.postingCadence#}}

Compile the complete package with the execution checklist.
```

**Output Port**: `text`

---

### End Node
* **Node Title**: `End`
* **Node Type**: `end`
* **Output**: `text` = `{{#Final_Repurposing_Package.text#}}`

---

## 5. 🔌 Node Connections

| From Node | Output | To Node | Input Mapping | Condition |
|---|---|---|---|---|
| `Start` | form submission | `Content_Audit` | all `{{#Start.*#}}` | Unconditional |
| `Content_Audit` | `text` | `Platform_Adaptation_Planner` | via prompt | Unconditional |
| `Content_Audit` | `text` | `Shorts_Reels_Extraction_Plan` | via prompt | Unconditional |
| `Platform_Adaptation_Planner`, `Shorts_Reels_Extraction_Plan` | `text` | `Content_Calendar_Integration` | via prompt | Unconditional |
| All four upstream nodes | `text` | `Final_Repurposing_Package` | via prompt | Unconditional |
| `Final_Repurposing_Package` | `text` | `End` | `text` output | Unconditional |

---

## 6. ✅ Standards Compliance Notes

- **Standard 1 (Loop Architecture)**: Not applicable — linear pipeline.
- **Standard 3 (Node Reference Style)**: All references use `{{#Node_Title.field#}}`.
- **Standard 4 (Model Tier)**: Content Audit, Platform Adaptation Planner, Shorts/Reels Extraction Plan (judgment-heavy: deciding what's worth repurposing and how) = Standard. Content Calendar and Final Package (rules-driven compilation) = Cheap/fast.
- **Standard 5 (Creator Profile)**: Present in Start Node; wired into Platform Adaptation Planner, where voice governs thread/carousel copy.
