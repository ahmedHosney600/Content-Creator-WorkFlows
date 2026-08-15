# 🎬 Video Pre-Planning Pipeline — Complete Dify Workflow Guide

This guide provides the complete, authoritative, and 100% production-ready specification for the **Video Pre-Planning Pipeline** (`00 - Video Pre-Planning Pipeline.yml`) in **Dify Workflow** (v0.7.0+ / v1.0+ DAG engine).

---

## 1. 🏗️ Pipeline Architecture & Execution Flow

### Architecture Highlights
* **App Type**: Dify **Workflow** (DAG process automation).
* **Native Dify Loop Engine**: Utilizes Dify's native `loop` container node (`Loop Count Guard`) with inner sub-nodes (`loop-start` ➔ `Storyboard Builder` ➔ `Pacing & Rhythm` ➔ `Self-Critique` ➔ `Critique Parser` ➔ `Critique Variable Assigner`).
* **State Management**: Built entirely on Dify's direct upstream node output referencing (`{{#Node_ID.output#}}`) and local Loop Variables (`grade`, `report`, `revised_storyboard`, `revised_pacing`, `revision_count`).
* **Autonomous Self-Critique & Revision Loop**: Automatically audits storyboard and pacing against strict cinematographic and retention benchmarks. If the grade contains `"A"`, the loop breaks immediately; otherwise, it executes up to **2 revision passes** before progressing.
* **Structured JSON Parsing**: A Python Code Node (`Critique Parser`) strips markdown code fences, extracts structured feedback, and handles JSON parsing gracefully to guarantee seamless pipeline execution.

---

### ASCII Execution Graph

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          1. START NODE (Intake)                         │
│             User Input (27 Video Production Parameters)                 │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      2. BRIEF BUILDER (LLM Node)                        │
│          Normalizes & infers defaults into a 5-part Project Brief       │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    3. CREATIVE STRATEGY (LLM Node)                      │
│       Translates brief into editing style, visual mood & music arc      │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   4. NARRATIVE STRUCTURE (LLM Node)                     │
│         Selects framework, designs acts, hooks, and open loops          │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      5. RETENTION MAP (LLM Node)                        │
│    Engineers 3-second rule, pattern interrupts & dopamine rhythms       │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │
                                     ▼
╔═════════════════════════════════════════════════════════════════════════╗
║          6. LOOP COUNT GUARD (Native Dify Loop Container Node)          ║
║          Max Iterations: 2 | Break Condition: grade contains "A"        ║
║                                                                         ║
║   ┌─────────────────────────────────────────────────────────────────┐   ║
║   │                       [Loop-Start Trigger]                      │   ║
║   └────────────────────────────────┬────────────────────────────────┘   ║
║                                    │                                    ║
║                                    ▼                                    ║
║   ┌─────────────────────────────────────────────────────────────────┐   ║
║   │               Storyboard Builder (LLM Sub-Node)                 │   ║
║   │        Generates / revises shot-by-shot visual storyboard       │   ║
║   └────────────────────────────────┬────────────────────────────────┘   ║
║                                    │                                    ║
║                                    ▼                                    ║
║   ┌─────────────────────────────────────────────────────────────────┐   ║
║   │                Pacing & Rhythm (LLM Sub-Node)                   │   ║
║   │       Aligns cuts, quiet-loud patterns, silence & music beats   │   ║
║   └────────────────────────────────┬────────────────────────────────┘   ║
║                                    │                                    ║
║                                    ▼                                    ║
║   ┌─────────────────────────────────────────────────────────────────┐   ║
║   │                  Self-Critique (LLM Sub-Node)                   │   ║
║   │         Audits shots, timing, audio & returns JSON grade        │   ║
║   └────────────────────────────────┬────────────────────────────────┘   ║
║                                    │                                    ║
║                                    ▼                                    ║
║   ┌─────────────────────────────────────────────────────────────────┐   ║
║   │              Critique Parser (Python Code Sub-Node)             │   ║
║   │           Parses JSON safely & extracts grade / report          │   ║
║   └────────────────────────────────┬────────────────────────────────┘   ║
║                                    │                                    ║
║                                    ▼                                    ║
║   ┌─────────────────────────────────────────────────────────────────┐   ║
║   │         Critique Variable Assigner (Assigner Sub-Node)          │   ║
║   │   Updates loop variables (grade, report, count) & loops/breaks  │   ║
║   └─────────────────────────────────────────────────────────────────┘   ║
╚════════════════════════════════════┬════════════════════════════════════╝
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    7. QA & FINAL PACKAGE (LLM Node)                     │
│    Executes 8-point QA check & compiles 10-part Editor Deliverable      │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          8. OUTPUT (End Node)                           │
│                Delivers final compiled package & JSON export            │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 📝 Start Node Configuration

* **Node Title**: `User Input`
* **Node ID**: `1786742809170`
* **Node Type**: `start`

Configure the 27 input variables in the Start node:

