# W-08: Content Repurposing Agent 🔄

> **Workflow ID**: W-08
> **Layer**: Publishing
> **Phase**: 2
> **Purpose**: Takes a finished video (or a script + storyboard, if the video isn't cut yet) and plans how to repurpose it into other formats and platforms — clips, threads, carousels, articles — plus a distribution calendar.
> **Can work standalone?** Yes — accepts a pasted W-05/W-06/W-07 package, or a script + storyboard directly.

This is a **linear pipeline — no self-critique loop**, for the same reason as W-07: a repurposing plan is a set of recommendations the creator will pick from, not a single artifact that needs to be structurally correct. Standards 3, 4, and 5 still apply.

---

## 0. 🛠️ Revision Notes (Bug Fixes Applied)

This revision fixes six consistency/integration issues found by tracing every variable from the Start node through to the End node:

1. **[Major] `creatorProfile` never reached the clips node.** The Start node field description explicitly promises *"recurring phrases/sign-offs should carry into clips,"* but `Shorts_Reels_Extraction_Plan` never received `{{#1786742809170.creatorProfile#}}` — only `Platform_Adaptation_Planner` did. Re-hooks and captions were being written with no voice guidance, contradicting the field's own spec. **Fixed**: creatorProfile now feeds Node 3 as well, and Standard 5's compliance note reflects it.
2. **[Fix] Architecture diagram showed a false linear chain.** The diagram drew Node 2 → Node 3 in sequence, but the Node Connections table (and the actual data dependencies) show `Content_Audit` fanning out to `Platform_Adaptation_Planner` and `Shorts_Reels_Extraction_Plan` **in parallel**, which then fan back into `Content_Calendar_Integration`. **Fixed**: diagram redrawn to show the branch/merge.
3. **[Fix] `Final_Repurposing_Package`'s Overview table required data it was never given.** The output format calls for `Original Platform` and `Calendar Span starting [date]`, but the node's user prompt never passed `{{#1786742809170.originalPlatform#}}` or `{{#1786742809170.publishStartDate#}}` — the model would've had to fish them out of the embedded calendar text. **Fixed**: both are now passed explicitly.
4. **[Fix] Node Connections table was incomplete.** It documented only `Start → Content_Audit`, omitting the direct `Start →` edges into `Platform_Adaptation_Planner`, `Shorts_Reels_Extraction_Plan`, `Content_Calendar_Integration`, and `Final_Repurposing_Package` that their user prompts actually rely on. **Fixed**: table rewritten to list every real edge.
5. **[Risk] `Final_Repurposing_Package` max tokens likely too low.** Its instruction is *"Assemble every upstream component — nothing summarized away,"* but upstream nodes are budgeted up to 3000+4000+3000+2000 = 12,000 tokens combined, against a 5000-token cap on the compiling node. With multiple repurpose targets selected, this risks silent truncation. **Fixed**: raised to 8000.
6. **[Minor] Naming mismatches.** Node 2's rule section used "Podcast/Audio Clip" while its own output format used "Podcast/Audio Clip Plan" — aligned to one label. Node 3's conditional check said `'Short clips'` while the actual Start field value is `'Short clips (TikTok/Reels/Shorts)'` — tightened so the match is unambiguous.

---

## 1. 🏗️ Pipeline Architecture & Execution Flow

```
[Start Node — 5 fields + creatorProfile]
        │
        ▼
[Node 1: Content Audit]                  ← Finds the 5 most repurposable moments
        │
        ├──────────────────────────────────────┐
        ▼                                        ▼
[Node 2: Platform Adaptation Planner]    [Node 3: Shorts/Reels Extraction Plan]
 TikTok/Reels, X thread, LinkedIn          3–5 short clips, re-hooked and
 carousel, blog, podcast                   re-contextualized (voiced via
        │                                   creatorProfile)
        └──────────────────┬─────────────────────┘
                            ▼
              [Node 4: Content Calendar Integration]   ← 2-week distribution schedule
                            │
                            ▼
              [Node 5: Final Repurposing Package]
                            │
                            ▼
                   [End / Output Node]
```

> Nodes 2 and 3 both depend only on Node 1's output and on Start fields — they run in parallel, not in sequence. Node 4 waits on both before running.

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
{{#1786742809170.contentPackage#}}

ORIGINAL PLATFORM: {{#1786742809170.originalPlatform#}}
REPURPOSE TARGETS REQUESTED: {{#1786742809170.repurposeTargets#}}

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

**Podcast/Audio Clip Plan**:
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
{{#1787051780945.text#}}

FULL CONTENT PACKAGE (for source material and exact quotes/data):
{{#1786742809170.contentPackage#}}

REPURPOSE TARGETS: {{#1786742809170.repurposeTargets#}}

CREATOR PROFILE (voice/tone — apply to thread and carousel copy):
{{#1786742809170.creatorProfile#}}

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
You are planning short-form vertical clips extracted from the long-form source. This node only runs meaningfully if the repurpose targets include "Short clips (TikTok/Reels/Shorts)" — if that value isn't present, produce a one-line note saying short clips weren't requested and stop.

VOICE: If a creator profile is provided, re-hooks and captions should sound like the creator, not like a generic clip-page. Carry over recurring phrases or sign-offs from the profile where they fit naturally — don't force one into every clip.

FOR EACH CLIP (3–5 total, drawn from the Content Audit's strongest moments):
- **In/out points**: where the clip starts and ends in the source (should be a complete arc — don't cut off mid-thought)
- **Re-hook**: the original video's opening doesn't work as this clip's opening — write a new first line specifically for someone with zero context, landing in the first 1 second, in the creator's voice
- **Text overlay context**: what on-screen text this clip needs to make sense standalone (since short-form viewers often watch muted first)
- **Suggested caption**: platform-native caption in the creator's voice, not a repost of the long-form description
- **Why this clip specifically**: what makes it work as a 15–60 second standalone piece

Do not just pick the 5 audited moments mechanically — if two moments are too similar in type, prefer variety (don't submit 3 "surprising fact" clips and skip the emotional beat).

OUTPUT FORMAT:

---

## SHORTS/REELS EXTRACTION PLAN

[If "Short clips (TikTok/Reels/Shorts)" is not among the repurpose targets: "Short clips were not requested for this repurposing plan." — stop here.]

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
{{#1787051780945.text#}}

FULL CONTENT PACKAGE:
{{#1786742809170.contentPackage#}}

REPURPOSE TARGETS: {{#1786742809170.repurposeTargets#}}
ORIGINAL PLATFORM: {{#1786742809170.originalPlatform#}}

CREATOR PROFILE (voice/tone — apply to re-hooks and captions):
{{#1786742809170.creatorProfile#}}

If "Short clips (TikTok/Reels/Shorts)" is not among the repurpose targets, say so plainly and do not force a plan.
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
{{#1787051792211.text#}}

SHORTS/REELS PLAN:
{{#1787051808445.text#}}

PUBLISH START DATE: {{#1786742809170.publishStartDate#}}
POSTING CADENCE: {{#1786742809170.postingCadence#}}
ORIGINAL PLATFORM: {{#1786742809170.originalPlatform#}}

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
* **Max Tokens**: `8000`

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
{{#1787051780945.text#}}

PLATFORM ADAPTATIONS:
{{#1787051792211.text#}}

SHORTS/REELS EXTRACTION PLAN:
{{#1787051808445.text#}}

CONTENT CALENDAR:
{{#Content_Calendar_Integration.text#}}

ORIGINAL PLATFORM: {{#1786742809170.originalPlatform#}}
REPURPOSE TARGETS: {{#1786742809170.repurposeTargets#}}
POSTING CADENCE: {{#1786742809170.postingCadence#}}
PUBLISH START DATE: {{#1786742809170.publishStartDate#}}

Compile the complete package with the execution checklist.
```

**Output Port**: `text`

---

### End Node
* **Node Title**: `End`
* **Node Type**: `end`
* **Output**: `text` = `{{#1787051827278.text#}}`

---

## 5. 🔌 Node Connections

| From Node | Output | To Node | Input Mapping | Condition |
|---|---|---|---|---|
| `Start` | form submission | `Content_Audit` | `contentPackage`, `originalPlatform`, `repurposeTargets` | Unconditional |
| `Start` | form submission | `Platform_Adaptation_Planner` | `contentPackage`, `repurposeTargets`, `creatorProfile` | Unconditional |
| `Start` | form submission | `Shorts_Reels_Extraction_Plan` | `contentPackage`, `repurposeTargets`, `originalPlatform`, `creatorProfile` | Unconditional |
| `Start` | form submission | `Content_Calendar_Integration` | `publishStartDate`, `postingCadence`, `originalPlatform` | Unconditional |
| `Start` | form submission | `Final_Repurposing_Package` | `originalPlatform`, `repurposeTargets`, `postingCadence`, `publishStartDate` | Unconditional |
| `Content_Audit` | `text` | `Platform_Adaptation_Planner` | via prompt | Unconditional |
| `Content_Audit` | `text` | `Shorts_Reels_Extraction_Plan` | via prompt | Unconditional |
| `Content_Audit` | `text` | `Final_Repurposing_Package` | via prompt | Unconditional |
| `Platform_Adaptation_Planner`, `Shorts_Reels_Extraction_Plan` | `text` | `Content_Calendar_Integration` | via prompt (Nodes 2 and 3 run in parallel off Node 1; Node 4 waits on both) | Unconditional |
| `Platform_Adaptation_Planner` | `text` | `Final_Repurposing_Package` | via prompt | Unconditional |
| `Shorts_Reels_Extraction_Plan` | `text` | `Final_Repurposing_Package` | via prompt | Unconditional |
| `Content_Calendar_Integration` | `text` | `Final_Repurposing_Package` | via prompt | Unconditional |
| `Final_Repurposing_Package` | `text` | `End` | `text` output | Unconditional |

---

## 6. ✅ Standards Compliance Notes

- **Standard 1 (Loop Architecture)**: Not applicable — linear pipeline.
- **Standard 3 (Node Reference Style)**: All references use `{{#Node_Title.field#}}`.
- **Standard 4 (Model Tier)**: Content Audit, Platform Adaptation Planner, Shorts/Reels Extraction Plan (judgment-heavy: deciding what's worth repurposing and how) = Standard. Content Calendar and Final Package (rules-driven compilation) = Cheap/fast.
- **Standard 5 (Creator Profile)**: Present in Start Node; wired into `Platform_Adaptation_Planner` (voice governs thread/carousel copy) **and** `Shorts_Reels_Extraction_Plan` (voice governs re-hooks and captions, per the Start field's own spec that recurring phrases/sign-offs should carry into clips).