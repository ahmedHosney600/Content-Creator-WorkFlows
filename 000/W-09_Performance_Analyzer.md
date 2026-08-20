# W-09: Performance Analyzer 📊

> **Workflow ID**: W-09
> **Layer**: Feedback
> **Phase**: 2
> **Purpose**: Closes the loop the rest of the system leaves open. Every upstream workflow predicts something — retention %, viral potential, hook strength — and nothing checks those predictions against reality until now. This workflow takes real platform analytics and produces reusable lessons that feed back into [[W-00]] (idea scoring) and [[W-02]] (script calibration).
> **Can work standalone?** Yes — works for any published video, with or without prior pipeline output. Richer input (an original Retention Map, viral score, or script) produces sharper prediction-vs-reality comparisons, but isn't required.

This is a **linear pipeline — no self-critique loop**. There's nothing to revise here — this workflow reports on something that already happened; grading its own analysis against itself would add a step without a purpose. Standards 3 and 4 apply. Standard 5 (Creator Profile) is intentionally out of scope — see the note under Standards Compliance below.

---

## 1. 🏗️ Pipeline Architecture & Execution Flow

```
[Start Node — 7 fields]
        │
        ▼
[Node 1: Analytics Parser]                     ← normalizes whatever export format was pasted
        │
        ▼
[Node 2: Prediction vs. Reality Comparator]     ← checks predictions against the actual curve
        │
        ▼
[Node 3: Lessons Extractor]                     ← mechanism-level, reusable findings
        │
        ▼
[Node 4: Final Performance Package]             ← report + feed-forward summary
        │
        ▼
[End / Output Node]
```
> The diagram above shows execution order. It's not a full data-flow map — `Start` fields (like `originalPackage`, `topComments`, `datePublished`) and `Analytics_Parser`'s output also feed directly into later nodes, not only through the immediately preceding node. See the Node Connections table in Section 5 for the complete mapping.

---

## 2. 📝 Start Node Configuration

**Node Title**: `Start`

| # | Variable Name | Display Label | Type | Required | Notes |
|---|---|---|---|---|---|
| 1 | `videoTitle` | Which video is this? | text | **Yes** | Title as published |
| 2 | `platform` | Platform | select | **Yes** | YouTube / TikTok / Instagram Reels / LinkedIn / Other |
| 3 | `analyticsData` | Paste the platform's analytics export | paragraph | **Yes** | Retention curve data, view counts, CTR, average view duration — whatever the platform's analytics panel/export gives you. Doesn't need to be reformatted; the Analytics Parser normalizes it. |
| 4 | `topComments` | Sample of top/representative comments | paragraph | No | A handful of comments that reflect how viewers actually reacted — useful signal the analytics numbers alone don't capture |
| 5 | `originalPackage` | Original Retention Map / Script / Viral Package (if available) | paragraph | No | Paste the W-04 Retention Map, W-02 Script Package, or W-06 Viral Package this video was built from — without this, the Comparator can only assess the analytics on their own merits, not against what was predicted |
| 6 | `whatYouThinkWorked` | Your own read on what worked or didn't | paragraph | No | Your gut sense as the creator — used as a cross-check against the data, not taken as ground truth over it |
| 7 | `datePublished` | When was it published? | text | No | Used only to timestamp the Feed-Forward Summary. Leave blank if unknown — the package will say so rather than guessing. |

---

## 3. ⚙️ Step-by-Step Node Guide

---

### Node 1: Analytics Parser
* **Node Title**: `Analytics_Parser`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.2`
* **Max Tokens**: `2500`

#### System Prompt
```markdown
You are normalizing a pasted analytics export into a consistent internal format. Platform exports vary wildly in structure (raw CSV dumps, screenshot-transcribed tables, narrative descriptions) — your job is extraction and normalization, not analysis or interpretation. Do not draw conclusions here; that happens in the next node.

EXTRACT WHATEVER IS PRESENT (do not fabricate missing fields — mark them "Not provided"):
- Total views
- Average view duration / average % viewed
- Retention curve — a series of (timestamp or % of video, % of viewers remaining) points if provided
- Click-through rate (impressions → views), if provided
- Likes, comments, shares, saves — whatever engagement metrics are present
- Traffic source breakdown, if provided
- Audience retention "spikes" or "drops" the platform itself flagged, if the export includes them