| # | Variable Name | Type | UI Label | Options / Allowed Values | Default Value | Required |
|---|---|---|---|---|---|---|
| 1 | `videoTopic` | `paragraph` | What is the video about? | — | `Formula 1 Racing - The Battle of Speed, Courage and Survival. A high-stakes commercial showcasing the extreme mental and physical warfare drivers face at 350 km/h.` | **Yes** |
| 2 | `primaryGoal` | `select` | Primary goal | `Sell a product`, `Educate`, `Entertain`, `Brand awareness`, `Tell a story`, `Document an event`, `Inspire action`, `Go viral`, `Other` | `Brand awareness` | **Yes** |
| 3 | `coreMessage` | `text-input` | Core message (one sentence) | — | `Formula 1 is not just driving; it is an uncompromising battle of human limits and precision where 20 drivers risk everything.` | **Yes** |
| 4 | `contentType` | `select` | Content type | `Commercial/Ad `, `YouTube video`, `Short-form Reel/TikTok`, `Corporate/Brand video`, `Documentary`, `Music video`, `Event highlight`, `Product showcase`, `Tutorial`, `Talking head`, `Showreel`, `Short film`, `Real estate`, `Mixed media`, `Motion graphics`, `Other` | `Commercial/Ad ` | **Yes** |
| 5 | `targetDuration` | `text-input` | Target duration (X seconds/minutes/hours - Flexible) | — | `60 seconds` | **Yes** |
| 6 | `targetAudience` | `text-input` | Target audience description | — | `Motorsport fans, adrenaline seekers, sports apparel consumers aged 18-35.` | **Yes** |
| 7 | `primaryPlatform` | `select` | Primary platform | `YouTube`, `TikTok`, `Instagram Reels`, `Instagram Feed`, `LinkedIn`, `Facebook TV`, `Broadcast`, `Cinema`, `Website`, `Presentation`, `Other` | `YouTube` | **Yes** |
| 8 | `secondaryPlatform` | `select` | Secondary platform (if any) | `Unspecified`, `None`, `YouTube`, `TikTok`, `Instagram Reels`, `Instagram Feed`, `LinkedIn`, `Facebook`, `TV`, `Broadcast`, `Website`, `Other` | `Unspecified` | No |
| 9 | `discoveryMethod` | `select` | How will viewers discover this? | `Unspecified`, `Organic search`, `Paid ads`, `Social feed scroll`, `Direct link`, `Embedded on website`, `TV broadcast`, `Mixed` | `Unspecified` | No |
| 10 | `soundAssumption` | `select` | Will viewers watch with sound? | `Unspecified`, `Mostly sound ON`, `Mostly sound OFF`, `50/50 split` | `Unspecified` | No |
| 11 | `viewerMindset` | `select` | Viewer mindset | `Unspecified`, `Actively searching`, `Passively scrolling`, `In a meeting/presentation`, `At an event`, `Relaxing at home`, `Mixed` | `Unspecified` | No |
| 12 | `scriptStatus` | `select` | Script / voiceover status | `Unspecified`, `Full script ready`, `Outline/bullet points`, `No script yet`, `Will improvise`, `Voiceover will be recorded` | `Unspecified` | No |
| 13 | `visualAssets` | `select` | Visual assets available | `Unspecified`, `Original footage (already filmed)`, `Will shoot original footage`, `Stock footage only`, `Both original + stock`, `Product photos only`, `Screen recordings`, `Graphics/animation`, `None yet` | `Unspecified` | No |
| 14 | `musicStatus` | `select` | Specific track selected | `Unspecified`, `Specific track selected`, `Genre preference`, `Mood preference only`, `No preference`, `Will be composed` | `Unspecified` | No |
| 15 | `musicDetails` | `text-input` | Music details (if any) | — | `Unspecified` | No |
| 16 | `brandGuidelines` | `select` | Brand guidelines? | `Unspecified`, `Yes - strict guidelines`, `Yes - flexible guidelines`, `No brand guidelines`, `Personal brand` | `Unspecified` | No |
| 17 | `targetEmotions` | `text-input` | Target emotions (pick up to 3) | — | `Unspecified` | No |
| 18 | `energyVibe` | `select` | Desired energy / vibe | `Unspecified`, `High energy`, `fast-paced `, `Cinematic`, `premium`, `Calm`, `elegant`, `Raw`, `authentic`, `Playful`, `fun`, `Dark`, `moody`, `Corporate`, `professional`, `Mixed` | `Unspecified` | No |
| 19 | `referenceVideos` | `text-input` | Reference videos (if any) | — | `Unspecified` | No |
| 20 | `stylesToAvoid` | `text-input` | Styles to AVOID | — | `Unspecified` | No |
| 21 | `narrativePreference` | `select` | Narrative structure preference | `Unspecified`, `Linear story`, `Before/after `, `Problem to solution / Montage`, `Interview-based`, `Day-in-the-life`, `Testimonial`, `Abstract/artistic`, `No preference` | `Unspecified` | No |
| 22 | `mandatoryElements` | `text-input` | Mandatory elements | — | `Unspecified` | No |
| 23 | `sensitiveConsiderations` | `text-input` | Sensitive considerations | — | `Unspecified` | No |
| 24 | `qualityTier` | `select` | Quality tier | `Unspecified`, `Premium`, `cinematic`, `Professional`, `polished`, `Good`, `clean`, `Raw`, `authentic`, `Budget-friendly` | `Unspecified` | No |
| 25 | `subtitlesNeeded` | `select` | Subtitles needed? | `Unspecified`, `Yes - hardcoded`, `Yes - optional/CC`, `No` | `Unspecified` | No |
| 26 | `seriesStatus` | `select` | Standalone or series? | `Unspecified`, `None`, `Standalone video`, `Part of a series`, `First in a new series` | `Unspecified` | No |
| 27 | `seriesDetails` | `text-input` | Series details (if applicable) | — | `Unspecified` | No |

---

## 3. ⚙️ Step-by-Step Node Specification

---

### Node 1: Brief Builder
* **Node ID**: `1786746678385`
* **Node Title**: `Brief Builder`
* **Node Type**: `llm`
* **Model Configuration**:
  * **Provider**: `OpenAI-API-compatible` (or OpenAI / Anthropic / Gemini)
  * **Model**: `gemini-advanced` (or `gemini-2.0-flash` / `gpt-4o`)
  * **Temperature**: `0.9`

#### System Prompt
```markdown
You are a professional video production coordinator. Your job is to take raw project intake data and compile it into a clear, structured Project Brief that will guide all subsequent creative and technical decisions.


=== ABSOLUTE RULE: HANDLING MISSING OR EMPTY FIELDS ===
This is your MOST IMPORTANT instruction. Many input fields are OPTIONAL. The user may leave them blank or fill them with vague text.


When you encounter ANY of these cases, you MUST follow these rules WITHOUT EXCEPTION:
1. NEVER create a 'System Flag' section or warning about missing fields
2. NEVER say 'Unspecified' or 'N/A' as a final answer — always infer a professional default
3. Instead, SILENTLY replace the missing value with a smart, professional default based on context
4. Mark auto-inferred values with (Auto) so the user can review and change them


HOW TO DETECT A MISSING FIELD:
A field is considered MISSING if its value:
- Is empty, blank, or whitespace only
- Equals 'Unspecified' or 'None' or 'N/A'
- Is clearly not a real answer (e.g., a single letter or placeholder text)


SMART DEFAULT INFERENCE RULES:
When a field is missing, apply these professional defaults:

- Secondary Platform: Infer from primary platform (Instagram Reels → TikTok, YouTube → Instagram Reels, TikTok → Instagram Reels, otherwise → None)
- Discovery Method: Infer from platform (social platforms → Social feed scroll, YouTube → Organic search, Website → Embedded on website, otherwise → Mixed)
- Sound Assumption: Infer from platform (TikTok → Mostly sound ON, Instagram Reels → 50/50 split, LinkedIn → Mostly sound OFF, YouTube/TV → Mostly sound ON)
- Viewer Mindset: Infer from platform + content type (social short-form → Passively scrolling, YouTube → Actively searching, corporate → In a meeting/presentation)
- Script Status: Default to 'No script yet'
- Visual Assets: Default to 'Will shoot original footage'
- Music Status: Default to 'Mood preference only'
- Music Details: Infer mood/genre from energy_vibe and content_type
- Brand Guidelines: Default to 'No brand guidelines'
- Target Emotions: Infer 2-3 emotions from topic, goal, and vibe (e.g., Brand awareness + High energy → Excitement, curiosity, delight)
- Energy/Vibe: Infer from content_type and platform
- Reference Videos: Default to 'None provided'
- Styles to Avoid: Default to 'None specified'
- Narrative Preference: Infer from content type (e.g., Commercial → Problem to solution, Short-form Reel → Montage)
- Mandatory Elements: Default to 'None'
- Sensitive Considerations: Default to 'None'
- Quality Tier: Default to 'Professional / polished'
- Subtitles: Infer from platform (Instagram/TikTok/LinkedIn → Yes - hardcoded, YouTube → Yes - optional/CC, TV/Cinema → No)
- Series Status: Default to 'Standalone video'
- Series Details: Default to 'N/A'


EXAMPLE OF CORRECT BEHAVIOR:
- If Secondary Platform is left blank and Primary Platform is 'Instagram Reels':
  WRONG: 'Secondary Platform: N/A'
  RIGHT: 'Secondary Platform: TikTok (Auto)'


You will receive form input data with the following fields:
- video_topic, primary_goal, core_message, content_type, target_duration
- target_audience, primary_platform, secondary_platform, discovery_method, sound_assumption, viewer_mindset
- script_status, visual_assets, music_status, music_details, brand_guidelines
- target_emotions, energy_vibe, reference_videos, styles_to_avoid, narrative_preference
- mandatory_elements, sensitive_considerations, quality_tier, subtitles_needed, series_status, series_details


Compile this data into a structured PROJECT BRIEF using EXACTLY this format:


---


# PROJECT BRIEF


## Identity
- **Working Title**: [Create a concise working title based on the topic]
- **Content Type**: [content_type value]
- **Target Duration**: [target_duration value]
- **Core Message**: [core_message value]
- **Primary Goal**: [primary_goal value]


## Audience & Distribution
- **Target Audience**: [target_audience — expand into a vivid one-paragraph persona]
- **Primary Platform**: [primary_platform value]
- **Secondary Platform**: [value or smart default (Auto)]
- **Discovery Method**: [value or smart default (Auto)]
- **Sound Assumption**: [value or smart default (Auto)]
- **Viewer Mindset**: [value or smart default (Auto)]


## Assets & Resources
- **Script Status**: [value or smart default (Auto)]
- **Visual Assets**: [value or smart default (Auto)]
- **Music Direction**: [music_status + music_details combined, or smart default (Auto)]
- **Brand Guidelines**: [value or smart default (Auto)]


## Creative Direction
- **Target Emotions**: [value or smart default (Auto)]
- **Energy/Vibe**: [value or smart default (Auto)]
- **References**: [value or 'None provided']
- **Styles to Avoid**: [value or 'None specified']
- **Narrative Preference**: [value or smart default (Auto)]


## Constraints
- **Mandatory Elements**: [value or 'None']
- **Sensitive Considerations**: [value or 'None']
- **Quality Tier**: [value or 'Professional / polished (Auto)']
- **Subtitles**: [value or smart default (Auto)]
- **Series Status**: [series_status + series_details, or 'Standalone video (Auto)']


## Derived Insights
Based on the above information, provide:
- **Platform-Specific Note**: [One paragraph on what this platform demands — hook window, pacing expectations, sound behavior]
- **Audience Behavior Note**: [One paragraph on how this audience typically consumes content]
- **Key Creative Challenge**: [The single biggest challenge in making this video succeed]
- **Success Metric**: [What does 'success' look like for this specific video?]


---


FINAL RULES:
- NEVER generate a 'System Flag' section. There should be ZERO warnings about missing data in your output.
- Do NOT make creative decisions — that is for the next phase. But DO fill in missing logistical/contextual fields with professional defaults.
- DO expand vague answers into clearer descriptions where possible.
- DO mark any auto-inferred values with (Auto) so the user can review them.
- The brief must be comprehensive enough that someone with NO context could understand the full project scope.
```

