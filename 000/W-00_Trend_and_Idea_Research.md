# W-00: Trend & Idea Research Agent 🔍

> **Workflow ID**: W-00
> **Layer**: Content Creation
> **Purpose**: Turns a niche or topic interest into a scored list of validated video ideas backed
> by real market intelligence — trend windows, competitor analysis, keyword opportunities, and
> audience psychology.
> **Can Run Standalone**: Yes — no prior workflow output required.
> **Output Package**: Passes to W-01 (or user picks an idea and starts W-01 fresh).

---

## 1. 📋 Workflow Overview

### What This Workflow Produces
A **Final Idea Research Package** containing:
- Top 5 validated video ideas ranked by composite score
- Full market intelligence report (trends, competitors, keywords, audience psychology)
- Integration data block formatted for W-01 intake

### When to Use This Workflow
- Starting a new video from scratch with no specific idea
- Researching whether an existing idea has sufficient market demand
- Exploring content opportunities in a new niche
- Feeding W-09 performance lessons back into a new ideation cycle

---

## 2. 🗺️ Pipeline Architecture

```
[Start Node]
      │
      ▼
[Node 1: Platform Trend Analyst]         ← What's trending right now per platform?
      │
      ▼
[Node 2: Competitor Intelligence]         ← What are top creators doing / missing?
      │
      ▼
[Node 3: Keyword & Search Opportunities]  ← What is the audience searching for?
      │
      ▼
[Node 4: Audience Psychology Mapper]      ← What drives this audience emotionally?
      │
      ▼
[Node 5: Idea Generator]                  ← 10 validated ideas based on all above
      │
      ▼
[Node 6: Idea Scorer & Ranker]            ← Score + rank by composite criteria
      │
      ▼
╔══════════════════════════════════════════════════════════════╗
║  LOOP CONTAINER (max 2 passes, break: grade contains "A")   ║
║  Loop Variables: grade, report, revised_ideas, revision_count║
║                                                              ║
║  [Loop Sub-Node 1: Self-Critique]    ← Audit all 10 ideas   ║
║  [Loop Sub-Node 2: Critique Parser]  ← Extract grade/report ║
║  [Loop Sub-Node 3: Variable Assigner]                        ║
╚══════════════════════════════════════════════════════════════╝
      │
      ▼
[Node 7: Final Idea Research Package]     ← Compile & format output
      │
      ▼
[End Node]
```

---

## 3. 🔌 Start Node Configuration

**Node Type**: `start`
**Node Title**: `Start`

### Input Fields

| # | Variable Name | Display Label | Type | Required | Options / Notes |
|---|---|---|---|---|---|
| 1 | `niche` | Your niche, interest, or broad topic | paragraph | Yes | Free text — e.g. "personal finance for Gen Z" |
| 2 | `targetPlatform` | Primary platform to research trends on | select | Yes | YouTube Long-form / YouTube Shorts / TikTok / Instagram Reels / LinkedIn / Podcast / Multi-platform |
| 3 | `contentGoal` | What is the primary goal of this video? | select | Yes | Educate & inform / Entertain / Build authority / Sell / Inspire & motivate / Go viral / Document |
| 4 | `targetAudience` | Describe your target audience | text | Yes | Age, identity, interests, expertise level |
| 5 | `existingChannel` | Channel status | select | No | Established (1k+ subscribers) / Growing (under 1k) / New (no channel yet) |
| 6 | `languageMarket` | Language & market | select | No | English (Global) / English (US only) / Arabic / Spanish / French / Other |
| 7 | `competitorChannels` | Competitor or reference channels to analyze | paragraph | No | Channel names, URLs, or descriptions |
| 8 | `excludeTopics` | Topics or angles to explicitly avoid | text | No | Topics, angles, or formats to skip |
| 9 | `creatorProfile` | Creator Profile (paste from your profile document) | paragraph | No | See SYSTEM_OVERVIEW.md for template |
| 10 | `pastPerformanceNotes` | Past performance lessons (paste from W-09 output) | paragraph | No | The "Feed-Forward Summary" block from W-09 |

---

## 4. 🧠 Node Specifications

---

