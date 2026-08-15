# 🎬 Video Production System — System Overview

> **Version**: 1.0 | **Status**: Phase 1 Active
> All nine workflow guides in this folder form one integrated system.
> Each can run independently or chain with others via standardized output packages.

---

## 📐 Complete System Map

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          CONTENT CREATION LAYER                              │
│                                                                              │
│  W-00              W-01              W-02              W-03                  │
│  Trend &     ──▶   Topic       ──▶   Script      ──▶   Production            │
│  Idea              Research &        Writer             Planning &            │
│  Research          Info              (Egyptian          Filming Guide         │
│  (+ W-09           Collector         Arabic layer)      (+ AI footage         │
│  feedback)         [web search                          prompt writer)        │
│                    required]                                                  │
└─────────────────────────────────────────────────┬────────────────────────────┘
                                                  │ raw footage
                                                  ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           VIDEO EDITING LAYER                                │
│                                                                              │
│  W-04                                                                        │
│  Video Pre-Planning Pipeline (UPGRADED)                                      │
│  storyboard + pacing + retention + thumbnail ideation + B-Roll Studio        │
│  (native Loop — reference implementation for all other workflows)            │
│                   │                                     │                    │
│                   ▼                                     ▼                    │
│  W-05                                    W-06                                │
│  Post-Production Execution               Viral Speed Ramp Pipeline           │
│  (native Loop rebuild, fail-to-F,        (native Loop rebuild, compound      │
│   rough cut gate, export presets)         loop-quality gate, fail-to-F)      │
└──────────────────────────────────────────────────────────────────────────────┘
                                                  │
┌──────────────────────────────────────────────────────────────────────────────┐
│                           PUBLISHING LAYER                                   │
│                                                                              │
│  W-07  SEO & Publishing Optimizer (+ Rights & Compliance Check)              │
│  W-08  Content Repurposing Agent                                             │
└──────────────────────────────────────────────────────────────────────────────┘
                                                  │ after publishing
┌──────────────────────────────────────────────────────────────────────────────┐
│                            FEEDBACK LAYER                                    │
│                                                                              │
│  W-09  Performance Analyzer                                                  │
│  real analytics in → prediction-vs-reality lessons → feeds W-00 + W-02      │
└──────────────────────────────────────────────────────────────────────────────┘