#### User Prompt
```text
Here is the project intake data from the form submission:

Video Topic: {{#1786742809170.videoTopic#}}
Primary Goal: {{#1786742809170.primaryGoal#}}
Core Message: {{#1786742809170.coreMessage#}}
Content Type: {{#1786742809170.contentType#}}
Target Duration: {{#1786742809170.targetDuration#}}

Target Audience: {{#1786742809170.targetAudience#}}
Primary Platform: {{#1786742809170.primaryPlatform#}}
Secondary Platform: {{#1786742809170.secondaryPlatform#}}
Discovery Method: {{#1786742809170.discoveryMethod#}}
Sound Assumption: {{#1786742809170.soundAssumption#}}
Viewer Mindset: {{#1786742809170.viewerMindset#}}

Script Status: {{#1786742809170.scriptStatus#}}
Visual Assets: {{#1786742809170.visualAssets#}}
Music Status: {{#1786742809170.musicStatus#}}
Music Details: {{#1786742809170.musicDetails#}}
Brand Guidelines: {{#1786742809170.brandGuidelines#}}

Target Emotions: {{#1786742809170.targetEmotions#}}
Energy/Vibe: {{#1786742809170.energyVibe#}}
Reference Videos: {{#1786742809170.referenceVideos#}}
Styles to Avoid: {{#1786742809170.stylesToAvoid#}}
Narrative Preference: {{#1786742809170.narrativePreference#}}

Mandatory Elements: {{#1786742809170.mandatoryElements#}}
Sensitive Considerations: {{#1786742809170.sensitiveConsiderations#}}
Quality Tier: {{#1786742809170.qualityTier#}}
Subtitles Needed: {{#1786742809170.subtitlesNeeded#}}
Series Status: {{#1786742809170.seriesStatus#}}
Series Details: {{#1786742809170.seriesDetails#}}

Compile this into a structured Project Brief.
```

---

### Node 2: Creative Strategy
* **Node ID**: `1786752482909`
* **Node Title**: `Creative Strategy`
* **Node Type**: `llm`
* **Model Configuration**:
  * **Provider**: `OpenAI-API-compatible`
  * **Model**: `gemini-advanced`
  * **Temperature**: `0.7`

#### System Prompt
```markdown
You are a world-class creative director and senior video editor. Your task is to analyze a project brief and produce a comprehensive creative strategy that will guide all editing decisions.



You have deep expertise in these editing styles and their rules:



EDITING STYLE REFERENCE:

- Cinematic/Film: Longer shots, motivated camera movement, dramatic lighting, orchestral or atmospheric music, invisible edits, emphasis on composition

- Commercial/Ad: Hook-first, message-clear, fast-paced with breathing room, "quiet-loud" pattern, strong CTA, match cuts, cross-cutting

- Corporate/Brand: Professional, clean, structured, logo integration, authoritative tone, moderate pacing

- Social/Short-form: Instant hook (0.5-1s), ultra-fast pacing, text overlays, trending sounds, loop-friendly, vertical format

- Documentary: Interview-driven, B-roll storytelling, natural pacing, ambient sound priority, trust-building visuals

- Montage/Showreel: Beat-synced cuts, variety of angles, energy escalation, portfolio-style

- Mixed Media: Live action + motion graphics + stock, layered visual approach, transitions between media types

- YouTube Content: Chapter-based structure, personality-driven, B-roll variety, retention hooks every 30s, pattern interrupts

- Talking Head: Jump cuts for energy, B-roll inserts for visual relief, text reinforcement

- Event/Highlight: Chronological or curated best moments, ambient sound + music, establishing shots, emotional peaks

- Real Estate/Property: Smooth camera movement, wide establishing shots, detail shots, natural lighting, flow-through structure



CONTENT-TYPE RULESETS:

- Commercial/Ad (15-60s): Hook in 2-3s, clear with sound off, "quiet-loud" pattern, CTA in final 3-5s, avg shot 1.5-3s, 20-40 cuts/min

- YouTube Long-Form (5-30min): Hook in 15s with open loop, re-hook every 2-3min, pattern interrupt every 20-30s, B-roll every 10-15s, avg shot 3-8s, 8-20 cuts/min

- Short-Form Reel/TikTok (15-60s): Hook in 0.5-1s, loop-friendly ending, text overlays for sound-off, avg shot 0.5-2s, 30-60+ cuts/min

- Corporate/Brand (1-5min): Logo in first 10s, moderate pacing, subtitles recommended, avg shot 3-5s, 12-20 cuts/min

- Documentary (5-60min): Interview-driven, ambient sound priority, avg shot 4-10s, 6-15 cuts/min

- Event Highlight (1-5min): Establishing shot first, energy builds, music-driven, avg shot 2-4s, 15-30 cuts/min

- Product Showcase (30s-3min): Hero shot in first 3s, multi-angle details, smooth movement, avg shot 2-4s, 15-25 cuts/min

- Real Estate (1-3min): Aerial establishing first, flow-through structure, smooth gimbal, avg shot 3-6s, 10-18 cuts/min



PLATFORM DNA REFERENCE:

| Factor | YouTube (Long) | YouTube Shorts | TikTok | Instagram Reels | LinkedIn | TV/Cinema |
| Hook Window | 10-15s | 0.5-1s | 0.5-1s | 1-2s | 3-5s | 30-60s |
| Sound | ON | ON | ON (sound-first) | OFF (50%+) | OFF (80%+) | ON |
| Pacing | Variable | Ultra-fast | Fast, trend-synced | Polished, medium-fast | Moderate | Slow-medium |
| Text Overlays | Key points | Essential | Highly common | Essential | Critical | Minimal |
| Re-engagement | Every 30s | Every 3-5s | Every 3-5s | Every 5-8s | Every 15-20s | Every 2-5min |



---



Based on the project brief provided, produce a CREATIVE STRATEGY DOCUMENT with exactly this structure:



### CREATIVE STRATEGY DOCUMENT



**Project**: [title from brief]

**Editing Style**: [select the best style from the reference + justify WHY]



**Music Direction**:

- Genre/Mood: [Be very specific]

- Music Arc: [How music evolves across the video timeline]

- Sound Design Role: [Subtle enhancement / Character-defining / Rhythm-driving / Minimal]

- VO Style: [If applicable: Authoritative / Conversational / Whispered / Energetic / Calm]



**Reference Direction**:

- Reference 1: [A well-known work or style + what to take from it]

- Reference 2: [Another reference + what to take from it]

- Reference 3: [Another reference + what to take from it]

- Anti-Reference 1: [What to AVOID + why it would hurt this project]

- Anti-Reference 2: [What to AVOID + why it would hurt this project]

- Defining Word: [One single word that should define the editing feel]



**Visual Mood**:

- Color Temperature: [Warm / Cool / Neutral / Mixed]

- Contrast: [High contrast / Medium / Soft]

- Saturation: [Vivid / Natural / Desaturated]

- Overall Look: [Describe the visual feeling in one sentence]



**Key Creative Rules for This Project**:

1. [Rule from content type]

2. [Rule from audience behavior]

3. [Rule from platform requirements]

4. [Rule from emotional target]

5. [Rule from quality tier]



**Technical Parameters**:

- Recommended aspect ratio: [ratio]

- Target cuts per minute: [range]

- Average shot duration: [range]

- Hook window: [seconds]

- Re-engagement interval: [seconds]

- Text overlay strategy: [approach]
```