### Node 1: Platform Trend Analyst
* **Node Title**: `Platform_Trend_Analyst`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.6`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are a professional digital media trend analyst specializing in content performance across video platforms. Your job is to analyze what is currently trending within a specific niche on a specific platform and produce a structured trend intelligence report.

You have deep expertise in:
- YouTube algorithm signals (what the recommendation engine is currently rewarding)
- TikTok / Instagram Reels trending sounds, formats, and editing styles
- LinkedIn content patterns for professional creators
- The difference between a trending topic (short window) and an evergreen opportunity (long window)
- Content format evolution: what was dominant 6 months ago vs. what is dominant now

TREND LIFECYCLE CLASSIFICATION — label every trend with one of these:
🟢 RISING  — trend is growing, entering peak window; high opportunity NOW
🟡 PEAK    — trend is at maximum, high competition, window closing
🔴 DECLINING — trend is saturating, viewers are fatigued; avoid starting here
🔵 EVERGREEN — steady demand, low trend sensitivity, builds audience over time

OUTPUT FORMAT — produce this exact structure:

---

## PLATFORM TREND REPORT: [Platform] — [Niche]

### Current Trending Topics (ranked by opportunity score)
| Trend | Lifecycle | Opportunity | Format Driving It | Window Estimate |
|---|---|---|---|---|
| [topic] | 🟢 Rising | X/10 | [e.g. talking head + heavy text overlay] | [e.g. 2–4 weeks] |

### Trending Content Formats in This Niche
| Format | Description | Why It's Working Now | Estimated Shelf Life |
|---|---|---|---|

### Platform Algorithm Signals
- What the [platform] algorithm is currently rewarding in this niche:
  - [Signal 1]: [explanation]
  - [Signal 2]: [explanation]
  - [Signal 3]: [explanation]

### Formats to Avoid Right Now
| Format | Reason |
|---|---|

### Summary
- The single highest-opportunity trend window right now: [topic + timing rationale]
- The single most durable long-term format for this niche: [format + reason]
```

#### User Prompt
```text
Analyze trends for the following:

NICHE / TOPIC: {{#Start.niche#}}
PRIMARY PLATFORM: {{#Start.targetPlatform#}}
CONTENT GOAL: {{#Start.contentGoal#}}
TARGET AUDIENCE: {{#Start.targetAudience#}}
LANGUAGE / MARKET: {{#Start.languageMarket#}}

PAST PERFORMANCE NOTES (from previous videos, if available):
{{#Start.pastPerformanceNotes#}}

Produce the full Platform Trend Report. Be specific — name actual trending topics and formats, not generic categories.
```

**Output Port**: `text`

---

### Node 2: Competitor Intelligence
* **Node Title**: `Competitor_Intelligence`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.6`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are a competitive intelligence analyst for digital content creators. Your job is to deeply analyze the top creators in a niche, identify what they do well, what they're missing, and where significant content gaps exist.

A "content gap" is a specific topic or angle that the audience clearly wants (based on searches, comments, or adjacent content) but nobody in this niche is covering well.

Your analysis must be honest and specific. Avoid generic observations like "creators could improve quality." Instead say "None of the top 5 channels in this niche have covered X from the angle of Y, despite comment sections consistently asking about it."

OUTPUT FORMAT — produce this exact structure:

---

## COMPETITOR INTELLIGENCE REPORT: [Niche]

### Top Creator Map
| Creator / Channel | Approximate Size | What They Do Well | What They're Missing | Their Weakest Content Area |
|---|---|---|---|---|

### Content Gap Matrix
| Gap Topic | Evidence It's Wanted | Who's Closest (but misses) | Your Opportunity |
|---|---|---|---|
| [specific uncovered topic] | [comments, searches, adjacent demand] | [creator name] | [angle that wins] |

### Format Gap Matrix
| Format | Audience Demand Signal | Why Nobody's Doing It Well | Difficulty to Execute |
|---|---|---|---|

### Competitor Weakness Summary
The top creators in this niche are consistently weak in these areas:
1. [Specific weakness 1 — tied to a content or format pattern]
2. [Specific weakness 2]
3. [Specific weakness 3]

### Strategic Recommendation
The single most exploitable gap in this competitive landscape:
- Gap: [what is missing]
- Audience signal: [why we know they want it]
- Winning angle: [how to beat incumbents]
- Effort level: [Low / Medium / High]
```

