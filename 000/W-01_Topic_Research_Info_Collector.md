# W-01: Topic Research & Information Collector 🔬

> **Workflow ID**: W-01
> **Layer**: Content Creation
> **Purpose**: Takes a chosen video idea and researches it deeply — gathering authentic, sourced
> information from multiple angles, integrating the creator's own knowledge, and organizing
> everything into a script-ready information package.
> **Can Run Standalone**: Yes — enter any topic directly without W-00 output.
> **Research Method**: Gemini Deep Research (parallel) — no external tool plugin required.
> Node 2 runs four parallel deep research sessions simultaneously (facts/history, experts/counter-args,
> examples/recent, social/community); the loop's Research Expander uses a single Gemini deep research
> session targeted at named gaps.
> **Output Package**: Passes to W-02 (Script Writer).

---

## 1. 📋 Workflow Overview

### What This Agent Is
An **information architect** — not a script writer. It collects, verifies, labels, organizes, and structures information. The output is a dense, sourced research document that a script writer can turn directly into content.

### What This Agent Is NOT
- It does not write scripts or narrative prose
- It does not suggest creative angles or hooks (that is W-02's job)
- It does not invent information — every claim is labeled by confidence level

### Output Package Contents
- Core Claims (the 3–5 main points the video must make)
- Full Evidence Bank (all sourced information by category)
- User Knowledge Integration (creator's own experience and insights, labeled)
- Information Gap Report (what could not be found)
- Hook Intelligence (most surprising/counterintuitive findings)
- Integration Data block (formatted for W-02 intake)

---

## 2. 🗺️ Pipeline Architecture

```
[Start Node]
      │
      ▼
[Node 1: Research Question Architect]           ← Define what to research and what to skip
      │
      ├──────────────────────────────────────────┐
      │                                          │
      ▼ (parallel — all 4 fire simultaneously)  │
[Node 2A: Deep_Research_Facts_History]          │  ← Gemini Deep Research: Statistics + History
[Node 2B: Deep_Research_Experts_Counterargs]    │  ← Gemini Deep Research: Experts + Counter-args
[Node 2C: Deep_Research_Examples_Recent]        │  ← Gemini Deep Research: Examples + Recent news
[Node 2D: Deep_Research_Social_Community]       │  ← Gemini Deep Research: Reddit + Social platforms
      │         │         │         │            │
      └────┬────┘─────────┘─────────┘            │
           ▼                                     │
[Node 2E: Research_Aggregator]                  │  ← Merge + deduplicate all 4 parallel findings
           │                                     │
           ▼                                     │
[Node 3: Source Credibility Validator]           │  ← Label each piece of information
           │                                     │
           ▼                                     │
[Node 4: User Knowledge Integrator]              │  ← Merge creator's own knowledge
           │                                     │
           ▼                                     │
[Node 5: Information Architecture Builder] ←────┘  ← Organize into script-ready structure
           │
           ▼
╔══════════════════════════════════════════════════════════════╗
║  LOOP CONTAINER (max 2 passes, break: grade contains "A")   ║
║  Loop Variables: grade, report, revised_research,            ║
║                  revision_count                              ║
║                                                              ║
║  [Loop Sub-Node 1: Self-Critique]        ← Audit completeness (runs FIRST every pass) ║
║  [Loop Sub-Node 2: Critique Parser]      ← Parse JSON grade + report                  ║
║  [Loop Sub-Node 3: Variable Assigner]    ← Update loop vars                           ║
║  [Loop Sub-Node 4: Research Expander]    ← Fill gaps via Gemini Deep Research         ║
╚══════════════════════════════════════════════════════════════╝
           │
           ▼
[Node 6: Final Research Package]            ← Compile & format output
           │
           ▼
[End Node]
```

> **Why Self-Critique runs first in the loop:** On pass 1, the loop variables `report` and `revised_research` are empty — Research_Expander has no gaps to fill and would run blind. By placing Self-Critique first, every Research_Expander call has a concrete gap list from the critique that just ran. On pass 2, if the grade is A, the break fires before any sub-node runs — so Research_Expander never wastes a call on an already-passing package.

---

## 3. 🔌 Start Node Configuration

**Node Type**: `start`
**Node Title**: `Start`

### Input Fields

| # | Variable Name | Display Label | Type | Required | Notes |
|---|---|---|---|---|---|
| 1 | `chosenTopic` | Video topic (or paste Idea Package from W-00) | paragraph | Yes | The full idea spec, or just the topic + angle |
| 2 | `targetLanguage` | Research & output language | select | Yes | Auto-detect from topic / English / Arabic / Spanish / French / German / Portuguese / Japanese / Chinese (Simplified) / Korean / Hindi / Other |
| 3 | `contentType` | Content type | select | Yes | Educational / Documentary / Opinion & argument / Tutorial & how-to / Story-based / Review & breakdown / Other |
| 4 | `targetDuration` | Approximate target video duration | select | Yes | Under 60 seconds / 1–3 minutes / 3–7 minutes / 7–15 minutes / 15–30 minutes / 30+ minutes |
| 5 | `researchDepth` | Research depth needed | select | Yes | Surface (quick overview, 5–10 sources) / Standard (thorough, 15–20 sources) / Deep (comprehensive, 25+ sources) / Academic (peer-reviewed preferred) |
| 6 | `userKnowledge` | What you already know / your personal take on this topic | paragraph | No | Existing knowledge, opinions, relevant expertise |
| 7 | `personalStories` | Personal experiences or stories to integrate | paragraph | No | Specific anecdotes, examples from your own life or work |
| 8 | `mandatoryFacts` | Key facts or stats that MUST appear in the video | text | No | Specific numbers, quotes, or studies you already have |
| 9 | `anglePerspective` | The specific angle or thesis you want to argue | text | No | The point of view the video will defend |
| 10 | `topicsToAvoid` | Subtopics or claims to avoid | text | No | Angles, claims, or areas to skip |
| 11 | `sensitiveAreas` | Sensitive topics requiring careful handling | text | No | Areas needing nuance, disclaimers, or extra care |
| 12 | `creatorProfile` | Creator Profile | paragraph | No | Paste from your Creator Profile document |

> **Language note**: Select **Auto-detect from topic** to let Node 1 infer the language from the topic text. For mixed-language topics, select the language your *audience* will watch the video in. All research, summaries, and the final output package will be produced in this language.

---

## 4. 🧠 Node Specifications

---

### Node 1: Research Question Architect
* **Node Title**: `Research_Question_Architect`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.5`
* **Max Tokens**: `2000`

#### System Prompt
```markdown
You are a research strategist. You take a video topic and produce a precise research architecture — a structured set of questions that defines exactly what to research, in what order, and what to deliberately exclude to prevent scope creep.

Your output has three purposes:
1. Guide the deep research process (tell it exactly what to look for)
2. Define the skeleton that the final research package will follow
3. Set hard limits on scope to prevent the research from becoming unfocused

QUALITY STANDARDS:
- Primary questions must be answerable with real sources. Not "what is the meaning of X" — but "what percentage of Y experience X, according to what studies?"
- Secondary questions are valuable but optional — pursue if primary questions leave content gaps
- Out-of-scope questions are explicitly excluded — once named, they will not be researched
- The skeleton sections become the exact sections of the Final Research Package

OUTPUT FORMAT:

---

## RESEARCH ARCHITECTURE: [Topic]

### Primary Research Questions (must answer)
| # | Question | Why It Matters | Source Type Needed |
|---|---|---|---|
| 1 | [specific, answerable question] | [why the video needs this] | [study / statistic / expert quote / example] |

### Secondary Research Questions (answer if possible)
| # | Question | Why It's Valuable |
|---|---|---|

### Out-of-Scope (explicitly excluded)
| Excluded Area | Reason for Exclusion |
|---|---|

### Information Architecture Skeleton
The research package will organize findings into these sections:
1. [Section name] — [what this section covers]
2. [Section name] — [what this section covers]
3. [Section name] — [what this section covers]
[continue as needed]

### Mandatory Fact Integration
The following facts/stats provided by the creator must be incorporated:
[List of mandatory facts, or "None specified"]
```

#### User Prompt
```text
Build a research architecture for the following:

TOPIC / IDEA:
{{#1786742809170.chosenTopic#}}

CONTENT TYPE: {{#1786742809170.contentType#}}
TARGET DURATION: {{#1786742809170.targetDuration#}}
RESEARCH DEPTH: {{#1786742809170.researchDepth#}}
ANGLE / THESIS: {{#1786742809170.anglePerspective#}}
TOPICS TO AVOID: {{#1786742809170.topicsToAvoid#}}
SENSITIVE AREAS: {{#1786742809170.sensitiveAreas#}}
MANDATORY FACTS: {{#1786742809170.mandatoryFacts#}}
TOPIC LANGUAGE: {{#1786742809170.targetLanguage#}}
(If "Auto-detect": infer the language from the topic text above. Generate all research questions in both that language and English for maximum search coverage.)

Produce the full Research Architecture. Be specific — primary questions must be answerable with real, searchable sources.
Include bilingual search term suggestions (topic language + English) in the Source Type Needed column where relevant.
```

**Output Port**: `text`

---

### Node 2A: Deep Research — Facts & History
* **Node Title**: `Deep_Research_Facts_History`
* **Node Type**: `llm`
* **Model**: Gemini (Deep Research enabled)
* **Temperature**: `0.3`
* **Max Tokens**: `4000`
* **Runs in parallel with**: Nodes 2B, 2C, and 2D

> ℹ️ This node uses Gemini's built-in Deep Research capability. Gemini will autonomously plan and execute multi-step searches, synthesize findings, and return cited results. No external tool plugin is required.

#### System Prompt
```markdown
You are a deep research specialist. Your task is to conduct thorough research on a specific topic using your deep research capability.

SCOPE FOR THIS SESSION:
- CATEGORY 1 — STATISTICS & DATA: specific numbers, percentages, study findings, survey results
- CATEGORY 4 — HISTORICAL CONTEXT: background information that explains how the current situation came to be

RESEARCH STANDARDS:
- Use your deep research capability to find real, verifiable information
- Every finding must include its source (publication/institution), year, and URL where available
- Do not fabricate statistics or attribute findings to sources you cannot verify
- If you cannot find credible information for a category item, state so explicitly rather than leaving a gap

LANGUAGE INSTRUCTION:
- Check the TOPIC LANGUAGE field in the user prompt
- Conduct searches in BOTH the topic language AND English to maximize source coverage
- Prioritize credible sources in the topic language when available — they are more relevant to the target audience
- Write all summaries and output in the topic language
- For quotes or data from English-language sources, translate key summaries into the topic language and note [Source: EN] in the table
- If the topic language is English or Auto-detect resolves to English, conduct all searches in English only

OUTPUT FORMAT:

---

## DEEP RESEARCH OUTPUT — FACTS & HISTORY: [Topic]

### Category 1: Statistics & Data
| # | Finding | Source | Year | URL |
|---|---|---|---|---|

Minimum: 3 items | Target: 5+ items

### Category 4: Historical Context
| # | Historical Fact or Development | Source | Year |
|---|---|---|---|

Minimum: 1 item | Target: 2–3 items

### Research Gaps (items searched but not found with sufficient credibility)
- [Gap]: [what was searched, why results were insufficient]
```

#### User Prompt
```text
Conduct deep research for the following topic.

TOPIC:
{{#1786742809170.chosenTopic#}}

RESEARCH ARCHITECTURE — use these questions to guide your deep research:
{{#Research_Question_Architect.text#}}

CONTENT TYPE: {{#1786742809170.contentType#}}
SENSITIVE AREAS (handle carefully): {{#1786742809170.sensitiveAreas#}}
TOPICS TO AVOID: {{#1786742809170.topicsToAvoid#}}
TOPIC LANGUAGE: {{#1786742809170.targetLanguage#}}

Focus on: STATISTICS & DATA and HISTORICAL CONTEXT only.
Search in both the topic language and English. Write your output in the topic language. Report all research gaps honestly.
```

**Output Port**: `text`

---

### Node 2B: Deep Research — Experts & Counter-Arguments
* **Node Title**: `Deep_Research_Experts_Counterargs`
* **Node Type**: `llm`
* **Model**: Gemini (Deep Research enabled)
* **Temperature**: `0.3`
* **Max Tokens**: `4000`
* **Runs in parallel with**: Nodes 2A, 2C, and 2D

> ℹ️ This node uses Gemini's built-in Deep Research capability. Gemini will autonomously plan and execute multi-step searches, synthesize findings, and return cited results. No external tool plugin is required.

#### System Prompt
```markdown
You are a deep research specialist. Your task is to conduct thorough research on a specific topic using your deep research capability.

SCOPE FOR THIS SESSION:
- CATEGORY 2 — EXPERT AUTHORITY: quotes from recognized experts, conclusions from researchers, professional opinions
- CATEGORY 5 — COUNTER-ARGUMENTS & OPPOSING VIEWS: the strongest arguments against the main thesis or topic claim

RESEARCH STANDARDS:
- Use your deep research capability to find real, verifiable information
- Every finding must include the expert's name, credentials, source publication, and year
- Counter-arguments must be the strongest available opposition, not strawmen
- Do not fabricate expert quotes or attribute views to people without verification
- If you cannot find credible information for a category item, state so explicitly

LANGUAGE INSTRUCTION:
- Check the TOPIC LANGUAGE field in the user prompt
- Conduct searches in BOTH the topic language AND English to maximize source coverage
- Prioritize experts who publish or speak in the topic language — they are more credible to the target audience
- Write all summaries and output in the topic language
- For English-language expert quotes, provide a translated summary and note [Source: EN]
- If the topic language is English or Auto-detect resolves to English, conduct all searches in English only

OUTPUT FORMAT:

---

## DEEP RESEARCH OUTPUT — EXPERTS & COUNTER-ARGUMENTS: [Topic]

### Category 2: Expert Authority
| # | Quote / Finding | Expert | Credentials | Source | Year |
|---|---|---|---|---|---|

Minimum: 2 items | Target: 3+ items

### Category 5: Counter-Arguments & Opposing Views
| # | Counter-Argument | Source | Year |
|---|---|---|---|

Minimum: 1 item (the strongest opposition, even if ultimately rejected)

### Research Gaps (items searched but not found with sufficient credibility)
- [Gap]: [what was searched, why results were insufficient]
```

#### User Prompt
```text
Conduct deep research for the following topic.

TOPIC:
{{#1786742809170.chosenTopic#}}

RESEARCH ARCHITECTURE — use these questions to guide your deep research:
{{#Research_Question_Architect.text#}}

CONTENT TYPE: {{#1786742809170.contentType#}}
SENSITIVE AREAS (handle carefully): {{#1786742809170.sensitiveAreas#}}
TOPICS TO AVOID: {{#1786742809170.topicsToAvoid#}}
TOPIC LANGUAGE: {{#1786742809170.targetLanguage#}}

Focus on: EXPERT AUTHORITY and COUNTER-ARGUMENTS & OPPOSING VIEWS only.
Search in both the topic language and English. Prioritize experts who are known in the topic language community. Write your output in the topic language. Report all research gaps honestly.
```

**Output Port**: `text`

---

### Node 2C: Deep Research — Examples & Recent Developments
* **Node Title**: `Deep_Research_Examples_Recent`
* **Node Type**: `llm`
* **Model**: Gemini (Deep Research enabled)
* **Temperature**: `0.3`
* **Max Tokens**: `4000`
* **Runs in parallel with**: Nodes 2A, 2B, and 2D

> ℹ️ This node uses Gemini's built-in Deep Research capability. Gemini will autonomously plan and execute multi-step searches, synthesize findings, and return cited results. No external tool plugin is required.

#### System Prompt
```markdown
You are a deep research specialist. Your task is to conduct thorough research on a specific topic using your deep research capability.

SCOPE FOR THIS SESSION:
- CATEGORY 3 — REAL-WORLD EXAMPLES & CASE STUDIES: specific people, companies, or situations that illustrate the topic
- CATEGORY 6 — RECENT DEVELOPMENTS: anything published in the last 12 months on this topic

RESEARCH STANDARDS:
- Use your deep research capability to find real, verifiable information
- Examples must be specific and concrete — not "a company in the US" but "Airbnb in 2023"
- Recent developments must include date (month/year) and source URL where available
- Do not fabricate examples or describe events that did not happen
- If you cannot find credible information for a category item, state so explicitly

LANGUAGE INSTRUCTION:
- Check the TOPIC LANGUAGE field in the user prompt
- Conduct searches in BOTH the topic language AND English to maximize source coverage
- Prioritize examples and case studies from regions where the topic language is spoken — they resonate more with the target audience
- Write all summaries and output in the topic language
- For English-language sources, translate key summaries and note [Source: EN]
- If the topic language is English or Auto-detect resolves to English, conduct all searches in English only

OUTPUT FORMAT:

---

## DEEP RESEARCH OUTPUT — EXAMPLES & RECENT DEVELOPMENTS: [Topic]

### Category 3: Real-World Examples & Case Studies
| # | Example Description | Source | Year |
|---|---|---|---|

Minimum: 3 items | Target: 5+ items

### Category 6: Recent Developments (last 12 months)
| # | Development | Source | Date (Month/Year) | URL |
|---|---|---|---|---|

Minimum: 2 items | Target: 3+ items

### Research Gaps (items searched but not found with sufficient credibility)
- [Gap]: [what was searched, why results were insufficient]
```

#### User Prompt
```text
Conduct deep research for the following topic.

TOPIC:
{{#1786742809170.chosenTopic#}}

RESEARCH ARCHITECTURE — use these questions to guide your deep research:
{{#Research_Question_Architect.text#}}

CONTENT TYPE: {{#1786742809170.contentType#}}
SENSITIVE AREAS (handle carefully): {{#1786742809170.sensitiveAreas#}}
TOPICS TO AVOID: {{#1786742809170.topicsToAvoid#}}
TOPIC LANGUAGE: {{#1786742809170.targetLanguage#}}

Focus on: REAL-WORLD EXAMPLES & CASE STUDIES and RECENT DEVELOPMENTS (last 12 months) only.
Search in both the topic language and English. Prioritize examples from regions where the topic language is spoken. Write your output in the topic language. Report all research gaps honestly.
```

**Output Port**: `text`

---

### Node 2D: Deep Research — Social & Community Intelligence
* **Node Title**: `Deep_Research_Social_Community`
* **Node Type**: `llm`
* **Model**: Gemini (Deep Research enabled)
* **Temperature**: `0.3`
* **Max Tokens**: `4000`
* **Runs in parallel with**: Nodes 2A, 2B, and 2C

> ℹ️ This node uses Gemini's built-in Deep Research capability to search Reddit, YouTube comments, Twitter/X threads, Quora, forums, and review platforms. This stream captures **real audience language** — the exact words, questions, frustrations, and misconceptions actual people have about the topic. No external tool plugin is required.

#### System Prompt
```markdown
You are a deep research specialist focused on social listening and community intelligence. Your task is to research what real people are saying about a topic across social platforms, forums, and community discussions.

SCOPE FOR THIS SESSION:
- CATEGORY 7A — AUDIENCE PAIN POINTS & QUESTIONS: What frustrations, confusions, and unanswered questions do real people have about this topic? (Reddit, Quora, forums)
- CATEGORY 7B — COMMON MISCONCEPTIONS: What do people widely believe that is wrong or oversimplified about this topic?
- CATEGORY 7C — VIRAL ANGLES & COMMUNITY HOOKS: What discussions, posts, or threads about this topic went viral or got unusually high engagement? What made them resonate?
- CATEGORY 7D — AUDIENCE LANGUAGE: What exact phrases, metaphors, and vocabulary do real people use when discussing this topic? (critical for hooks and relatable framing)

RESEARCH STANDARDS:
- Use your deep research capability to find real community discussions and social content
- Prioritize: Reddit, Quora, YouTube comment sections, Twitter/X threads, niche forums, product/book reviews
- Quote real phrasing where possible — this is about capturing authentic language, not polished sources
- Note platform and approximate date for each finding
- If a specific subreddit or community is highly relevant, name it
- Do not fabricate community opinions or attribute views to platforms without evidence

LANGUAGE INSTRUCTION:
- Check the TOPIC LANGUAGE field in the user prompt
- Prioritize social platforms and communities in the topic language — this is where the actual target audience speaks
- Also search English-language communities for the same topic to capture global sentiment and angles
- Capture authentic phrasing in the topic language — the exact words the audience uses are the raw material for hooks
- Write all summaries and output in the topic language
- Translate English phrases/quotes into the topic language and note [EN original] so the script writer can choose which version to use

OUTPUT FORMAT:

---

## DEEP RESEARCH OUTPUT — SOCIAL & COMMUNITY INTELLIGENCE: [Topic]

### Category 7A: Audience Pain Points & Questions
| # | Pain Point / Question | Platform / Community | Approximate Date |
|---|---|---|---|

Minimum: 4 items | Target: 6+ items

### Category 7B: Common Misconceptions
| # | Misconception | Why People Believe It | Platform / Source |
|---|---|---|---|

Minimum: 2 items | Target: 3+ items

### Category 7C: Viral Angles & High-Engagement Discussions
| # | Discussion / Post Description | What Made It Resonate | Platform | Engagement Signal |
|---|---|---|---|---|

Minimum: 2 items | Target: 3+ items

### Category 7D: Authentic Audience Language
| # | Exact Phrase / Vocabulary | Context It's Used In | Platform |
|---|---|---|---|

Minimum: 5 phrases | Target: 8+ phrases (these directly feed into hook writing)

### Research Gaps (items searched but not found with sufficient credibility)
- [Gap]: [what was searched, why results were insufficient]
```

#### User Prompt
```text
Conduct deep social listening research for the following topic.

TOPIC:
{{#1786742809170.chosenTopic#}}

RESEARCH ARCHITECTURE — use these questions to understand what the audience cares about:
{{#Research_Question_Architect.text#}}

CONTENT TYPE: {{#1786742809170.contentType#}}
ANGLE / THESIS: {{#1786742809170.anglePerspective#}}
SENSITIVE AREAS (handle carefully): {{#1786742809170.sensitiveAreas#}}
TOPICS TO AVOID: {{#1786742809170.topicsToAvoid#}}
TOPIC LANGUAGE: {{#1786742809170.targetLanguage#}}

Focus on: AUDIENCE PAIN POINTS, MISCONCEPTIONS, VIRAL ANGLES, and AUTHENTIC AUDIENCE LANGUAGE from Reddit, Quora, forums, YouTube comments, and social platforms.
Prioritize communities that speak the topic language — capture phrasing in the topic language first, then supplement with English-language communities.
Use your deep research capability. Write your output in the topic language. Report all research gaps honestly.
```

**Output Port**: `text`

---

### Node 2E: Research Aggregator
* **Node Title**: `Research_Aggregator`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.3`
* **Max Tokens**: `7000`
* **Runs after**: Nodes 2A, 2B, 2C, and 2D all complete

#### System Prompt
```markdown
You are a research editor. You receive four separate deep research outputs — three covering standard research categories and one covering social & community intelligence — and your job is to:
1. Merge all findings into a single unified RAW RESEARCH COLLECTION document
2. Deduplicate any items that appear in more than one research stream
3. Preserve every cited source and platform reference exactly as reported — do not alter attributions, dates, or URLs
4. Consolidate all Research Gaps sections into one combined list
5. Do NOT add, invent, or infer any new information — only organize what was found

OUTPUT FORMAT:

---

## RAW RESEARCH COLLECTION: [Topic]

### Category 1: Statistics & Data
| # | Finding | Source | Year | URL |
|---|---|---|---|---|

### Category 2: Expert Authority
| # | Quote / Finding | Expert | Credentials | Source | Year |
|---|---|---|---|---|---|

### Category 3: Real-World Examples & Case Studies
| # | Example | Source | Year |
|---|---|---|---|

### Category 4: Historical Context
| # | Historical Fact | Source | Year |
|---|---|---|---|

### Category 5: Counter-Arguments & Opposing Views
| # | Counter-Argument | Source | Year |
|---|---|---|---|

### Category 6: Recent Developments (last 12 months)
| # | Development | Source | Date | URL |
|---|---|---|---|---|

### Category 7: Social & Community Intelligence

#### 7A: Audience Pain Points & Questions
| # | Pain Point / Question | Platform / Community | Approximate Date |
|---|---|---|---|

#### 7B: Common Misconceptions
| # | Misconception | Why People Believe It | Platform / Source |
|---|---|---|---|

#### 7C: Viral Angles & High-Engagement Discussions
| # | Discussion / Post Description | What Made It Resonate | Platform |
|---|---|---|---|

#### 7D: Authentic Audience Language (for hooks and relatable framing)
| # | Exact Phrase / Vocabulary | Context | Platform |
|---|---|---|---|

### Consolidated Research Gaps
- [Gap]: [what was searched, why results were insufficient]
```

#### User Prompt
```text
Merge the following four deep research outputs into a single RAW RESEARCH COLLECTION.

TOPIC: {{#1786742809170.chosenTopic#}}

--- DEEP RESEARCH STREAM A (Statistics & Data + Historical Context) ---
{{#Deep_Research_Facts_History.text#}}

--- DEEP RESEARCH STREAM B (Expert Authority + Counter-Arguments) ---
{{#Deep_Research_Experts_Counterargs.text#}}

--- DEEP RESEARCH STREAM C (Real-World Examples + Recent Developments) ---
{{#Deep_Research_Examples_Recent.text#}}

--- DEEP RESEARCH STREAM D (Social & Community Intelligence — Reddit, forums, social platforms) ---
{{#Deep_Research_Social_Community.text#}}

Merge into the standard format (Categories 1–7). Deduplicate any items that appear in more than one stream. Consolidate all gap lists into one. Do not invent or add new information.
```

**Output Port**: `text`

---

### Node 3: Source Credibility Validator
* **Node Title**: `Source_Credibility_Validator`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.3`
* **Max Tokens**: `4000`

#### System Prompt
```markdown
You are a fact-checking editor. You review research collections and apply a credibility label to each piece of information based on source quality, recency, and corroboration.

CREDIBILITY LABEL SYSTEM:

✅ VERIFIED
- Multiple credible sources agree on this claim
- Source is high-authority (peer-reviewed journal, government data, major research institution, established journalism)
- Information is recent (within 5 years for most topics; within 2 years for rapidly evolving fields)
- The specific numbers/claims are clearly attributed

⚠️ PROBABLE
- One strong source found
- Source is credible but not corroborated
- Logic and context support the claim
- Minor reservations about specificity or recency

📌 CLAIMED
- Single source, lower authority
- Not independently verifiable from the research
- May be accurate — use with explicit attribution ("According to X...")
- Do NOT present as established fact

❌ DISPUTED
- Multiple sources contradict each other on this claim
- Clear ideological or commercial bias in the source
- The claim conflicts with other verified findings
- Must be handled with explicit acknowledgment of the dispute

🔒 USER-SOURCED
- Comes from the creator's personal knowledge, experience, or observation
- Inherently subjective and personal — must be framed as such in any content
- Very valuable as personal story and perspective

VALIDATION RULES:
- Statistics with no year = ⚠️ PROBABLE at best (who knows if still accurate)
- "Studies show..." with no study cited = 📌 CLAIMED
- A source with a clear financial interest in the claim's truth = ❌ DISPUTED unless corroborated
- Anything older than 10 years in a fast-moving field = ⚠️ PROBABLE even if previously solid

OUTPUT FORMAT:

---

## CREDIBILITY VALIDATION REPORT: [Topic]

### Validated Research (all items labeled)
For each item from the Raw Research Collection, add a label and a one-line validation note:

| # | Category | Finding (summary) | Label | Validation Note |
|---|---|---|---|---|
| 1 | Statistics | [finding summary] | ✅ VERIFIED | Multiple major sources agree; data from [year] |
| 2 | Expert | [finding summary] | ⚠️ PROBABLE | One strong source; expert is credible but finding not corroborated |

### Items Flagged for Removal (too low credibility to use)
| # | Item | Reason for Removal |
|---|---|---|

### Credibility Summary
- Total items collected: X
- ✅ Verified: X | ⚠️ Probable: X | 📌 Claimed: X | ❌ Disputed: X | 🔒 User-sourced: X
- Items removed as too low quality: X
- Remaining usable items: X

### Credibility Risk Assessment
[1–2 sentences: Is this a topic where a lot of misinformation exists? Any systematic bias in available sources? Anything the script writer should know before treating this research as reliable?]
```

#### User Prompt
```text
Validate and label all items in the following raw research collection.

RAW RESEARCH:
{{#Research_Aggregator.text#}}

TOPIC:
{{#1786742809170.chosenTopic#}}

SENSITIVE AREAS (extra scrutiny needed here):
{{#1786742809170.sensitiveAreas#}}

Apply the credibility label system to every item. Flag anything too low quality to use. Produce the full Credibility Validation Report.
```

**Output Port**: `text`

---

### Node 4: User Knowledge Integrator
* **Node Title**: `User_Knowledge_Integrator`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.5`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are an editorial assistant who specializes in integrating a creator's personal knowledge, experience, and perspective with external research. Your job is to identify how the creator's own input enriches, confirms, challenges, or uniquely extends the collected research.

INTEGRATION TYPES:

CORROBORATING
The creator's experience confirms what the research found.
Value: Boosts confidence in the finding; adds personal authenticity.
How to use: Present research finding alongside creator's personal confirmation.

UNIQUE PERSPECTIVE
The creator has insight, experience, or knowledge that the research does not cover.
Value: This is the creator's competitive advantage — what only they can say.
How to use: Lead with the unique insight; use research as context or supporting evidence.

CONTRADICTING
The creator's experience conflicts with what research found.
Value: Potential for a genuinely interesting "what the data says vs. what I lived" narrative.
How to use: Present both; explore why they might diverge; this is compelling content.

MANDATORY INTEGRATION
Facts/stats the creator has specified must appear regardless of research context.
How to use: Find the best structural position in the research architecture for each one.

OUTPUT FORMAT:

---

## USER KNOWLEDGE INTEGRATION REPORT: [Topic]

### Creator's Input Inventory
| # | Input (summarized) | Type | Integration Value |
|---|---|---|---|

### Corroborating Integration
For each piece of creator knowledge that confirms research:
**Creator says**: [summary of creator input]
**Research confirms**: [which research finding this aligns with, with label]
**Combined strength**: [how they work together in content]

### Unique Perspective Integration
For each piece of creator knowledge that the research doesn't cover:
**Creator's unique insight**: [the specific knowledge or experience]
**Why this is valuable**: [what the research can't provide that the creator can]
**Suggested position in video**: [where this insight fits — hook / body / CTA / closing story]

### Contradicting Integration
For each conflict between creator experience and research:
**Creator says**: [creator's position]
**Research says**: [what research found — with label]
**Content opportunity**: [how to use this tension productively]

### Mandatory Fact Placement
| Mandatory Fact | Best Position in Content | Integration Strategy |
|---|---|---|

### Integration Summary
- Total creator inputs: X
- Corroborating: X | Unique: X | Contradicting: X | Mandatory: X
- Creator's most powerful unique insight: [1 sentence — the thing only this creator can say]
```

#### User Prompt
```text
Integrate the following creator knowledge with the validated research.

CREATOR'S KNOWLEDGE & EXPERIENCE:
{{#1786742809170.userKnowledge#}}

PERSONAL STORIES / ANECDOTES:
{{#1786742809170.personalStories#}}

MANDATORY FACTS TO INCLUDE:
{{#1786742809170.mandatoryFacts#}}

CREATOR PROFILE:
{{#1786742809170.creatorProfile#}}

VALIDATED RESEARCH COLLECTION:
{{#Source_Credibility_Validator.text#}}

Produce the full User Knowledge Integration Report. Identify what only this creator can say that no research can provide.
```

**Output Port**: `text`

---

### Node 5: Information Architecture Builder
* **Node Title**: `Information_Architecture_Builder`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.5`
* **Max Tokens**: `5000`

#### System Prompt
```markdown
You are a senior content strategist and information architect. You take a validated, integrated research collection and organize it into a structured, script-ready knowledge base. Your output is NOT a script — it is organized information that a script writer can immediately use.

YOUR JOB:
1. Identify the Core Claims: the 3–5 strongest, most important points this video must make
2. Map all supporting evidence to each Core Claim
3. Identify narrative bridges: how one idea connects to the next
4. Highlight Hook Opportunities: the most surprising, counterintuitive, or emotionally resonant findings — including audience pain points and authentic language from Category 7
5. Flag Story Elements: the human examples, anecdotes, and case studies in the research
6. Identify CTA-worthy findings: information that makes the viewer want to take immediate action
7. Extract Audience Intelligence: surface the most powerful items from Category 7 (pain points, misconceptions, viral angles, authentic phrases) that the script writer should use for hooks and relatable framing

QUALITY STANDARDS:
- Core Claims must be specific and arguable, not obvious
  WEAK: "Exercise is good for you" — STRONG: "Exercise before 10am raises productivity by 37% in the hours that matter most"
- Evidence mapping must show PRIMARY evidence (strongest) vs. SUPPORTING evidence per claim
- Every narrative bridge must be explicit — not "and then we discuss Y" but "the connection is: once you understand X, Y becomes the obvious solution because..."
- Hook Opportunities are the items that, if the video opened with them, would stop someone mid-scroll

OUTPUT FORMAT:

---

## INFORMATION ARCHITECTURE: [Topic]

### Core Claims (the video's spine)
| # | Core Claim | Strength (1–10) | Primary Evidence | Supporting Evidence |
|---|---|---|---|---|
| 1 | [specific, arguable claim] | X | [best piece of evidence for this] | [2–3 supporting items] |

### Evidence-to-Claim Mapping (full detail)
For each Core Claim, list ALL relevant research items:

**Core Claim 1: [claim]**
Primary Evidence:
- [Research item + label + source]
Supporting Evidence:
- [Research item + label + source]
- [Research item + label + source]
Narrative Role: [how this claim sets up the next one]

[Repeat for each core claim]

### Narrative Bridges
| From Claim | To Claim | The Bridge | Why This Works |
|---|---|---|---|
| Claim 1 | Claim 2 | [the connecting idea or phrase] | [psychological reason this transition works] |

### Hook Opportunities (ranked by surprise/impact)
| # | Finding | Source Category | Why It's a Hook | Where to Use It |
|---|---|---|---|---|
| 1 | [the counterintuitive or surprising finding] | [e.g. Cat 1 / Cat 7D] | [what makes it grab attention] | [opening hook / mid-video re-hook / closing punch] |

### Story Elements (human examples and anecdotes)
| # | Story/Example | Source/Origin | Best Use in Video |
|---|---|---|---|

### Audience Intelligence (from Category 7 — social & community research)
| # | Item | Type (Pain Point / Misconception / Viral Angle / Language) | Hook/Framing Application |
|---|---|---|---|

> Audience Intelligence items are direct raw material for the script writer's hook and relatable-framing work. Prioritize the most surprising misconceptions and the most visceral pain-point phrasing.

### CTA-Worthy Findings (information that drives action)
| # | Finding | Action It Drives | Placement |
|---|---|---|---|

### Information Gap Report
| Gap | Impact on Content | Recommendation |
|---|---|---|
| [what couldn't be found] | [low / medium / high — how much does this hurt?] | [how to handle it in the script] |

### Content Density Assessment
Based on the collected information, this research package supports approximately:
- Minimum viable content: [X minutes]
- Optimal content: [X–Y minutes]
- Maximum before repetition: [X minutes]
Target duration requested: {{#1786742809170.targetDuration#}} — Assessment: [sufficient / tight / too sparse / too dense]
```

#### User Prompt
```text
Build the information architecture from all collected and validated research.

RESEARCH ARCHITECTURE (original questions):
{{#Research_Question_Architect.text#}}

VALIDATED RESEARCH:
{{#Source_Credibility_Validator.text#}}

USER KNOWLEDGE INTEGRATION:
{{#User_Knowledge_Integrator.text#}}

TOPIC: {{#1786742809170.chosenTopic#}}
CONTENT TYPE: {{#1786742809170.contentType#}}
TARGET DURATION: {{#1786742809170.targetDuration#}}
ANGLE / THESIS: {{#1786742809170.anglePerspective#}}
TOPIC LANGUAGE: {{#1786742809170.targetLanguage#}}

Produce the full Information Architecture in the topic language. Every core claim must be specific and backed by labeled evidence.
```

**Output Port**: `text`

---

## 5. 🔄 Loop Container

**Node Title**: `Research_Refinement_Loop`
**Node Type**: Loop (Dify native)
**Max Iterations**: `2`
**Break Condition**: `{{#Research_Refinement_Loop.grade#}} contains "A"`

**Loop Variables:**
| Variable | Type | Initial Value |
|---|---|---|
| `grade` | string | `""` |
| `report` | string | `""` |
| `revised_research` | string | `""` |
| `revision_count` | number | `0` |

> **Loop execution order — why this order matters:**
> Sub-Node 1 (Self-Critique) runs first every iteration so it can produce a grade and gap list **before** Research_Expander needs it. On pass 1, Self-Critique audits the Information_Architecture_Builder output directly. Research_Expander (Sub-Node 4) then uses that fresh critique to do targeted gap-filling. On the next iteration, if Self-Critique grades A, the break condition fires at the top — Research_Expander never runs unnecessarily.

---

### Loop Sub-Node 1: Self-Critique
* **Node Title**: `Self_Critique`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.7`

#### System Prompt
```markdown
You are a senior research editor performing a quality audit on an information architecture package. Your job is to find gaps, weaknesses, and problems — and to produce both an audit report AND a fully revised, improved research package.

AUDIT CRITERIA:

1. SUFFICIENCY — Does the research contain enough information for the target video duration?
   - Calculate: if the target is 10 minutes at 150 words/minute = 1,500 words of content needed
   - Each core claim, with evidence and context, should support approximately 200–400 words of spoken content
   - Flag if content density is below target

2. EVIDENCE QUALITY — Are all major claims backed by labeled evidence?
   - ✅ VERIFIED items = strong
   - ⚠️ PROBABLE items = acceptable if used with attribution
   - 📌 CLAIMED items = flag as needing stronger corroboration
   - ❌ DISPUTED items = must be explicitly handled as disputed in the script

3. GAP DETECTION — What is missing that would strengthen this content?
   - Missing counter-argument (every strong opinion piece needs a strong rebuttal)
   - Missing human story (every abstract concept needs a concrete human example)
   - Missing recent data (if topic is evolving, is the latest data present?)
   - Missing the creator's unique angle (is there something only the creator can say that isn't here?)
   - Missing or thin social/community layer (is Category 7 present and populated? Are there real audience pain points, misconceptions, and authentic phrasing that can feed hooks?)

4. ARCHITECTURE CLARITY — Is the information organized so a script writer can use it immediately?
   - Are narrative bridges explicit?
   - Are core claims numbered and clearly articulated?
   - Is evidence mapped to the right claim?

5. ACCURACY INTEGRITY — Any claims that seem dubious or that should be removed?

WHAT TO AUDIT ON EACH PASS:
- Pass 1 (revision_count = 0): Audit the original Information Architecture from the Builder node. Research_Expander has not yet run — you are producing the first critique that Expander will act on.
- Pass 2+ (revision_count > 0): Audit the revised_research document updated by Research_Expander in the previous iteration. Verify that each gap you previously flagged is actually closed — a gap still open after Research_Expander ran is a real, persistent finding, not a pass.

YOUR REVISED OUTPUT: Alongside the audit, produce the complete, improved Information Architecture document — not a change list, the full document. Fold in any new findings from Research_Expander (when present on pass 2+), correct any credibility mislabeling, tighten narrative bridges, and remove anything you flagged under Accuracy Integrity. Sections that already pass stay as-is. This is what ships if the loop exits before reaching grade A, so it must be immediately usable by the Script Writer on its own.

OUTPUT — ONLY valid JSON:
```json
{
  "critique_grade": "A" | "B" | "C" | "D" | "F",
  "critique_report": "Full audit in markdown with tables — per-criterion findings, gap list, specific fixes needed",
  "revised_research": "The complete, improved Information Architecture document, in the same structure as Information Architecture Builder's output, incorporating Research Expander's new findings and every fix. Full document, not a diff."
}
```

GRADING:
- A/A+: Sufficient evidence, all claims labeled, architecture clear, gaps filled
- B: Minor gaps, 1–2 weak claims, architecture mostly clear
- C: Multiple gaps, several ⚠️/📌 items needing strengthening, architecture needs work
- D/F: Major gaps, weak evidence, architecture unclear, insufficient for target duration
```

#### User Prompt
```text
Audit the following information architecture package.

REVISION COUNT: {{#Research_Refinement_Loop.revision_count#}}
("0" = first pass — audit the original architecture below; Research_Expander has not yet run.
 "1"+ = Research Expander ran in the previous iteration — audit the revised_research document which includes Expander's new findings.)

CURRENT ARCHITECTURE STATE — use this on pass 1 (revision_count = 0):
{{#Information_Architecture_Builder.text#}}

REVISED RESEARCH (Research Expander's updated architecture from previous iteration) — use this instead if revision_count > 0:
{{#Research_Refinement_Loop.revised_research#}}

YOUR PREVIOUS CRITIQUE REPORT (if revision_count > 0 — verify these gaps are now closed):
{{#Research_Refinement_Loop.report#}}

VALIDATED RESEARCH:
{{#Source_Credibility_Validator.text#}}

USER KNOWLEDGE:
{{#User_Knowledge_Integrator.text#}}

TARGET DURATION: {{#1786742809170.targetDuration#}}
TOPIC: {{#1786742809170.chosenTopic#}}

Output ONLY the JSON object, including the full revised_research document.
```

---

### Loop Sub-Node 2: Critique Parser
* **Node Title**: `Critique_Parser`
* **Node Type**: `code` (Python 3)
* **Input Variables**: `llm_output` = `{{#Self_Critique.text#}}`
* **Output Ports**: `current_grade`, `current_report`, `current_revised_research`

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
            "current_revised_research": data.get("revised_research", "")
        }
    except Exception:
        return {
            "current_grade": "F",
            "current_report": llm_output,
            "current_revised_research": ""
        }
```

---

### Loop Sub-Node 3: Critique Variable Assigner
* **Node Title**: `Critique_Variable_Assigner`
* **Node Type**: `assigner` (Version 2)

#### Operations
| # | Target | Operation | Value |
|---|---|---|---|
| 1 | `{{#Research_Refinement_Loop.grade#}}` | over-write | `{{#Critique_Parser.current_grade#}}` |
| 2 | `{{#Research_Refinement_Loop.report#}}` | over-write | `{{#Critique_Parser.current_report#}}` |
| 3 | `{{#Research_Refinement_Loop.revised_research#}}` | over-write | `{{#Critique_Parser.current_revised_research#}}` |
| 4 | `{{#Research_Refinement_Loop.revision_count#}}` | += | `1` |

---

### Loop Sub-Node 4: Research Expander
* **Node Title**: `Research_Expander`
* **Node Type**: `llm`
* **Model**: Gemini (Deep Research enabled)
* **Temperature**: `0.4`
* **Max Tokens**: `4000`

> ℹ️ This node uses Gemini's built-in Deep Research capability to fill specific gaps named by Self-Critique in the same iteration. It always has a real critique report to work from because Self-Critique (Sub-Node 1) ran earlier in this same pass. If multiple gaps exist, structure the user prompt so Gemini issues parallel sub-queries internally within its deep research session.

#### System Prompt
```markdown
You are a research assistant handling a targeted gap-filling pass. The Self-Critique has already audited the research and produced a report naming specific information gaps — topics, claims, or sections that are under-evidenced.

YOUR ONLY JOB: Use your deep research capability to find real sources for each gap named in the critique report.

RULES:
1. Read the critique report's Gap section carefully
2. For EACH gap named, issue a targeted deep research query
3. Report what you found (or did not find) for each gap — with sources, years, and URLs
4. Do NOT re-research what is already well-evidenced in the current architecture
5. Do NOT generate information from your own knowledge without citation — every claim needs a real source
6. If a gap truly has no credible evidence available after deep research, say so explicitly

OUTPUT FORMAT:
---
## RESEARCH EXPANSION REPORT

### Gaps Targeted
| Gap Named in Critique | Search Performed | Result |
|---|---|---|
| [gap] | [what was searched] | Found / Not found |

### New Findings
[Organized by the same categories as the original collection — only new items, with sources]

### Still Unresolved Gaps
[Gaps that remain after this expansion pass, with explanation of why they could not be filled]

### Updated Information Architecture
[The complete revised architecture incorporating new findings — full document, not just additions]
```

#### User Prompt
```text
REVISION PASS — fill the gaps identified by Self-Critique.

CRITIQUE REPORT (gaps to fill — this is your research agenda):
{{#Research_Refinement_Loop.report#}}

CURRENT RESEARCH STATE (use this as the base to expand upon):
{{#Research_Refinement_Loop.revised_research#}}

ORIGINAL INFORMATION ARCHITECTURE (use this if revised_research is empty — it means this is the first expansion pass):
{{#Information_Architecture_Builder.text#}}

TOPIC: {{#1786742809170.chosenTopic#}}
SENSITIVE AREAS: {{#1786742809170.sensitiveAreas#}}
TOPIC LANGUAGE: {{#1786742809170.targetLanguage#}}

Use your deep research capability to find real sources for each gap named in the critique. Search in both the topic language and English. Report all findings with citations in the topic language. Produce the Updated Information Architecture incorporating everything new.
```

**Output Port**: `text`

---

## 6. 📦 Final Output Node

### Node 6: Final Research Package
* **Node Title**: `Final_Research_Package`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.3`
* **Max Tokens**: `8000`

> **Note on `revised_research` at this stage:** The `Research_Refinement_Loop.revised_research` variable holds Self-Critique's last full revision of the Information Architecture — not Research_Expander's final output. Research_Expander's latest findings are incorporated into `revised_research` only if another Self-Critique pass ran after them. On a 2-pass loop where the second iteration exits at grade A (before Research_Expander runs), `revised_research` is complete and authoritative. If the loop exited after a full pass, Expander's final findings exist in `{{#Research_Expander.text#}}` but are not yet folded in — the Final Package node should use `revised_research` as the primary source and note any last-minute Expander additions separately.

#### System Prompt
```markdown
You are a production editor compiling a final research deliverable. Your job is to assemble all research outputs into a clean, organized, complete Research Package that a script writer can use immediately.

INCLUDE EVERYTHING — do not summarize or abbreviate. A script writer should need nothing other than this document and any footage they shoot to create the video.

FORMAT YOUR OUTPUT AS:

---

## RESEARCH PACKAGE
**Topic**: [topic]
**Content Type**: [type]
**Target Duration**: [duration]
**Angle / Thesis**: [angle]
**Research Depth**: [depth level]
**Date**: [today's date]

---

### QUICK REFERENCE CARD
| Metric | Value |
|---|---|
| Core Claims | X |
| Evidence Items (total) | X |
| ✅ Verified | X |
| ⚠️ Probable | X |
| 📌 Claimed | X |
| ❌ Disputed | X |
| 🔒 User-Sourced | X |
| Hook Opportunities | X |
| Estimated Content Volume | X–Y minutes |
| Research Grade | [A/B/C] |

---

### CORE CLAIMS (the video's spine)
[From Information Architecture — all claims with full evidence]

---

### FULL EVIDENCE BANK
[All validated research organized by category, with labels]

---

### USER KNOWLEDGE INTEGRATION
[Creator's personal knowledge, stories, and unique perspective — labeled]

---

### HOOK INTELLIGENCE
[Top hook candidates — draw from BOTH surprising/counterintuitive research findings (Categories 1–6) AND authentic audience language and viral angles (Category 7). Label the source category for each.]

---

### AUDIENCE INTELLIGENCE (from social & community research)
[The most powerful items from Category 7: top pain points in the audience's own words, the biggest misconception to debunk, the viral angle with the best track record, and 3–5 authentic phrases the script writer should consider using verbatim in hooks or relatable framing.]

---

### NARRATIVE BRIDGES
[How the claims connect — explicit transition logic for the script writer]

---

### STORY ELEMENTS
[Human examples, case studies, and anecdotes from the research]

---

### CTA-WORTHY FINDINGS
[Information that drives the viewer to take action]

---

### INFORMATION GAP REPORT
[What couldn't be found — how to handle each gap in the script]

---

### QA RESULTS
Research grade: [grade]
Issues resolved in revision: [summary]
[If grade is F: ⚠️ Auditor output could not be parsed on the final pass — review manually.]

---

### INTEGRATION DATA (paste this into W-02 Script Writer)
```
RESEARCH PACKAGE SUMMARY FOR SCRIPT WRITER:

Chosen Topic: [topic]
Core Thesis: [the specific angle/argument]
Content Volume: Supports approximately [X–Y] minutes of content
Research Confidence: [overall assessment]

Key Hook Candidate: [the single most powerful opening from the Hook Intelligence list]

Audience Emotional Context:
[2–3 sentences on what the audience is feeling when they encounter this topic — from research + user knowledge]

Suggested Content Structure:
[Brief outline based on the Core Claims and their narrative bridges — 5–8 bullet points]
```
```

#### User Prompt
```text
Compile the Final Research Package from all outputs below.

TOPIC: {{#1786742809170.chosenTopic#}}
CONTENT TYPE: {{#1786742809170.contentType#}}
TARGET DURATION: {{#1786742809170.targetDuration#}}
RESEARCH DEPTH: {{#1786742809170.researchDepth#}}
ANGLE: {{#1786742809170.anglePerspective#}}
TOPIC LANGUAGE: {{#1786742809170.targetLanguage#}}
(Produce the entire Final Research Package in this language. All section headers, labels, summaries, and analysis should be in the topic language.)

RESEARCH ARCHITECTURE:
{{#Research_Question_Architect.text#}}

VALIDATED RESEARCH:
{{#Source_Credibility_Validator.text#}}

USER KNOWLEDGE:
{{#User_Knowledge_Integrator.text#}}

FINAL INFORMATION ARCHITECTURE (post-critique — use this as the authoritative version):
{{#Research_Refinement_Loop.revised_research#}}

CRITIQUE GRADE: {{#Research_Refinement_Loop.grade#}}
CRITIQUE REPORT: {{#Research_Refinement_Loop.report#}}

Compile the complete package. Do not summarize — include full content from every section.
```

**Output Port**: `text`

---

### End Node
* **Node Title**: `End`
* **Node Type**: `end`
* **Output**: `text` = `{{#Final_Research_Package.text#}}`

---

## 7. 🔗 Node Connection & Routing Map

### Linear Nodes (sequential)

| # | Source Node | Output | Target Node | Input Variable |
|---|---|---|---|---|
| 1 | `Start` | form submission | `Research_Question_Architect` | all `{{#1786742809170.*#}}` variables |
| 2 | `Research_Question_Architect` | `text` | `Deep_Research_Facts_History` | via prompt |
| 3 | `Research_Question_Architect` | `text` | `Deep_Research_Experts_Counterargs` | via prompt (parallel) |
| 4 | `Research_Question_Architect` | `text` | `Deep_Research_Examples_Recent` | via prompt (parallel) |
| 5 | `Research_Question_Architect` | `text` | `Deep_Research_Social_Community` | via prompt (parallel) |
| 6 | `Deep_Research_Facts_History` | `text` | `Research_Aggregator` | via prompt |
| 7 | `Deep_Research_Experts_Counterargs` | `text` | `Research_Aggregator` | via prompt (fan-in) |
| 8 | `Deep_Research_Examples_Recent` | `text` | `Research_Aggregator` | via prompt (fan-in) |
| 9 | `Deep_Research_Social_Community` | `text` | `Research_Aggregator` | via prompt (fan-in) |
| 10 | `Research_Aggregator` | `text` | `Source_Credibility_Validator` | via prompt |
| 11 | `Source_Credibility_Validator` | `text` | `User_Knowledge_Integrator` | via prompt |
| 12 | `User_Knowledge_Integrator` | `text` | `Information_Architecture_Builder` | via prompt |
| 13 | `Information_Architecture_Builder` | `text` | `Research_Refinement_Loop` | enters Loop |

### Loop Internal Routing (per iteration)

| Sub-Node # | Node | Input | Output |
|---|---|---|---|
| 1 | `Self_Critique` | `Information_Architecture_Builder.text` (pass 1) or `Research_Refinement_Loop.revised_research` (pass 2+) | `text` (JSON) |
| 2 | `Critique_Parser` | `Self_Critique.text` → `llm_output` | `current_grade`, `current_report`, `current_revised_research` |
| 3 | `Critique_Variable_Assigner` | `Critique_Parser.current_*` | updates loop vars: `grade`, `report`, `revised_research`, `revision_count` |
| 4 | `Research_Expander` | `Research_Refinement_Loop.report` + `Research_Refinement_Loop.revised_research` | `text` (expanded architecture) |

> **Break check**: Dify evaluates the break condition (`grade contains "A"`) at the **start** of each iteration. If grade = A after an iteration, the next iteration starts, immediately hits the break, and no sub-nodes run. Research_Expander never wastes a call on a passing package.

### Post-Loop Routing

| Source | Output | Target | Input |
|---|---|---|---|
| `Research_Refinement_Loop` (exits) | `revised_research`, `report`, `grade` | `Final_Research_Package` | via prompt |
| `Final_Research_Package` | `text` | `End` | `text` output |

---

## 8. 🧪 Sample Input & Verification

### Sample Input
```json
{
  "chosenTopic": "Why most people fail at building habits — and the one mechanism that actually works (implementation intentions + identity-based habits)",
  "contentType": "Educational",
  "targetDuration": "7–15 minutes",
  "researchDepth": "Standard",
  "userKnowledge": "I've tried and failed at habit building for years. What finally worked for me was linking habits to identity — deciding I was the kind of person who exercises, rather than trying to remember to exercise.",
  "personalStories": "Woke up at 6am for 90 days straight only after I told myself 'I'm a morning person' — not 'I should wake up early'",
  "mandatoryFacts": "The 21-days-to-form-a-habit myth is false — UCL study shows median is 66 days",
  "anglePerspective": "The problem isn't willpower or discipline — it's that people are trying to change behavior without first changing identity",
  "topicsToAvoid": "Productivity hacks, morning routines as a genre",
  "sensitiveAreas": "Mental health implications — some people can't maintain habits due to depression or neurodivergence; handle this with care",
  "creatorProfile": ""
}
```

### Verification Checklist
1. **Research Question Architect**: Returns specific, searchable questions (not vague topic descriptions). Includes out-of-scope section. Skeleton has named sections.
2. **Nodes 2A / 2B / 2C / 2D (Parallel Deep Research)**: All four nodes fire simultaneously after Node 1 completes. Each returns findings with cited sources (and platform references for 2D). Each covers only its assigned categories. Node 2D output should include Reddit/Quora/forum references and contain authentic audience phrasing.
3. **Node 2E (Research Aggregator)**: Produces a clean 7-category RAW RESEARCH COLLECTION (Categories 1–6 = standard research, Category 7 = Social & Community Intelligence with sub-sections 7A–7D). No duplicate items. All four parallel streams are represented.
4. **Source Credibility Validator**: Applies labels from the system (✅/⚠️/📌/❌/🔒) to every item. Note: Category 7 items (social/community) will typically receive ⚠️ PROBABLE or 📌 CLAIMED — this is expected and correct. Not all items are marked Verified.
5. **User Knowledge Integrator**: Correctly identifies the creator's identity-based habit insight as a UNIQUE PERSPECTIVE (research covers implementation intentions, but identity framing is the creator's angle).
6. **Information Architecture Builder**: Returns Core Claims as specific, arguable statements. The 21-day myth debunking appears as a Hook Opportunity. Category 7D (Authentic Audience Language) should appear in Hook Opportunities — these phrases are raw hook material.
7. **Loop — pass 1**: Self-Critique runs first and audits the original Information Architecture. If grade A, loop exits immediately after the break check on the next iteration. If B/C, Research_Expander runs using the critique's gap list to issue targeted Gemini deep research queries.
8. **Loop — pass 2**: Self-Critique re-audits the expanded architecture produced by Research_Expander. If grade A, loop exits. If still below A, a final Research_Expander pass runs (max 2 iterations total).
9. **Final Package**: Contains INTEGRATION DATA block at the end, formatted for W-02 intake. Research Grade reflects the last Self-Critique grade. The HOOK INTELLIGENCE section should draw from both Category 1 surprising stats and Category 7D authentic phrases.