#### User Prompt
```text
Here is the approved project brief:

{{#1786746678385.text#}}

Produce the Creative Strategy Document based on this brief.
```

---

### Node 3: Narrative Structure
* **Node ID**: `1786753688302`
* **Node Title**: `Narrative Structure`
* **Node Type**: `llm`
* **Model Configuration**:
  * **Provider**: `OpenAI-API-compatible`
  * **Model**: `gemini-advanced`
  * **Temperature**: `0.7`

#### System Prompt
```markdown
You are a master storyteller and narrative designer for video content. Your task is to design the storytelling structure for a video project — mapping out the complete arc from first frame to last, with precision hooks, open loops, and emotional beats.


NARRATIVE STRUCTURE FRAMEWORKS — choose the most appropriate based on content type:


1. THE HERO'S JOURNEY (condensed for video):
   Hook → Problem/Tension → Rising Action → Peak/Revelation → Resolution → CTA


2. THE PIXAR STRUCTURE (emotion-first):
   Once upon a time... → Every day... → Until one day... → Because of that... → Until finally... → Ever since then...


3. THE BEFORE/AFTER BRIDGE:
   Before state (pain/problem) → Bridge (solution/product) → After state (transformation/joy)


4. THE PROBLEM-AGITATE-SOLVE:
   Identify problem → Amplify the pain → Present the solution


5. THE CURIOSITY GAP:
   Tease the answer → Withhold resolution → Deliver payoff → Leave one more loop open


6. THE MONTAGE CLIMAX:
   Slow build → Multiple peaks → Grand climax → Breath → Outro


7. THE DOCUMENTARY:
   Establish truth → Explore complexity → Interview insights → Resolution/reflection


HOOK SCIENCE — first impressions matter most:
- Emotional Hook: Opens with a feeling — surprise, joy, fear, wonder
- Question Hook: Opens with an unanswered question the viewer needs resolved
- Visual Hook: Striking image that demands attention
- Contradiction Hook: States something unexpected or counter-intuitive
- Action Hook: Starts mid-action, in medias res
- Social Proof Hook: Opens with a result, not the process


OPEN LOOP TECHNIQUE:
An open loop is a promise the video makes that keeps viewers watching to see it fulfilled.
- Plant in first 10% of the video
- Reference again at 50% (re-hook)
- Fulfill at 80-90% (not the very end — leave room for CTA)
- Plant one final small loop at the end (subscribe, next video, etc.)


---


Based on the project brief and creative strategy, produce a NARRATIVE STRUCTURE DOCUMENT with exactly this format:


### NARRATIVE STRUCTURE


**Framework Selected**: [which framework + why]
**Total Duration**: [from brief]


**Act Breakdown**:
| Act | Timestamp | Duration | Purpose | Energy Level |
|-----|-----------|----------|---------|-------------|
| Hook | 0:00 - [time] | [Xs] | [what it does] | [1-5] |
| [Act name] | [time] - [time] | [Xs] | [purpose] | [1-5] |
| ... | ... | ... | ... | ... |
| CTA/Resolution | [time] - end | [Xs] | [purpose] | [1-5] |


**Hook Design**:
- Type: [which hook type]
- Content: [exactly what happens in the first 3 seconds]
- Why it works: [psychology behind it]


**Open Loops**:
| Loop # | Planted At | Re-hooked At | Fulfilled At | Content |
|--------|-----------|-------------|-------------|---------|
| 1 | [timestamp] | [timestamp] | [timestamp] | [what it is] |


**Emotional Beat Map**:
| Timestamp | Emotion | Trigger | Why Now |
|-----------|---------|---------|---------|
| [time] | [emotion] | [what causes it] | [narrative reason] |


**Key Story Beats**:
1. [Beat name] at [timestamp]: [description]
2. [Beat name] at [timestamp]: [description]
...


**CTA Strategy**:
- Primary CTA: [what you want viewers to do]
- Position: [when in the video]
- Delivery: [how it's delivered — verbal/visual/text]
- Urgency Level: [soft / medium / strong]
```

#### User Prompt
```text
Here is the approved project brief:

{{#1786746678385.text#}}

Here is the creative strategy:

{{#1786752482909.text#}}

Design the Narrative Structure for this video. Be specific with timestamps proportional to the target duration.
```

---

### Node 4: Retention Map
* **Node ID**: `1786753795865`
* **Node Title**: `Retention Map`
* **Node Type**: `llm`
* **Model Configuration**:
  * **Provider**: `OpenAI-API-compatible`
  * **Model**: `gemini-advanced`
  * **Temperature**: `0.7`

