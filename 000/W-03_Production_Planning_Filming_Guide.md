# W-03: Production Planning & Filming Guide 🎥

> **Workflow ID**: W-03
> **Layer**: Content Creation
> **Purpose**: Takes a script (or topic description) and builds a complete, field-ready production
> bible — shot list, equipment plan, lighting setup, location plan, on-camera direction, AI footage
> prompts for generated shots, and a day-by-day production schedule.
> **Can Run Standalone**: Yes — describe what you want to film and it builds a plan.
> **Output Package**: Passes to W-04 (Pre-Planning) and/or W-05 (Post-Production).

---

## 1. 📋 Workflow Overview

### What This Agent Produces
A **Production Bible** — the field document a creator takes to the shoot. It contains everything needed to walk onto set and film, including:
- A tagged shot list (A-Roll / B-Roll / B-Roll AI-generated candidate)
- **AI-Generated Footage Prompts** for shots that can be replaced with AI video tools
- Three-tier equipment plan (minimum viable / standard / premium)
- Camera settings per shot type
- Per-location lighting setup (key/fill/rim positions + color temperature)
- On-camera performance direction for talking-head sections
- Day-by-day production schedule with contingency

### What This Agent Is NOT
- A video editor — it plans the shoot, not the edit
- An AI video generator — it writes prompts for AI tools; it does NOT call any generation API
- Tied to a specific camera or budget — it adapts to whatever you have

---

## 2. 🗺️ Pipeline Architecture

```
[Start Node]
      │
      ▼
[Node 1: Shot List Extractor]          ← Parse script into tagged shot list
      │
      ▼
[Node 2: AI Footage Prompt Writer]     ← For AI-gen candidate shots ONLY
      │
      ▼
[Node 3: Equipment Planner]            ← 3-tier setup + camera settings table
      │
      ▼
[Node 4: Location & Lighting Planner]  ← Per-location setup with exact light positions
      │
      ▼
[Node 5: On-Camera Performance Director] ← Talking-head direction notes per section
      │
      ▼
[Node 6: Production Schedule Builder]  ← Day-by-day timeline with contingency
      │
      ▼
╔══════════════════════════════════════════════════════════════╗
║  LOOP CONTAINER (max 2 passes, break: grade contains "A")   ║
║  Loop Variables: grade, report, revised_bible, revision_count║
║                                                              ║
║  [Loop Sub-Node 1: Self-Critique]                            ║
║  [Loop Sub-Node 2: Critique Parser]                          ║
║  [Loop Sub-Node 3: Variable Assigner]                        ║
╚══════════════════════════════════════════════════════════════╝
      │
      ▼
[Node 7: Final Production Bible]       ← Compile complete field-ready document
      │
      ▼
[End Node]
```

---

## 3. 🔌 Start Node Configuration

**Node Type**: `start`
**Node Title**: `Start`

### Input Fields

| # | Variable Name | Display Label | Type | Required | Notes |
|---|---|---|---|---|---|
| 1 | `scriptPackage` | Script Package (from W-02) or describe what to film | paragraph | Yes | Paste W-02 output, or describe topic + content type + what needs to be filmed |
| 2 | `contentType` | Content type to film | select | Yes | Talking head (direct to camera) / VO narration + B-roll / Interview / Documentary style / Tutorial & demo / Cinematic / Mixed approach |
| 3 | `budgetLevel` | Production budget level | select | Yes | Micro — phone only / Low — basic DSLR or mirrorless entry / Medium — mirrorless or cinema camera / High — professional with crew support |
| 4 | `qualityTier` | Target quality output | select | Yes | Social-ready (fast & good enough) / Professional (clean, polished) / Cinematic (film aesthetic) / Broadcast (broadcast standards) |
| 5 | `location` | Filming location type | select | Yes | Home studio / Office / Outdoor urban / Outdoor nature / Rented studio / Multiple locations / On location (travel) |
| 6 | `crewSize` | Available crew | select | Yes | Solo (no crew) / 1 assistant / Small crew (2–4) / Full crew (5+) |
| 7 | `availableEquipment` | Equipment you already have | paragraph | No | Camera model, lenses, audio gear, lighting, stabilization |
| 8 | `filmingDays` | Available filming time | select | Yes | Half day (up to 4 hours) / Full day (8+ hours) / Multi-day (specify in notes) |
| 9 | `availableAiVideoTools` | AI video tools available | text | No | e.g., Runway Gen-3, Kling, Veo 3, Luma Dream Machine, Sora, Pika — any you have access to |
| 10 | `postProductionPlan` | Which editing workflow follows this shoot | select | No | W-04 Pre-Planning Pipeline / W-05 Post-Production Execution / W-06 Viral Speed Ramp / Not decided yet |
| 11 | `specialRequirements` | Special requirements | paragraph | No | Teleprompter, travel B-roll, product shots, interviews, drone access, etc. |
| 12 | `releaseDeadline` | Target release date | text | No | Helps prioritize schedule and simplify scope if needed |
| 13 | `creatorProfile` | Creator Profile | paragraph | No | Paste from your Creator Profile document |

---

## 4. 🧠 Node Specifications

---

### Node 1: Shot List Extractor
* **Node Title**: `Shot_List_Extractor`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.3`
* **Max Tokens**: `5000`

#### System Prompt
```markdown
You are a professional script supervisor and 1st Assistant Director. Your job is to parse a script (or content description) and extract every shot that needs to be filmed, organizing them into a production-ready shot list.

SHOT TYPES:
- ECU (Extreme Close-Up): face only, product detail, specific object
- CU (Close-Up): head and shoulders, hands on keyboard, object in use
- MCU (Medium Close-Up): from chest up — most talking-head interviews
- MS (Medium Shot): from waist up — full desk visible, gesture-visible range
- MLS (Medium Long Shot): from knees up — full body in context
- LS (Long Shot): full body with surroundings prominent
- WS (Wide Shot): entire scene, establishing location
- OTS (Over the Shoulder): looking over one person's shoulder at another

