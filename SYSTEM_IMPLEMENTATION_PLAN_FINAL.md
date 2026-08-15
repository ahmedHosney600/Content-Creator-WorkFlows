# 🎬 Complete Video Production System — Final Implementation Plan

> **This is the authoritative plan.** It merges the v1 architecture with all v2 refinements
> and decisions. Nothing is marked "open question" — every decision is resolved here.

---

## The Core Philosophy

Each workflow is a **standalone agent** that produces a **standardized output package**.
Any workflow can consume another's output package as input OR run with its own fresh intake.
No workflow forces another to run — they are **integratively designed, independently operable**.

Two governing principles:

- **One system, one set of conventions.** Ten independently-built workflows stay maintainable only
  when they share the same loop mechanism, the same Critique Parser schema, and the same reference
  style. The System-Wide Standards section below defines these — every workflow assumes them.

- **Prompts, not pixels.** Where this system touches AI image or video generation, its job stops at
  producing a ready-to-use prompt (and, for images, an optional low-cost preview via a tool call).
  It never calls a video-generation API directly — that step stays a deliberate, human-triggered
  action outside these flows.

---

## 🧱 System-Wide Standards

These apply to every workflow — new and upgraded — so the system reads as one thing built by one
team, not ten.

---

### Standard 1 — Loop Architecture: Dify Native Loop Container, Everywhere

No workflow hand-rolls a loop with IF/ELSE back-edges and a Python string counter. Every
self-critique/revision cycle uses Dify's native **Loop** container node:

- **Loop variables declared inside the container**: `grade` (string, initial `""`), `report`
  (string, initial `""`), one `revised_<artifact>` per artifact being refined, `revision_count` (number, initial `0`).
- **Break condition**: `{{#Loop.grade#}} contains "A"`. **Max iterations: 2.**
- **Sub-nodes inside the loop, in order**: `[Content-generating node(s)]` → `Self-Critique (LLM)` → `Critique Parser (Code)` → `Critique Variable Assigner`
- The Assigner overwrites each loop variable from the Critique Parser's `current_*` outputs every pass and increments `revision_count` by `+= 1`.
- Content-generating nodes read `revision_count` and the prior `report`/`revised_*` values to decide whether they are on an initial pass or a revision pass — the "MODE: REVISION PASS" pattern W-04's Storyboard Builder already uses correctly. This is the reference all other workflows copy.

**W-04** already implements this correctly — it becomes the template.
**W-05 and W-06** require a loop rebuild: delete `Revision_Applier`, `Revision_Counter`, `Loop_Count_Guard`, and the manual back-edge. Replace with one Loop container (details in each workflow section).

---

### Standard 2 — Critique Parser: One Schema, Fail Closed to `F`

Every Critique Parser follows one contract:

**Self-Critique (LLM) output** — JSON with exactly:
- `critique_report` (string) — full markdown audit
- `critique_grade` (string) — one of: `A+` / `A` / `B` / `C` / `D` / `F`
- Workflow-specific extra fields (W-06 adds `loop_score` and `viral_score`; W-02 adds `dialect_authenticity_score` in Arabic mode)

**Critique Parser (Code) output** — matching Loop container variable names:
- `current_grade` (string)
- `current_report` (string)
- `current_revised_<artifact>` — one per artifact in that workflow's loop

**On any parse failure — malformed JSON, missing keys, exception — return `current_grade = "F"`. Never `"A"`. Never `"B"`.**

**Canonical Python template:**

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
            # "current_revised_<artifact>": data.get("revised_<artifact>", "")
        }
    except Exception:
        return {
            "current_grade": "F",
            "current_report": llm_output,
            # "current_revised_<artifact>": ""
        }