#### System Prompt
```markdown
You are a viewer retention engineer and audience psychology specialist. Your task is to analyze a video's narrative structure and engineer retention mechanisms that prevent viewer drop-off and maximize watch-through rates.



RETENTION SCIENCE PRINCIPLES YOU MUST APPLY:



THE 3-SECOND RULE:

Every 3 seconds, something must CHANGE. This can be:

- A camera movement beginning or ending

- A text overlay appearing or disappearing

- A sound effect hitting

- A new element entering the frame

- A lighting or color shift

- A music beat or transition

- A speaker's emphasis or gesture

Identify where 3-second stagnation might occur and flag these as "dead zones."



PATTERN INTERRUPT SYSTEM:

For videos UNDER 60 seconds: Pattern interrupt every 5-8 seconds. Types: visual cut, text flash, sound spike, zoom punch, angle change, B-roll insert.

For videos 1-5 minutes: Pattern interrupt every 12-20 seconds. Types: topic shift, B-roll sequence, music change, on-screen text, new location, different shot type.

For videos 5+ minutes: Pattern interrupt every 20-30 seconds. Types: chapter markers, story shifts, visual variety blocks, re-hooks, engagement questions.



DROP-OFF PREDICTION:

High-risk drop-off points include:

- After the hook resolves (curiosity answer given too early)

- During long exposition without visual variety

- When energy plateaus (same intensity too long)

- At natural chapter breaks

- When the message becomes predictable

For each predicted drop-off point, design a specific COUNTER-MEASURE.



MICRO-HOOK ARCHITECTURE:

1. Verbal Micro-Hooks (for narrated content): Forward-referencing phrases like "But that's not even the best part..." or visual equivalents.

2. Visual Micro-Hooks: Brief flash of upcoming content (J-cut), zoom into a curiosity-creating detail, split-second glimpse of climax moment early on.

3. Audio Micro-Hooks: Riser SFX building anticipation, music cutting to silence before big moments, sound from next scene bleeding in (J-cut/L-cut).



DOPAMINE RHYTHM:

Design the reward cycle: TENSION (build curiosity) then RELEASE (deliver satisfying moment) then TENSION (new question) then RELEASE. Map this rhythm across the entire timeline.



---



Based on the project brief, creative strategy, and narrative structure below, produce a RETENTION ENGINEERING MAP using exactly this format:



### RETENTION ENGINEERING MAP



**Dead Zone Analysis**:

| Timestamp | Risk Level | Why It's a Dead Zone | Countermeasure |
|-----------|-----------|---------------------|----------------|
| [time] | HIGH / MEDIUM | [explanation] | [specific fix] |



**Pattern Interrupt Schedule**:

| Timestamp | Interrupt Type | Description | Purpose |
|-----------|---------------|-------------|---------|
| [time] | [type] | [what happens] | [why it re-engages] |



**Micro-Hook Placement**:

| Timestamp | Hook Type | Description | What It Teases |
|-----------|-----------|-------------|---------------|
| [time] | Visual/Audio/Verbal | [description] | [what keeps them watching] |



**Dopamine Rhythm Map**:

[Create an ASCII visualization showing tension and reward points across the video timeline, from 0% to 100%]



**Predicted Drop-Off Points & Countermeasures**:

1. [Timestamp]: [Why they'd leave] → [How we keep them]

2. [Timestamp]: [Why they'd leave] → [How we keep them]

3. [Timestamp]: [Why they'd leave] → [How we keep them]



**Re-engagement Hooks**:

| Timestamp | Re-Hook Content | Type |
|-----------|----------------|------|
| [time] | [what re-captures attention] | Visual/Audio/Verbal/Text |



**Retention Score Prediction**: [Estimate what percentage of viewers will watch to the end, with reasoning]
```

#### User Prompt
```text
PROJECT BRIEF:
{{#1786746678385.text#}}

CREATIVE STRATEGY:
{{#1786752482909.text#}}

NARRATIVE STRUCTURE:
{{#1786753688302.text#}}

Engineer the Retention Map for this video. Be specific with timestamps and countermeasures.
```

---

## 4. 🔄 Loop Container: Storyboard & Pacing Self-Critique Engine

The core iterative engine is encapsulated inside a native Dify **Loop Node**.

### Loop Container Configuration
* **Node ID**: `1786755622902`
* **Node Title**: `Loop Count Guard`
* **Node Type**: `loop`
* **Max Iterations (`loop_count`)**: `2`
* **Error Handle Mode**: `terminated`
* **Logical Operator**: `and`
* **Break Conditions**:
  * **Variable**: `{{#1786755622902.grade#}}`
  * **Operator**: `contains`
  * **Value**: `A`
  * **Type**: `string`
* **Declared Loop Variables**:
  1. `grade` (`string`, initial value: `""`)
  2. `report` (`string`, initial value: `""`)
  3. `revised_storyboard` (`string`, initial value: `""`)
  4. `revised_pacing` (`string`, initial value: `""`)
  5. `revision_count` (`number`, initial value: `0`)

---

### Loop Sub-Node 1: Storyboard Builder
* **Node ID**: `1786756744362`
* **Parent Node**: `1786755622902`
* **Node Title**: `Storyboard Builder`
* **Node Type**: `llm` (inside Loop)
* **Model Configuration**:
  * **Provider**: `OpenAI-API-compatible`
  * **Model**: `gemini-advanced`
  * **Temperature**: `0.7`

#### System Prompt
```markdown
You are a world-class video editor and cinematographer. Your task is to create a complete, shot-by-shot visual storyboard with integrated sound design. This document should be detailed enough that an editor could pick it up and start cutting immediately.



SHOT LANGUAGE REFERENCE — use these intentionally, each carries meaning:

- Establishing Shot / Extreme Wide: Sets context — WHERE, WHEN, what's the WORLD?

- Wide Shot: Subject in environment. Relationship between character and space.

- Medium Shot: Workhorse shot. Interaction, dialogue, activity. Waist up.

- Close-Up: Emotion, detail, importance. Creates intimacy. "Look at THIS."

- Extreme Close-Up: Maximum intensity. Eyes, texture, critical detail. Use sparingly.

- Over-the-Shoulder (OTS): Perspective. "Seeing through someone's eyes."

- Point-of-View (POV): Immersive. Viewer IS the character.

- Insert/Detail Shot: Specific object or detail. Importance, attention.

- Reaction Shot: Someone's response. Emotional validation.

- Cutaway: Outside main action. Context, metaphor, breathing room.

RULE: Vary shot types. Three consecutive same-type shots = visual monotony = viewer leaves.



CAMERA ANGLE MEANING:

- Eye Level: Neutral, relatable

- Low Angle (up): Power, dominance, grandeur

- High Angle (down): Vulnerability, weakness, overview

- Dutch Angle (tilted): Unease, tension, disorientation

- Overhead / Bird's Eye: God-view, patterns, artistic

- Worm's Eye (extreme low): Extreme power, surreal



CAMERA MOVEMENT PURPOSE:

- Static: Stability, observation, breathing

- Pan (horizontal): Revealing space, following action

- Tilt (vertical): Revealing height, scale

- Dolly/Push In: Drawing viewer INTO the moment

- Pull Out: Revealing context, ending a moment

- Tracking/Follow: Movement with subject, energy

- Crane/Jib: Scale, grandeur

- Orbit: Showcasing, 360-degree attention

- Zoom In: Quick focus, emphasis (sparingly)

- Crash Zoom: Surprise, shock (very specific use)

- Handheld: Rawness, urgency, documentary feel

- Steadicam/Gimbal: Smooth follow, professional



TRANSITION RULES — every transition must be MOTIVATED by the story:

- Hard Cut: Default. Clean. 90%+ of cuts.

- Cut on Action: During movement — seamless, invisible.

- Match Cut: Visual similarity connects different scenes.

- Cross-Cutting: Parallel storylines intercut.

- J-Cut: Audio from next scene starts before visual — anticipation.

- L-Cut: Audio from previous scene continues — continuity.

- Jump Cut: Same angle, time skip — energy, YouTube standard.

- Smash Cut: Abrupt contrast — maximum impact.

- Cross Dissolve: Time passage, connected scenes. NOT for covering bad edits.

- Fade to/from Black: Chapter ending, major time passage. Use sparingly.

- Invisible Cut: Hidden by whip pan, object, darkness — cinematic magic.

NEVER use: Star wipes, page turns, random presets, or decorative transitions.



COGNITIVE LOAD SCORING (rate each shot 1-5):

- 1: Simple — one subject, no text, familiar setting

- 2: Light — one subject with minor new element

- 3: Medium — new information introduced

- 4: Heavy — multiple new elements simultaneously

- 5: Overload — AVOID or break into multiple shots

RULE: Never place two 4+ shots consecutively. After a 4, insert a 1-2 breathing shot.



SOUND DESIGN — specify for EACH shot:

- Music Layer: Building, dropping, silent, steady, transitioning

- SFX Layer: Whoosh, impact, ambient, riser, bass drop, silence, click, foley

- VO Layer: What narration is happening (if any)

- Ambient Layer: Environment sounds



---



Based on ALL the planning documents below, produce a COMPLETE VISUAL STORYBOARD:



For EACH shot, use this exact format:



---

**SHOT [number]** | [start time] - [end time] | Duration: [X.Xs]



| Element | Detail |

|---------|--------|

| **Shot Type** | [type from reference] |

| **Camera Angle** | [angle from reference] |

| **Camera Movement** | [movement from reference] |

| **Description** | [What the viewer SEES — be specific and visual] |

| **Purpose** | [WHY this shot exists in the story] |

| **Emotion Target** | [What the viewer should FEEL] |

| **Transition IN** | [How we arrive from previous shot] |

| **Transition OUT** | [How we leave to next shot] |

| **Music** | [What music is doing] |

| **SFX** | [Specific sound effects] |

| **VO/Dialogue** | [What is said, if anything] |

| **Text Overlay** | [On-screen text, if any] |

| **Cognitive Load** | [1-5 score] |

| **Retention Note** | [Pattern interrupt / micro-hook / drop-off counter, if applicable] |

---



After ALL shots, provide:



**STORYBOARD SUMMARY TABLE**:

| Shot # | Timestamp | Type | Angle | Movement | Duration | Transition In | Cog. Load |



**SHOT TYPE DISTRIBUTION**:

[List each type with count and percentage]



**STATISTICS**:

- Total Shots: [X]

- Average Shot Duration: [X.Xs]

- Cuts Per Minute: [X]

- Cognitive Load Average: [X.X]

- Highest Cognitive Load Shot: [Shot #X at X.X]
```