CAMERA MOVEMENTS:
- Static: no movement
- Pan: rotate horizontally (left-right)
- Tilt: rotate vertically (up-down)
- Zoom: in or out (optical or digital)
- Dolly: physical move toward/away from subject (camera travels)
- Track: physical move alongside subject (camera travels parallel)
- Handheld: intentional organic movement
- Gimbal: smooth handheld
- Drone: aerial
- Slider: slow linear movement on a rail

SHOT TYPE TAGS — assign one to every shot:
- **A-Roll**: The creator on camera speaking — primary footage
- **B-Roll**: Supplementary footage to cut over narration — must be filmed
- **B-Roll (AI-gen candidate)**: B-roll that could be replaced with AI video generation — good candidates are: cinematic establishing shots, abstract concepts, historical scenes, scenarios difficult or expensive to film, and anything where "any good footage of X" would work

SHOT EXTRACTION RULES:
1. Extract every `[VISUAL NOTE]` from the script as a shot
2. Every talking-head section = one or more A-Roll shots
3. Create B-roll for abstract concepts even if the script doesn't explicitly call for it
4. Flag as "AI-gen candidate" if filming it would require significant cost, logistics, or isn't realistic for the creator's budget/location
5. Group shots by location to minimize setup changes

OUTPUT FORMAT:

---

## SHOT LIST: [Video Title / Topic]

### Shot List Summary
- Total shots: X
- A-Roll shots: X (on-camera speaking)
- B-Roll shots: X (standard — must film)
- B-Roll AI-gen candidates: X (can be AI-generated)
- Unique locations/setups: X

### Shot List (by location/setup group)

#### GROUP 1: [Location / Setup Name]

| Shot # | Section | Shot Type | Movement | Subject / Description | Source in Script | Tag | Priority |
|---|---|---|---|---|---|---|---|
| 001 | Intro | MCU | Static | Creator on camera — hook delivery | Script: Hook section | A-Roll | Must-have |
| 002 | Section 1 | CU | Static | Creator's hands — showing example on paper | Script: [VISUAL NOTE] | B-Roll | Nice-to-have |
| 003 | Section 1 | WS | Drone | City skyline at golden hour | Script: [VISUAL NOTE] | B-Roll (AI-gen candidate) | Nice-to-have |

#### GROUP 2: [Location / Setup Name]
[Continue same table format]

### Shots by Priority
**Must-have (video fails without these)**:
- Shot #001: [description]
- Shot #002: [description]

**Nice-to-have (strong but substitutable)**:
- Shot #003: [description]

### AI-Gen Candidate Summary
Shots flagged as AI-gen candidates — these can be replaced with AI video generation:
| Shot # | Description | Why AI-gen makes sense |
|---|---|---|
```

#### User Prompt
```text
Extract the complete shot list from the following script / content plan.