```

When the loop escapes on grade `F`, the Final Package must say so explicitly:
*"⚠️ Auditor output could not be parsed on the final pass — review this section manually before publishing."*

---

### Standard 3 — Node Reference Style

New and rebuilt guides reference upstream nodes by **title** (`{{#Node_Title.field#}}`), matching W-05/W-06's existing convention. W-04's ID-based references are left as-is; the pattern does not propagate to new guides.

---

### Standard 4 — Model Tier Assignment

| Tier | Use For | Examples |
|---|---|---|
| **Best available** | Creative judgment, quality gating | Self-Critique (every workflow), Hook Writer, Script Body Writer, Idea Generator, Egyptian Dialect Authenticity Layer |
| **Standard** | Structured planning following clear rules | Storyboard Builder, Shot List Extractor, Creative Strategy, most first-pass generation nodes |
| **Cheap/fast** | Formatting, extraction, compilation | Tags & Metadata Optimizer, Cross-Platform Plan, Critique Parser paths, Final Package compilers |

Under-provisioning Self-Critique is the one substitution that quietly undermines the whole architecture.

---

### Standard 5 — Creator Profile (Shared Context, Not a Workflow)

A `creatorProfile` (paragraph, optional) field is available on every workflow's Start Node. The user maintains one Creator Profile document and pastes it per run.

**Creator Profile template:**
```
CREATOR PROFILE
- Channel / brand name & niche:
- Target audience (who, and why they watch):
- Voice & tone (3–5 adjectives, plus what to avoid):
- Recurring phrases / catchphrases / sign-offs:
- Dialect / language register (see W-02 for Arabic specifics):
- Topics or angles permanently off-limits:
- Reference scripts/videos that nailed the voice (link or paste):
- Visual brand (colors, fonts, logo usage rules):
```

---

## 📐 Complete System Map

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          CONTENT CREATION LAYER                              │
│                                                                              │
│  W-00 Trend &    ──▶  W-01 Topic      ──▶  W-02 Script  ──▶  W-03 Production │
│  Idea Research         Research &            Writer             Planning &    │
│  (+ past lessons       Info Collector        (Egyptian          Filming Guide │
│  from W-09)            [web search           Arabic layer)      (+ AI footage │
│                         required]                                prompt writer)│
└──────────────────────────────────────────────────────────────────────────────┘
                                              │ raw footage
                                              ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           VIDEO EDITING LAYER                                │
│                                                                              │
│  W-04 Video Pre-Planning Pipeline (UPGRADED)                                 │
│  storyboard + pacing + retention + thumbnails + B-Roll Studio                │
│  (native Loop — reference implementation)                                    │
│                 │                                       │                    │
│                 ▼                                       ▼                    │
│  W-05 Post-Production Execution          W-06 Viral Speed Ramp Pipeline      │
│  (native Loop rebuild, fail-to-F,         (native Loop rebuild, compound     │
│   rough cut gate, export presets)          loop-quality gate, fail-to-F)     │
└──────────────────────────────────────────────────────────────────────────────┘
                                              │
┌──────────────────────────────────────────────────────────────────────────────┐
│                           PUBLISHING LAYER                                   │
│  W-07 SEO & Publishing Optimizer (+ Rights & Compliance Check)               │
│  W-08 Content Repurposing Agent                                              │
└──────────────────────────────────────────────────────────────────────────────┘
                                              │ after publishing
┌──────────────────────────────────────────────────────────────────────────────┐
│                            FEEDBACK LAYER                                    │
│  W-09 Performance Analyzer                                                   │
│  real retention/CTR data → lessons → feeds back into W-00 & W-02            │
└──────────────────────────────────────────────────────────────────────────────┘

Creator Profile (user-maintained, outside any workflow) feeds every Start Node above.
```

### Entry Points — Where You Can Start

| You Have | Start At |
|---|---|
| Nothing — just an interest or niche | W-00 |
| An idea, no research yet | W-01 |
| Research notes, no script | W-02 |
| A script, no footage | W-03 |
| A script + raw footage | W-04 |
| Raw footage only (no script) | W-04 |
| Edited rough cut needing polish | W-05 |
| Short clips for viral content | W-06 |
| A finished video needing publishing | W-07 |
| A published video and its analytics | W-09 |

---

## W-00: Trend & Idea Research Agent 🔍

Turns a niche / topic / interest into a **scored list of validated video ideas** with market intelligence, trend windows, competition analysis, and audience insights. Receives lessons from W-09 on runs after the first.

**Can work standalone? Yes.**

### Architecture

```
[Start Node — 8 fields + creatorProfile + pastPerformanceNotes]
          │
          ▼
[Platform Trend Analyst (LLM — Standard)]
[Competitor Intelligence (LLM — Standard)]
[Keyword & Search Opportunity Finder (LLM — Standard)]
[Audience Psychology Mapper (LLM — Standard)]
[Idea Generator (LLM — Best) — 10 ideas]
[Idea Scorer & Ranker (LLM + Code — Standard)]
          │
╔══════════════════════════════════════════╗
║  LOOP (max 2, break: grade contains "A") ║
║  Self-Critique (Best) → Critique Parser  ║
║  → Critique Variable Assigner            ║
╚══════════════════════════════════════════╝
          │
          ▼
[Final Idea Research Package (LLM — Cheap/fast)]
[End / Output]
```

### Start Node Fields

| Variable | Type | Label | Required |
|---|---|---|---|
| `niche` | paragraph | Your niche, interest, or broad topic | Yes |
| `targetPlatform` | select | Primary platform for trend research | Yes |
| `contentGoal` | select | Educate / entertain / sell / inspire / document / go viral | Yes |
| `targetAudience` | text | Describe your target audience | Yes |
| `existingChannel` | select | Established / new / none | No |
| `languageMarket` | select | English Global / Arabic / Spanish / French / Other | No |
| `competitorChannels` | paragraph | Competitor or reference channels to analyze | No |
| `excludeTopics` | text | Topics or angles to avoid | No |
| `creatorProfile` | paragraph | Paste your Creator Profile | No |
| `pastPerformanceNotes` | paragraph | Paste W-09 feed-forward summary | No |

### Key Node Specifications

**Platform Trend Analyst** — trend name, platform, lifecycle (🟢 Rising / 🟡 Peak / 🔴 Declining / 🔵 Evergreen), opportunity window, content format driving it.

**Competitor Intelligence** — top 5–10 creator map, content gap matrix, format gap matrix.

**Keyword & Search Opportunity Finder** — keyword opportunity matrix: term | demand (1–10) | competition (1–10) | opportunity score.

**Audience Psychology Mapper** — 5 pain points, 5 desires, 3 fears, 3 aspirations, sharing psychology, the ONE emotional promise this niche content fulfills.

**Idea Generator** — exactly 10 ideas. Per idea:
```
Idea #X: [Working Title]
- Format:          [talking head / documentary / tutorial / story / etc.]
- Hook Concept:    [exact first 3 seconds — word-for-word or visual description]
- Angle:           [specific, defensible take — not "the basics of X"]
- Why Now:         [trend timing rationale]
- Effort Level:    [Low / Medium / High]
- Platform Fit:    [primary / secondary]
- Target Emotion:  [the feeling the viewer takes away]
- Differentiator:  [how it beats existing content on this topic]
```

**Idea Scorer & Ranker** — composite scoring matrix per idea:

| Criterion | Weight | Score (1–10) |
|---|---|---|
| Search / discovery demand | 25% | |
| Virality potential | 20% | |
| Competition gap (uniqueness) | 20% | |
| Production feasibility | 15% | |
| Trend timing | 10% | |
| Alignment with creator strengths | 10% | |
| **Composite Score** | 100% | **X.X / 10** |

**Self-Critique Audit Criteria:**
- Are ideas based on actual trend data or speculation?
- Are hook concepts specific enough to be immediately actionable?
- Is competition analysis realistic?
- Are audience pain points validated or assumed?
- Is each idea sufficiently differentiated?

**Output Package:**
```
## IDEA RESEARCH PACKAGE

### Executive Summary
Best opportunity: [niche + timing rationale]
Recommended platform strategy: [which platform and why]

### Top 5 Validated Ideas (ranked by composite score)
[Full spec for each]

### Market Intelligence Report
- Platform Trend Analysis
- Competitor Intelligence Map
- Content Gap Matrix
- Keyword Opportunities
- Audience Psychology Profile

### Integration Data (for W-01)
- chosen_idea: [user fills before passing to W-01]
- key_research_questions: [auto-generated questions W-01 should answer]
- audience_psychology: [summary for W-02]
```

---

## W-01: Topic Research & Information Collector 🔬

Takes a chosen video idea and researches it deeply — gathering authentic, sourced information from multiple angles and integrating the creator's own knowledge.

**This agent is an information architect, not a script writer.** It collects, verifies, organizes, and structures. It does not write scripts.

**Can work standalone? Yes.**

### Non-Negotiable: Real Tool Access

The VERIFIED/PROBABLE/CLAIMED/DISPUTED labeling system is the entire value of this workflow. Without a live web-search tool wired into Multi-Source Research Collector, those labels are decoration on hallucinated content. This node is `LLM + Tool Call`. **W-01 does not ship without a working Dify web-search node or external search API (Tavily, SerpAPI, or equivalent) connected.**

The **Research Expander** node (revision loop) must also be `LLM + Tool Call` — on a revision pass it must issue new searches for specific gaps the critique named, not generate plausible-sounding filler.

### Architecture

```
[Start Node — 10 fields + creatorProfile]
          │
          ▼
[Research Question Architect (LLM — Standard)]
[Multi-Source Research Collector (LLM + Tool Call — Standard)]
[Source Credibility Validator (LLM — Standard)]
[User Knowledge Integrator (LLM — Standard)]
[Information Architecture Builder (LLM — Standard)]
          │
╔═══════════════════════════════════════════════════════╗
║  LOOP (max 2, break: grade contains "A")              ║
║  Research Expander (LLM + Tool Call — Standard)       ║
║  ← performs NEW searches for named gaps;              ║
║    never generates filler from model knowledge        ║
║  Self-Critique (Best) → Critique Parser → Assigner    ║
╚═══════════════════════════════════════════════════════╝
          │
          ▼
[Final Research Package (LLM — Cheap/fast)]
[End / Output]
```

### Start Node Fields

| Variable | Type | Label | Required |
|---|---|---|---|
| `chosenTopic` | paragraph | Video topic or paste Idea Package from W-00 | Yes |
| `contentType` | select | Educational / documentary / opinion / tutorial / story / review / other | Yes |
| `targetDuration` | select | Approximate target video length | Yes |
| `researchDepth` | select | Surface / Standard / Deep / Academic | Yes |
| `userKnowledge` | paragraph | What you already know / your personal take | No |
| `personalStories` | paragraph | Personal experiences or stories to integrate | No |
| `mandatoryFacts` | text | Key facts/stats that must be included | No |
| `anglePerspective` | text | The specific angle or thesis you want to argue | No |
| `topicsToAvoid` | text | Subtopics or claims to avoid | No |
| `sensitiveAreas` | text | Sensitive topics requiring careful handling | No |
| `creatorProfile` | paragraph | Paste your Creator Profile | No |

### Key Node Specifications

**Research Question Architect:**
- Primary research questions (must answer)
- Secondary research questions (answer if possible)
- Out-of-scope questions (explicitly excluded to prevent scope creep)
- Information architecture skeleton: sections the research will populate

**Multi-Source Research Collector — Six Categories:**

| Category | What to Find | Minimum |
|---|---|---|
| Statistics & Data | Credible statistics with source, date, URL | 3–5 items |
| Expert Authority | Expert quotes or studies with credentials | 2–3 items |
| Real-World Examples | Concrete case studies or examples | 3–5 items |
| Historical Context | Background that explains the current state | 1–2 items |
| Counter-Arguments | Strongest argument against the main thesis | 1 item |
| Recent Developments | Anything from the last 12 months | 2–3 items |

**Source Credibility Validator — Label System:**

| Label | Meaning |
|---|---|
| ✅ VERIFIED | Multiple credible sources agree, recent, high authority |
| ⚠️ PROBABLE | One strong source, logic supports it, minor reservations |
| 📌 CLAIMED | Single source, not independently verified — use with attribution |
| ❌ DISPUTED | Contradicted by other sources — note contradiction explicitly |
| 🔒 USER-SOURCED | Creator's personal knowledge/experience — marked clearly as personal POV |

**User Knowledge Integrator — Three Integration Types:**
1. **Corroborating** — user's experience confirms the research → boosts confidence
2. **Unique perspective** — user has insights the research doesn't cover → valuable differentiator
3. **Contradicting** — user's experience conflicts with research → flag for script writer to handle

**Information Architecture Builder:**
- Core Claims: the 3–5 main points the video MUST make
- Supporting Evidence per claim
- Narrative bridges: how ideas connect
- Hook opportunities: most surprising or counterintuitive findings
- Story elements: human examples, anecdotes, case studies
- CTA-worthy findings: what's immediately actionable for the audience

**Self-Critique Audit Criteria:**
- Enough content for the target duration?
- All major claims backed by sourced evidence?
- Information gaps explicitly named?
- Information organized so a script writer can use it immediately?
- User insights clearly distinguished from external research?

**Output Package:**
```
## RESEARCH PACKAGE

### Quick Reference Card
Topic: [topic]
Items collected: X | Verified: X | Probable: X | Claimed: X | Disputed: X
Estimated content: enough for X minutes

### Core Claims (prioritized)
1. [Claim] — [Evidence summary] — [Source]
...

### Full Evidence Bank
[By category — all sourced]

### User Knowledge Integration
[Personal stories, perspectives, insider knowledge — labeled]

### Information Gap Report
[What couldn't be found / what the creator should still research]

### Hook Intelligence
[Top 3 most surprising/counterintuitive findings — likely hook candidates]

### Integration Data (for W-02)
- research_depth_score: X/10
- content_density: enough for X–Y minutes
- suggested_content_structure: [brief outline based on information flow]
```

---

## W-02: Script Writer Agent ✍️

Transforms research and information into a **production-ready script** for any content type, style, platform, and language. Calibrated to the creator's voice, the platform's demands, and the audience's psychology. Understands talking-head pacing, VO narration rhythm, documentary structure, platform-specific hooks, and language localization.

**Can work standalone? Yes.**

### Architecture

```
[Start Node — 14 fields + creatorProfile]
          │
          ▼
[Script Strategy Planner (LLM — Standard)]
[Hook Writer (LLM — Best) — 3 variations]
[Script Body Writer (LLM — Best)]
[CTA & Retention Layer Writer (LLM — Standard)]
[Egyptian Dialect & Voice Authenticity Layer (LLM — Best)]  ← Arabic mode only
[Script Refinement (LLM — Standard)]
          │
╔══════════════════════════════════════════════════════════════╗
║  LOOP (max 2)                                                ║
║  Break condition (English): grade contains "A"               ║
║  Break condition (Arabic):  grade contains "A"               ║
║                             AND dialect_authenticity_score>=8║
║  Self-Critique (Best) → Critique Parser → Assigner           ║
╚══════════════════════════════════════════════════════════════╝
          │
          ▼
[Final Script Package (LLM — Cheap/fast)]
[End / Output]
```

### Start Node Fields

| Variable | Type | Label | Required |
|---|---|---|---|
| `researchPackage` | paragraph | Research Package from W-01 or your notes/outline | Yes |
| `contentType` | select | Talking head / VO narration / documentary / tutorial / opinion essay / interview / story / other | Yes |
| `scriptLanguage` | select | English / Arabic (Egyptian) / Spanish / French / Other | Yes |
| `dialectRegister` | select | *(Arabic only)* Cairene media-standard / Street & youth slang / Educated professional / Sa'idi | Conditional |
| `codeSwitchingLevel` | select | *(Arabic only)* None / Light (occasional tech & brand terms) / Moderate (bilingual style) | Conditional |
| `targetDuration` | text | Target video duration (e.g., 8 minutes) | Yes |
| `targetPlatform` | select | YouTube / TikTok / Instagram Reels / LinkedIn / other | Yes |
| `voiceStyle` | select | Conversational / authoritative / energetic / calm / educational / storytelling / raw-authentic | Yes |
| `audienceLevel` | select | General public / beginners / intermediate / expert | Yes |
| `scriptFormat` | select | Full word-for-word / bullet-point outline / hybrid | Yes |
| `ctaGoal` | select | Subscribe / visit link / buy / follow / watch next / share / none | No |
| `mandatoryMentions` | text | Things that MUST appear in the script | No |
| `referenceScripts` | paragraph | Reference scripts/channels whose style to emulate | No |
| `sensitiveHandling` | text | Topics requiring careful/nuanced handling | No |
| `creatorProfile` | paragraph | Paste your Creator Profile | No |

### Key Node Specifications

**Script Strategy Planner:**
- Narrative structure selection with justification (Hero's Journey / Problem-Solution / Before-After / Curiosity Gap / Documentary / Storytelling / Tutorial / Opinion Essay)
- Word count target: `duration × WPM rate` (120–160 WPM narration / 140–180 WPM high-energy talking head)
- Hook window: [X seconds]
- Act breakdown with approximate word counts per act
- Retention mechanism schedule: loop plants, re-hooks, pattern interrupts
- Platform-specific rules applied to this script
- Language/cultural calibration notes

**Hook Writer — 3 variations:**
```
HOOK A — Type: [Question / Contradiction / Action / Visual / Social Proof]
[Full scripted hook — word for word, with performance cues]
Hook science score: X/10 — [reasoning]

HOOK B — [different type] ...
HOOK C — [different type] ...
RECOMMENDED: A/B/C — [reason]
```

**Script Body Writer — format per section:**
```
=== [SECTION NAME] | Timestamp: [0:00–0:45] | Energy: [1–5] ===

[VISUAL NOTE: B-roll of X / Camera: talking head / etc.]

[Script text — word for word]

[PAUSE: 1 second]

[EMPHASIS: the following line should land with power]

[More script text]

[VISUAL NOTE: cut to X / text overlay: "Y"]

---
```

**CTA & Retention Layer Writer:**
```
OPEN LOOPS:
- Loop 1 planted at [0:15]: "[exact line]"
- Loop 1 re-referenced at [50%]: "[exact line]"
- Loop 1 resolved at [80%]: "[exact payoff]"

RE-ENGAGEMENT HOOKS:
- At [2:30]: "[line that creates curiosity spike]"
- At [5:00]: "[pattern interrupt — topic shift or challenge]"

CTA SECTION (3 variations):
- SOFT CTA:   [scripted, natural, low pressure]
- MEDIUM CTA: [scripted, clear ask with value framing]
- STRONG CTA: [scripted, urgency-based, direct]
```

---

### Egyptian Dialect & Voice Authenticity Layer

> Runs **only** when `scriptLanguage = Arabic (Egyptian)`, immediately after Script Refinement.

**Core instruction:** Write in **عامية مصرية** (Egyptian colloquial), never **فصحى** (Modern Standard Arabic), unless the content type explicitly calls for a formal register (e.g., documentary narrator).

**Register mapping:**
- `Cairene media-standard` — Egyptian grammar and vocabulary, calmer rhythm, fewer street idioms; this is what Egyptian film, TV, and most YouTube content uses.
- `Street & youth slang` — more idioms, faster rhythm, full youth-media register.
- `Educated professional` — Egyptian grammar throughout; register affects word choice, not the underlying dialect.

**MSA vs. Egyptian Contrastive Reference:**

| Avoid (MSA-leaning) | Target (Native Egyptian) | Context |
|---|---|---|
| كيف حالك؟ | إزيك؟ / عامل إيه؟ | Greeting |
| أريد أن أتحدث عن... | عايز أتكلم عن... | Intro |
| هذا الأمر مهم جدًا | الموضوع ده مهم جدًا | Emphasis |
| سأشرح لكم كيف... | هوريكم إزاي... | Explainer opener |
| في الحقيقة | بجد / بصراحة | Sincerity marker |
| والآن، دعونا ننتقل إلى | طب تعالوا بقى نشوف | Transition |
| يفعل | بيعمل | Present tense |
| سأذهب | هروح | Future |
| لكن | بس | Conjunction |
| الآن | دلوقتي | Adverb of time |

**Sentence rhythm:** Egyptian spoken Arabic uses shorter clauses with more discourse markers (`يعني`, `طب`, `خلاص`, `على فكرة`, `بجد`). A script that reads like an English script translated into correct Arabic is a **failure state**, even if the grammar is flawless.

**Code-switching per `codeSwitchingLevel`:** Egyptian creators naturally drop English nouns for tech, business, and global brand terms (`الـ engagement بتاع الفيديو`, `عملت subscribe`). Forcing purist Arabic equivalents for terms native speakers wouldn't translate is itself a tell. Applies to nouns only — never verb conjugation or core grammar.

**Hard gate in Self-Critique (Arabic mode only):**
- `dialect_authenticity_score` (1–10): would a native Cairene viewer assume a human wrote this?
- **A grade is blocked** if `dialect_authenticity_score < 8`. Loop break condition (Arabic mode): `grade contains "A" AND dialect_authenticity_score >= 8`

> **Calibration note:** Track every correction a native-speaker editor makes on the first 10–15 scripts through this pipeline and feed those corrections back into the contrastive table above. The table improves with each batch.

**Cross-cutting Arabic note:** RTL rendering, subtitle direction, and thumbnail text-overlay direction are affected in W-04/W-05 (subtitles) and W-07 (thumbnail brief) — those guides must account for this when written.

---

**Self-Critique Audit Criteria (all languages):**
- Hook grabs attention in 3 seconds; works with sound off
- Narrative arc: setup → build → climax → resolution
- Word count within ±10% of target duration
- Sounds natural when read aloud, not written
- Platform compliance: content rules and pacing norms
- Open loops: planted, referenced, resolved per schedule
- Factual claims traceable to the research package
- CTA: natural, at right timing, matches `ctaGoal`

**Output Package:**
```
## SCRIPT PACKAGE

### Script Metadata
Content Type, Target Duration, Word Count (~X min at Xwpm), Platform, Language
Narrative Structure, Hook Type
Dialect Authenticity Score: X/10 (Arabic only)
Authenticity Notes: [lines flagged — for native-speaker review]

### Production Script
[Full formatted script with visual notes, performance cues, timestamps]

### Retention Architecture
Open loops map (planted / referenced / resolved)
Re-engagement hook schedule
CTA versions (soft / medium / strong)

### B-Roll Shot List
[Auto-extracted from all [VISUAL NOTE] markers]

### Integration Data (for W-03 / W-04)
- script_duration_estimate: X minutes X seconds
- shot_list_extracted: yes
- visual_complexity: low / medium / high
- filming_requirements: [summary of what needs to be filmed]
```

---

## W-03: Production Planning & Filming Guide 🎥

Takes a script and plans **exactly how to film it** — shot by shot, equipment, locations, settings, lighting, performance direction, and production schedule. Includes an AI-Generated Footage Prompt Writer for shots the creator would rather generate than film.

**Can work standalone? Yes.**

### Architecture

```
[Start Node — 12 fields + creatorProfile]
          │
          ▼
[Shot List Extractor (LLM — Standard)]
  ← tags each shot: A-Roll / B-Roll / B-Roll (AI-gen candidate)
          │
          ▼
[AI-Generated Footage Prompt Writer (LLM — Standard)]
  ← runs ONLY for B-Roll (AI-gen candidate) shots
  ← produces text prompts ONLY — no API call, no generation
          │
          ▼
[Equipment Planner (LLM — Standard)]
[Location & Lighting Planner (LLM — Standard)]
[On-Camera Performance Director (LLM — Standard)]
[Production Schedule Builder (LLM — Standard)]
          │
╔══════════════════════════════════════════════════╗
║  LOOP (max 2, break: grade contains "A")         ║
║  Self-Critique (Best) → Critique Parser → Assigner║
╚══════════════════════════════════════════════════╝
          │
          ▼
[Final Production Bible (LLM — Cheap/fast)]
[End / Output]
```

### Start Node Fields

| Variable | Type | Label | Required |
|---|---|---|---|
| `scriptPackage` | paragraph | Script Package from W-02 or describe what to film | Yes |
| `contentType` | select | Talking head / VO + B-roll / interview / documentary / tutorial demo / cinematic / mixed | Yes |
| `budgetLevel` | select | Micro (phone only) / Low (basic DSLR) / Medium (mirrorless) / High (pro + crew) | Yes |
| `qualityTier` | select | Social-ready / Professional / Cinematic / Broadcast | Yes |
| `location` | select | Home studio / Office / Outdoor / Rented studio / Multiple / On location | Yes |
| `crewSize` | select | Solo / 1 assistant / Small crew 2–4 / Full crew 5+ | Yes |
| `availableEquipment` | paragraph | Camera, lens, audio, lighting gear available | No |
| `filmingDays` | select | Half day / Full day / Multi-day | Yes |
| `availableAiVideoTools` | text | Video generation tools available (Runway, Veo, Kling, Luma, Sora, Pika…) | No |
| `postProductionPlan` | select | W-04 / W-05 / W-06 — which editing workflow follows | No |
| `specialRequirements` | paragraph | Teleprompter, travel B-roll, product shots, interviews | No |
| `releaseDeadline` | text | Target release date | No |
| `creatorProfile` | paragraph | Paste your Creator Profile | No |

### Key Node Specifications

**Shot List Extractor — Output table:**

| Shot # | Scene | Shot Type | Camera Movement | Description | Source | Type Tag |
|---|---|---|---|---|---|---|
| 1 | Intro | ECU | Static | [description] | Script line 4 | A-Roll |
| 5 | B-roll | Wide | Drone | [description] | [VISUAL NOTE] | B-Roll (AI-gen candidate) |

**AI-Generated Footage Prompt Writer — per AI-gen-candidate shot:**
```
SHOT #[X] — AI-GENERATED FOOTAGE PROMPT

Subject & Action: [exactly what happens in frame]
Camera: [movement + angle + lens feel matching the shot plan]
Style & Lighting: [cinematic / mood / lighting — phrased for video-gen tools]
Duration: [matches planned shot length]
Negative Prompt: [no text artifacts, no warped limbs, generic boilerplate]
Suggested Tool: [from availableAiVideoTools, or general recommendation]
```

Output appended to Production Bible as table: `Shot # | Full Prompt | Suggested Tool | Duration`

**Equipment Planner — Three-tier output:**
```
MINIMUM VIABLE SETUP:
- Camera: [model or phone]   Lens: [spec]   Audio: [setup]
- Lighting: [setup]          Stabilization: [tripod / gimbal]
- Estimated cost: $X

STANDARD SETUP (recommended for target quality tier): ...

PREMIUM SETUP (if budget allows): ...

CAMERA SETTINGS PER SHOT TYPE:
| Shot Type             | ISO | Shutter | Aperture | FPS | Resolution |
|----------------------|-----|---------|----------|-----|------------|
| Talking head (indoor)| 800 | 1/50    | f2.0     | 24  | 4K         |
| B-roll (outdoor day) | 100 | 1/200   | f4.0     | 60  | 4K         |
| Slow-motion action   | 400 | 1/500   | f2.8     | 120 | 4K         |
```

**Location & Lighting Planner — per location:**
```
LOCATION: [Name]
Purpose: [which shots]   Setup time: [X min]   Best time of day: [X — reason]

LIGHTING SETUP:
- Key Light:  [position, type, color temp (K), distance]
- Fill Light: [position, intensity ratio vs. key — e.g. 2:1]
- Rim Light:  [position, purpose]
- Ambient:    [window blocking / supplementing strategy]
- Camera WB:  set to [X K] to match
```

**Production Schedule Builder — day-by-day:**
```
FILMING DAY 1 — [Date] — Location: [X]

07:30  Arrive, unload equipment
08:00  Setup & lighting check (30 min)
08:30  Camera test, audio levels (15 min)
08:45  SHOT 1–4: Main talking head section (60 min)
09:45  Setup change for close-ups (15 min)
10:00  SHOT 5–8: Close-up details (45 min)
10:45  Break — review footage (15 min)
11:00  SHOT 9–12: B-roll sequences (90 min)
12:30  Pack up, travel to Location 2

TOTAL SHOTS TODAY: X   ESTIMATED FOOTAGE: X GB
REVIEW NOTE: Check focus on talking head before leaving location
CONTINGENCY: If [weather / access issue], move shots [X–Y] to indoor fallback
```

**Self-Critique Audit Criteria:**
- Every shot from the script accounted for?
- Schedule realistic for available crew and time?
- Equipment matched to quality tier goal?
- Lighting plan specific (not just "natural light")?
- Locations with noise, access, or permit issues?
- Weather/contingency accounted for?

---

## W-04: Video Pre-Planning Pipeline (UPGRADED) 🎬

The existing pipeline is **architecturally correct** and becomes the reference all other workflows copy. All changes are **additive only** — no structural changes.

### New Optional Start Node Fields

| Variable | Type | Purpose |
|---|---|---|
| `scriptPackage` | paragraph | Brief Builder reads it; Storyboard Builder translates script sections to shots directly |
| `researchPackage` | paragraph | Narrative Structure and Retention Map use it for depth |
| `availableFootage` | paragraph | Storyboard Builder maps shots to actual available footage |
| `enableImageConcepts` | select Yes/No | Opts into Storyboard Visualizer image calls (default No — requires API access) |
| `imageGenProvider` | text | Image endpoint to call (swappable without rewriting the node) |

### New Node: Thumbnail Ideation
Runs after Retention Map, before Loop. Generates 3 thumbnail concepts:
- Emotion-driven / Curiosity-driven / Value-driven
- Per concept: hero image description, text overlay (max 5 words), color palette, click psychology, A/B test recommendation
- If `enableImageConcepts = Yes`: calls Storyboard Visualizer and returns 3 draft thumbnail images

### New Node: Storyboard Visualizer (tool/http-request, not LLM)
Calls an image-generation model with shot or B-roll concept descriptions. **Runs selectively** — not every shot: hook shot (first 3 seconds), 3 thumbnail concepts, any B-Roll Concept Studio row tagged `AI-generated image`. Text description remains the source of truth; image is a visual aid.

### New Node: B-Roll Concept Studio
Runs **after** the self-critique loop completes, before QA & Final Package.
- For every B-roll-needing section: 2–3 concepts beyond the literal illustration
- Per-project **B-Roll Style Guide**: which shot types the project leans on so all inserts read as one visual thread
- Per concept: timestamp range, visual concept, style note, and **Fulfillment Path** tag: `Film it` / `Stock footage` / `AI-generated image` / `AI-generated video`
- AI-generated video concepts: same prompt format as W-03's AI-Generated Footage Prompt Writer (text only, no API call)
- AI-generated image concepts: calls the Storyboard Visualizer
- Output feeds W-05's Asset Organization / Footage Sourcing Plan directly

### QA & Final Package Additions
Two new sections: "B-Roll Concepts & Style Guide" and (when `enableImageConcepts = Yes`) generated preview images alongside their source concepts.

### What Does NOT Change
The self-critique loop architecture, all existing node IDs, existing prompt texts, the 27-field intake form, the JSON storyboard export format, the QA checklist.

---

## W-05: Post-Production Execution Flow (UPGRADED) 🖥️

### Loop Architecture Rebuild (Standard 1)

**Delete:** `Grade_Check → Revision_Applier → Revision_Counter → Loop_Count_Guard → manual back-edge to Effects_and_Transition_Designer`

**Replace with a single Dify Loop container:**

Container wraps: `Effects_and_Transition_Designer` → `Motion_Graphics_Planner` → `Sound_Design_Architect` → `Audio_Mixing_and_Mastering` → `Color_Grading_and_Finishing` → `Self_Critique` → `Critique_Parser` → Assigner.

Loop variables: `grade`, `report`, `revised_effects`, `revised_motion_graphics`, `revised_sound`, `revised_mixing`, `revised_color`, `revision_count`. Break condition: `grade contains "A"`. Max iterations: 2.

### Other Changes

**Critique Parser fail-closed:** exception-path fallback changed from `"B"` to `"F"`.

**New optional Start Node fields:**
- `scriptPackage` (paragraph) — First Cuts Strategist uses it for dialogue sync
- `roughCutFeedback` (paragraph) — enables human-in-the-loop for rough cut review

**New Node: Rough Cut Review Gate** (between First Cuts Strategist and the Loop container):
- Generates a "Rough Cut Review Document" — structured checklist the editor follows before effects begin
- If `roughCutFeedback` is provided (not empty): reads it and feeds corrections into the first Effects pass
- If empty: generates AI auto-review and proceeds
- Soft gate: does not block execution; produces a clear review artifact

**First Cuts in Revision Loop:** Loop-back now reaches `First_Cuts_Strategist` if Self-Critique flags cut timing issues.

**Footage Sourcing Plan update:** Asset Organization table gets new column **AI Prompt (if applicable)** — when source is `AI Generated`, pull prompt from W-03/W-04 if upstream, or generate a basic prompt as fallback.

**Platform Export Presets Table** (added to Color Grading & Finishing node):

| Platform | Codec | Resolution | FPS | Bitrate | Color Space | Audio LUFS |
|---|---|---|---|---|---|---|
| YouTube | H.264 / H.265 | Up to 4K | Match source | 35–68 Mbps | Rec.709 | −14 |
| TikTok | H.264 | 1080×1920 | 30 | 15–25 Mbps | Rec.709 | −14 |
| Instagram Reels | H.264 | 1080×1920 | 30 | 15–20 Mbps | Rec.709 | −14 |
| LinkedIn | H.264 | 1920×1080 | 30 | 5–10 Mbps | Rec.709 | −16 |
| Broadcast | DNxHR/ProRes | 1920×1080 | Delivery spec | — | Rec.709/2020 | −24 |

**Revision Changelog Section** added to Final Execution Package: "What Changed in Revision Pass X" auto-generated per pass.

**Asset Readiness Alert** added to Asset Organization output:
```
⚠️ MISSING ASSETS — DO NOT BEGIN EDITING UNTIL RESOLVED
[Exact list of unconfirmed shots/assets]
```

**What does NOT change:** 4-layer sound design, LUFS targets, Python parsers, time budget calculator, 25-step execution order, all node names.

---

## W-06: Viral Speed Ramp Pipeline (UPGRADED) ⚡

### Loop Architecture Rebuild (Standard 1)

**Delete:** `Revision_Applier`, `Revision_Counter`, `Loop_Count_Guard`, and the manual back-edge to `Speed_Ramp_Designer`.

**Replace with a single Dify Loop container:**

Container wraps: `Speed_Ramp_Designer` → `Viral_Effects_and_Transitions` → `Sound_Design_and_Finishing` → `Self_Critique_Audit` → `Critique_Parser` → Assigner.

Loop variables: `grade`, `report`, `revised_speed_ramp`, `revised_effects`, `revised_sound`, `loop_score`, `revision_count`.

**Break condition — compound:**
```
{{#Loop.grade#}} contains "A"   AND   {{#Loop.loop_score#}} >= 7
```

Without the `loop_score` condition, a video can carry an A-grade and a 4/10 loop quality and still ship — precisely the failure this gate exists to catch.

### Other Changes

**Critique Parser fail-closed:** exception-path fallback changed from `"B"` to `"F"`. `loop_score`, `viral_score`, and `issues_summary` outputs remain as previously specified.

**New Start Node field:** `trendStyleDetailed` (paragraph) replaces single-line `trendStyle`:
```
- Current viral sound name + BPM (if known):
- Trending effect or transition style:
- Competitor reels to reference (links or descriptions):
- What makes this trend work (your read on it):
```

**New Node: Clip Intelligence** (between Start and Clip Arrangement) — per-clip scoring:

| Metric | Score | Notes |
|---|---|---|
| Speed Ramp Suitability | X/10 | Clear slow→fast moment? |
| Slow-Mo Quality | X/10 | Frame rate assessment |
| Peak Moment | [timestamp] | The single best frame in this clip |
| Visual Uniqueness | X/10 | How different from other clips? |
| Recommended Pattern | V-Ramp / Freeze→Ramp / Multi-Ramp / Skip | |

Ranks clips by viral potential. Flags unusable clips before any editing time is spent.

**Platform Format Matrix** (added to Sound & Finishing node):

| Platform | Aspect | Resolution | Safe Zone (bottom) | Caption Area | Export |
|---|---|---|---|---|---|
| TikTok | 9:16 | 1080×1920 | Bottom 250px (TikTok UI) | 250–700px | H.264, 15–25 Mbps |
| Instagram Reels | 9:16 | 1080×1920 | Bottom 200px | 200–650px | H.264, 15–20 Mbps |
| YouTube Shorts | 9:16 | 1080×1920 | Bottom 150px | 150–600px | H.264, 35+ Mbps |

**Expanded Viral Potential Scoring:**

| Criterion | Weight | Score (1–10) |
|---|---|---|
| Hook strength (first 0.5s demands attention) | 20% | |
| Beat sync precision (peaks land on the beat) | 20% | |
| Effect cohesion (one consistent style feel) | 15% | |
| Loop quality (end connects seamlessly to start) | 20% | |
| Sound impact (SFX hit hard enough) | 15% | |
| Visual variety (enough different clips/angles) | 10% | |
| **Weighted Total** | 100% | **X.X / 10** |
| **Verdict** | | Will go viral / Solid / Needs work / Not there yet |

**What does NOT change:** Speed ramp curve methodology, CC Particle World parameters, 3D Camera Tracker workflow, pre-compose strategy, clip arrangement Energy Wave pattern, loop-ability rules, all node names.

---

## W-07: SEO & Publishing Optimizer (BONUS) 📢

Takes a finished video + research + script and produces a complete publishing package.

**Can work standalone? Yes.**

### Architecture (Linear — no critique loop)

```
[Start Node — 8 fields + creatorProfile]
[Title Generator (LLM — Best)] — 10 variations, SEO score + CTR prediction, A/B pair
[Description & Chapter Writer (LLM — Standard)]
[Tags & Metadata Optimizer (LLM — Cheap/fast)]
[Thumbnail Brief Generator (LLM — Standard)]
[Rights & Compliance Check (LLM — Standard)]
  ← Music licensing risk
  ← Disclosure requirements (sponsors, affiliate links, paid promotion)
  ← Platform policy risk areas (medical claims, financial advice, etc.)
  ← Output: flagged-items list (most videos show none)
[Cross-Platform Publishing Plan (LLM — Cheap/fast)]
[Final Publishing Package (LLM — Cheap/fast)]
[End / Output]
```

---

## W-08: Content Repurposing Agent (BONUS) 🔄

Takes a finished video (or script + storyboard) and plans repurposing across formats and platforms.

**Can work standalone? Yes.**

### Architecture (Linear)

```
[Start Node — 5 fields + creatorProfile]
[Content Audit (LLM — Standard)] — 5 most repurposable moments
[Platform Adaptation Planner (LLM — Standard)]
  ← TikTok/Reels clip extraction
  ← Twitter/X thread (key insights)
  ← LinkedIn carousel (key points as slides)
  ← Blog post outline (script → article)
  ← Podcast clip / audio version plan
[Shorts/Reels Extraction Plan (LLM — Standard)]
  ← 3–5 short clips from long-form, with re-hooks, text overlay context
[Content Calendar Integration (LLM — Cheap/fast)]
  ← 2-week distribution schedule
  ← Pre-publish teaser + post-publish promotion plan
[Final Repurposing Package (LLM — Cheap/fast)]
[End / Output]
```

---

## W-09: Performance Analyzer 📊

Closes the loop the rest of the system leaves open. Every workflow upstream predicts something — retention %, viral potential, hook strength — and nothing checks those predictions against reality. This workflow takes real platform analytics and produces reusable lessons that feed back into W-00 (idea scoring) and W-02 (script calibration).

**Can work standalone? Yes — works for any published video, with or without prior pipeline output.**

### Architecture (Linear)

```
[Start Node — 6 fields]
[Analytics Parser (LLM — Standard)] — normalizes platform analytics export format
[Prediction vs. Reality Comparator (LLM — Best)]
  ← pulls original Retention Map / hook design / viral score if provided
  ← compares predicted drop-off points to actual retention curve
  ← flags: where prediction was right, where it missed, by how much
[Lessons Extractor (LLM — Best)]
  ← specific, reusable lessons tied to mechanism, not outcome
  ← "the pattern interrupt at 0:45 didn't land — the topic shift was too abrupt"
    not "retention was low"
[Final Performance Package (LLM — Cheap/fast)]
  ← full report + feed-forward summary for W-00's pastPerformanceNotes
    and W-02's brand-voice context
[End / Output]
```

### Start Node Fields

| Variable | Type | Label | Required |
|---|---|---|---|
| `videoTitle` | text | Which video is this? | Yes |
| `platform` | select | YouTube / TikTok / Instagram Reels / other | Yes |
| `analyticsData` | paragraph | Paste the platform's analytics export | Yes |
| `topComments` | paragraph | Sample of top/representative comments | No |
| `originalPackage` | paragraph | Original Retention Map / Script / Viral Package | No |
| `whatYouThinkWorked` | paragraph | Your own read on what worked or didn't | No |

### Output Package
```
## PERFORMANCE ANALYSIS PACKAGE

### Quick Reference Card
Video: [title] | Platform: [platform]
Actual retention at key markers vs. predicted
CTR: X% | Avg. view duration: X

### Prediction vs. Reality
[Table: predicted drop-off | actual behavior | hit/miss]

### Lessons Extracted
[Specific, mechanism-tied findings]

### Feed-Forward Summary
[Block formatted to paste into W-00's pastPerformanceNotes and W-02 context]
```

---

## 🔗 Integration Matrix

| From | Output Contains | Passes To | Key Fields |
|---|---|---|---|
| W-00 | Idea list, market intelligence, audience psychology | W-01 | `chosen_idea`, `key_research_questions`, `audience_psychology` |
| W-01 | Research report, evidence bank, user knowledge | W-02 | `researchPackage` |
| W-02 | Script, shot list, B-roll list, dialect score (Arabic) | W-03, W-04 | `scriptPackage`, `shot_list_extracted` |
| W-03 | Production bible, schedule, equipment, AI-footage prompts | W-04, W-05 | `preplanning_package` |
| W-04 | Storyboard, pacing, retention, thumbnails, B-roll concepts | W-05, W-06 | `preplanningPackage` |
| W-05 | Final execution package | W-07 | Video + metadata |
| W-06 | Final viral edit package | W-07 | Video + metadata |
| W-07 | Publishing package + compliance flags | — | Titles, descriptions, thumbnails, schedule |
| W-08 | Repurposing plan | — | Platform-adapted content calendar |
| *(after publishing)* | Real analytics | W-09 | `analyticsData` |
| W-09 | Lessons + feed-forward summary | W-00, W-02 | `pastPerformanceNotes` |
| *(user-maintained)* | Creator Profile | Every workflow | `creatorProfile` |

---

## 📋 Files To Create / Modify

### Phase 1 — Core Pipeline (build in this order)

| File | Lines Est. | Status |
|---|---|---|
| `SYSTEM_OVERVIEW.md` | ~400 | NEW — workflow map, entry-point selector, Standards reference, model-tier table |
| `W-00_Trend_and_Idea_Research.md` | ~1,400 | NEW |
| `W-01_Topic_Research_Info_Collector.md` | ~1,200 | NEW |
| `W-02_Script_Writer.md` | ~1,900 | NEW — Egyptian dialect layer adds significant length |
| `W-03_Production_Planning_Filming_Guide.md` | ~1,400 | NEW — AI-footage prompt writer adds a node |
| `W-04_Video_Pre-Planning_Pipeline.md` | ~2,100 | UPGRADE of `00 - Video Pre-Planning Pipeline.md` |
| `W-05_Post_Production_Execution_Flow.md` | ~2,400 | **REBUILD** of `01_post_production_execution_flow.md` |
| `W-06_Viral_Speed_Ramp_Pipeline.md` | ~2,000 | **REBUILD** of `02_speed_ramp_viral_flow.md` |

> W-05 and W-06 are **rebuilds** (loop architecture change), not additive upgrades. Budget accordingly.

### Phase 2 — Bonus Agents (after core pipeline validates)

| File | Lines Est. | Status |
|---|---|---|
| `W-07_SEO_Publishing_Optimizer.md` | ~900 | NEW (bonus) |
| `W-08_Content_Repurposing_Agent.md` | ~700 | NEW (bonus) |
| `W-09_Performance_Analyzer.md` | ~600 | NEW |

---

## ✅ All Decisions — Resolved

| Question | Decision |
|---|---|
| Keep existing filenames or rename? | **Renamed.** All guides use the W-number convention. |
| Build all 10 now or core first? | **Core 7 first** (W-00 through W-04, W-05, W-06), bonus agents (W-07, W-08, W-09) after the core pipeline validates end-to-end. |
| Same depth as existing guides for new ones? | **Yes for W-00 through W-03** — reference tables and rulesets stop these from producing generic output. **Lighter for W-07/W-08/W-09** — a mediocre publishing description is a much lower-stakes failure. |
| Is web search available for W-01? | **Required, not optional.** W-01 does not ship without a working search tool. |
| Does W-02 need Arabic support? | **Yes — full Egyptian dialect authenticity layer** with dedicated node, intake fields, contrastive reference table, and hard gate. |
| Loop architecture: native Loop or hand-rolled IF/ELSE? | **Native Loop container, system-wide.** W-05 and W-06 rebuilt to match W-04. |
| Critique Parser fail-safe default? | **Grade `F`**, system-wide — replaces W-04's fail-open `A` and W-05/W-06's fail-closed `B`. |
| AI image/video generation: built in or prompt-only? | **Prompt-only for video.** Optional image preview for thumbnails and storyboard via Storyboard Visualizer (opt-in, requires API access) in W-04. |
| W-09 Performance Analyzer: build it? | **Yes.** The feedback layer closes the prediction loop; without it the system never learns from results. |
