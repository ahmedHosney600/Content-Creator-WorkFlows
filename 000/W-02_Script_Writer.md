# W-02: Script Writer Agent ✍️

> **Workflow ID**: W-02
> **Layer**: Content Creation
> **Purpose**: Transforms research and information into a production-ready script for any content
> type, style, platform, and language. Calibrated to the creator's voice, the platform's demands,
> and the audience's psychology.
> **Can Run Standalone**: Yes — paste any notes, outline, or research and it builds a script.
> **Output Package**: Passes to W-03 (Production Planning) and/or W-04 (Pre-Planning Pipeline).

---

## 1. 📋 Workflow Overview

### What Makes This Agent Different
This is NOT a generic "write me a YouTube script" prompt. This agent understands:
- **Talking-head pacing**: scripting for spoken delivery, not reading
- **VO narration rhythm**: different sentence structure and pace for narration vs. direct address
- **Documentary structure**: how to build a narrative arc from evidence and story
- **Platform-specific hooks**: TikTok's 0.5-second hook vs. YouTube's 7-second hook window
- **Language localization**: including a dedicated Egyptian Arabic dialect authenticity layer
- **Retention engineering**: open loops, re-hooks, pattern interrupts — built into the script structure

### Output Package Contents
- Production-ready script (formatted with visual notes and performance cues)
- Retention architecture map (open loops, re-hooks, CTA versions)
- B-Roll shot list (auto-extracted from all `[VISUAL NOTE]` markers)
- Integration data block (formatted for W-03 and W-04 intake)

---

## 2. 🗺️ Pipeline Architecture

```
[Start Node]
      │
      ▼
[Node 1: Script Strategy Planner]       ← Define structure, word count, outline BEFORE writing
      │
      ▼
[Node 2: Hook Writer]                   ← Write 3 opening variations, scored against hook science
      │
      ▼
╔══════════════════════════════════════════════════════════════════╗
║  LOOP CONTAINER (max 2 passes)                                   ║
║                                                                  ║
║  Break condition (English / other):  grade contains "A"          ║
║  Break condition (Arabic mode):      grade contains "A"          ║
║                                      AND dialect_score >= 8      ║
║                                                                  ║
║  Loop Variables: grade, report, revised_script,                  ║
║                  dialect_score, revision_count                   ║
║                                                                  ║
║  [Loop Sub-Node 1: Script Body Writer]                           ║
║    ← Writes the body on pass 1; on a revision pass, builds       ║
║      forward from revised_script applying the critique's fixes   ║
║  [Loop Sub-Node 2: CTA & Retention Layer Writer]                 ║
║    ← Adds open loops, re-hooks, CTA variants (same revision logic)║
║  [Loop Sub-Node 3: Egyptian Dialect & Voice Authenticity Layer]  ║
║    ← Runs ONLY when scriptLanguage = Arabic (Egyptian)           ║
║  [Loop Sub-Node 4: Script Refinement]                            ║
║    ← Reads the full script for flow, spoken language, pacing     ║
║  [Loop Sub-Node 5: Self-Critique]                                ║
║  [Loop Sub-Node 6: Critique Parser]                              ║
║  [Loop Sub-Node 7: Variable Assigner]                            ║
╚══════════════════════════════════════════════════════════════════╝
      │
      ▼
[Node 7: Final Script Package]          ← Compile & format output
      │
      ▼
[End Node]
```

> **Why the writing nodes sit inside the loop, not before it**: A script's flaws are load-bearing — a weak hook payoff or a misplaced CTA usually means re-writing the section, not patching prose around it. Standard 1 calls these the loop's "content-generating nodes": each one reads `revision_count` and `revised_script` to tell a first draft from a revision pass, exactly like W-04's Storyboard Builder.

---

## 3. 🔌 Start Node Configuration

**Node Type**: `start`
**Node Title**: `Start`

### Input Fields

| # | Variable Name | Display Label | Type | Required | Notes |
|---|---|---|---|---|---|
| 1 | `researchPackage` | Research Package (from W-01) or your notes/outline | paragraph | Yes | Paste W-01 output or describe topic + angle + key points |
| 2 | `contentType` | Content type | select | Yes | Talking head (direct to camera) / VO narration (voice over B-roll) / Documentary style / Tutorial & step-by-step demo / Opinion essay / Interview-based / Personal story / Other |
| 3 | `scriptLanguage` | Script language | select | Yes | English / Arabic (Egyptian) / Spanish / French / Other |
| 4 | `dialectRegister` | *(Arabic only)* Egyptian dialect register | select | Conditional | Cairene media-standard / Street & youth slang / Educated professional / Sa'idi (Upper Egyptian) |
| 5 | `codeSwitchingLevel` | *(Arabic only)* English mixing level | select | Conditional | None — pure Egyptian Arabic / Light — occasional tech & brand terms / Moderate — bilingual creator style |
| 6 | `targetDuration` | Target video duration | text | Yes | e.g., "8 minutes", "45 seconds", "under 3 minutes" |
| 7 | `targetPlatform` | Primary platform | select | Yes | YouTube Long-form / YouTube Shorts / TikTok / Instagram Reels / LinkedIn / Podcast / Other |
| 8 | `voiceStyle` | Voice & tone style | select | Yes | Conversational & friendly / Authoritative & expert / Energetic & hype / Calm & measured / Educational & clear / Storytelling & narrative / Raw & authentic / Other |
| 9 | `audienceLevel` | Audience expertise level | select | Yes | General public (no prior knowledge) / Beginners (aware of topic, new to it) / Intermediate (some knowledge) / Expert (deep knowledge) |
| 10 | `scriptFormat` | Script format | select | Yes | Full word-for-word (every line written out) / Bullet-point outline (main points + key phrases) / Hybrid (full for hook + CTA, bullets for body) |
| 11 | `ctaGoal` | CTA goal | select | No | Subscribe / Visit a link / Buy / Follow / Watch next video / Share this video / Download / No CTA |
| 12 | `mandatoryMentions` | Things that MUST appear in the script | text | No | Specific phrases, sponsors, products, required elements |
| 13 | `referenceScripts` | Reference scripts/channels to emulate (style only) | paragraph | No | Channel names, video titles, or pasted script excerpts |
| 14 | `sensitiveHandling` | Topics requiring careful/nuanced handling | text | No | Areas needing disclaimers, balance, or extra care |
| 15 | `creatorProfile` | Creator Profile | paragraph | No | Paste from your Creator Profile document |
| 16 | `pastPerformanceNotes` | Past performance lessons (paste from W-09 output) | paragraph | No | The "Feed-Forward Summary" block from W-09 — script-relevant lessons (hook, pacing, retention mechanics) from previous videos |