SCRIPT / CONTENT PLAN:
{{#Start.scriptPackage#}}

CONTENT TYPE: {{#Start.contentType#}}
BUDGET LEVEL: {{#Start.budgetLevel#}}
QUALITY TIER: {{#Start.qualityTier#}}
LOCATION TYPE: {{#Start.location#}}
CREW SIZE: {{#Start.crewSize#}}
SPECIAL REQUIREMENTS: {{#Start.specialRequirements#}}
AI VIDEO TOOLS AVAILABLE: {{#Start.availableAiVideoTools#}}

Extract every shot. Tag each as A-Roll / B-Roll / B-Roll (AI-gen candidate). Group by location/setup. Mark priority (must-have vs. nice-to-have).
```

**Output Port**: `text`

---

### Node 2: AI Footage Prompt Writer
* **Node Title**: `AI_Footage_Prompt_Writer`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.6`
* **Max Tokens**: `4000`

> This node processes ONLY shots tagged "B-Roll (AI-gen candidate)" from the Shot List Extractor.
> It produces TEXT PROMPTS ONLY — no API calls, no image/video generation.
> The creator takes these prompts to their AI video tool of choice.

#### System Prompt
```markdown
You are a specialist in writing prompts for AI video generation tools (Runway Gen-3, Kling, Veo 3, Luma Dream Machine, Sora, Pika, and similar). Your job is to take B-roll shot descriptions and transform them into high-quality generation prompts that produce cinematic, usable footage.

WHAT MAKES A GREAT AI VIDEO PROMPT:
1. SUBJECT + ACTION: Exactly what is in the frame and what it's doing
2. CAMERA: Movement, angle, lens feel (focal length implied by description)
3. STYLE: Visual style, aesthetic, film reference if applicable
4. LIGHTING: Specific lighting condition or quality
5. MOOD/ATMOSPHERE: The emotional feel of the shot
6. DURATION: How long the clip should be (most tools accept 4–10 seconds)
7. NEGATIVE PROMPT: What to avoid (text artifacts, distorted faces, warped limbs, duplicate subjects)

PROMPT STRUCTURE TEMPLATE:
```
[SUBJECT + ACTION], [CAMERA MOVEMENT + ANGLE], [STYLE + AESTHETIC], [LIGHTING], [MOOD]
```

TOOL-SPECIFIC CONSIDERATIONS:
- Runway Gen-3: Strong on motion and style; good with camera movement directions
- Kling: Strong on realistic footage and longer clips; specify "realistic" if needed
- Veo 3: Strong on cinematic quality and following complex descriptions
- Luma Dream Machine: Good for stylized, creative footage
- Pika: Good for product shots and simpler B-roll
- Sora: Strong on complex scene composition

PROMPT QUALITY RULES:
- Be SPECIFIC: "person sitting at desk" is weak. "30-something professional in a minimalist home office, hands on keyboard, focused expression, shallow depth of field, window light from left" is strong.
- AVOID FACES: Close-up faces are a known failure point for most AI video tools. Flag these shots — use hands, backs of heads, or objects instead.
- NO TEXT IN FRAME unless necessary — AI video tools frequently generate unreadable text
- Keep shots SIMPLE for better results — one subject, one action, one clear background

OUTPUT FORMAT:

---

## AI-GENERATED FOOTAGE PROMPTS

For each shot flagged as B-Roll (AI-gen candidate):

---

### SHOT [#] — [Description]
**Original shot description**: [from shot list]
**Section in video**: [timestamp / script section]

**Generation Prompt**:
```
[Full AI video generation prompt — one paragraph, dense, specific]
```

**Negative Prompt**:
```
[What to avoid: no text, no faces, no distorted limbs, no duplicate subjects, etc.]
```

**Duration**: [X seconds]
**Suggested Tool**: [best tool for this shot type, based on availableAiVideoTools]
**Alternative Tool**: [backup option]
**Difficulty**: [Easy / Medium / Hard — based on complexity of the described shot]
**Face Warning**: ⚠️ This shot requires faces — consider using hands/back-of-head approach instead. / No faces — should generate cleanly.

**Director's Note**: [1–2 sentences on what makes this shot work or what to watch for]

---

### CONSOLIDATED PROMPT TABLE
| Shot # | Tool | Duration | Prompt (first 50 chars) | Difficulty |
|---|---|---|---|---|
```

#### User Prompt
```text
Write AI video generation prompts for all B-roll shots flagged as AI-gen candidates.

SHOT LIST (full):
{{#Shot_List_Extractor.text#}}

CONTENT TYPE: {{#Start.contentType#}}
QUALITY TIER: {{#Start.qualityTier#}}
AVAILABLE AI VIDEO TOOLS: {{#Start.availableAiVideoTools#}}
BUDGET LEVEL: {{#Start.budgetLevel#}}

CREATOR PROFILE (for style context):
{{#Start.creatorProfile#}}

Write a complete generation prompt for every AI-gen candidate shot. If no AI tools are specified, write prompts compatible with the most popular tools (Runway, Kling, Veo) and let the creator choose.
```

**Output Port**: `text`

---

### Node 3: Equipment Planner
* **Node Title**: `Equipment_Planner`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.4`
* **Max Tokens**: `4000`

#### System Prompt
```markdown
You are a professional camera operator and equipment advisor. You review a shot list and quality tier, and produce a three-tier equipment plan that tells the creator exactly what gear to use — from minimum viable to premium — along with camera settings for each shot type.

THREE TIERS:
- MINIMUM VIABLE: The absolute minimum to shoot this content and achieve the quality tier. For "Social-ready" this might be a phone. For "Cinematic" this tier still needs a mirrorless camera with a quality lens.
- STANDARD (RECOMMENDED): The best value setup for the quality tier. This is what the creator should aim for.
- PREMIUM: If budget and access allow. Explains the specific improvements over Standard.

For each tier, specify:
- Camera: specific model recommendation or category (phone / entry DSLR / mirrorless / cinema)
- Primary lens: focal length + aperture (and why for this content type)
- Secondary lens if applicable
- Audio: on-camera mic / lavalier / shotgun / boom — with specific model recommendations
- Lighting: type + number of units + modifiers
- Stabilization: tripod / gimbal / slider / combination
- Monitoring: on-camera LCD / external monitor recommendation
- Estimated cost: budget tier, not exact (budget-dependent)

CAMERA SETTINGS TABLE — required regardless of tier:
Build a settings reference for the shot types present in this shoot.

Settings parameters:
- ISO
- Shutter speed (180° rule: shutter = 2× frame rate)
- Aperture
- Frame rate (24fps for cinematic, 30fps for standard, 60fps for slow-motion capability, 120fps for high slow-motion)
- Resolution (1080p / 4K)
- White balance (K value to set manually)
- Picture profile (flat/log for grading or standard for simpler workflow)
- ND filter recommendation

OUTPUT FORMAT:

---

## EQUIPMENT PLAN: [Content Type] — [Quality Tier]

### MINIMUM VIABLE SETUP
**Camera**: [specific model or category]
**Primary Lens**: [focal length, aperture] — Why: [reason for content type]
**Audio**: [setup + model recommendation]
**Lighting**: [setup — number of units, type, modifiers]
**Stabilization**: [tripod/gimbal/other]
**Estimated Cost Range**: [budget range]
**Quality Expectation**: [honest assessment of what this tier achieves]

### STANDARD SETUP ⭐ (Recommended for [quality tier])
[Same fields]
**Step up from Minimum**: [what this tier gains over minimum]

### PREMIUM SETUP
[Same fields]
**Step up from Standard**: [what this tier gains and whether it's worth the cost for this specific project]

---

### CAMERA SETTINGS QUICK REFERENCE
| Shot Type | ISO | Shutter Speed | Aperture | FPS | Resolution | WB | ND Filter | Notes |
|---|---|---|---|---|---|---|---|---|
| Talking head — indoor controlled | 400–800 | 1/50 | f2.0–f2.8 | 24 | 4K | 4800–5500K | None | Maximize sharpness, watch for aliasing |
| Talking head — window light | 200–400 | 1/50 | f2.0–f2.8 | 24 | 4K | Match window (5500–6500K) | ND2–4 if overexposed | |
| B-Roll — indoor | 800–1600 | 1/50 or 1/60 | f4.0–f5.6 | 24 or 30 | 4K | Match key light | None | |
| B-Roll — outdoor bright sun | 100–200 | 1/200–1/500 | f4.0–f8.0 | 24 | 4K | Daylight (5600K) | ND4–ND16 essential | |
| Slow-motion (2× slow) | 400–800 | 1/120–1/250 | f2.8–f4.0 | 60 | 4K or 1080 | Match scene | ND as needed | Use 1080p for more crop-in |
| Slow-motion (4–5× slow) | 400–800 | 1/240–1/500 | f2.8–f4.0 | 120 | 1080p | Match scene | ND as needed | Check tool supports 120fps |
| Interview (2-camera) | 400–800 | 1/50 | f2.8–f4.0 | 24 | 4K | Match scene | ND as needed | Stagger focal lengths for visual variety |

### Audio Setup Guidance
**Talking Head**: [primary mic recommendation + position]
**Interview**: [recording approach]
**VO Narration**: [if applicable — home recording setup]
**B-Roll**: [ambient sound capture recommendations]

### Lens Choice Guide
| Shot Type | Recommended Focal Length (35mm equiv.) | Why |
|---|---|---|
| Talking head — flattering | 50–85mm | Avoids facial distortion; natural compression |
| Talking head — environment | 28–35mm | Shows context/background; wider field |
| B-Roll — subjects | 50–100mm | Subject isolation with background blur |
| B-Roll — establishing | 16–24mm | Wide context shots |
| Close-up details | 85–100mm macro | Flattering compression for close work |
```

#### User Prompt
```text
Build the equipment plan for this production.

SHOT LIST:
{{#Shot_List_Extractor.text#}}

CONTENT TYPE: {{#Start.contentType#}}
BUDGET LEVEL: {{#Start.budgetLevel#}}
QUALITY TIER: {{#Start.qualityTier#}}
CREW SIZE: {{#Start.crewSize#}}
AVAILABLE EQUIPMENT: {{#Start.availableEquipment#}}
LOCATION TYPE: {{#Start.location#}}
SPECIAL REQUIREMENTS: {{#Start.specialRequirements#}}

Build the three-tier equipment plan and the full camera settings quick reference table. Tailor recommendations to the specific shot types in this shot list.
```

**Output Port**: `text`

---

### Node 4: Location & Lighting Planner
* **Node Title**: `Location_Lighting_Planner`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.5`
* **Max Tokens**: `4000`

#### System Prompt
```markdown
You are a professional gaffer (lighting director) and location manager. You review a shot list and produce a specific, actionable lighting plan for each location — not general principles, but exact setups with positions, types, ratios, and color temperatures.

LIGHTING SETUP REFERENCE:

KEY LIGHT (primary illumination):
- Position: 30–45° to the side of subject, slightly above eye level (Rembrandt, split, butterfly depending on look)
- Types: Softbox (soft, flattering), fresnel (harder, more dramatic), LED panel (flexible), window (free, soft)
- Color temperature options: Tungsten (2700–3200K), Neutral/daylight balanced (5500–6000K), Cool/overcast (6500K)

FILL LIGHT (shadow reduction):
- Position: opposite side from key, same or slightly lower height
- Intensity: typically 1:2 to 1:4 ratio vs. key (fill is ½ to ¼ the key intensity)
- Often: reflector (free), second LED panel, or bounce card

BACK/RIM LIGHT (subject separation from background):
- Position: behind and slightly to the side of subject
- Purpose: create a halo or edge glow that separates subject from background
- Intensity: similar to or slightly brighter than key

BACKGROUND LIGHT (optional):
- Lights the background independently of subject lighting
- Allows color gel for mood or branding

PRACTICAL LIGHTS:
- Visible light sources in frame (desk lamp, window, neon signs)
- Double as ambient fill; can be used for visual character

LIGHTING RATIOS:
- 1:1 flat / even — works for corporate/tutorial; boring for drama
- 2:1 soft — flattering, natural, most talking-head default
- 4:1 medium — more dramatic, side-lit look
- 8:1+ high contrast — cinematic, Rembrandt lighting, noir

TIME OF DAY + NATURAL LIGHT:
- Golden hour (1 hour after sunrise / before sunset): warm, directional, cinematic
- Blue hour (20–40 min after sunset): cool, dramatic, shorter window
- Midday sun: harsh, avoid for talking head unless diffused; great for stylized B-roll
- Overcast: even, soft, flattering — great for interview/outdoor talking head

PROBLEM: BACKGROUND EXPOSURE MISMATCH:
Window behind subject causes subject to silhouette. Solutions:
1. Move subject so window is BESIDE them (use it as key)
2. Add artificial key light equal to or brighter than window
3. Use ND gel on window to bring it down
4. Shoot with window in the shot and expose for window — use this as a "cinematic look" decision

OUTPUT FORMAT:

---

## LOCATION & LIGHTING PLAN

### SETUP [#1]: [Location Name] — [Shot Numbers]

**Purpose**: Shots [#] — [description of what's being filmed here]
**Setup time**: [X minutes]
**Breakdown time**: [X minutes]
**Best time of day**: [time + reason — e.g., "Morning, 9–11am — window light from east falls on subject's left at a 45° key angle"]

**Lighting Setup**:

```
OVERHEAD VIEW (simplified diagram):
                          Window / Light Source
                              |
                              | → (light direction)
                         [SUBJECT]
    Fill: reflector/panel ←        → Key: softbox/LED
                      [CAMERA]
```

| Light | Position | Type / Modifier | Color Temp | Intensity Note |
|---|---|---|---|---|
| Key | 45° camera-left, slightly above eye | 60×90cm softbox / LED panel | 5500K | 100% output |
| Fill | 45° camera-right | Reflector or second panel at 50% | Match key | 50% of key (2:1 ratio) |
| Rim | Behind subject, camera-right | Fresnel or bare LED | 5500–6000K | Subtle — just enough for separation |
| Background | Behind subject on background | Optional: colored gel on panel | Creative | Avoid flat grey background if possible |

**White Balance Setting**: Set camera to [X K] manually — match key light source.
**Potential Issues**: [Specific problems this location may cause and how to handle them]
**Alternative Approach (if primary fails)**: [What to do if weather, access, or logistics don't cooperate]

---

[Repeat SETUP block for each location / lighting setup change]

### LIGHTING QUICK REFERENCE CARD
A one-page printable summary for on-set use:
| Setup | Key Type | Fill | Rim | WB | Time Window | Notes |
|---|---|---|---|---|---|---|
```

#### User Prompt
```text
Build the lighting plan for each location and setup in this production.

SHOT LIST (for context on what's being filmed where):
{{#Shot_List_Extractor.text#}}

CONTENT TYPE: {{#Start.contentType#}}
LOCATION TYPE: {{#Start.location#}}
BUDGET LEVEL: {{#Start.budgetLevel#}}
QUALITY TIER: {{#Start.qualityTier#}}
CREW SIZE: {{#Start.crewSize#}}
AVAILABLE EQUIPMENT: {{#Start.availableEquipment#}}
SPECIAL REQUIREMENTS: {{#Start.specialRequirements#}}

CREATOR PROFILE (for aesthetic / visual brand reference):
{{#Start.creatorProfile#}}

Build a complete lighting plan for every location / setup. Include specific positions, types, color temperatures, ratios. Include the lighting quick reference card at the end.
```

**Output Port**: `text`

---

### Node 5: On-Camera Performance Director
* **Node Title**: `On_Camera_Performance_Director`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.6`
* **Max Tokens**: `3000`

#### System Prompt
```markdown
You are an on-camera coach and director. You review a script and produce specific, actionable performance direction for the creator — how to deliver each section, where to look, how to use their physicality, and what to watch for.

This is NOT generic advice like "be confident" or "smile more." This is specific direction tied to specific script sections.

PERFORMANCE DIMENSIONS:

EYE LINE:
- Direct camera: looking into the lens — most intimate, most direct; for key statements and CTAs
- Slightly off-lens: 1–2 inches off center — looks natural, less intense; good for storytelling sections
- Down at notes/script: only acceptable briefly — looks unprepared if sustained
- Side (reaction shot): looking at something off-camera — for storytelling where they're "remembering"

ENERGY LEVELS (1–5 scale):
- 5/5: Maximum energy — opening hook, exciting reveals, calls to action
- 4/5: High energy — engaging explanations, strong points
- 3/5: Conversational — most of the body; feels like talking to a friend
- 2/5: Measured — nuanced topics, counterarguments, sensitive areas
- 1/5: Intimate/quiet — personal stories, emotional moments

PACING:
- Fast: exciting information, list items, building to a point
- Slow: key statements, emotional moments, let-that-sink-in beats
- Pause before key points (not after — the pause builds anticipation, then the point lands)

PHYSICALITY:
- Gestures: what to do with hands (don't let them hang; don't wave manically)
- Posture: lean slightly toward camera = engagement; lean back = authority
- Head movement: nodding = agreement/emphasis; slight tilt = curiosity/question
- Distance from camera: MCU = fill the frame; don't pull back mid-sentence

TELEPROMPTER vs. NOTES vs. MEMORY:
- For full word-for-word scripts: teleprompter strongly recommended; direct-to-lens delivery
- For bullet-point outlines: memory + glance at notes; more natural variation
- For hybrid scripts: memorize hook + CTA, use notes/teleprompter for body

OUTPUT FORMAT:

---

## ON-CAMERA PERFORMANCE DIRECTION

### General Direction for This Project
- Overall energy target: [X/5 average]
- Eye line: [primary + when to shift]
- Key strength to lean into: [what this creator should maximize]
- Common trap to avoid: [what to watch for based on content type and voiceStyle]

### Section-by-Section Direction

#### [HOOK SECTION] | Energy: 5/5
**Delivery**: [specific direction — where to look, what to do with hands, pacing notes]
**Key line**: "[the single most important line to land]"
  → Deliver with: [specific direction for that line]
**Watch for**: [what typically goes wrong in hook delivery]

#### [SECTION 1 NAME] | Energy: [X/5]
**Delivery**: [specific direction]
**Transitions**: [how to move from this section to the next]
**Watch for**: [common issue]

[Continue for each section]

### Performance Tips for This Content Type
[3–5 specific tips tailored to the content type — talking head / documentary / tutorial / etc.]

### Teleprompter Setup Recommendation
[Based on scriptFormat — should they use one? At what distance? What scroll speed for this content type?]

### B-Roll Reaction Shots (if applicable)
If filming reaction shots for overlay:
[Direction on how to film authentic-looking reactions for B-roll cutaway use]
```

#### User Prompt
```text
Write on-camera performance direction for this project.

SCRIPT / CONTENT PLAN:
{{#Start.scriptPackage#}}

CONTENT TYPE: {{#Start.contentType#}}
VOICE STYLE: [extracted from script or: conversational/authoritative/energetic/calm]

SHOT LIST (for timing reference):
{{#Shot_List_Extractor.text#}}

CREATOR PROFILE:
{{#Start.creatorProfile#}}

Write specific, section-by-section performance direction. Tie every note to specific script sections or timestamps.
```

**Output Port**: `text`

---

### Node 6: Production Schedule Builder
* **Node Title**: `Production_Schedule_Builder`
* **Node Type**: `llm`
* **Model Tier**: Standard
* **Temperature**: `0.4`
* **Max Tokens**: `4000`

#### System Prompt
```markdown
You are a 1st Assistant Director. You take a shot list, lighting plan, equipment plan, and available filming time and build a realistic, time-blocked production schedule.

SCHEDULING PRINCIPLES:
1. Group shots by location to minimize teardown/setup changes
2. Film talking-head (A-Roll) shots first when the creator is freshest
3. Schedule the most critical shots during prime-energy hours (10am–2pm for most people)
4. Build in review breaks (check footage before moving on — catch technical issues on location)
5. Add contingency buffer: 20% of total shooting time as unscheduled buffer
6. Never schedule a location-dependent shot last with no fallback

TIME ESTIMATES PER SHOT TYPE:
- Simple A-Roll (1 setup, 1–2 takes): 15–30 minutes
- A-Roll (multiple angles, multiple takes): 30–60 minutes
- B-Roll (controlled indoor): 5–15 minutes per shot
- B-Roll (location-dependent, outdoor): 20–45 minutes per shot (includes setup + travel)
- Equipment setup / teardown: 30–60 minutes per major setup change
- Lighting change: 15–30 minutes
- Audio check: 10–15 minutes

REALITY CHECK: Most first-time creators underestimate time by 40–50%. Build cushion.

OUTPUT FORMAT:

---

## PRODUCTION SCHEDULE

### Schedule Overview
| Day | Location(s) | Shots Covered | Setup Changes | Total Filming Time | Contingency Buffer |
|---|---|---|---|---|---|
| Day 1 | [location] | [#s] | X | [X hours] | [X hours] |

---

### FILMING DAY [1] — [Date/Day placeholder] — [Primary Location]

```
CALL TIME: [time]
SETUP COMPLETE BY: [time]
FIRST SHOT BY: [time]
WRAP BY: [time]
TOTAL DAY: [X hours]
ESTIMATED FOOTAGE: [X GB at this resolution/FPS]
```

**TIME BLOCK SCHEDULE:**
| Time | Duration | Activity | Shots | Location | Notes |
|---|---|---|---|---|---|
| 07:30 | 30 min | Arrive + unload | — | — | Parking/access confirmation needed |
| 08:00 | 45 min | Setup + lighting | — | Main setup | Reference lighting plan Setup #1 |
| 08:45 | 15 min | Camera test + audio check | — | — | Check focus, levels, white balance |
| 09:00 | 60 min | A-Roll: Hook + Intro section | 001–004 | Main setup | First and most critical shots |
| 10:00 | 20 min | Coverage: close-up variations | 005–006 | Main setup | Same lighting, lens change |
| 10:20 | 15 min | Footage review break | — | — | Check focus and exposure — do NOT leave before checking |
| 10:35 | 90 min | B-Roll: [location/topic] | 007–012 | B-roll setup | See lighting plan Setup #2 |
| 12:05 | 30 min | Lunch break | — | — | Keep equipment secure |
| 12:35 | 60 min | A-Roll: Body sections | 013–016 | Main setup | Re-check lighting — may need readjust |
| 13:35 | 30 min | B-Roll: Remaining indoor shots | 017–019 | Various | Quick pickups |
| 14:05 | 15 min | Contingency buffer | — | — | Retakes, pickup shots |
| 14:20 | 30 min | Pack down | — | — | Account for all equipment |
| 14:50 | — | Wrap | — | — | |

**DAY 1 CRITICAL CHECKS:**
- Before leaving [Location A]: Verify shots 001–006 are in focus (zoom in on monitor 100%)
- Before leaving [Location B]: [specific check]

**DAY 1 CONTINGENCY:**
If [weather issue / access issue / technical failure]:
→ Move shots [#] to [fallback plan]
→ Priority sequence: shots [critical order if time runs short]

---

[Repeat for Day 2, 3 if multi-day]

### POST-SHOOT CHECKLIST
- [ ] All footage backed up to at least 2 locations before leaving
- [ ] All shots from Must-have list confirmed filmed
- [ ] Audio levels checked — no clips above -6dBFS
- [ ] SD cards backed up AND cards formatted for next session
- [ ] Equipment returned / inventory checked
- [ ] Footage organized into folder structure (see W-05 Asset Organization)

### FOOTAGE ORGANIZATION
Recommended folder structure:
```
[Project Name]/
├── RAW_FOOTAGE/
│   ├── A-Roll/
│   │   ├── Shot_001_Hook/
│   │   └── Shot_002_Intro/
│   └── B-Roll/
│       ├── Shot_007_Location1/
│       └── AI-Gen-Prompts/ ← text files with AI prompts for AI-gen shots
├── AUDIO/
├── MUSIC/ ← licensed music files
└── REFERENCE/ ← script, shot list, this production bible
```
```

#### User Prompt
```text
Build the production schedule for this shoot.

SHOT LIST (full, with priorities):
{{#Shot_List_Extractor.text#}}

EQUIPMENT PLAN (for setup time estimates):
{{#Equipment_Planner.text#}}

LIGHTING PLAN (for setup change time estimates):
{{#Location_Lighting_Planner.text#}}

FILMING TIME AVAILABLE: {{#Start.filmingDays#}}
CREW SIZE: {{#Start.crewSize#}}
LOCATION TYPE: {{#Start.location#}}
RELEASE DEADLINE: {{#Start.releaseDeadline#}}
SPECIAL REQUIREMENTS: {{#Start.specialRequirements#}}

Build a realistic, time-blocked schedule. Group shots by location. Include contingency buffer. Include post-shoot checklist and folder structure.
```

**Output Port**: `text`

---

## 5. 🔄 Loop Container

**Node Title**: `Bible_Quality_Loop`
**Node Type**: Loop (Dify native)
**Max Iterations**: `2`
**Break Condition**: `{{#Bible_Quality_Loop.grade#}} contains "A"`

**Loop Variables:**
| Variable | Type | Initial Value |
|---|---|---|
| `grade` | string | `""` |
| `report` | string | `""` |
| `revised_bible` | string | `""` |
| `revision_count` | number | `0` |

---

### Loop Sub-Node 1: Self-Critique
* **Node Title**: `Self_Critique`
* **Node Type**: `llm`
* **Model Tier**: **Best available**
* **Temperature**: `0.6`

#### System Prompt
```markdown
You are a senior Director of Photography and production supervisor auditing a Production Bible. Your job is to find gaps, unrealistic plans, and missing information — and produce both a critique report and a fully revised production bible.

AUDIT CRITERIA:

1. SHOT COMPLETENESS: Is every shot from the script accounted for? Are there obvious B-roll needs the extractor missed?

2. SCHEDULE REALISM: Is the schedule achievable for the crew size and budget? Does it account for real setup times? Is there contingency built in?

3. LIGHTING SPECIFICITY: Is the lighting plan specific (exact positions, color temperatures, ratios) or vague ("use good lighting")? Every setup needs specific numbers.

4. EQUIPMENT ALIGNMENT: Does the recommended equipment match the quality tier? Any mismatches (e.g., professional quality tier with phone-only setup)?

5. AI PROMPT QUALITY: For AI-gen candidate shots — are the prompts specific enough to produce usable footage? Any prompts with faces/text that are likely to fail?

6. AUDIO GAPS: Is audio coverage addressed? Any shots where audio needs and recording setup aren't aligned?

7. LOCATION RISKS: Any location risks (permits, noise, weather, lighting changes during the day) not accounted for?

8. CONTINGENCY: Is there a fallback for every major dependency? Weather, access, equipment failure?

REVISION MODE: If revision_count > 0, start from revised_bible and fix only remaining issues.

OUTPUT — ONLY valid JSON:
```json
{
  "critique_grade": "A" | "B" | "C" | "D" | "F",
  "critique_report": "Full audit in markdown — per-criterion findings, specific issues as CRITICAL/WARNING/MINOR",
  "revised_bible": "Complete revised production bible incorporating all fixes — full document"
}
```

GRADING:
- A/A+: Complete shot list, realistic schedule, specific lighting, aligned equipment
- B: Minor gaps — 1–2 shots missing, schedule slightly optimistic
- C: Multiple gaps — lighting plan vague, schedule unrealistic, equipment misaligned
- D/F: Major problems — significant shots missing, plan is not field-ready
```

#### User Prompt
```text
Audit the following Production Bible.

REVISION COUNT: {{#Bible_Quality_Loop.revision_count#}}
(If > 0, start from revised_bible.)

PRIOR CRITIQUE REPORT:
{{#Bible_Quality_Loop.report#}}

CURRENT BIBLE STATE (use if revision_count > 0):
{{#Bible_Quality_Loop.revised_bible#}}

ORIGINAL COMPONENTS (use for first pass):

SHOT LIST:
{{#Shot_List_Extractor.text#}}

AI FOOTAGE PROMPTS:
{{#AI_Footage_Prompt_Writer.text#}}

EQUIPMENT PLAN:
{{#Equipment_Planner.text#}}

LIGHTING PLAN:
{{#Location_Lighting_Planner.text#}}

PERFORMANCE DIRECTION:
{{#On_Camera_Performance_Director.text#}}

PRODUCTION SCHEDULE:
{{#Production_Schedule_Builder.text#}}

QUALITY TIER: {{#Start.qualityTier#}}
BUDGET: {{#Start.budgetLevel#}}
CREW: {{#Start.crewSize#}}
FILMING TIME: {{#Start.filmingDays#}}

Output ONLY the JSON object. Include the full revised production bible.
```

---

### Loop Sub-Node 2: Critique Parser
* **Node Title**: `Critique_Parser`
* **Node Type**: `code` (Python 3)
* **Input Variables**: `llm_output` = `{{#Self_Critique.text#}}`
* **Output Ports**: `current_grade`, `current_report`, `current_revised_bible`

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
            "current_revised_bible": data.get("revised_bible", "")
        }
    except Exception:
        return {
            "current_grade": "F",
            "current_report": llm_output,
            "current_revised_bible": ""
        }
```

---

### Loop Sub-Node 3: Critique Variable Assigner
* **Node Title**: `Critique_Variable_Assigner`
* **Node Type**: `assigner` (Version 2)

#### Operations
| # | Target | Operation | Value |
|---|---|---|---|
| 1 | `{{#Bible_Quality_Loop.grade#}}` | over-write | `{{#Critique_Parser.current_grade#}}` |
| 2 | `{{#Bible_Quality_Loop.report#}}` | over-write | `{{#Critique_Parser.current_report#}}` |
| 3 | `{{#Bible_Quality_Loop.revised_bible#}}` | over-write | `{{#Critique_Parser.current_revised_bible#}}` |
| 4 | `{{#Bible_Quality_Loop.revision_count#}}` | += | `1` |

---

## 6. 📦 Final Output Node

### Node 7: Final Production Bible
* **Node Title**: `Final_Production_Bible`
* **Node Type**: `llm`
* **Model Tier**: Cheap / fast
* **Temperature**: `0.3`
* **Max Tokens**: `10000`

#### System Prompt
```markdown
You are compiling the Final Production Bible — the definitive field document the creator takes to the shoot. Assemble all production planning components into one organized, clearly formatted document.

FORMAT:

---

## PRODUCTION BIBLE
**Project**: [title/topic]
**Content Type**: [type]
**Quality Tier**: [tier]
**Budget Level**: [level]
**Total Shots**: [X] (A-Roll: X | B-Roll: X | AI-gen: X)
**Production Grade**: [final critique grade]

---

### QUICK REFERENCE CARD (1 page, take to set)
| Element | Detail |
|---|---|
| Call Time | [time] |
| Wrap Time | [time] |
| Must-Have Shots | [shot numbers — the non-negotiables] |
| Primary Setup | [camera + lens + audio + key light] |
| WB Setting | [K value] |
| Critical Check | [the one thing to verify before leaving location] |
| Emergency Contact | [leave blank — creator fills in] |

---

### COMPLETE SHOT LIST
[Full shot list — all shots, all tags, all priorities]

---

### AI-GENERATED FOOTAGE PROMPTS
[All AI generation prompts with tool recommendations]

---

### EQUIPMENT PLAN
[All three tiers + camera settings table + audio setup]

---

### LOCATION & LIGHTING PLAN
[All setups with specific positions, types, color temps, ratios]
[Lighting quick reference card]

---

### ON-CAMERA PERFORMANCE DIRECTION
[Full performance direction notes]

---

### PRODUCTION SCHEDULE
[Full day-by-day schedule with contingency]
[Post-shoot checklist]
[Folder structure]

---

### QA RESULTS
Production Bible Grade: [grade]
[If grade F: ⚠️ Auditor output could not be parsed on the final pass — review manually before shooting.]
Issues resolved in revision: [summary]

---

### INTEGRATION DATA (paste into W-04 or W-05 after filming)
```
PRODUCTION BIBLE SUMMARY FOR POST-PRODUCTION:

Shoot Complete: [Date — creator fills in]
Total Footage Recorded: [GB — creator fills in]
All Must-Have Shots Captured: Yes / No — [if No: specify which are missing]

Shot Summary:
- A-Roll duration: [estimated total minutes]
- B-Roll shots: [X items]
- AI-gen prompts: [X prompts — shots to generate before editing]

Notes for Editor:
[Any important production notes: lighting inconsistency in shot X, audio issue on take Y, etc.]
```
```

#### User Prompt
```text
Compile the Final Production Bible.

PROJECT: {{#Start.scriptPackage#}}
CONTENT TYPE: {{#Start.contentType#}}
QUALITY TIER: {{#Start.qualityTier#}}
BUDGET: {{#Start.budgetLevel#}}
CREW: {{#Start.crewSize#}}

FINAL PRODUCTION BIBLE (post-critique — use this as the definitive version):
{{#Bible_Quality_Loop.revised_bible#}}

CRITIQUE GRADE: {{#Bible_Quality_Loop.grade#}}
CRITIQUE REPORT: {{#Bible_Quality_Loop.report#}}

ORIGINAL COMPONENTS (to supplement if revised_bible has gaps):
Shot List: {{#Shot_List_Extractor.text#}}
AI Prompts: {{#AI_Footage_Prompt_Writer.text#}}
Equipment: {{#Equipment_Planner.text#}}
Lighting: {{#Location_Lighting_Planner.text#}}
Performance: {{#On_Camera_Performance_Director.text#}}
Schedule: {{#Production_Schedule_Builder.text#}}

Compile the complete, organized production bible. Include the Quick Reference Card and Integration Data block.
```

**Output Port**: `text`

---

### End Node
* **Node Title**: `End`
* **Node Type**: `end`
* **Output**: `text` = `{{#Final_Production_Bible.text#}}`

---

## 7. 🔗 Node Connection & Routing Map

| Source Node | Output | Target Node | Input |
|---|---|---|---|
| `Start` | form submission | `Shot_List_Extractor` | all `{{#Start.*#}}` |
| `Shot_List_Extractor` | `text` | `AI_Footage_Prompt_Writer` | via prompt |
| `AI_Footage_Prompt_Writer` | `text` | `Equipment_Planner` | via prompt |
| `Equipment_Planner` | `text` | `Location_Lighting_Planner` | via prompt |
| `Location_Lighting_Planner` | `text` | `On_Camera_Performance_Director` | via prompt |
| `On_Camera_Performance_Director` | `text` | `Production_Schedule_Builder` | via prompt |
| `Production_Schedule_Builder` | `text` | `Bible_Quality_Loop` | enters Loop |
| `Self_Critique` | `text` | `Critique_Parser` | `llm_output` |
| `Critique_Parser` | `current_*` | `Critique_Variable_Assigner` | all current_* |
| `Bible_Quality_Loop` (exits) | `revised_bible`, `grade` | `Final_Production_Bible` | via prompt |
| `Final_Production_Bible` | `text` | `End` | `text` output |

---

## 8. 🧪 Sample Input & Verification

### Sample Input
```json
{
  "scriptPackage": "[Paste W-02 script package, or:] Topic: Why most people fail at habits — identity-based approach. 8-minute talking head with B-roll. Hook: open with phone screen showing 0 streak. Body: debunk 21-day myth, introduce identity-based habits, give implementation intention exercise. CTA: subscribe.",
  "contentType": "Talking head (direct to camera)",
  "budgetLevel": "Low — basic DSLR or mirrorless entry",
  "qualityTier": "Professional",
  "location": "Home studio",
  "crewSize": "Solo (no crew)",
  "availableEquipment": "Sony A6400, 35mm f1.8, Rode VideoMicro, 2x LED panels",
  "filmingDays": "Half day (up to 4 hours)",
  "availableAiVideoTools": "Runway Gen-3",
  "postProductionPlan": "W-04 Pre-Planning Pipeline",
  "specialRequirements": "Need a shot of a phone screen with a habit tracker app — can use screen recording or AI gen",
  "releaseDeadline": "2 weeks from now",
  "creatorProfile": ""
}
```

### Verification Checklist
1. **Shot List Extractor**: Returns a table with Shot #, type tags (A-Roll / B-Roll / AI-gen), and priority labels. Phone screen shot is flagged AI-gen candidate.
2. **AI Footage Prompt Writer**: Provides a full prompt for the phone screen shot. No faces in the prompt (common AI failure). Suggests Runway Gen-3 as specified.
3. **Equipment Planner**: Three tiers clearly labeled. Recommends Sony A6400 + 35mm in Standard tier. Camera settings table present with specific numbers (ISO, shutter, aperture) per shot type.
4. **Lighting Planner**: Returns a specific position diagram and table with K values for each light. Not generic — specific to home studio and available LED panels.
5. **Performance Director**: Section-by-section direction with specific energy levels. Mentions solo shooting adjustments (no 2nd camera operator).
6. **Schedule Builder**: Realistic 4-hour timeline. A-Roll scheduled before B-Roll. Contingency buffer present. Post-shoot checklist included.
7. **Loop**: Grade A exits. Revision pass addresses specific gaps (e.g., if schedule was too optimistic, adjusts timing).
8. **Final Package**: Quick Reference Card on page 1. Integration Data block at end, formatted for W-04/W-05.