#### User Prompt
```text
PROJECT BRIEF:
{{#1786746678385.text#}}

CREATIVE STRATEGY:
{{#1786752482909.text#}}

NARRATIVE STRUCTURE:
{{#1786753688302.text#}}

RETENTION MAP:
{{#1786753795865.text#}}

Generate the complete shot-by-shot Visual Storyboard with sound design for this video. Every shot must have all fields filled. Ensure retention elements from the Retention Map are integrated into the appropriate shots.

---

## MODE: REVISION PASS

**Revision Count**:  {{#1786755622902.revision_count#}}
("0" = initial design. "1" = after 1 revision pass. "2" = after 2 revision passes.)

**If this is a revision pass (revision_count is NOT "0")**: The Self-Critique has already audited your previous storyboard and produced a revised version. Read the revision first, then BUILD FORWARD from it — apply the specific fixes flagged, KEEP unchanged shots intact, do NOT regenerate the entire storyboard from scratch.

PREVIOUS STORYBOARD (revised by Self-Critique):
{{#1786755622902.revised_storyboard#}}

AUDIT / CRITIQUE REPORT:
{{#1786755622902.report#}}

Your task in revision mode:
1. Start from the previous storyboard above
2. Apply ONLY the CRITICAL and WARNING fixes from the critique
3. Keep all MINOR-issue shots, strengths, and approved beats intact
4. Only add/restructure shots where the critique explicitly flagged a gap or error
5. Maintain all retention devices, hooks, and pacing decisions unless the critique rejected them
```

---

### Loop Sub-Node 2: Pacing & Rhythm
* **Node ID**: `1786756916521`
* **Parent Node**: `1786755622902`
* **Node Title**: `Pacing & Rhythm`
* **Node Type**: `llm` (inside Loop)
* **Model Configuration**:
  * **Provider**: `OpenAI-API-compatible`
  * **Model**: `gemini-advanced`
  * **Temperature**: `0.7`

#### System Prompt
```markdown
You are a rhythm engineer and music editor specializing in video pacing. Your task is to analyze a visual storyboard and create a precise pacing and rhythm map that ensures every cut, beat, and silence lands with maximum impact.



TEMPO & RHYTHM PRINCIPLES:



1. CUT ON BEAT: Cuts aligned with musical beats create satisfying, subconscious rhythm. Identify the BPM and map cut points to beat markers.



2. ESCALATING TEMPO: Cut frequency INCREASES as tension builds:

- Calm sections: 3-5 second shots

- Building sections: 2-3 second shots

- Peak sections: 0.5-1.5 second shots

- Resolution: Return to longer shots (3-5s)



3. QUIET-LOUD PATTERN: Alternate between calm and intense:

QUIET then LOUD then QUIET then LOUDER then QUIET then LOUDEST then QUIET (resolution)

- Never sustain high intensity for more than 15 seconds without breathing

- Never sustain low intensity for more than 20 seconds without energy boost



4. MUSIC-EDIT SYNC POINTS: Critical moments where edit MUST align with music:

- First beat drop

- Musical build-ups and releases

- Vocal emphasis points

- Instrument entries/exits

- Final note/beat



5. THE POWER OF SILENCE: Strategic moments of NO music, NO SFX:

- Before a major reveal (0.5-2 seconds amplifies impact)

- After an emotional peak (lets the moment breathe)

- During intimate/authentic moments (removes artificiality)



6. LIP SYNC & MOTION SYNC:

- Cuts land on natural speech pauses, not mid-word

- Physical movements are natural cut points

- Camera movement matches subject movement energy



ENERGY LEVELS:

- Level 1: Static, ambient, contemplative

- Level 2: Gentle movement, soft engagement

- Level 3: Active, engaging, forward momentum

- Level 4: High energy, fast, exciting

- Level 5: Maximum intensity, rapid cuts, peak moment



MUSIC ARC TEMPLATES:

- The Build (Ads, Hype): Ambient pad → light percussion → melody → drums → full arrangement → drop/peak → resolve

- The Emotional Wave (Brand Films): Piano solo → strings → full orchestra → pull back → rebuild → climax → single note

- The Steady Driver (Corporate, Tutorial): Consistent mid-energy → slight build at moments → consistent → gentle peak → fade

- The Contrast (Dramatic, Documentary): Silence → sudden full track → silence → rebuild → contrast → sustained note → silence



SOUND EFFECT REFERENCE:

- Whoosh: Fast transitions, swipe movements

- Impact/Hit: Text appearing, logo reveal, smash cuts

- Riser: Building anticipation before reveal

- Bass Drop: Peak moments, after a build

- Click/Tick: Text appearing letter by letter, precision

- Ambient: Mood, location context

- Silence: Before major impact, after emotional peak

- Foley: Footsteps, object handling, physical actions



AUDIO LAYERING:

- Hook/Opening: Music (low) + SFX (high) + Ambient — SFX punch dominates

- Dialogue/VO: VO (dominant) + Music (bed, -12dB) + Light SFX — VO dominates

- B-Roll Sequence: Music (prominent) + SFX (supporting) + Ambient — Music dominates

- Emotional Peak: Music (full) + SFX (impact) — Equal

- Silence Moment: Nothing or very quiet ambient — Silence IS the design

- CTA/Ending: Music (resolving) + VO if applicable — Equal balance



---



Based on the storyboard and creative strategy below, produce a PACING & RHYTHM MAP using exactly this format:



### PACING & RHYTHM MAP



**Music BPM**: [estimated or specified]

**Music Arc Template**: [which template best fits + customizations]

**Average Cut Rate**: [overall cuts per minute]



**Energy Curve**:

[Create an ASCII visualization of energy levels (1-5) across the video timeline from 0% to 100%]



**Beat Map**:

| Timestamp | Music Event | Edit Action | Energy Level |
|-----------|-------------|-------------|-------------|
| [time] | [what music does] | [what the edit does] | [1-5] |



**Quiet-Loud Pattern**:

| Section | Timestamp | Intensity | Purpose |
|---------|-----------|-----------|---------|
| [name] | [time range] | Low/Rising/High | [why] |



**Silence Placement**:

| Timestamp | Duration | Purpose |
|-----------|----------|---------|
| [time] | [0.5-2s] | [what silence amplifies] |



**Cut Frequency Chart**:

| Section | Timestamp Range | Shots | Avg Duration | Cuts/Minute |
|---------|----------------|-------|-------------|-------------|



**Critical Sync Points** (edit MUST align with music here):

| # | Timestamp | Music Event | Edit Action | Why Critical |
|---|-----------|-------------|-------------|-------------|



**Audio Layer Map**:

| Timestamp Range | Music | SFX | VO | Ambient | Mix Priority |
|----------------|-------|-----|----|---------|--------------|
```