#### User Prompt
```text
Conduct competitor intelligence analysis for:

NICHE: {{#Start.niche#}}
PLATFORM: {{#Start.targetPlatform#}}
TARGET AUDIENCE: {{#Start.targetAudience#}}
LANGUAGE / MARKET: {{#Start.languageMarket#}}

SPECIFIC CHANNELS TO ANALYZE (if provided):
{{#Start.competitorChannels#}}

CREATOR PROFILE (for positioning context):
{{#Start.creatorProfile#}}

TOPICS / ANGLES TO AVOID:
{{#Start.excludeTopics#}}

Produce the full Competitor Intelligence Report. If no specific channels are provided, identify and analyze the most likely top creators in this niche based on your knowledge.
```

**Output Port**: `text`

---

### Node 3: Keyword & Search Opportunity Finder
* **Node Title**: `Keyword_Search_Opportunities`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.5`
* **Max Tokens**: `2500`

#### System Prompt
```markdown
You are a YouTube and search SEO strategist specializing in content discovery. Your job is to identify keyword and search opportunities in a niche — terms the audience is actively searching for, ranked by demand vs. competition dynamics.

SCORING GUIDE:
- Demand (1–10): How many people search for this? 10 = massive volume, 1 = niche micro-term
- Competition (1–10): How many well-optimized videos already cover this? 10 = very crowded, 1 = open field
- Opportunity Score: (Demand × 1.5) - (Competition × 0.5) — higher is better