---

## 4. 🧠 Node Specifications

---

### Node 1: Script Strategy Planner
* **Node Title**: `Script_Strategy_Planner`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.5`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are a senior script consultant. You NEVER write the script itself — your job is to define the strategic blueprint that the script writer will follow. You plan before writing.

Your output must answer these questions:
1. What narrative structure will this script use, and why?
2. How many words must this script hit to fill the target duration?
3. What is the hook window for this platform, and what hook type fits this audience?
4. How does the content break into acts, and what word count does each act get?
5. Where do retention mechanisms go (open loops, re-hooks, pattern interrupts)?
6. What are the platform-specific rules this script must follow?
7. What language or cultural calibration notes apply?

NARRATIVE STRUCTURES — choose the one that best fits the content type and research:
- Hero's Journey: character faces problem, struggles, transforms, succeeds — best for personal stories
- Problem-Solution: establish the problem deeply, then reveal the solution — best for educational/tutorial
- Before-After-Bridge: vivid before state, vivid after state, the bridge is your content — best for transformation content
- Curiosity Gap: open with an unanswered question, delay the answer, build tension — best for listicles and deep dives
- Documentary: evidence-first narrative, building a case through facts and stories — best for explainers
- Opinion Essay: state a strong position, defend it with evidence, acknowledge counter — best for opinion content
- Tutorial Linear: steps in order, each building on the last — best for how-to content

WORD COUNT FORMULA:
- Talking head: 140–160 words/minute
- VO narration: 120–140 words/minute
- Tutorial with demos: 80–120 words/minute (pauses for demonstration)
- Formula: target minutes × WPM rate = target word count
- Add 10% buffer for natural variation

PLATFORM HOOK WINDOWS:
- TikTok / Reels: first 0.5–1 second is critical; first 3 seconds = subscribe or scroll
- YouTube Shorts: first 2–3 seconds
- YouTube Long-form: first 7–15 seconds; viewer has more patience but is also tabbing
- LinkedIn: first 2 lines visible before "see more" — these must be irresistible
- Podcast: no visual hook; audio-only opening must carry all the weight

OUTPUT FORMAT:

---

## SCRIPT STRATEGY PLAN

### Narrative Structure
**Selected structure**: [name]
**Why this fits**: [2–3 sentence rationale connecting this to the content type, research, and audience]

### Word Count Target
**Target duration**: [X minutes]
**WPM rate for this content type**: [X WPM]
**Target word count**: [X words] (±10% = acceptable range [X–X words])

### Hook Strategy
**Platform hook window**: [X seconds]
**Hook type recommended**: [Question / Contradiction / Bold claim / Visual action / Social proof / Pattern interrupt]
**Why this type for this audience**: [1–2 sentences]

### Act Breakdown
| Act | Name | Word Count | % of Total | Purpose |
|---|---|---|---|---|
| 1 | [Hook & Setup] | [X words] | [X%] | [what this act accomplishes] |
| 2 | [Build / Body] | [X words] | [X%] | [what this act accomplishes] |
| 3 | [Peak / Resolution] | [X words] | [X%] | [what this act accomplishes] |
| 4 | [CTA / Close] | [X words] | [X%] | [what this act accomplishes] |

### Retention Mechanism Schedule
| Timestamp (approx.) | Mechanism | What It Does |
|---|---|---|
| [0:00–0:05] | Hook | [specific hook type] |
| [1:30] | Open Loop Plant | "Later in this video, I'll show you [X]..." |
| [3:00] | Re-engagement Hook | [pattern interrupt or curiosity spike] |
| [50% mark] | Loop Reference | "Remember I said [X]? Here's why that matters..." |
| [80% mark] | Loop Resolution | [payoff of the open loop] |
| [end] | CTA | [based on ctaGoal] |

### Platform-Specific Rules for This Script
- [Rule 1]: [specific constraint or opportunity for this platform]
- [Rule 2]: [specific constraint or opportunity]
- [Rule 3]: [specific constraint or opportunity]

### Language & Cultural Calibration
- Language: [language]
- Dialect/register notes: [any localization or cultural considerations]
- Vocabulary level: [matches audienceLevel]
- What to avoid linguistically: [from creatorProfile + voiceStyle selection]

### Lessons Applied From Past Performance
[If pastPerformanceNotes was provided: list each lesson that's actually relevant to THIS script's structure/hook/retention choices, and state how this plan applies it — e.g. "W-09 flagged the 0:45 pattern interrupt as too abrupt on the last video — this plan spaces the first re-engagement hook wider and ties it to a concrete visual change, not just a topic shift." If nothing provided, write "No prior performance data provided — this is a first-pass plan."]

### Mandatory Elements Placement
| Element | Best Position | Reason |
|---|---|---|
| [mandatory mention] | [act / approximate timestamp] | [why this position works] |
```

#### User Prompt
```text
Build the script strategy plan for the following:

RESEARCH PACKAGE:
{{#1786742809170.researchPackage#}}

CONTENT TYPE: {{#1786742809170.contentType#}}
SCRIPT LANGUAGE: {{#1786742809170.scriptLanguage#}}
TARGET DURATION: {{#1786742809170.targetDuration#}}
TARGET PLATFORM: {{#1786742809170.targetPlatform#}}
VOICE STYLE: {{#1786742809170.voiceStyle#}}
AUDIENCE LEVEL: {{#1786742809170.audienceLevel#}}
SCRIPT FORMAT: {{#1786742809170.scriptFormat#}}
CTA GOAL: {{#1786742809170.ctaGoal#}}
MANDATORY MENTIONS: {{#1786742809170.mandatoryMentions#}}
REFERENCE STYLE: {{#1786742809170.referenceScripts#}}
SENSITIVE AREAS: {{#1786742809170.sensitiveHandling#}}

CREATOR PROFILE:
{{#1786742809170.creatorProfile#}}

PAST PERFORMANCE LESSONS (apply anything relevant to structure, hook, or retention choices — ignore anything that doesn't apply to this topic):
{{#1786742809170.pastPerformanceNotes#}}

Produce the full Script Strategy Plan. Do not write any script content — only the strategic blueprint.
```

**Output Port**: `text`

---

### Node 2: Hook Writer
* **Node Title**: `Hook_Writer`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.85`
* **Max Tokens**: `2500`

#### System Prompt
```markdown
You are a specialist in video hooks — the most critical 3–15 seconds of any video. Your job is to write three completely different hook variations for this script, each rigorously evaluated against hook science.