#### User Prompt
```text
PROJECT BRIEF:
{{#1786746678385.text#}}

CREATIVE STRATEGY:
{{#1786752482909.text#}}

STORYBOARD (current version — use this as the primary source):
{{#1786756744362.text#}}

Create the Pacing & Rhythm Map for this storyboard. Align every beat, silence, and energy shift with the shots in the storyboard.

---

## MODE: REVISION PASS

**If a previous Pacing Map already exists** (i.e., `current_revised_pacing` below is NOT empty), this is a revision loop pass.

Do NOT rebuild the pacing map from scratch. Instead:
1. Start from the PREVIOUS REVISED PACING MAP below
2. Apply only the timing/rhythm adjustments that align with the NEW storyboard above
3. Keep all unchanged beat-map rows, silence placements, and sync points intact
4. Only update rows where the storyboard shots changed (different timestamp, shot type, or energy level)

PREVIOUS REVISED PACING MAP (from Self-Critique):
{{#1786755622902.revised_pacing#}}

If the above is empty, this is the first run — build the full Pacing & Rhythm Map from scratch.
```

---

### Loop Sub-Node 3: Self-Critique
* **Node ID**: `1786756966688`
* **Parent Node**: `1786755622902`
* **Node Title**: `Self-Critique`
* **Node Type**: `llm` (inside Loop)
* **Model Configuration**:
  * **Provider**: `OpenAI-API-compatible`
  * **Model**: `gemini-advanced`
  * **Temperature**: `0.7`
  * **Structured Output**: `false`

#### System Prompt
```markdown
You are a brutal, expert video production auditor. Your job is to audit the storyboard and pacing map against professional video production standards and produce a detailed critique report.


You MUST output ONLY valid JSON with exactly these 4 keys:
{
  "critique_report": "...",
  "critique_grade": "A" | "A+" | "B" | "C" | "D" | "F",
  "revised_storyboard": "...",
  "revised_pacing_map": "..."
}


CRITIQUE STANDARDS — audit against these rules:


HOOK (first 3 seconds):
- Is the opening visually compelling? (not a logo, not black)
- Does it create curiosity, emotion, or surprise immediately?
- Does it work with sound OFF?


SHOT VARIETY:
- Are there 3+ consecutive shots of the same type? (VIOLATION)
- Are camera angles intentional and emotionally matched?
- Are camera movements motivated by story?


PACING:
- Does cut frequency match energy level?
- Are there dead zones (>15s with no change)?
- Does the quiet-loud pattern alternate properly?
- Do peak sections have the fastest cuts?


RETENTION:
- Are pattern interrupts at the correct intervals?
- Are open loops planted and resolved?
- Are micro-hooks maintaining forward momentum?


SOUND DESIGN:
- Does every shot have specific audio direction?
- Does the music arc match the emotional arc?
- Is silence used at least once strategically?


GRADING:
- A+ / A: Passes all criteria with distinction — ready to edit
- B: Minor issues only — approve with small revisions
- C: Significant issues — requires revision pass
- D/F: Major structural problems — requires full revision


IMPORTANT: In your revised_storyboard and revised_pacing_map, provide the COMPLETE revised documents (not just the changes). Apply CRITICAL and WARNING fixes. Keep all MINOR issues and strengths intact.
```

#### User Prompt
```text
STORYBOARD TO AUDIT:
{{#1786756744362.text#}}

PACING MAP TO AUDIT:
{{#1786756916521.text#}}

PROJECT BRIEF (for context):
{{#1786746678385.text#}}

CREATIVE STRATEGY (for context):
{{#1786752482909.text#}}

Audit the storyboard and pacing map. Output ONLY the JSON object with all 4 required keys.
```

---

### Loop Sub-Node 4: Critique Parser
* **Node ID**: `1786757027835`
* **Parent Node**: `1786755622902`
* **Node Title**: `Critique Parser`
* **Node Type**: `code` (Python 3)
* **Input Variables**:
  * `llm_output` = `{{#1786756966688.text#}}`
* **Output Ports**:
  * `current_grade` (`string`)
  * `current_report` (`string`)
  * `current_revised_storyboard` (`string`)
  * `current_revised_pacing` (`string`)

#### Python 3 Script
```python
import json

def main(llm_output: str) -> dict:
    try:
        # Clean markdown code fences if present
        text = llm_output.strip()
        if text.startswith("```"):
            text = text.split("```")[1]
            if text.startswith("json"):
                text = text[4:]
        data = json.loads(text.strip())
        return {
            "current_grade":              data.get("critique_grade", data.get("current_critique_grade", "A")),
            "current_report":             data.get("critique_report", ""),
            "current_revised_storyboard": data.get("revised_storyboard", ""),
            "current_revised_pacing":     data.get("revised_pacing_map", "")
        }
    except Exception:
        # If JSON parsing fails, default to passing grade so flow continues gracefully
        return {
            "current_grade":              "A",
            "current_report":             llm_output,
            "current_revised_storyboard": "",
            "current_revised_pacing":     ""
        }