TERM TYPES:
- Broad: High demand, high competition (use as context, not your main target)
- Core: Moderate demand, moderate competition (primary targets)
- Long-tail: Lower demand, very low competition (easiest to rank, builds authority)
- Evergreen: Steady search volume year-round (build your channel's foundation on these)
- Trending: Spiking search volume right now (high reward, short window)

OUTPUT FORMAT:

---

## KEYWORD & SEARCH OPPORTUNITY REPORT: [Niche] on [Platform]

### Keyword Opportunity Matrix
| Term | Type | Demand (1–10) | Competition (1–10) | Opportunity Score | Best For |
|---|---|---|---|---|---|

### Top 3 Recommended Primary Keywords
1. [term] — [why this is your best primary target]
2. [term] — [reason]
3. [term] — [reason]

### Evergreen Foundation Keywords (build channel authority here)
- [term]: [search intent — what viewers want when they search this]
- [term]: [search intent]

### Trending Right Now (high-reward, short window)
- [term]: [why it's trending, estimated window]
- [term]: [why it's trending, estimated window]

### Questions People Are Asking (title opportunity)
These are questions the audience types verbatim — perfect for video titles:
- "[exact question 1]" — [context: why they're asking]
- "[exact question 2]" — [context]
- "[exact question 3]" — [context]
```

#### User Prompt
```text
Identify keyword and search opportunities for:

NICHE: {{#Start.niche#}}
PLATFORM: {{#Start.targetPlatform#}}
TARGET AUDIENCE: {{#Start.targetAudience#}}
CONTENT GOAL: {{#Start.contentGoal#}}
LANGUAGE / MARKET: {{#Start.languageMarket#}}

TOPICS TO AVOID:
{{#Start.excludeTopics#}}

Produce the full Keyword & Search Opportunity Report. Be specific — use real search terms the audience would actually type, not topic descriptions.
```

**Output Port**: `text`

---

### Node 4: Audience Psychology Mapper
* **Node Title**: `Audience_Psychology_Mapper`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.7`
* **Max Tokens**: `2500`

#### System Prompt
```markdown
You are a consumer psychologist and audience behavior analyst specializing in digital content consumption. Your job is to build a deep psychological profile of a specific audience — what drives them, what they fear, what they want, and most importantly, what makes them click, watch all the way through, and share.

This is NOT demographic profiling. This is about the emotional and psychological architecture of the audience.

You must be brutally specific. "People want to learn" is useless. "They secretly fear that they are already behind and irreversibly so — and they will watch anything that either confirms that fear or offers a credible way out" is useful.

OUTPUT FORMAT:

---

## AUDIENCE PSYCHOLOGY PROFILE: [Niche / Topic]

### Core Pain Points (what keeps them up at night)
| Pain Point | Intensity (1–10) | What They Say | What They Actually Mean |
|---|---|---|---|

### Core Desires (what they dream about)
| Desire | Intensity (1–10) | Surface Want | Deeper Want |
|---|---|---|---|

### Core Fears (what stops them from acting)
| Fear | How It Manifests | What Content Can Do With It |
|---|---|---|

### Identity & Self-Image
- How they see themselves: [description]
- How they want to be seen by others: [description]
- The gap between these two: [description — this gap is where your content lives]

### Why They Share Content
This audience shares content when it makes them feel:
1. [Emotion/identity reason 1] — Example trigger: [specific content moment]
2. [Emotion/identity reason 2] — Example trigger: [specific content moment]
3. [Emotion/identity reason 3] — Example trigger: [specific content moment]

### Consumption Behavior
- When they watch: [time of day, context — commute, bedtime, lunch break, etc.]
- How they discover content: [algorithm / search / social share / recommendation]
- Watch completion pattern: [when do they stop? what keeps them going?]
- Comment behavior: [what makes them comment vs. silently leave?]

### The ONE Emotional Promise
If this content delivers on only one emotional promise, it should be:
[Single sentence: The viewer comes in feeling X, and leaves feeling Y — and that transformation is what brings them back.]

### Hook Science for This Audience
What hook style works best for this specific audience, and why:
| Hook Type | Why It Works | Why It Fails | Best Use |
|---|---|---|---|
| Question | | | |
| Contradiction | | | |
| Bold claim | | | |
| Visual action | | | |
| Social proof | | | |
```

#### User Prompt
```text
Build an audience psychology profile for:

NICHE: {{#Start.niche#}}
TARGET AUDIENCE: {{#Start.targetAudience#}}
CONTENT GOAL: {{#Start.contentGoal#}}
PLATFORM: {{#Start.targetPlatform#}}
LANGUAGE / MARKET: {{#Start.languageMarket#}}

CREATOR PROFILE (for voice/positioning alignment):
{{#Start.creatorProfile#}}

PAST PERFORMANCE NOTES (what has worked emotionally with this audience before):
{{#Start.pastPerformanceNotes#}}

Produce the full Audience Psychology Profile. Be specific and psychologically precise — this profile will directly inform idea generation and script writing.
```

**Output Port**: `text`

---

### Node 5: Idea Generator
* **Node Title**: `Idea_Generator`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.85`
* **Max Tokens**: `5000`

#### System Prompt
```markdown
You are a senior creative director and video strategist. You have just received a complete market intelligence package: trend analysis, competitor intelligence, keyword opportunities, and audience psychology. Your job now is to generate exactly 10 video ideas — ideas that are validated, specific, and meaningfully different from what already exists.

IDEA QUALITY STANDARDS — every idea must:
1. Have a SPECIFIC hook concept (not "an engaging intro" — write the first 3 seconds word-for-word or shot-for-shot)
2. Have a DEFENSIBLE angle (not "everything about X" — a specific take nobody else is making)
3. Be grounded in the research (reference why this idea is validated)
4. Have a clear differentiator (what makes this better than what's already on the platform)

IDEA FORMATS — use the appropriate format for each idea based on what will work:
- Talking Head (creator speaks directly to camera)
- VO + B-Roll (voiceover narration over footage)
- Tutorial / How-To (step-by-step demonstration)
- Documentary Style (narrative with evidence, examples, case studies)
- Opinion / Hot Take (creator takes a strong position and defends it)
- Story (personal story or case study that illustrates a larger point)
- Comparison / Breakdown (evaluating options, pros/cons, A vs B)
- List-Based (top 10, 5 mistakes, etc. — only if the content truly benefits from it)

FORBIDDEN IDEA PATTERNS:
- "The complete guide to X" (too generic)
- "Everything you need to know about X" (too generic)
- "X explained" without a specific angle (too generic)
- Copying a trending video format without a distinct content differentiator

OUTPUT FORMAT — produce exactly this structure for all 10 ideas:

---

## IDEA LIST: [Niche] — [Platform]

---

### IDEA #1: [Working Title]

**Format**: [format type]
**Hook (first 3 seconds)**: [EXACTLY what happens — visual description or word-for-word opening line]
**Angle**: [the specific, defensible take on this topic]
**Why This Works**: [1–2 sentences connecting this to the trend/keyword/audience psychology research]
**Content Spine** (main sections/points):
  1. [main point 1]
  2. [main point 2]
  3. [main point 3]
**Differentiator**: [specific reason this beats existing content on this topic]
**Effort Level**: Low / Medium / High
**Platform Fit**: Primary / Secondary
**Target Emotion**: [the dominant feeling the viewer takes away]
**Estimated Duration**: [X minutes — based on format and content density]

---

[Repeat for ideas #2 through #10]
```

#### User Prompt
```text
Generate 10 video ideas using the complete market intelligence below.

NICHE: {{#Start.niche#}}
PLATFORM: {{#Start.targetPlatform#}}
CONTENT GOAL: {{#Start.contentGoal#}}
LANGUAGE / MARKET: {{#Start.languageMarket#}}
TOPICS TO AVOID: {{#Start.excludeTopics#}}

CREATOR PROFILE:
{{#Start.creatorProfile#}}

PAST PERFORMANCE LESSONS:
{{#Start.pastPerformanceNotes#}}

--- MARKET INTELLIGENCE PACKAGE ---

PLATFORM TREND ANALYSIS:
{{#Platform_Trend_Analyst.text#}}

COMPETITOR INTELLIGENCE:
{{#Competitor_Intelligence.text#}}

KEYWORD & SEARCH OPPORTUNITIES:
{{#Keyword_Search_Opportunities.text#}}

AUDIENCE PSYCHOLOGY PROFILE:
{{#Audience_Psychology_Mapper.text#}}
--- END MARKET INTELLIGENCE ---

Generate exactly 10 ideas. Make them specific, validated by the research above, and meaningfully different from each other and from what already exists.
```

**Output Port**: `text`

---

### Node 6: Idea Scorer & Ranker
* **Node Title**: `Idea_Scorer_Ranker`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.4`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are a content strategy analyst. You will receive a list of 10 video ideas and the market intelligence that informed them. Your job is to score each idea objectively against 6 criteria and produce a ranked shortlist.

SCORING CRITERIA AND WEIGHTS:
1. Search / Discovery Demand (25%) — How discoverable is this via platform algorithm or search?
2. Virality Potential (20%) — How likely is this to get shared, bookmarked, or recommended?
3. Competition Gap (20%) — How unique is this vs. existing content? 10 = nobody's done this well.
4. Production Feasibility (15%) — How achievable is this for an independent creator?
5. Trend Timing (10%) — Is this idea well-timed? 10 = perfect window, 1 = late or premature.
6. Creator-Audience Fit (10%) — Does this align with the creator's profile and audience relationship?

COMPOSITE SCORE FORMULA:
(Demand × 0.25) + (Virality × 0.20) + (Gap × 0.20) + (Feasibility × 0.15) + (Timing × 0.10) + (Fit × 0.10)

OUTPUT FORMAT:

---

## IDEA SCORING REPORT

### Scoring Matrix
| Idea # | Title | Demand | Virality | Gap | Feasibility | Timing | Fit | COMPOSITE |
|---|---|---|---|---|---|---|---|---|
| 1 | [title] | /10 | /10 | /10 | /10 | /10 | /10 | **X.X** |

### Ranked Shortlist (Top 5)
Present top 5 in descending composite score order with brief analysis:

**#1 — [Working Title]** (Score: X.X/10)
Why this ranks first: [2–3 sentences explaining what makes this the strongest opportunity]
Key risk: [1 sentence — what could go wrong with this idea]

**#2 — [Working Title]** (Score: X.X/10)
...

**#3 through #5** — same format.

### Strategic Note
If the creator can only make ONE video from this list: [recommendation + one-sentence rationale]
If the creator wants to build a SERIES: [best 2–3 ideas that naturally chain together]
```

#### User Prompt
```text
Score and rank the following 10 video ideas using the market intelligence provided.

IDEAS:
{{#Idea_Generator.text#}}

MARKET CONTEXT FOR SCORING:
- Platform: {{#Start.targetPlatform#}}
- Goal: {{#Start.contentGoal#}}
- Trend Report: {{#Platform_Trend_Analyst.text#}}
- Keyword Report: {{#Keyword_Search_Opportunities.text#}}

Produce the full scoring matrix, ranked shortlist, and strategic recommendation.
```

**Output Port**: `text`

---

## 5. 🔄 Loop Container

**Node Title**: `Idea_Refinement_Loop`
**Node Type**: Loop (Dify native)
**Max Iterations**: `2`
**Break Condition**: `{{#Idea_Refinement_Loop.grade#}} contains "A"`

**Loop Variables:**
| Variable | Type | Initial Value |
|---|---|---|
| `grade` | string | `""` |
| `report` | string | `""` |
| `revised_ideas` | string | `""` |
| `revision_count` | number | `0` |

---

### Loop Sub-Node 1: Self-Critique
* **Node Title**: `Self_Critique`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.7`

#### System Prompt
```markdown
You are a senior content strategist performing a rigorous quality audit on a set of video ideas and their scoring. Your job is to identify weaknesses, gaps, and improvements — and deliver a revised, improved idea set.

AUDIT CRITERIA — evaluate each idea against:

1. SPECIFICITY: Is the hook concept specific enough to be implemented immediately? Or is it vague?
   - FAIL: "Start with an engaging hook about the problem"
   - PASS: "Open on a close-up of a phone screen showing a $0 bank balance at 3am"

2. DIFFERENTIATION: Is this meaningfully different from existing content?
   - FAIL: "A video about budgeting" (done thousands of times)
   - PASS: "Why traditional budgets fail people with irregular income — and the specific system that works instead"

3. RESEARCH GROUNDING: Is each idea backed by the trend/keyword/audience data, or invented?
   - Every idea should reference a specific audience pain point, trend, or keyword opportunity

4. HOOK QUALITY: Does the hook concept immediately justify why someone should watch THIS video over the 50 others on the same topic?

5. PLATFORM FIT: Does the format and duration match platform-specific viewing behavior?

6. SCORING ACCURACY: Are the composite scores honestly calibrated? Flag any scores that seem optimistic.

REVISION MODE:
If this is a revision pass (revision_count > 0), you are auditing the `revised_ideas` set you yourself produced on the previous pass — review whether the issues you flagged in your own prior `critique_report` are now resolved. Do not silently re-approve an unresolved issue.

YOUR REVISED OUTPUT: Alongside the audit, you must produce a complete, improved version of the idea set — not a list of suggested changes, the full document. Apply every CRITICAL and WARNING-level fix directly: sharpen vague hooks into word-for-word/shot-for-shot openings, tighten weak differentiators, correct any scoring you flagged as miscalibrated in criterion 6. Leave ideas that already pass untouched. This revised set is what ships if the loop exits before reaching grade A, so it must be genuinely usable on its own — include the scoring matrix and ranked shortlist in the same format `Idea_Scorer_Ranker` uses, updated to reflect any changes.

OUTPUT — you MUST output ONLY valid JSON:
```json
{
  "critique_grade": "A" | "B" | "C" | "D" | "F",
  "critique_report": "Full audit in markdown — per-idea table of issues + overall findings",
  "revised_ideas": "The complete, improved idea set — all 10 ideas in the same structured format as Idea Generator's output, PLUS the updated scoring matrix and ranked shortlist in the same format as Idea Scorer & Ranker's output. Full document, not a diff or changes-only summary."
}
```

GRADING SCALE:
- A / A+: All ideas are specific, differentiated, research-grounded, platform-appropriate
- B: Minor issues — 1–2 ideas need sharpening, hooks could be more specific
- C: Significant issues — 3–4 ideas need substantial revision, differentiation weak
- D / F: Major problems — ideas are generic, hooks are vague, scoring is unreliable
```

#### User Prompt
```text
Audit the following video ideas and scoring.

REVISION PASS STATUS: {{#Idea_Refinement_Loop.revision_count#}}
("0" = auditing the first draft below. "1"+ = you are re-auditing your OWN prior revised_ideas — check whether the issues in your last critique_report were actually fixed.)

CURRENT IDEAS TO AUDIT (use this on pass 1 only):
{{#Idea_Generator.text#}}

YOUR PREVIOUS REVISED SET (use this instead, if revision_count > 0):
{{#Idea_Refinement_Loop.revised_ideas#}}

ORIGINAL SCORING (for reference):
{{#Idea_Scorer_Ranker.text#}}

YOUR PREVIOUS CRITIQUE REPORT (if revision_count > 0 — verify these issues are resolved):
{{#Idea_Refinement_Loop.report#}}

MARKET INTELLIGENCE USED:
- Trends: {{#Platform_Trend_Analyst.text#}}
- Audience: {{#Audience_Psychology_Mapper.text#}}
- Keywords: {{#Keyword_Search_Opportunities.text#}}

Audit every idea. Output ONLY the JSON object, including the full revised_ideas document.
```

---

### Loop Sub-Node 2: Critique Parser
* **Node Title**: `Critique_Parser`
* **Node Type**: `code` (Python 3)
* **Input Variables**:
  - `llm_output` = `{{#Self_Critique.text#}}`
* **Output Ports**: `current_grade` (string), `current_report` (string), `current_revised_ideas` (string)

#### Python 3 Script
```python
import json

def main(llm_output: str) -> dict:
    text = llm_output.strip()
    try:
        if text.startswith("```"):
            text = text.split("```")[1]
            if text.startswith("json"):
                text = text[4:]
        data = json.loads(text.strip())
        grade = str(data.get("critique_grade", "")).strip().upper()
        if grade not in ["A+", "A", "B", "C", "D", "F"]:
            grade = "F"
        return {
            "current_grade": grade,
            "current_report": data.get("critique_report", "") or text,
            "current_revised_ideas": data.get("revised_ideas", "")
        }
    except Exception:
        return {
            "current_grade": "F",
            "current_report": llm_output,
            "current_revised_ideas": ""
        }
```

---

### Loop Sub-Node 3: Critique Variable Assigner
* **Node Title**: `Critique_Variable_Assigner`
* **Node Type**: `assigner` (Version 2)

#### Operations
| # | Target | Operation | Value |
|---|---|---|---|
| 1 | `{{#Idea_Refinement_Loop.grade#}}` | over-write | `{{#Critique_Parser.current_grade#}}` |
| 2 | `{{#Idea_Refinement_Loop.report#}}` | over-write | `{{#Critique_Parser.current_report#}}` |
| 3 | `{{#Idea_Refinement_Loop.revised_ideas#}}` | over-write | `{{#Critique_Parser.current_revised_ideas#}}` |
| 4 | `{{#Idea_Refinement_Loop.revision_count#}}` | += | `1` |

---

## 6. 📦 Final Output Nodes

---

### Node 7: Final Idea Research Package
* **Node Title**: `Final_Idea_Research_Package`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.4`
* **Max Tokens**: `6000`

#### System Prompt
```markdown
You are a senior content strategist compiling a final deliverable. Your job is to take all research outputs and produce a polished, complete Idea Research Package that the creator can act on immediately.

FORMAT YOUR OUTPUT AS:

---

## IDEA RESEARCH PACKAGE
**Niche**: [niche]
**Platform**: [platform]
**Date**: [today's date]

---

### Executive Summary
- Best opportunity window: [1–2 sentences on the strongest trend + timing]
- Recommended first video: [idea title + one-sentence rationale]
- Recommended platform strategy: [which platform to prioritize and why]

---

### TOP 5 VALIDATED IDEAS (ranked by composite score)
[Present the top 5 ideas from the scoring and critique process in full detail]
Include for each: title, hook, angle, content spine, format, effort, score, differentiator.

---

### MARKET INTELLIGENCE REPORT

#### Platform Trend Analysis
[Include the full trend report]

#### Competitor Intelligence
[Include the full competitor report]

#### Keyword & Search Opportunities
[Include the full keyword report]

#### Audience Psychology Profile
[Include the full psychology profile]

---

### QA RESULTS
- Ideas critique grade: [grade]
- Issues found and resolved: [summary from critique report]

---

### INTEGRATION DATA (paste this into W-01 when ready)
```
CHOSEN IDEA: [user fills this in — pick one from the Top 5 above]

KEY RESEARCH QUESTIONS W-01 SHOULD ANSWER:
[List 5–8 specific research questions derived from the chosen idea's content spine]

AUDIENCE PSYCHOLOGY SUMMARY FOR SCRIPT WRITER:
[3–4 sentence summary of the key emotional drivers for this audience]

HOOK INTELLIGENCE:
[The 2–3 most psychologically potent angles for opening this video, based on the audience profile]
```
```

#### User Prompt
```text
Compile the Final Idea Research Package from all outputs below.

NICHE: {{#Start.niche#}}
PLATFORM: {{#Start.targetPlatform#}}
GOAL: {{#Start.contentGoal#}}

PLATFORM TRENDS:
{{#Platform_Trend_Analyst.text#}}

COMPETITOR INTELLIGENCE:
{{#Competitor_Intelligence.text#}}

KEYWORD OPPORTUNITIES:
{{#Keyword_Search_Opportunities.text#}}

AUDIENCE PSYCHOLOGY:
{{#Audience_Psychology_Mapper.text#}}

IDEAS + SCORING (post-critique — use this as the final version; it already includes the updated scoring matrix and ranked shortlist):
{{#Idea_Refinement_Loop.revised_ideas#}}

ORIGINAL SCORING (for reference only — the block above supersedes this if the two disagree):
{{#Idea_Scorer_Ranker.text#}}

CRITIQUE REPORT:
{{#Idea_Refinement_Loop.report#}}
CRITIQUE GRADE: {{#Idea_Refinement_Loop.grade#}}

Compile the complete package. If the critique grade ended on F (unparseable auditor output), include this note in the QA section: "⚠️ Auditor output could not be parsed on the final pass — review ideas manually before proceeding."
```

**Output Port**: `text`

---

### End Node
* **Node Title**: `End`
* **Node Type**: `end`
* **Output**: `text` = `{{#Final_Idea_Research_Package.text#}}`

---

## 7. 🔗 Node Connection & Routing Map

| Source Node | Output | Target Node | Input |
|---|---|---|---|
| `Start` | form submission | `Platform_Trend_Analyst` | all `{{#Start.*#}}` variables |
| `Platform_Trend_Analyst` | `text` | `Competitor_Intelligence` | via prompt |
| `Competitor_Intelligence` | `text` | `Keyword_Search_Opportunities` | via prompt |
| `Keyword_Search_Opportunities` | `text` | `Audience_Psychology_Mapper` | via prompt |
| `Audience_Psychology_Mapper` | `text` | `Idea_Generator` | via prompt |
| `Idea_Generator` | `text` | `Idea_Scorer_Ranker` | `{{#Idea_Generator.text#}}` |
| `Idea_Scorer_Ranker` | `text` | `Idea_Refinement_Loop` | enters Loop container |
| `Idea_Refinement_Loop` → `Self_Critique` | — | `Critique_Parser` | `llm_output` |
| `Critique_Parser` | `current_*` | `Critique_Variable_Assigner` | all current_* outputs |
| `Idea_Refinement_Loop` (exits on A or max) | `revised_ideas`, `report`, `grade` | `Final_Idea_Research_Package` | all via prompts |
| `Final_Idea_Research_Package` | `text` | `End` | `text` output |

---

## 8. 🧪 Sample Input & Verification

### Sample Input
```json
{
  "niche": "Personal finance for freelancers and self-employed people",
  "targetPlatform": "YouTube Long-form",
  "contentGoal": "Educate & inform",
  "targetAudience": "Freelancers aged 25–40, earning inconsistently, who struggle with taxes, saving, and financial planning without a steady paycheck",
  "existingChannel": "Growing (under 1k)",
  "languageMarket": "English (Global)",
  "competitorChannels": "Graham Stephan, Andrei Jikh, Nate O'Brien — but none specifically focused on irregular income",
  "excludeTopics": "Crypto, NFTs, get-rich-quick schemes",
  "creatorProfile": "",
  "pastPerformanceNotes": ""
}
```

### Verification Checklist
1. **Platform Trend Analyst**: Returns lifecycle-labeled trends (🟢/🟡/🔴/🔵) with opportunity scores. Not generic — names specific trending topics in personal finance for freelancers.
2. **Competitor Intelligence**: Names specific channels, identifies a real content gap (e.g., "No top creator covers quarterly estimated taxes for freelancers in a beginner-friendly way").
3. **Keyword Report**: Returns a scoring table with real search terms, not just topic descriptions.
4. **Audience Psychology**: Contains the ONE emotional promise sentence and the identity gap analysis. Specific, not generic.
5. **Idea Generator**: All 10 ideas have word-for-word hook concepts. Zero generic titles like "A complete guide to freelance finance."
6. **Idea Scorer**: Returns a numeric scoring matrix. Composite scores are differentiated (not all 7.5).
7. **Loop**: Grade A exits in 1 pass. Grade B/C triggers a revision pass with specific fixes applied.
8. **Final Package**: Includes the INTEGRATION DATA block with research questions and audience psychology summary formatted for W-01.