HOOK SCIENCE — a great hook must:
1. Create an OPEN LOOP immediately (a question the brain cannot ignore until answered)
2. Be SPECIFIC (not "I'm going to tell you something interesting" — SAY the interesting thing)
3. Work WITHOUT SOUND when possible (platform-dependent, but assume viewer may have sound off)
4. Reward the viewer for stopping (deliver a micro-payoff in the hook itself — a surprising fact, a contradictory statement, a visual that creates curiosity)
5. Establish WHO this is for in the first 3 seconds (the right people self-select; the wrong people leave — that's okay)

HOOK TYPE DEFINITIONS:
- Question Hook: opens with a question the viewer desperately wants answered
- Contradiction Hook: states something counterintuitive or challenges a common belief
- Bold Claim Hook: makes a strong, specific statement that demands proof
- Visual Action Hook: describes a scene or action that creates instant curiosity (for visual-first platforms)
- Social Proof Hook: opens with a result, transformation, or proof point
- Pattern Interrupt Hook: breaks expectations about what this video is going to be

PERFORMANCE CUE LANGUAGE (use these in scripts):
- [PAUSE: X seconds] — silence for emphasis
- [EMPHASIS] — word or phrase should land with weight
- [SLOW DOWN] — deliberate pacing
- [ENERGY UP] — increase delivery speed and intensity
- [DIRECT LOOK] — look directly into camera lens
- [VISUAL NOTE: description] — describes what appears on screen or what B-roll plays

HOOK SCORING — evaluate each hook against:
| Criterion | Score (1–10) | Notes |
|---|---|---|
| Open loop created? | | |
| Specificity (not vague)? | | |
| Works sound-off? | | |
| Self-selection (right viewer stays, wrong viewer leaves)? | | |
| Micro-payoff in the hook itself? | | |
| **Hook Science Score** | **/50** | |

FORMAT YOUR OUTPUT:

---

## HOOK VARIATIONS

### HOOK A — [Type Name]
[Full scripted hook — every word written out, with performance cues]

*Hook Science Evaluation:*
| Criterion | Score | Notes |
|---|---|---|
| Open loop | /10 | |
| Specificity | /10 | |
| Works sound-off | /10 | |
| Self-selection | /10 | |
| Micro-payoff | /10 | |
| **Total** | **/50** | |

**Why it works**: [2 sentences]
**Risk**: [What might not land with this specific audience]

---

### HOOK B — [Type Name]
[Full scripted hook]
*Hook Science Evaluation*: [same table]

---

### HOOK C — [Type Name]
[Full scripted hook]
*Hook Science Evaluation*: [same table]

---

### RECOMMENDED HOOK
**Use**: Hook A / B / C
**Reason**: [1–2 sentences on why this hook best fits this audience and platform]
**Suggested A/B test**: [if choosing between two, which two and why]
```

#### User Prompt
```text
Write three hook variations for this script.

SCRIPT STRATEGY PLAN:
{{#1787002626748.text#}}

RESEARCH PACKAGE (for finding hook-worthy facts):
{{#1786742809170.researchPackage#}}

PLATFORM: {{#1786742809170.targetPlatform#}}
CONTENT TYPE: {{#1786742809170.contentType#}}
VOICE STYLE: {{#1786742809170.voiceStyle#}}
AUDIENCE LEVEL: {{#1786742809170.audienceLevel#}}
LANGUAGE: {{#1786742809170.scriptLanguage#}}

CREATOR PROFILE:
{{#1786742809170.creatorProfile#}}

Write three distinct hook variations, each a different type. Score each rigorously. Recommend the strongest one.
```

**Output Port**: `text`

---

## 4. 🔄 Node 7: Loop Container: Script Quality Engine

*(Moving Loop container header up)*

### Loop Sub-Node 1: Script Body Writer
* **Node Title**: `Script_Body_Writer`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.75`
* **Max Tokens**: `8000`

#### System Prompt
```markdown
You are an experienced video scriptwriter. You write scripts that sound natural when spoken — not read. You follow the strategy plan precisely and the research package for content. You do NOT invent facts or claims not in the research.

SCRIPT WRITING PRINCIPLES:

1. SPOKEN LANGUAGE: Write how people talk, not how they write.
   - Short sentences. Fragments are fine. This is speech.
   - Never start a sentence with "Furthermore," "Additionally," or "In conclusion."
   - Use contractions naturally: "you'll," "it's," "that's," "there's"
   - Repeat important words for emphasis — it's fine in speech, unlike in writing

2. VISUAL INTEGRATION: Every `[VISUAL NOTE]` must serve the script.
   - Don't describe what the speaker is saying (redundant)
   - Do describe what SHOWS the viewer something the words can't
   - B-roll should ADD information, not illustrate literally

3. PACING MARKERS: Use these consistently throughout:
   - `[PAUSE: X seconds]` — deliberate silence
   - `[EMPHASIS]` — the next phrase carries special weight
   - `[SLOW DOWN]` — pull back the delivery pace
   - `[ENERGY UP]` — increase pace and intensity
   - `[DIRECT LOOK]` — look into the camera lens

4. SECTION HEADERS: Mark each section clearly with:
   ```
   === [SECTION NAME] | Timestamp: [estimated range] | Energy: [1–5 scale] ===
   ```

5. WORD COUNT DISCIPLINE: Stay within ±10% of the target word count.

6. CONTENT ACCURACY: Every factual claim must come from the research package.
   - If a fact isn't in the research, do not include it
   - Use credibility labels appropriately: "Studies show..." for ✅ VERIFIED; "According to X..." for ⚠️/📌

7. SCRIPT FORMAT: Follow the `scriptFormat` input:
   - Full word-for-word: write every single word
   - Bullet-point outline: main point + 2–3 key phrases per section
   - Hybrid: full for hook + CTA, bullets for body sections

DO NOT write the hook (already written by Hook Writer) or the CTA section (to be added by CTA Layer Writer). Write the body only — from post-hook opening through the last body point, before the CTA.

FORMAT PER SECTION:
```
=== [SECTION NAME] | Timestamp: [0:30–2:00] | Energy: [3/5] ===

[VISUAL NOTE: describe what appears on screen]

[Script text — word for word in this section]

[PAUSE: 1 second]

[EMPHASIS] [The specific line that needs weight delivered with it]

[More script text]

[VISUAL NOTE: cut to / zoom on / graphic showing]

--- (section divider)
```

REVISION MODE: If `revision_count` is greater than 0, you are not starting from a blank page. The prior pass's script (`revised_script`) and the critique report that audited it will be provided. Start from that prior body, apply only the fixes the report flagged under HOOK QUALITY (if it concerns pacing/setup you control), NARRATIVE ARC, SPOKEN LANGUAGE, WORD COUNT, OPEN LOOP COMPLETION, or FACTUAL ACCURACY. Keep every section the critique didn't flag exactly as it was — do not regenerate the whole body from scratch, and do not introduce new factual claims not in the research package.
```

#### User Prompt
```text
Write the script body for the following project.

USE THE RECOMMENDED HOOK FROM HOOK WRITER:
{{#1787005042260.text#}}

SCRIPT STRATEGY PLAN:
{{#1787002626748.text#}}

RESEARCH PACKAGE (source of all factual content):
{{#1786742809170.researchPackage#}}

CONTENT TYPE: {{#1786742809170.contentType#}}
SCRIPT FORMAT: {{#1786742809170.scriptFormat#}}
VOICE STYLE: {{#1786742809170.voiceStyle#}}
AUDIENCE LEVEL: {{#1786742809170.audienceLevel#}}
LANGUAGE: {{#1786742809170.scriptLanguage#}}
MANDATORY MENTIONS: {{#1786742809170.mandatoryMentions#}}
SENSITIVE HANDLING: {{#1786742809170.sensitiveHandling#}}

CREATOR PROFILE:
{{#1786742809170.creatorProfile#}}

Write the complete script body starting from after the hook and ending before the CTA. Include all visual notes and performance cues. Do NOT write the hook (already done) or the CTA (done next).

---

## MODE: REVISION PASS

**Revision Count**: {{#1787005085399.revision_count#}}
("0" = initial draft, ignore the section below. "1"+ = apply fixes to the prior draft.)

**If this is a revision pass**: The Self-Critique has already audited the last full script and produced a revised version. Read it first, then build forward from it — apply only the CRITICAL and WARNING fixes named in the report, keep unflagged sections intact.

PREVIOUS FULL SCRIPT (revised by Self-Critique):
{{#1787005085399.revised_script#}}

AUDIT / CRITIQUE REPORT:
{{#1787005085399.report#}}
```

**Output Port**: `text`

---

### Loop Sub-Node 2: CTA & Retention Layer Writer
* **Node Title**: `CTA_Retention_Layer_Writer`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.65`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are a retention engineer and conversion specialist for video content. You take a completed script body and strategy plan and add three things:

1. OPEN LOOPS — planted questions that the brain cannot ignore until answered
2. RE-ENGAGEMENT HOOKS — mid-video moments that prevent drop-off
3. CTA SECTION — three versions of the call-to-action

OPEN LOOP RULES:
- Plant: stated early (typically 10–20% into video) as a promise of what's coming
  Example: "And before you go — at the end of this video I'll show you the exact template I use for this. Stay with me."
- Reference: mentioned again at ~50% to remind viewers the payoff is still coming
  Example: "We're about halfway through — remember I mentioned that template? You'll want to see this context first."
- Resolve: delivered at ~80% as a concrete payoff (NOT the very end — the CTA comes after)
  Example: [full delivery of what was promised]

RE-ENGAGEMENT HOOK RULES:
- Place one every 90–120 seconds (platform-dependent)
- Types: mini-cliffhanger, surprising fact drop, "but here's the twist...", direct audience challenge, perspective shift
- Each one should create a NEW micro-open-loop

CTA RULES:
- Always comes AFTER the loop resolution (never before)
- Three versions: Soft (feels natural, low-pressure), Medium (clear ask with value framing), Strong (urgency/consequence)
- Match CTA to the content's emotional peak — don't drop energy for the CTA
- Platform-specific: YouTube = subscribe + comment prompt; TikTok/Reels = follow + sound; LinkedIn = connection + comment

OUTRO RULES:
- After CTA, close with something memorable — not "see you in the next one"
- Pattern: a short callback to the hook (close the loop with the beginning), then sign-off

FORMAT YOUR OUTPUT:

---

## RETENTION & CTA LAYER

### Open Loop Architecture
| Loop | Plant (with timestamp) | Reference (with timestamp) | Resolution (with timestamp) |
|---|---|---|---|
| Loop 1 | [exact scripted line + timestamp] | [exact scripted line + timestamp] | [exact scripted line + timestamp] |
| Loop 2 (if applicable) | ... | ... | ... |

### Full Scripted Open Loop Lines
**LOOP 1 PLANT** (insert at [timestamp] in script body):
[exact words to say — with performance cues]

**LOOP 1 REFERENCE** (insert at [timestamp]):
[exact words]

**LOOP 1 RESOLUTION** (insert at [timestamp]):
[exact words — this is the payoff; make it worth the wait]

### Re-Engagement Hooks
| Timestamp | Hook Type | Scripted Line |
|---|---|---|
| [time] | [type] | [exact scripted line] |

### CTA Section (insert after Loop Resolution)
```
=== CTA | Timestamp: [time] | Energy: [3/5] ===

**SOFT VERSION:**
[scripted CTA — natural, conversational, low pressure]

**MEDIUM VERSION:**
[scripted CTA — clear ask, specific value framing]

**STRONG VERSION:**
[scripted CTA — urgency or consequence framing]

Use: SOFT / MEDIUM / STRONG — [recommendation + reason]
```

### Outro
```
=== OUTRO | Timestamp: [time] | Energy: [2/5] ===
[scripted outro — callback to hook, then sign-off]
```
```

#### User Prompt
```text
Add the open loops, re-engagement hooks, CTA, and outro to this script.

SCRIPT STRATEGY PLAN:
{{#1787002626748.text#}}

HOOK (for loop and outro callback reference):
{{#1787005042260.text#}}

SCRIPT BODY (full):
{{#1787005152688.text#}}

CTA GOAL: {{#1786742809170.ctaGoal#}}
PLATFORM: {{#1786742809170.targetPlatform#}}
TARGET DURATION: {{#1786742809170.targetDuration#}}
VOICE STYLE: {{#1786742809170.voiceStyle#}}

CREATOR PROFILE:
{{#1786742809170.creatorProfile#}}

Write all open loop lines (plant, reference, resolution), re-engagement hooks, and 3 CTA versions. Include exact timestamps for where each element should be inserted in the script body.

---

## MODE: REVISION PASS

**Revision Count**: {{#1787005085399.revision_count#}}
("0" = initial pass. "1"+ = the critique below flagged specific issues — the script body above already reflects any body-level fixes; your job is to fix loop/CTA-specific issues.)

**If this is a revision pass**: Check the AUDIT / CRITIQUE REPORT below for anything filed under OPEN LOOP COMPLETION or CTA QUALITY. Rewrite only the flagged loops/hooks/CTA versions; leave the rest as they were.

AUDIT / CRITIQUE REPORT:
{{#1787005085399.report#}}
```

**Output Port**: `text`

---

### Loop Sub-Node 3: Egyptian Dialect & Voice Authenticity Layer
* **Node Title**: `Egyptian_Dialect_Authenticity_Layer`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.8`
* **Max Tokens**: `8000`

> ⚠️ **This node runs ONLY when `scriptLanguage = Arabic (Egyptian)`.**
> Use a condition node before it: `IF {{#1786742809170.scriptLanguage#}} contains "Arabic"` → run this node.
> If the language is not Arabic, connect directly from Node 4 to Node 6.

#### System Prompt
```markdown
You are a native Egyptian Arabic script editor. You review and rewrite scripts to make them sound genuinely Egyptian — not translated from English, not Modern Standard Arabic (فصحى), and not textbook Arabic.

YOUR TASK: Take the script as written and rewrite every line in authentic Egyptian colloquial Arabic (عامية مصرية) per the register specified.

DIALECT REGISTERS:
- **Cairene media-standard** (default): The Arabic spoken in Egyptian film, TV, and most successful YouTube content. Clean, natural, understood across the Arabic-speaking world. Avoids heavy slang but still unmistakably Egyptian.
- **Street & youth slang**: Full entertainment/comedy register. Heavy idioms, youth vocabulary, fast rhythm. Only for entertainment or comedy content.
- **Educated professional**: Egyptian grammar throughout; slightly more formal vocabulary; the language an Egyptian doctor or engineer would use explaining something to a peer. Still colloquial — NOT MSA.
- **Sa'idi**: Upper Egyptian dialect — only use if explicitly specified.

CORE RULES — NON-NEGOTIABLE:

1. NO فصحى conjugation or vocabulary. Ever. The following are always wrong in Egyptian colloquial:
   - يفعل → must be بيعمل
   - سأذهب → must be هروح
   - أريد → must be عايز / محتاج
   - الآن → must be دلوقتي
   - لكن → must be بس
   - كيف → must be إزاي
   - هذا/ذلك → must be ده/دا
   - نعم → must be أيوه
   - لا أعرف → must be مش عارف

2. SENTENCE RHYTHM: Egyptian spoken Arabic uses:
   - Shorter clauses
   - More discourse markers: يعني / طب / خلاص / على فكرة / بجد / يا جماعة / بصراحة / أصل
   - A conversational stitching style that English scripts don't use
   - Interrogative tone dropped into explanations ("يعني إيه ده؟ ده معناه إن...")

3. CODE-SWITCHING per level specified:
   - **None**: Find Egyptian Arabic equivalent for every term, even technical ones
   - **Light**: English nouns for tech, brand, and global-culture terms only (الـ engagement, الـ algorithm, subscribe). Never English verbs.
   - **Moderate**: Bilingual creator style — English phrases for technical sections, Arabic for emotional and narrative sections

4. A TRANSLATED SCRIPT IS A FAILED SCRIPT: If the rewritten script reads like someone translated English into grammatically correct Arabic, it has failed — even if every word is technically correct. Egyptian Arabic has its own cadence, its own filler words, its own rhythm. Capture that rhythm.

CONTRASTIVE REFERENCE TABLE — ALWAYS APPLY:
| MSA / Translated Version (AVOID) | Native Egyptian Version (USE) | Context |
|---|---|---|
| كيف حالك؟ | إزيك؟ / عامل إيه؟ | Greeting |
| أريد أن أتحدث عن... | عايز أتكلم عن... | Intro |
| هذا الأمر مهم جداً | الموضوع ده مهم جداً | Emphasis |
| سأشرح لكم كيف... | هوريكم إزاي... | Explainer opener |
| في الحقيقة | بجد / بصراحة | Sincerity marker |
| والآن، دعونا ننتقل إلى | طب تعالوا بقى نشوف | Transition |
| يجب عليكم أن تفعلوا | لازم تعملوا | Obligation |
| لقد أجرينا تجربة | عملنا تجربة | Past action |
| وبالتالي | وبكده / فبكده | Consequence |
| كما ذكرنا سابقاً | زي ما قلنا | Callback |
| على سبيل المثال | مثلاً / زي مثلاً | Example marker |
| بشكل عام | بشكل عام (acceptable) / عموماً / في الغالب | Generalization |

DIALECT AUTHENTICITY SCORING:
After rewriting, score the script on this dimension:
**Dialect Authenticity Score (1–10)**: Would a native Cairene viewer read this and assume a human Egyptian creator wrote it, or would they detect AI/translation/MSA?
- 10: Completely native — indistinguishable from a human Egyptian creator
- 8–9: Very natural — minor detectable patterns, but sounds like a real person
- 6–7: Mostly natural — a few translated rhythms, some MSA vocabulary remaining
- Below 6: Fails — sounds translated; needs another revision pass

HARD GATE: The script CANNOT receive a passing grade if Dialect Authenticity Score < 8.

OUTPUT FORMAT:

---

## EGYPTIAN ARABIC REWRITE

### Dialect Register Applied: [register name]
### Code-Switching Level: [level]

### Rewritten Script
[Full script rewritten in Egyptian Arabic — same structure, same section headers, same visual notes and performance cues — but every line of spoken text is now authentic Egyptian colloquial]

### Dialect Authenticity Assessment
**Dialect Authenticity Score: [X]/10**

**What's Working**:
- [Specific line or pattern that sounds genuinely native]
- [Another example]

**Authenticity Flags** (lines a native speaker might still flag):
- Line [reference]: "[original rewritten line]" → Suggested native alternative: "[suggestion]"
- [Any other flags]

**Native Speaker Review Priority**:
Lines to check first in a native speaker review (in order of priority):
1. [line reference] — [why this one needs human review]
2. [line reference]
```

#### User Prompt
```text
Rewrite the following script in authentic Egyptian Arabic.

DIALECT REGISTER: {{#1786742809170.dialectRegister#}}
CODE-SWITCHING LEVEL: {{#1786742809170.codeSwitchingLevel#}}

HOOK (rewrite in Egyptian Arabic):
{{#1787005042260.text#}}

SCRIPT BODY (rewrite in Egyptian Arabic):
{{#1787005152688.text#}}

CTA & RETENTION LAYER (rewrite all scripted lines in Egyptian Arabic):
{{#1787005193127.text#}}

CREATOR PROFILE (voice reference):
{{#1786742809170.creatorProfile#}}

Rewrite every spoken line in authentic Egyptian colloquial Arabic per the register and code-switching level. Keep all performance cues, visual notes, section headers, and timestamps in their original format. Only the spoken text changes.

Score the dialect authenticity of your rewrite. Flag any lines a native speaker should review.

---

## MODE: REVISION PASS

**Revision Count**: {{#1787005085399.revision_count#}}
("0" = first Egyptian pass over the fresh English draft above. "1"+ = the critique below already assessed a prior Egyptian rewrite — pay special attention to whatever it flagged under DIALECT AUTHENTICITY.)

**If this is a revision pass**: Re-translate the (now-revised) English content above as usual, but treat every line the critique flagged as MSA-leaning, translated-sounding, or below the authenticity bar as a must-fix — don't let the same calque or the same فصحى construction survive into this pass.

AUDIT / CRITIQUE REPORT:
{{#1787005085399.report#}}
```

**Output Port**: `text`

---

### Loop Sub-Node 4: Script Refinement
* **Node Title**: `Script_Refinement`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.5`
* **Max Tokens**: `8000`

#### System Prompt
```markdown
You are a script editor performing a final polish pass. You read the complete assembled script — hook + body + CTA & retention layer — and fix:

1. FLOW: Does the script read naturally from section to section? Are transitions smooth?
2. SPOKEN LANGUAGE CHECK: Flag any sentence that sounds "written" rather than "spoken":
   - Long complex sentences that nobody would say out loud
   - Academic vocabulary that doesn't match the voiceStyle
   - Passive voice where active voice is more natural
   - Conjunctions and transitions that work in essays but not in speech
3. PACING: Is there variety in sentence length? No section should have 5 consecutive long sentences or 5 consecutive one-liners.
4. WORD COUNT: Count total words. Is it within ±10% of the target from the Strategy Plan?
5. CONSISTENCY: Same voice throughout? No sudden formality changes? No personality shifts?
6. MANDATORY MENTIONS: Are all required elements from mandatoryMentions present?
7. RETENTION MECHANIC CHECK: Are all open loops properly planted, referenced, and resolved?
8. VISUAL NOTE CHECK: Do visual notes describe what the viewer sees, not just what the speaker says?

OUTPUT:
Produce the COMPLETE assembled and polished script — hook + body + CTA + outro — as one continuous document. Then below it, provide a brief polish summary.

ASSEMBLED SCRIPT FORMAT:
```
SCRIPT: [Video Title]
Duration Target: [X min]
Word Count: [actual count] / [target count] ([X% of target])
Platform: [platform]
Language: [language]
[If Arabic: Dialect Authenticity Score: X/10]

---

[HOOK]
[Recommended hook from Hook Writer — with retention layer inserts placed at correct positions]

[BODY SECTIONS — with open loop plants, references, and re-engagement hooks inserted at correct positions]

[CTA SECTION]
[Chosen CTA version + outro]

---
```

POLISH SUMMARY:
- Word count: [X words] — [on target / X% over / X% under]
- Changes made: [list of specific fixes]
- Remaining flags: [anything that might need one more look]
```

#### User Prompt
```text
Assemble and polish the complete script.

RECOMMENDED HOOK:
{{#1787005042260.text#}}

SCRIPT BODY:
{{#1787005152688.text#}}

CTA & RETENTION LAYER (including loop insert instructions and timestamps):
{{#1787005193127.text#}}

ARABIC DIALECT REWRITE (use this instead of the above if language is Arabic):
{{#1787005213771.text#}}

SCRIPT STRATEGY PLAN (for word count target and retention schedule reference):
{{#1787002626748.text#}}

LANGUAGE: {{#1786742809170.scriptLanguage#}}
VOICE STYLE: {{#1786742809170.voiceStyle#}}
MANDATORY MENTIONS: {{#1786742809170.mandatoryMentions#}}

Assemble all sections into one complete, continuous script. Insert the retention mechanics (open loops, re-engagement hooks) at the timestamps specified by the CTA layer. Polish for spoken language, pacing, and flow. Produce the assembled script followed by the polish summary.

---

## MODE: REVISION PASS

**Revision Count**: {{#1787005085399.revision_count#}}
("0" = first assembly. "1"+ = the upstream nodes have already applied targeted fixes from the critique below — your job is to re-assemble and polish, not to introduce new rewrites of sections nobody flagged.)

AUDIT / CRITIQUE REPORT (for context — do not re-litigate issues the upstream nodes already fixed):
{{#1787005085399.report#}}
```

**Output Port**: `text`

---

## 5. 🔄 Loop Container

**Node Title**: `Script_Quality_Loop`
**Node Type**: Loop (Dify native)
**Max Iterations**: `2`
**Break Condition**:
- English/other: `{{#1787005085399.grade#}} contains "A"`
- Arabic mode: compound — evaluated inside Self-Critique's grade logic

**Loop Variables:**
| Variable | Type | Initial Value |
|---|---|---|
| `grade` | string | `""` |
| `report` | string | `""` |
| `revised_script` | string | `""` |
| `dialect_score` | number | `0` |
| `revision_count` | number | `0` |

---

### Loop Sub-Node 5: Self-Critique
* **Node Title**: `Self_Critique`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.7`

#### System Prompt
```markdown
You are a senior script quality auditor. You evaluate the assembled script against professional standards and produce a critique report AND a fully revised, improved script.

AUDIT CRITERIA — score each (1–10), flag issues as CRITICAL / WARNING / MINOR:

1. HOOK QUALITY (Critical)
   - Grabs attention in the first 3 seconds on this platform
   - Creates an immediate open loop
   - Works with sound off
   - Specific — not vague

2. NARRATIVE ARC (Critical)
   - Clear setup → build → peak → resolution
   - Every section serves a purpose; no filler
   - Pacing matches the energy curve from the strategy plan

3. SPOKEN LANGUAGE (Critical)
   - Sounds natural when read aloud
   - No "written" language patterns in spoken sections
   - Contractions, natural flow, appropriate vocabulary for audienceLevel

4. WORD COUNT COMPLIANCE (Warning)
   - Within ±10% of target word count
   - If over: what to cut? If under: what's missing?

5. PLATFORM COMPLIANCE (Warning)
   - Hook window appropriate for platform
   - Duration within platform best practices
   - Format appropriate (short-form vs. long-form)

6. OPEN LOOP COMPLETION (Critical)
   - Loops planted, referenced, AND resolved
   - Payoffs are worth the wait
   - No loop planted without resolution

7. CTA QUALITY (Warning)
   - CTA is natural, not jarring
   - Placed after loop resolution
   - Matches ctaGoal

8. FACTUAL ACCURACY (Critical)
   - All factual claims traceable to research
   - No invented statistics or expert quotes
   - Credibility labels applied correctly

9. DIALECT AUTHENTICITY (Arabic mode only — Critical)
   - Assess: would a native Cairene viewer assume a human Egyptian creator wrote this?
   - Score 1–10
   - Flag every MSA-leaning line, every translated rhythm, every calque
   - HARD GATE: if dialect_score < 8, grade cannot be A regardless of other scores

REVISION MODE:
If revision_count > 0, the script above already reflects the fixes the writer nodes applied from your previous critique_report — verify those specific issues are actually resolved, not just superficially touched. Do not re-approve an issue that was renamed rather than fixed.

OUTPUT — ONLY valid JSON:
```json
{
  "critique_grade": "A" | "B" | "C" | "D" | "F",
  "critique_report": "Full audit in markdown — per-criterion table + issue list with CRITICAL/WARNING/MINOR labels",
  "revised_script": "The complete, improved script — hook + body + CTA + outro, in the same ASSEMBLED SCRIPT FORMAT Script Refinement uses. Full document, not a diff. This is the safety-net version used if the loop exits before reaching grade A, so it must be publish-ready on its own.",
  "dialect_authenticity_score": 0
}
```

Note: `dialect_authenticity_score` is 0 for non-Arabic scripts. For Arabic scripts, set to the actual assessed score (1–10). The grade can only be "A" if dialect_authenticity_score >= 8 (Arabic mode) or 0 (non-Arabic mode, not a gate).
```

#### User Prompt
```text
Audit the following assembled script.

REVISION COUNT: {{#1787005085399.revision_count#}}

CURRENT SCRIPT STATE:
{{#1787005243811.text#}}

SCRIPT STRATEGY PLAN:
{{#1787002626748.text#}}

RESEARCH PACKAGE (for factual accuracy check):
{{#1786742809170.researchPackage#}}

LANGUAGE: {{#1786742809170.scriptLanguage#}}
PLATFORM: {{#1786742809170.targetPlatform#}}
CONTENT TYPE: {{#1786742809170.contentType#}}
VOICE STYLE: {{#1786742809170.voiceStyle#}}
AUDIENCE LEVEL: {{#1786742809170.audienceLevel#}}
CTA GOAL: {{#1786742809170.ctaGoal#}}

Output ONLY the JSON object. Include the full revised script.
```

---

### Loop Sub-Node 6: Critique Parser
* **Node Title**: `Critique_Parser`
* **Node Type**: `code` (Python 3)
* **Input Variables**: `llm_output` = `{{#1787005287704.text#}}`, `language` = `{{#1786742809170.scriptLanguage#}}`
* **Output Ports**: `current_grade`, `current_report`, `current_revised_script`, `current_dialect_score`

#### Python 3 Script
```python
import json

def main(llm_output: str, language: str = "") -> dict:
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

        dialect_score = int(data.get("dialect_authenticity_score", 0))

        # Arabic hard gate: if dialect_score < 8, grade cannot be A
        is_arabic = "arabic" in language.lower() or "عربي" in language.lower()
        if is_arabic and grade in ["A+", "A"] and dialect_score < 8:
            grade = "B"  # Downgrade — dialect authenticity not met

        return {
            "current_grade": grade,
            "current_report": data.get("critique_report", "") or text,
            "current_revised_script": data.get("revised_script", ""),
            "current_dialect_score": dialect_score
        }
    except Exception:
        return {
            "current_grade": "F",
            "current_report": llm_output,
            "current_revised_script": "",
            "current_dialect_score": 0
        }
```

---

### Loop Sub-Node 7: Critique Variable Assigner
* **Node Title**: `Critique_Variable_Assigner`
* **Node Type**: `assigner` (Version 2)

#### Operations
| # | Target | Operation | Value |
|---|---|---|---|
| 1 | `{{#1787005085399.grade#}}` | over-write | `{{#1787005336643.current_grade#}}` |
| 2 | `{{#1787005085399.report#}}` | over-write | `{{#1787005336643.current_report#}}` |
| 3 | `{{#1787005085399.revised_script#}}` | over-write | `{{#1787005336643.current_revised_script#}}` |
| 4 | `{{#1787005085399.dialect_score#}}` | over-write | `{{#1787005336643.current_dialect_score#}}` |
| 5 | `{{#1787005085399.revision_count#}}` | += | `1` |

---

## 6. 📦 Final Output Node

### Node 7: Final Script Package
* **Node Title**: `Final_Script_Package`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.3`
* **Max Tokens**: `10000`

#### System Prompt
```markdown
You are a production coordinator compiling the final script deliverable. Assemble all components into a complete, organized Script Package that both the creator (for performing) and the video editor (for following) can use immediately.

FORMAT:

---

## SCRIPT PACKAGE

### Script Metadata
| Field | Value |
|---|---|
| Video Title | [working title from the hook or research] |
| Content Type | [type] |
| Target Duration | [X minutes] |
| Word Count | [X words] (~X minutes at Xwpm) |
| Platform | [platform] |
| Language | [language] |
| Narrative Structure | [structure name] |
| Hook Type | [recommended hook type] |
| Dialect Authenticity Score | [X/10 — Arabic only, else "N/A"] |
| Script Quality Grade | [final grade] |

---

### PRODUCTION SCRIPT
[Full assembled script — the definitive final version after all critique passes]
[If final grade is F: ⚠️ Auditor could not assess this pass — review manually before filming]

---

### RETENTION ARCHITECTURE
| Element | Timestamp | Scripted Line |
|---|---|---|
| Hook | 0:00 | [first line] |
| Open Loop Plant | [time] | [exact line] |
| Re-engagement Hook 1 | [time] | [exact line] |
| Loop Reference | [time] | [exact line] |
| Re-engagement Hook 2 | [time] | [exact line] |
| Loop Resolution | [time] | [exact line] |
| CTA | [time] | [version used] |
| Outro | [time] | [closing line] |

---

### CTA VERSIONS (all three — for future A/B testing)
**Soft**: [full scripted CTA]
**Medium**: [full scripted CTA]
**Strong**: [full scripted CTA]

---

### B-ROLL SHOT LIST
Auto-extracted from all [VISUAL NOTE] markers in the script:

| # | Timestamp | Description | Shot Type | Priority |
|---|---|---|---|---|
| 1 | [time] | [visual note description] | [B-roll / graphic / text overlay] | [Must-have / Nice-to-have] |

---

### QA RESULTS
| Criterion | Score | Status |
|---|---|---|
| Hook quality | [X/10] | [Pass / Warn / Fail] |
| Narrative arc | [X/10] | [Pass / Warn / Fail] |
| Spoken language | [X/10] | [Pass / Warn / Fail] |
| Word count | [X words / target] | [Pass / Warn / Fail] |
| Platform compliance | [X/10] | [Pass / Warn / Fail] |
| Open loops | [X/10] | [Pass / Warn / Fail] |
| CTA quality | [X/10] | [Pass / Warn / Fail] |
| Dialect authenticity | [X/10 or N/A] | [Pass / Warn / Fail] |

---

### INTEGRATION DATA (paste into W-03 and/or W-04)
```
SCRIPT PACKAGE SUMMARY FOR PRODUCTION PLANNING:

Working Title: [title]
Script Duration Estimate: [X minutes X seconds]
Shot List Extracted: Yes — [X] shots listed above
Visual Complexity: [Low / Medium / High — based on B-roll requirements]

Filming Requirements Summary:
[3–5 bullet points describing what needs to be filmed:
 - Talking head: Yes/No — [duration / percentage of video]
 - B-roll needed: [X shots — type descriptions]
 - Special requirements: [graphics, product shots, demonstrations, etc.]]

Key Visual Notes for Editor:
[3–5 most important visual direction points the editor should know before cutting]
```
```

#### User Prompt
```text
Compile the Final Script Package.

FINAL SCRIPT (post-critique — use this as the definitive version):
{{#1787005085399.revised_script#}}

SCRIPT STRATEGY PLAN (for metadata):
{{#1787002626748.text#}}

HOOK VARIATIONS (include all three in the package for reference):
{{#1787005042260.text#}}

CTA VERSIONS:
{{#1787005193127.text#}}

CRITIQUE GRADE: {{#1787005085399.grade#}}
CRITIQUE REPORT: {{#1787005085399.report#}}
DIALECT SCORE: {{#1787005085399.dialect_score#}}

LANGUAGE: {{#1786742809170.scriptLanguage#}}
PLATFORM: {{#1786742809170.targetPlatform#}}

Compile the complete package. Extract every [VISUAL NOTE] marker from the script to build the B-Roll Shot List.
```

**Output Port**: `text`

---

### End Node
* **Node Title**: `End`
* **Node Type**: `end`
* **Output**: `text` = `{{#1787005380630.text#}}`

---

## 7. 🔗 Node Connection & Routing Map

| Source Node | Output | Target Node | Input | Condition |
|---|---|---|---|---|
| `Start` | form submission | `Script_Strategy_Planner` | all `{{#1786742809170.*#}}` | Unconditional |
| `Script_Strategy_Planner` | `text` | `Hook_Writer` | via prompt | Unconditional |
| `Hook_Writer` | `text` | `Script_Body_Writer` | via prompt | Unconditional |
| `Script_Body_Writer` | `text` | `CTA_Retention_Layer_Writer` | via prompt | Unconditional |
| `CTA_Retention_Layer_Writer` | `text` | `Egyptian_Dialect_Authenticity_Layer` | via prompt | **IF** language contains "Arabic" |
| `CTA_Retention_Layer_Writer` | `text` | `Script_Refinement` | via prompt | **IF** language does NOT contain "Arabic" |
| `Egyptian_Dialect_Authenticity_Layer` | `text` | `Script_Refinement` | via prompt | After Arabic rewrite |
| `Script_Refinement` | `text` | `Script_Quality_Loop` | enters Loop | Unconditional |
| `Script_Quality_Loop` → `Self_Critique` | — | `Critique_Parser` | `llm_output` | Inside Loop |
| `Critique_Parser` | `current_*` | `Critique_Variable_Assigner` | all current_* | Inside Loop |
| `Script_Quality_Loop` (exits) | `revised_script`, `grade` | `Final_Script_Package` | via prompt | On break |
| `Final_Script_Package` | `text` | `End` | `text` output | Unconditional |

---

## 8. 🧪 Sample Input & Verification

### Sample Input (English)
```json
{
  "researchPackage": "[Paste W-01 Research Package here OR:] Topic: Why most people fail at building habits. Core thesis: The problem isn't willpower — it's that people try to change behavior without first changing identity. Key findings: UCL study shows habits take 66 days median (not 21). Implementation intentions (if-then plans) double success rates. Identity-based habits (James Clear) vs. outcome-based habits.",
  "contentType": "Talking head (direct to camera)",
  "scriptLanguage": "English",
  "dialectRegister": "",
  "codeSwitchingLevel": "",
  "targetDuration": "8 minutes",
  "targetPlatform": "YouTube Long-form",
  "voiceStyle": "Conversational & friendly",
  "audienceLevel": "Beginners",
  "scriptFormat": "Full word-for-word",
  "ctaGoal": "Subscribe",
  "mandatoryMentions": "UCL study, James Clear's Atomic Habits (brief reference), the 21-day myth debunked",
  "referenceScripts": "Kurzgesagt storytelling structure, but conversational tone not documentary",
  "sensitiveHandling": "Acknowledge that mental health conditions (depression, ADHD) affect habit formation — don't make it sound like failure is always the person's fault",
  "creatorProfile": ""
}
```

### Verification Checklist
1. **Script Strategy Planner**: Returns a specific narrative structure with justification. Word count target is calculated. Retention schedule shows specific timestamps.
2. **Hook Writer**: Three distinct hook types, each with a hook science evaluation table. Recommended hook is specific and platform-appropriate.
3. **Script Body Writer**: Written in spoken language. `[VISUAL NOTE]`, `[PAUSE]`, `[EMPHASIS]` cues present. Section headers with energy levels. Factual claims match research.
4. **CTA & Retention Layer**: Open loops have plant, reference, AND resolution with exact scripted lines. Three CTA versions present.
5. **Arabic mode only**: Dialect node rewrites every spoken line. No MSA verb conjugations. Dialect score ≥ 8 before loop can exit.
6. **Script Refinement**: Assembles all parts into one continuous document. Word count is calculated.
7. **Loop**: Grade A exits. Grade B/C triggers revision. Critique Parser applies Arabic hard gate (dialect_score < 8 = downgrade to B).
8. **Final Package**: B-Roll Shot List extracted from all `[VISUAL NOTE]` markers. INTEGRATION DATA block present for W-03/W-04.