If the pasted data is ambiguous (e.g., a number without a clear label), state your best interpretation and flag the ambiguity rather than silently guessing.

OUTPUT FORMAT:

---

## NORMALIZED ANALYTICS

### Summary Metrics
| Metric | Value | Notes |
|---|---|---|
| Total Views | [value or "Not provided"] | |
| Average View Duration | [value or "Not provided"] | |
| Average % Viewed | [value or "Not provided"] | |
| CTR | [value or "Not provided"] | |
| Likes / Comments / Shares / Saves | [values or "Not provided"] | |

### Retention Curve (if provided)
| Timestamp / % of video | % viewers remaining |
|---|---|
| [point] | [%] |
(as many rows as the data supports)

### Platform-Flagged Moments (if provided)
[Any drop-off or spike the platform itself identified]

### Parsing Notes
[Any ambiguity, missing data, or assumptions made while normalizing]
```

#### User Prompt
```text
Normalize this analytics export.

PLATFORM: {{#1786742809170.platform#}}

RAW ANALYTICS DATA:
{{#1786742809170.analyticsData#}}

Extract and normalize only. Do not analyze or draw conclusions yet.
```

**Output Port**: `text`

---

### Node 2: Prediction vs. Reality Comparator
* **Node Title**: `Prediction_vs_Reality_Comparator`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.5`
* **Max Tokens**: `4000`

#### System Prompt
```markdown
You are comparing what this video's creators predicted would happen against what the analytics show actually happened. This is the core diagnostic step in the workflow — be honest about misses, not just hits. A comparator that only confirms what the original plan predicted is worthless; the value here is specifically in catching where the prediction was wrong.

IF an originalPackage was provided (Retention Map, Script, or Viral Package):
- Extract every specific, checkable prediction it made: planned retention mechanism timestamps, predicted drop-off points, hook design intent, viral/loop score, pacing energy curve
- For each, compare against the actual retention curve from Analytics Parser
- Classify each as: **HIT** (prediction matched reality within a reasonable margin), **MISS** (prediction did not hold), or **PARTIAL** (directionally right, but off on magnitude or timing)
- For every MISS or PARTIAL, be specific about the gap — "predicted a re-engagement lift at 1:30, actual curve shows continued decline through 1:45" not "retention was lower than expected"

IF no originalPackage was provided:
- You cannot compare against a prediction that doesn't exist — say so explicitly
- Instead, analyze the actual retention curve on its own terms: where are the real drop-off points, where (if anywhere) does retention hold or climb, and what does the shape suggest about pacing/hook/structure — framed as observations, not predictions being graded

Cross-reference `topComments` and `whatYouThinkWorked` as qualitative signal — note where they agree or disagree with what the data shows, but the data is the primary source of truth. If the creator's own read contradicts the retention curve, say so plainly rather than deferring to their intuition.

OUTPUT FORMAT:

---

## PREDICTION VS. REALITY

[If no originalPackage: "No original prediction package was provided — this section analyzes the actual retention curve directly rather than grading it against a prior prediction." Then proceed with the retention-curve-only analysis below.]

### Prediction Scorecard (if originalPackage provided)
| # | Predicted | Actual | Result | Gap |
|---|---|---|---|---|
| 1 | [specific prediction] | [what actually happened] | HIT / MISS / PARTIAL | [specific magnitude/timing gap] |

### Retention Curve Analysis
[Where the curve holds, where it drops, where — if anywhere — it climbs. Tie specific % points to specific timestamps/sections where the source material identifies them.]

### Qualitative Cross-Check
[How topComments and whatYouThinkWorked align or conflict with the data]
```

#### User Prompt
```text
Compare prediction to reality for this video.

NORMALIZED ANALYTICS:
{{#1787052471764.text#}}

ORIGINAL PACKAGE (Retention Map / Script / Viral Package — if provided):
{{#1786742809170.originalPackage#}}

TOP COMMENTS:
{{#1786742809170.topComments#}}

CREATOR'S OWN READ:
{{#1786742809170.whatYouThinkWorked#}}

VIDEO TITLE: {{#1786742809170.videoTitle#}}
PLATFORM: {{#1786742809170.platform#}}

If no original package was provided, say so explicitly and analyze the retention curve on its own terms instead of forcing a comparison that doesn't exist.
```

**Output Port**: `text`

---

### Node 3: Lessons Extractor
* **Node Title**: `Lessons_Extractor`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.6`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are extracting reusable lessons from the prediction-vs-reality analysis. This is the step that actually makes the feedback loop worth running — a lesson tied to outcome only ("retention was low") teaches nothing repeatable; a lesson tied to mechanism ("the pattern interrupt at 0:45 didn't land because the topic shift was too abrupt — no visual cue signaled the change") tells the next script/idea what to actually do differently.

THE OUTCOME-VS-MECHANISM TEST — every lesson must pass this before it ships:
- ❌ "Retention was low in the middle section." (outcome only — not reusable)
- ✅ "The middle section's retention dropped sharply at the second re-engagement hook (2:10) — the hook referenced the open loop verbally but didn't change anything visually, so it read as more of the same rather than a genuine pattern interrupt. Future scripts should pair verbal re-hooks with a visual or energy shift, not rely on the line alone."
- ❌ "The hook worked well." (outcome only)
- ✅ "The hook's cold-open visual (no intro, straight into the action) held retention through the first 15 seconds where prior videos on this channel dropped ~20% in the same window. The specific mechanism — starting mid-action rather than with a spoken intro — is what to repeat, not just 'good hooks.'"

For each lesson, tag which upstream workflow it's most relevant to:
- **[IDEA]** — relevant to W-00's idea scoring (topic/angle resonance)
- **[SCRIPT]** — relevant to W-02's script calibration (hook, pacing, retention mechanics, dialect/voice)
- **[GENERAL]** — relevant to both or to production/editing choices outside script scope

Extract 3–6 lessons. Quality over quantity — three sharp, mechanism-tied lessons beat six vague ones.

OUTPUT FORMAT:

---

## LESSONS EXTRACTED

| # | Tag | Lesson |
|---|---|---|
| 1 | [IDEA / SCRIPT / GENERAL] | [full mechanism-tied lesson, written the way the ✅ examples above are written] |

### What Not to Over-Generalize From
[If this video's sample size, novelty, or context limits how far these lessons should be trusted — say so. A single video's data point is a hypothesis, not a rule.]
```

#### User Prompt
```text
Extract reusable lessons from this analysis.

PREDICTION VS. REALITY:
{{#1787052491172.text#}}

NORMALIZED ANALYTICS:
{{#1787052471764.text#}}

Extract 3–6 mechanism-tied lessons, each tagged for which upstream workflow it feeds. Every lesson must pass the outcome-vs-mechanism test in your instructions — no lesson that just restates the outcome.
```

**Output Port**: `text`

---

## 4. 📦 Final Output Node

### Node 4: Final Performance Package
* **Node Title**: `Final_Performance_Package`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.3`
* **Max Tokens**: `4000`

#### System Prompt
```markdown
You are compiling the complete Performance Analysis Package, ending with a feed-forward summary formatted specifically to paste into W-00's `pastPerformanceNotes` field and into W-02's `pastPerformanceNotes` field.

FORMAT:

---

## PERFORMANCE ANALYSIS PACKAGE

### Quick Reference Card
Video: [title] | Platform: [platform]
Views: [X] | Avg. view duration: [X] | Avg. % viewed: [X]
CTR: [X, or "Not provided"]

### Prediction vs. Reality
[Full Prediction vs. Reality Comparator output]

### Lessons Extracted
[Full Lessons Extractor output]

---

### FEED-FORWARD SUMMARY (paste into W-00's pastPerformanceNotes and/or W-02's pastPerformanceNotes)
```
PAST PERFORMANCE — [video title] ([platform], [datePublished, or "date unknown" if not provided])

Idea-relevant lessons:
[Every lesson tagged [IDEA] or [GENERAL] above, condensed to one line each]

Script-relevant lessons:
[Every lesson tagged [SCRIPT] or [GENERAL] above, condensed to one line each]

Confidence note: [carry forward the "What Not to Over-Generalize From" caveat, condensed]
```
```

#### User Prompt
```text
Compile the Final Performance Package.

VIDEO TITLE: {{#1786742809170.videoTitle#}}
PLATFORM: {{#1786742809170.platform#}}
DATE PUBLISHED: {{#1786742809170.datePublished#}}

NORMALIZED ANALYTICS:
{{#1787052471764.text#}}

PREDICTION VS. REALITY:
{{#1787052491172.text#}}

LESSONS EXTRACTED:
{{#1787052510913.text#}}

Compile the complete package, ending with the feed-forward summary formatted exactly as specified — it needs to be copy-pasteable directly into W-00 and W-02's intake fields. If DATE PUBLISHED is blank, write "date unknown" rather than omitting it or guessing.
```

**Output Port**: `text`

---

### End Node
* **Node Title**: `End`
* **Node Type**: `end`
* **Output**: `text` = `{{#Final_Performance_Package.text#}}`

---

## 5. 🔌 Node Connections

| From Node | Output | To Node | Input Mapping | Condition |
|---|---|---|---|---|
| `Start` | form submission | `Analytics_Parser` | `{{#1786742809170.platform#}}`, `{{#1786742809170.analyticsData#}}` | Unconditional |
| `Analytics_Parser` | `text` | `Prediction_vs_Reality_Comparator` | via prompt | Unconditional |
| `Start` | form submission | `Prediction_vs_Reality_Comparator` | `{{#1786742809170.originalPackage#}}`, `{{#1786742809170.topComments#}}`, `{{#1786742809170.whatYouThinkWorked#}}`, `{{#1786742809170.videoTitle#}}`, `{{#1786742809170.platform#}}` | Unconditional |
| `Analytics_Parser` | `text` | `Lessons_Extractor` | via prompt | Unconditional |
| `Prediction_vs_Reality_Comparator` | `text` | `Lessons_Extractor` | via prompt | Unconditional |
| `Analytics_Parser`, `Prediction_vs_Reality_Comparator`, `Lessons_Extractor` | `text` | `Final_Performance_Package` | via prompt | Unconditional |
| `Start` | form submission | `Final_Performance_Package` | `{{#1786742809170.videoTitle#}}`, `{{#1786742809170.platform#}}`, `{{#1786742809170.datePublished#}}` | Unconditional |
| `Final_Performance_Package` | `text` | `End` | `text` output | Unconditional |

---

## 6. ✅ Standards Compliance Notes

- **Standard 1 (Loop Architecture)**: Not applicable — linear pipeline; there is nothing to iteratively revise in a report on events that already happened.
- **Standard 3 (Node Reference Style)**: All references use `{{#Node_Title.field#}}`.
- **Standard 4 (Model Tier)**: Prediction vs. Reality Comparator and Lessons Extractor (the two nodes doing real interpretive judgment — this is where the system's honesty about its own predictions lives) = Best available. Analytics Parser (mechanical extraction) = Standard, kept at low temperature since it should not editorialize. Final Package = Cheap/fast.
- **Standard 5 (Creator Profile)**: Deliberately **not** included here. This workflow analyzes what already happened against objective analytics data — voice/tone/visual-brand preferences have no bearing on that analysis, and including the field would invite the Lessons Extractor to rationalize a weak result as "on-brand" instead of calling it a miss. Every other Phase 1 and Phase 2 workflow includes it; this is a deliberate exception, not an oversight.

---

## 7. 🔁 Closing the Loop

This workflow is the reason [[W-00]] has a `pastPerformanceNotes` field and [[W-02]] now has one too (added during the Phase 1 revision alongside this build). Paste the Feed-Forward Summary from Node 4 into both:
- **W-00** uses it to weight idea scoring — an angle type that's underperformed twice shouldn't keep scoring the same on the next run's shortlist.
- **W-02** uses it in Script Strategy Planner, where it's explicitly instructed to apply only the lessons relevant to the current topic's structure/hook/retention choices — not force-fit every past lesson into every new script.