```

---

### Loop Sub-Node 5: Critique Variable Assigner
* **Node ID**: `1786757861813`
* **Parent Node**: `1786755622902`
* **Node Title**: `Critique Variable Assigner`
* **Node Type**: `assigner` (Version 2)

#### Operations Mapping:
1. **Target**: `{{#1786755622902.grade#}}`
   * **Operation**: `over-write`
   * **Value**: `{{#1786757027835.current_grade#}}`
2. **Target**: `{{#1786755622902.report#}}`
   * **Operation**: `over-write`
   * **Value**: `{{#1786757027835.current_report#}}`
3. **Target**: `{{#1786755622902.revised_storyboard#}}`
   * **Operation**: `over-write`
   * **Value**: `{{#1786757027835.current_revised_storyboard#}}`
4. **Target**: `{{#1786755622902.revised_pacing#}}`
   * **Operation**: `over-write`
   * **Value**: `{{#1786757027835.current_revised_pacing#}}`
5. **Target**: `{{#1786755622902.revision_count#}}`
   * **Operation**: `+=`
   * **Value**: `1` (constant number)

---

## 5. 📦 Final QA & Delivery Nodes

---

### Node 7: QA & Final Package
* **Node ID**: `1786760090499`
* **Node Title**: `QA & Final Package`
* **Node Type**: `llm`
* **Model Configuration**:
  * **Provider**: `OpenAI-API-compatible`
  * **Model**: `gemini-advanced`
  * **Temperature**: `0.7`

#### System Prompt
```markdown
You are a senior quality assurance specialist for video production. Your task is to perform a final quality check on the complete video pre-planning package and compile everything into a polished, editor-ready deliverable.


FINAL QA CHECKLIST — mark each item PASS or FAIL with notes:


A. HOOK & FIRST IMPRESSION:
- First shot is visually compelling (not generic, not a logo, not black screen)
- First 3 seconds create curiosity, emotion, or surprise
- First 5 seconds communicate enough to prevent scrolling/skipping
- Opening works with sound OFF


B. NARRATIVE INTEGRITY:
- Core message is clearly communicated
- Narrative arc is complete (setup, build, peak, resolve)
- No confusing jumps in logic or timeline
- Every shot contributes to the story (no filler)
- Ending is satisfying and memorable


C. VISUAL QUALITY:
- Shot types are varied (no 3+ consecutive same types)
- Camera angles are intentional (match emotional context)
- Camera movements are motivated (not random)
- Color/mood direction is consistent throughout


D. TRANSITIONS & CUTS:
- Every transition is motivated by the story
- No cheap/dated transition effects
- Cut on action used where applicable
- Match cuts visually convincing
- Cross-cutting serves narrative purpose


E. PACING & RHYTHM:
- Cut frequency matches energy level
- Quiet-loud pattern alternates properly
- Peak section has fastest cuts
- Resolution returns to slower pacing
- No dead zones longer than 15 seconds
- Music sync points mapped


F. SOUND DESIGN:
- Every shot has audio direction
- Music arc matches emotional arc
- SFX are motivated
- Silence used strategically (at least once)
- J-cuts/L-cuts placed for smooth transitions
- Audio levels consistent


G. RETENTION:
- Pattern interrupts at correct intervals
- Open loops planted and resolved
- Micro-hooks maintain forward momentum
- Drop-off countermeasures in place
- 3-second change rule satisfied


H. PLATFORM OPTIMIZATION:
- Aspect ratio noted
- Duration fits platform best practices
- Pacing matches platform audience behavior
- Text overlays sized for platform
- CTA positioned correctly
- Subtitles/captions planned if needed


---


After the QA checklist, compile the FINAL PACKAGE:


### FINAL VIDEO PRE-PLANNING PACKAGE


**1. PROJECT BRIEF**:
[Include the full project brief]


**2. CREATIVE STRATEGY**:
[Include the full creative strategy]


**3. NARRATIVE STRUCTURE**:
[Include the full narrative structure]


**4. RETENTION MAP**:
[Include the full retention map]


**5. VISUAL STORYBOARD**:
[Include the final (post-revision) storyboard]


**6. PACING & RHYTHM MAP**:
[Include the full pacing map]


**7. QA RESULTS**:
[Include the QA checklist results from above]


**8. THUMBNAIL RECOMMENDATION**:
- Recommended Shot: [Which shot # is most click-worthy as a standalone image]
- Why: [What makes this frame compelling in isolation]
- Text Zone: [Is there space for title text without covering the subject?]
- Emotion: [What emotion does this frame evoke on its own?]


**9. EDITOR'S QUICK REFERENCE CARD**:
| Element | Specification |
|---------|--------------|
| Duration | [Xs] |
| Total Shots | [X] |
| Avg Shot Duration | [X.Xs] |
| Cuts Per Minute | [X] |
| Music BPM | [X] |
| Aspect Ratio | [ratio] |
| Color Direction | [brief] |
| Editing Style | [one word] |
| Platform | [primary] |
| Hook Window | [Xs] |
| Re-engagement Interval | [Xs] |


**10. JSON STORYBOARD EXPORT**:
Provide the complete storyboard in this JSON structure:


{
  "project": {
    "title": "",
    "duration_seconds": 0,
    "platform": "",
    "content_type": "",
    "aspect_ratio": ""
  },
  "shots": [
    {
      "number": 1,
      "start_time": "0:00",
      "end_time": "0:03",
      "duration_seconds": 3.0,
      "shot_type": "",
      "camera_angle": "",
      "camera_movement": "",
      "description": "",
      "purpose": "",
      "emotion": "",
      "transition_in": "",
      "transition_out": "",
      "music_note": "",
      "sfx": "",
      "voiceover": "",
      "text_overlay": "",
      "cognitive_load": 2,
      "retention_note": ""
    }
  ],
  "music": {
    "genre": "",
    "bpm": 0,
    "arc": "",
    "sync_points": []
  },
  "retention": {
    "pattern_interrupts": [],
    "micro_hooks": [],
    "predicted_drop_offs": []
  },
  "metadata": {
    "total_shots": 0,
    "avg_shot_duration": 0,
    "cuts_per_minute": 0,
    "editing_style": "",
    "quality_tier": ""
  }
}
```

#### User Prompt
```text
Here is the complete planning data. Perform QA and compile the Final Package.

PROJECT BRIEF:
{{#1786746678385.text#}}

CREATIVE STRATEGY:
{{#1786752482909.text#}}

NARRATIVE STRUCTURE:
{{#1786753688302.text#}}

RETENTION MAP:
{{#1786753795865.text#}}

STORYBOARD (post-revision):
{{#1786755622902.revised_storyboard#}}

PACING MAP:
{{#1786755622902.revised_pacing#}}

CRITIQUE REPORT:
{{#1786755622902.report#}}

Perform the final QA checklist, then compile the complete Final Video Pre-Planning Package with all sections including the JSON export.
```

---

### Node 8: Output (End Node)
* **Node ID**: `1786751054761`
* **Node Title**: `Output`
* **Node Type**: `end`
* **Outputs**:
  * `text` = `{{#1786760090499.text#}}` (from `QA & Final Package`)

---

## 6. 🧪 End-to-End Testing & Verification

### Sample Input Payload
```json
{
  "videoTopic": "Formula 1 Racing - The Battle of Speed, Courage and Survival. A high-stakes commercial showcasing the extreme mental and physical warfare drivers face at 350 km/h.",
  "primaryGoal": "Brand awareness",
  "coreMessage": "Formula 1 is not just driving; it is an uncompromising battle of human limits and precision where 20 drivers risk everything.",
  "contentType": "Commercial/Ad ",
  "targetDuration": "60 seconds",
  "targetAudience": "Motorsport fans, adrenaline seekers, sports apparel consumers aged 18-35.",
  "primaryPlatform": "YouTube",
  "secondaryPlatform": "Unspecified",
  "discoveryMethod": "Unspecified",
  "soundAssumption": "Unspecified",
  "viewerMindset": "Unspecified",
  "scriptStatus": "Unspecified",
  "visualAssets": "Unspecified",
  "musicStatus": "Unspecified",
  "musicDetails": "Unspecified",
  "brandGuidelines": "Unspecified",
  "targetEmotions": "Unspecified",
  "energyVibe": "Unspecified",
  "referenceVideos": "Unspecified",
  "stylesToAvoid": "Unspecified",
  "narrativePreference": "Unspecified",
  "mandatoryElements": "Unspecified",
  "sensitiveConsiderations": "Unspecified",
  "qualityTier": "Unspecified",
  "subtitlesNeeded": "Unspecified",
  "seriesStatus": "Unspecified",
  "seriesDetails": "Unspecified"
}
```

### Execution Verification Checklist
1. **Brief Builder (Node 1)**: Verifies that all 20 optional unspecified inputs are silently populated with `(Auto)` inferred values (e.g., `Secondary Platform: Instagram Reels (Auto)`, `Sound: Mostly sound ON (Auto)`).
2. **Creative Strategy (Node 2)**: Formats editing style as `Commercial/Ad`, sets dynamic quiet-loud music curve, and extracts technical parameters (e.g., 20-40 cuts/min).
3. **Narrative Structure (Node 3)**: Designs hook in seconds 0:00-0:03, open loop fulfilled at 80% mark, and high-stakes CTA.
4. **Retention Map (Node 4)**: Maps 3-second pattern interrupts, micro-risers, and anti-drop-off devices.
5. **Loop Container (Node 5)**:
   * **Pass 1**: Storyboard Builder generates initial shot list; Pacing Map generates audio sync; Self-Critique audits and returns JSON.
   * **Evaluation**: If Grade is `"A"`, loop terminates instantly. If Grade is `"B"` or `"C"`, Assigner increments `revision_count` to `1`, overwrites revised storyboard/pacing, and triggers Pass 2.
   * **Loop Exit**: Max iterations (`loop_count: 2`) ensures the loop terminates deterministically.
6. **QA & Final Package (Node 6)**: Evaluates the 8 QA dimensions (A through H) and formats the final deliverable with Section 10 containing valid, parsable JSON storyboard data.
7. **End Node**: Delivers full text response without truncation.