Creator Profile (user-maintained, outside any workflow) — pastes into every Start Node.
```

---

## 🚪 Entry Points — Where You Can Start

| You Have | Start At | Skip |
|---|---|---|
| Nothing — just an interest or niche | W-00 | — |
| An idea, no research yet | W-01 | W-00 |
| Research notes but no script | W-02 | W-00, W-01 |
| A script but no footage yet | W-03 | W-00–W-02 |
| A script + raw footage ready | W-04 | W-00–W-03 |
| Raw footage only (no script) | W-04 | W-00–W-03 |
| An edited rough cut needing polish | W-05 | W-00–W-04 |
| Short clips for viral content | W-06 | W-00–W-04 |
| A finished video needing publishing | W-07 | W-00–W-06 |
| A published video and its analytics | W-09 | W-00–W-08 |

---

## 🧱 System-Wide Standards

Every workflow in this system follows these five standards. Read them once and apply to all guides.

---

### Standard 1 — Loop Architecture: Dify Native Loop Container

Every self-critique / revision cycle uses Dify's native **Loop** container node.
No workflow uses hand-rolled IF/ELSE back-edges or Python string counters.

**Loop container setup:**
- **Type**: Loop node (Dify native)
- **Loop variables** (declared inside the container):
  - `grade` (string, initial: `""`)
  - `report` (string, initial: `""`)
  - One `revised_<artifact>` per artifact being refined (string, initial: `""`)
  - `revision_count` (number, initial: `0`)
- **Break condition**: `{{#Loop.grade#}} contains "A"`
- **Max iterations**: `2`
- **Error handle mode**: `terminated`

**Sub-nodes inside every Loop container, in this order:**
```
[Content-generating node(s)]
         ↓
[Self-Critique LLM]
         ↓
[Critique Parser Code]
         ↓
[Critique Variable Assigner]
```

**Content-generating nodes** read `{{#Loop.revision_count#}}` and `{{#Loop.report#}}` /
`{{#Loop.revised_<artifact>#}}` to distinguish first pass from revision pass.
On revision pass (revision_count > 0), they apply the fixes listed in `report` and start
from the prior `revised_<artifact>` — the "MODE: REVISION PASS" pattern.

**Critique Variable Assigner** — always these operations:
1. Target: `{{#Loop.grade#}}` | Operation: `over-write` | Value: `{{#Critique_Parser.current_grade#}}`
2. Target: `{{#Loop.report#}}` | Operation: `over-write` | Value: `{{#Critique_Parser.current_report#}}`
3. Target: `{{#Loop.revised_<artifact>#}}` | Operation: `over-write` | Value: `{{#Critique_Parser.current_revised_<artifact>#}}`
4. Target: `{{#Loop.revision_count#}}` | Operation: `+=` | Value: `1`

**W-04** already implements this correctly — it is the reference pattern.
**W-05 and W-06** have been rebuilt to match.

---

### Standard 2 — Critique Parser Schema

**Self-Critique (LLM) must output ONLY valid JSON with these keys:**
```json
{
  "critique_grade": "A+" | "A" | "B" | "C" | "D" | "F",
  "critique_report": "full markdown audit text",
  "revised_<artifact>": "full revised document — not just changes"
}
```
Workflow-specific additions:
- W-06 adds: `"loop_score": 7` (integer 1–10) and `"viral_score": 8` (integer 1–10)
- W-02 adds: `"dialect_authenticity_score": 9` (integer 1–10, Arabic mode only)

**Critique Parser (Code node) outputs — matching Loop variable names:**
- `current_grade` (string)
- `current_report` (string)
- `current_revised_<artifact>` (string, one per artifact)

**On any parse failure: return `current_grade = "F"`. Never `"A"`. Never `"B"`.**

**Canonical Critique Parser Python template:**
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
            # Add: "current_revised_<artifact>": data.get("revised_<artifact>", "")
        }
    except Exception:
        return {
            "current_grade": "F",
            "current_report": llm_output,
            # Add: "current_revised_<artifact>": ""
        }
```

When the loop escapes on grade `F`, the Final Package must include:
> ⚠️ Auditor output could not be parsed on the final pass — review this section manually.

---

### Standard 3 — Node Reference Style

All new and rebuilt guides reference upstream nodes by **title**:
```
{{#Node_Title.output_field#}}
```
Example: `{{#Self_Critique.text#}}`, `{{#Critique_Parser.current_grade#}}`

Do NOT use numeric IDs in new guides. Numeric IDs (W-04's legacy style) are auto-generated
per Dify environment and can't be reproduced from a guide before the node is built.

---

### Standard 4 — Model Tier

| Tier | Use For | Temperature |
|---|---|---|
| **Best available** | Creative judgment, quality gating (Self-Critique every workflow, Hook Writer, Script Body Writer, Idea Generator, Dialect Layer) | 0.7–0.9 |
| **Standard** | Structured planning following clear rules (Storyboard Builder, Shot List, Creative Strategy, most first-pass nodes) | 0.5–0.7 |
| **Cheap / fast** | Formatting, extraction, compilation (Tags, Cross-Platform Plan, Final Package compilers) | 0.3–0.5 |

**Under-provisioning Self-Critique is the one substitution that quietly destroys the system.**
That node is where the entire quality story rests.

---

### Standard 5 — Creator Profile

Every Start Node has a `creatorProfile` (paragraph, optional) field.
User maintains one document and pastes it each run.

**Creator Profile template:**
```
CREATOR PROFILE
- Channel / brand name & niche:
- Target audience (who they are, why they watch):
- Voice & tone (3–5 adjectives + what to avoid):
- Recurring phrases / catchphrases / sign-offs:
- Dialect / language register:
- Topics or angles permanently off-limits:
- Reference scripts / videos that nailed the voice:
- Visual brand (colors, fonts, logo rules):
```

---

## 📋 File Index

| File | Workflow | Layer | Phase |
|---|---|---|---|
| `SYSTEM_OVERVIEW.md` | This document | — | 1 |
| `W-00_Trend_and_Idea_Research.md` | Trend & Idea Research Agent | Content Creation | 1 |
| `W-01_Topic_Research_Info_Collector.md` | Topic Research & Info Collector | Content Creation | 1 |
| `W-02_Script_Writer.md` | Script Writer Agent | Content Creation | 1 |
| `W-03_Production_Planning_Filming_Guide.md` | Production Planning & Filming Guide | Content Creation | 1 |
| `W-04_Video_Pre-Planning_Pipeline.md` | Video Pre-Planning Pipeline | Editing | 1 |
| `W-05_Post_Production_Execution_Flow.md` | Post-Production Execution Flow | Editing | 1 |
| `W-06_Viral_Speed_Ramp_Pipeline.md` | Viral Speed Ramp Pipeline | Editing | 1 |
| `W-07_SEO_Publishing_Optimizer.md` | SEO & Publishing Optimizer | Publishing | 2 |
| `W-08_Content_Repurposing_Agent.md` | Content Repurposing Agent | Publishing | 2 |
| `W-09_Performance_Analyzer.md` | Performance Analyzer | Feedback | 2 |

---

## 🔗 Integration Data Flow

```
W-00 → W-01 : chosen_idea, key_research_questions, audience_psychology
W-01 → W-02 : researchPackage (full research document)
W-02 → W-03 : scriptPackage, shot_list_extracted, filming_requirements
W-02 → W-04 : scriptPackage (optional — enables script-aware storyboarding)
W-03 → W-04 : preplanning_package (after filming completes)
W-03 → W-05 : preplanning_package (standalone post-production path)
W-04 → W-05 : preplanningPackage (storyboard + pacing + retention + B-roll concepts)
W-04 → W-06 : preplanningPackage (viral path)
W-05 → W-07 : finished video + execution metadata
W-06 → W-07 : finished video + viral metadata
W-07 → —    : publishing complete
W-08 → —    : repurposing calendar
(after publishing) → W-09 : analyticsData
W-09 → W-00 : pastPerformanceNotes (feed-forward)
W-09 → W-02 : pastPerformanceNotes (script calibration)
```

---

## ⚙️ Dify Implementation Notes

### Building Workflows in Order
Build in this recommended order — each workflow's output format becomes the next one's expected input:
1. W-04 (already built — use as Loop reference)
2. W-00 (standalone, no dependencies)
3. W-01 (depends on W-00 format understanding)
4. W-02 (depends on W-01 output format)
5. W-03 (depends on W-02 output format)
6. W-05 (depends on W-04 output format)
7. W-06 (depends on W-04 output format)

### The "Paste Package" Pattern
Every workflow's first large `paragraph` input field accepts:
- The previous workflow's full output package pasted directly, OR
- Fresh user input describing the same information from scratch

This is the key integration mechanism — no API-to-API connection required between workflows.

### Loop Node Setup in Dify UI
1. Add a **Loop** node to the canvas
2. Enter the loop, add sub-nodes inside
3. Set loop variables in the Loop node's settings panel
4. Set break condition: type `contains` and value `A` for the `grade` variable
5. Set max iterations to `2`
6. The Assigner node goes last inside the Loop, AFTER Critique Parser
