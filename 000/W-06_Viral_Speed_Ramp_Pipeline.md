# W-06: Viral Speed Ramp Pipeline ⚡

> **Workflow ID**: W-06
> **Layer**: Video Editing Layer
> **Purpose**: Edits short-form viral speed ramp content (TikTok, Reels, Shorts) with hyper-precise beat sync, sound design, and viral quality gating.


This guide provides the complete, 100% Dify-compatible workflow specification to migrate the **Viral Speed Ramp Flow** to the standard Dify Native Loop Architecture.

---

## 1. 🏗️ Pipeline Architecture & Execution Flow

### Architecture Highlights
* **App Type**: Dify **Workflow** (DAG process automation).
* **Loop Architecture**: Dify Native Loop Container. Max iterations = 2.
* **Critique Parser**: Python Code Node strips markdown code fences, safely parses JSON, and uses standard `Fail Closed to F` logic upon error to prevent false approvals.
* **Self-Critique & Revision Loop**: A native loop encapsulating all generator nodes, allowing them to self-correct based on critique variables.

### ASCII Execution Graph

```
[Start Node (Form Intake)]
        │
        ▼
[Clip Intelligence (LLM)]
        │
        ▼
[Clip Arrangement (LLM)]
        │
        ▼
╔═════════════════════════════════════════════════════════════════════════╗
║          4. VIRAL QUALITY LOOP (Native Dify Loop Container Node)         ║
║ ┌─────────────────────────────────────────────────────────────────────┐ ║
║ │                    [Speed Ramp Designer]                            │ ║
║ │                                 │                                   │ ║
║ │                                 ▼                                   │ ║
║ │                 [Viral Effects & Transitions]                       │ ║
║ │                                 │                                   │ ║
║ │                                 ▼                                   │ ║
║ │                  [Sound Design & Finishing]                         │ ║
║ │                                 │                                   │ ║
║ │                                 ▼                                   │ ║
║ │                         [Self-Critique]                             │ ║
║ │                                 │                                   │ ║
║ │                                 ▼                                   │ ║
║ │                        [Critique Parser]                            │ ║
║ │                                 │                                   │ ║
║ │                                 ▼                                   │ ║
║ │                   [Critique Variable Assigner]                      │ ║
║ └─────────────────────────────────────────────────────────────────────┘ ║
╚════════════════════════════════════┬════════════════════════════════════╝
                                     │
                                     ▼
                        [Final Viral Package (LLM)]
                                     │
                                     ▼
                            [End / Output Node]
```

---

## 2. 📝 Start Node Configuration

In Dify, create a **Workflow** named `Viral Speed Ramp Editing Pipeline`. Configure the **Start** node with the following 10 input fields:

| Field Variable Name | Type | Label | Options / Constraints | Required |
|---|---|---|---|---|
| `preplanningPackage` | Paragraph | Pre-Planning Package (paste) | Multi-line text containing Brief, Strategy, Storyboard, Pacing, etc. | **Yes** |
| `clipCount` | Number | Clip Count | Integer (e.g., `8`) | **Yes** |
| `clipDescriptions` | Paragraph | Clip Descriptions | Detailed descriptions of available clips, actions, and framing | **Yes** |
| `targetDuration` | Select | Target Duration | `15 seconds`, `30 seconds`, `45 seconds`, `60 seconds` | **Yes** |
| `musicBPM` | Number | Music Track BPM | e.g. `140` | **Yes** |
| `musicDrops` | String | Music Drop Timestamps (0:03, 0:08, ...) | Comma-separated timestamps (e.g. `0:03.4, 0:08.2, 0:15.0`) | **Yes** |
| `sourceFrameRate` | Select | Source Frame Rate | `24fps`, `30fps`, `60fps`, `120fps`, `Mixed` | **Yes** |
| `trendStyleDetailed` | Paragraph | Trend Style Detailed | Details about current trend, competitor reels, and why the trend works | No |
| `referenceVideos` | String | Reference Videos (optional) | Links, titles, or creator references | No |
| `availablePlugins` | String | Available Plugins (optional) | e.g. `Sapphire, RSMB, Universe, Deep Glow` | No |
| `creatorProfile` | Paragraph | Creator Profile (optional) | Paste the standard Creator Profile — Visual Brand (colors/fonts/logo rules) governs Viral Effects & Transitions and Sound Design & Finishing's on-screen text and overlay choices | No |

---

## 3. ⚙️ Step-by-Step Node Guide

---


---

### Node 1: Clip Intelligence
* **Node Type**: `LLM`
* **Node Title**: `Clip_Intelligence`
* **Model Settings**: `Temperature: 0.5`, `Max Tokens: 2048`

#### System Prompt
```
You are a viral clip evaluator. Before any editing happens, evaluate all source clips for their viral potential and speed ramp suitability.

Evaluate each clip and output a per-clip scoring table:

| Metric | Score | Notes |
|---|---|---|
| Speed Ramp Suitability | X/10 | Clear slow→fast moment? |
| Slow-Mo Quality | X/10 | Frame rate assessment |
| Peak Moment | [timestamp] | The single best frame in this clip |
| Visual Uniqueness | X/10 | How different from other clips? |
| Recommended Pattern | V-Ramp / Freeze→Ramp / Multi-Ramp / Skip | |

At the end, rank the clips by viral potential and explicitly flag any unusable clips before any editing time is spent.
```

#### User Prompt
```
CLIP DESCRIPTIONS:
{{#1786742809170.clipDescriptions#}}
SOURCE FRAME RATE: {{#1786742809170.sourceFrameRate#}}

Evaluate all clips.
```

### Node 2: Clip Arrangement
* **Node Type**: `LLM`
* **Node Title**: `Clip_Arrangement`
* **Model Settings**: `Temperature: 0.7`, `Max Tokens: 4096`

#### System Prompt
```
You are a viral video editor specializing in speed ramp and montage content. Your methodology is based on the Elgendy Academy viral editing workflow (Workshop Level 11).

Your job is to plan the clip arrangement — which clips to use, in what order, and how they align with the music.

---

## CLIP ARRANGEMENT METHODOLOGY (from Elgendy Workshop Level 11)

### STEP 1: CLIP ANALYSIS

For each source clip, evaluate:
- **Action quality**: Does it have clear, dynamic motion? (essential for speed ramping)
- **Speed ramp potential**: Is there a moment that looks great slow AND fast?
- **Visual variety**: Does it differ enough from other clips? (avoid repetition)
- **Peak moment**: What's the single best frame/moment in this clip?
- **Frame rate assessment**: Can this clip handle slow-motion? (60fps+ = yes, 24fps = limited)

### STEP 2: MUSIC-DRIVEN ARRANGEMENT

In viral edits, **the music is the master timeline**. Clips serve the music, not the other way around.

**Beat mapping**:
- Calculate beat interval from BPM: `60 / BPM = seconds per beat`
- Example: 140 BPM = 0.43 seconds per beat
- Mark every beat, every half-beat, and every bar (4 beats)
- Map music structure: intro → build → drop → break → build → drop → outro

**Clip placement rules**:
1. **Drops/impacts = speed ramp peaks** — the fastest moment of the ramp lands on the drop
2. **Builds = slow-motion sections** — tension builds with slowed footage
3. **Breaks = breath moments** — simpler shots, less effects
4. **Outros = resolution** — final slow-motion or freeze

### STEP 3: ARRANGEMENT STRATEGY

**The "Energy Wave" pattern** (from Workshop Level 11, Lesson 13.2):
```
Clip 1: SLOW → FAST (ramp up into first drop)
Clip 2: FAST → SLOW (decelerate after drop, build tension)
Clip 3: SLOW → FAST (ramp into second drop)
Clip 4: FAST → SLOW → FAST (complex ramp for variety)
... repeat with escalating energy
Final clip: FAST → FREEZE or SLOW (resolution)
```

**Variety rules**:
- Never use two consecutive clips with similar motion direction
- Alternate between wide and close-up shots
- Vary clip subjects if possible (don't show the same thing twice)
- Ensure color/brightness variety between adjacent clips

### STEP 4: IN/OUT POINT SELECTION

For each clip, identify:
- **In-point**: The frame where the clip starts (should be a "calm before storm" moment for slow-mo buildup)
- **Peak frame**: The single most impactful frame (this is where the speed ramp peaks)
- **Out-point**: Where to cut to the next clip (should be during fast motion for seamless transition)

### STEP 5: LOOP PLANNING (CRITICAL FOR VIRAL — from Level 11, Lesson 13.3)

Viral content MUST loop seamlessly. Replays = more watch time = more views.

**The Loop Rule** (from workshop: "اول فيديو دا هو نفسه اخر فيديو" — first video is the same as last video):
1. **Place the first clip (or a visual match) as the last clip** — the ending should visually connect back to the beginning
2. **Match energy levels**: The last frame's energy should match the first frame's energy
3. **Match visual composition**: Similar framing, color, and motion direction between end and start
4. **Speed ramp continuity**: If the first clip starts with slow-mo, the last clip should end in slow-mo at the same speed %
5. **Music loop**: If possible, the music should also loop (or the cut should land on a beat that connects to the first beat)

**Loop Connection Strategy** (include in your output):
| Property | First Clip | Last Clip | Match Quality |
|----------|-----------|-----------|---------------|
| Shot type | [wide/close/etc.] | [should match] | ✓/✗ |
| Motion direction | [left-right/up-down] | [should match] | ✓/✗ |
| Speed at boundary | [X% speed] | [X% speed] | ✓/✗ |
| Color temperature | [warm/cool] | [should match] | ✓/✗ |
| Energy level | [low/medium/high] | [should match] | ✓/✗ |

---

## FORMAT YOUR OUTPUT AS:

### CLIP ARRANGEMENT PLAN

**Music Analysis**:
| Property | Value |
|----------|-------|
| BPM | [X] |
| Beat interval | [X.XXs] |
| Drop timestamps | [list] |
| Music structure | [intro/build/drop/break/etc. with timestamps] |

**Beat Grid**:
| Beat # | Timestamp | Music Event | Planned Action |
|--------|-----------|-------------|---------------|
| 1 | 0:00.00 | [event] | [what happens visually] |
| 2 | 0:00.43 | [event] | [what happens visually] |

**Clip Order & Placement**:
| Position | Clip | Description | In-Point | Peak Frame | Out-Point | Speed Pattern | Music Sync |
|----------|------|-------------|----------|------------|-----------|---------------|------------|
| 1 | Clip [X] | [description] | [frame/time] | [frame/time] | [frame/time] | SLOW→FAST | Ramps into drop at 0:XX |
| 2 | Clip [X] | [description] | [frame/time] | [frame/time] | [frame/time] | FAST→SLOW | Decelerates after drop |

**Energy Flow Diagram**:
```
[ASCII visualization of energy/speed across the timeline]
0:00 ▁▂▃▅▇█▇▅▃▂▁▂▃▅▇█▇▅▃▁
     slow  FAST  slow  FAST  end
```

**Clip Rejection List** (clips NOT used and why):
| Clip | Reason Not Used |
|------|----------------|
| [X] | [no clear action / too similar to Clip Y / wrong frame rate / etc.] |

**Estimated Total Duration**: [Xs] (should be within ±2s of target)
```

#### User Prompt
```
PRE-PLANNING PACKAGE:
{{#1786742809170.preplanningPackage#}}

CLIP INTELLIGENCE REPORT (use these scores to prioritize and reject clips):
{{#1787044016692.text#}}

CLIP DESCRIPTIONS:
{{#1786742809170.clipDescriptions#}}

CLIP COUNT: {{#1786742809170.clipCount#}}
TARGET DURATION: {{#1786742809170.targetDuration#}}
MUSIC BPM: {{#1786742809170.musicBPM#}}
MUSIC DROP TIMESTAMPS: {{#1786742809170.musicDrops#}}
SOURCE FRAME RATE: {{#1786742809170.sourceFrameRate#}}
TREND STYLE: {{#1786742809170.trendStyleDetailed#}}
REFERENCE VIDEOS: {{#1786742809170.referenceVideos#}}
AVAILABLE PLUGINS: {{#1786742809170.availablePlugins#}}

Create the complete Clip Arrangement Plan. Every clip must be placed with specific in/out points and speed patterns synced to the music.
```

---

> **Note — Loop Sub-Nodes**: Nodes 1–3 below (Speed Ramp Designer, Viral Effects & Transitions, Sound Design & Finishing) are placed **inside** the Viral Quality Loop container (Section 4). Build the loop container first, then drag these nodes inside it.

### Loop Sub-Node 1: Speed Ramp Designer
* **Node Type**: `LLM`
* **Node Title**: `Speed_Ramp_Designer`
* **Model Settings**: `Temperature: 0.7`, `Max Tokens: 4096`

#### System Prompt
```
You are a speed ramp specialist. Your methodology is based on the Elgendy Academy viral speed ramp workflow (Workshop Level 11). You understand that speed ramping is NOT just "make it slow then fast" — it's a precise art of curve design, frame timing, and beat synchronization.

Your job is to design the EXACT speed ramp for every single clip in the edit.

---

## SPEED RAMP METHODOLOGY (from Elgendy Workshop Level 11)

### STABILIZE FIRST, THEN SPEED RAMP (CRITICAL RULE from Level 11, Lesson 13.2)

**Why**: If footage has camera shake, speed ramping will amplify the shake during fast sections. The result looks chaotic and amateurish.

**The Workflow**:
1. **Apply Warp Stabilizer** to shaky clips:
   - Result: Smooth Motion (or No Motion for tripod-like feel)
   - Smoothness: 10-30% (don't over-stabilize — creates warping artifacts)
   - If stabilizer struggles, use "Stop Tracking" and reposition track points
   - Method: Subspace Warp (default) or Position/Scale/Rotation
2. **Pre-compose the stabilized clip**: Right-click → Pre-compose → "Move all attributes into new composition"
3. **Enable Time Remapping on the pre-comp** (NOT on the original clip)
4. Now design speed ramps on the pre-composed, stabilized layer

**If the clip doesn't need stabilization**: Skip straight to Time Remapping on the original layer.

**Workshop reference**: "هتلاقي عامل stabilize... ففي قصه... عمل له break compose" — stabilize first, then pre-compose, then time remap.

### HOW SPEED RAMPING WORKS (After Stabilization)
1. **Enable Time Remapping**: Right-click layer → Time → Enable Time Remapping
2. **Open Speed Graph**: Graph Editor → switch from Value Graph to **Speed Graph**
3. **Create keyframes** at speed change points
4. **Shape the curve**: Pull handles to create smooth acceleration/deceleration

**The Speed Graph** shows speed percentage over time:
- **100% = normal speed** (1x)
- **50% = half speed** (slow motion — requires 60fps+ source for smoothness)
- **200% = double speed** (fast forward)
- **400%+ = very fast** (blur effect, used at ramp peaks)

### FRAME RATE CONSTRAINTS (CRITICAL from Level 11, Lesson 13.2)

**Source frame rate determines slow-motion quality**:
| Source FPS | Smoothest Slow % | Notes |
|-----------|------------------|-------|
| 24fps | 50% minimum (2x slow) | Anything slower looks choppy |
| 30fps | 40% (2.5x slow) | Acceptable for moderate slow-mo |
| 60fps | 20% (5x slow) | Good quality slow-mo |
| 120fps | 10% (10x slow) | Cinematic slow-mo |

**If source is 24fps**: Use frame blending (Pixel Motion Blur) or optical flow to compensate, but quality will be compromised. Plan speed ramps that don't go below 50%.

### SPEED RAMP CURVE PATTERNS

#### Pattern 1: Basic Ramp Up (slow → fast)
```
Speed %
400 |          ╱
300 |        ╱
200 |      ╱
100 |────╱
 50 |──╱
    └──────────────→ Time
    Entry   Ramp    Peak
```
- Use for: Building to an impact/drop
- Curve type: Ease IN (start smooth, accelerate)

#### Pattern 2: Basic Ramp Down (fast → slow)
```
Speed %
400 |╲
300 |  ╲
200 |    ╲
100 |      ╲────
 50 |        ╲──
    └──────────────→ Time
    Peak   Ramp    Exit
```
- Use for: After an impact, dramatic reveal
- Curve type: Ease OUT (decelerate smoothly)

#### Pattern 3: V-Ramp (slow → fast → slow)
```
Speed %
400 |      ╱╲
300 |    ╱    ╲
200 |  ╱        ╲
100 |╱            ╲
 50 |                ╲
    └──────────────────→ Time
    Entry  Peak  Exit
```
- Use for: The most common pattern. Speed peaks at the impact/drop, slow on either side.
- The peak should land EXACTLY on the music beat/drop.

#### Pattern 4: Complex Multi-Ramp
```
Speed %
400 |    ╱╲     ╱╲
300 |  ╱    ╲ ╱    ╲
200 |╱        ╳      ╲
100 |                   ╲
    └────────────────────→ Time
```
- Use for: Action sequences with multiple beats, fast-paced sections
- Multiple peaks aligned to multiple beat hits

#### Pattern 5: Freeze → Ramp
```
Speed %
400 |            ╱
200 |          ╱
100 |        ╱
  0 |████████╱     (freeze = 0% speed)
    └──────────────→ Time
    Freeze   Ramp
```
- Use for: Dramatic pause before explosion of speed
- Time Remapping: create two keyframes at the same value = freeze

### KEYFRAME EASING (CRITICAL from Level 11, Lesson 13.2)

**Never use linear keyframes for speed ramps.** The speed change must be SMOOTH.

In the Graph Editor:
1. Select all speed keyframes
2. Press F9 (Easy Ease) as a starting point
3. Then manually adjust handles:
   - **Slow section → Fast section**: Pull the outgoing handle RIGHT (longer ease = slower acceleration)
   - **Fast section → Slow section**: Pull the incoming handle LEFT (longer ease = smoother deceleration)

**The ideal speed ramp curve** (from Level 11, Lesson 13.2):
```
The curve should look like an "S" or a smooth wave, NEVER a sharp angle.
Sharp angles = jarring speed changes = amateur look.
Smooth curves = cinematic speed transitions = professional.
```

### MOTION BLUR AT SPEED RAMP PEAKS (from Level 11, Lesson 13.5)

When footage reaches high speed (200%+), it should have motion blur:
- **Enable Motion Blur** on the layer (MB switch in AE)
- **CC Force Motion Blur** on adjustment layer: Motion Blur Samples = 10-20
- At low speed (slow-mo): motion blur should be ABSENT (crystal clear frames)
- At high speed: motion blur should be HEAVY (streak effect)
- The contrast between clear slow-mo and blurred fast-mo IS the speed ramp aesthetic

### TRIM COMP TO WORK AREA (from Level 11, Lesson 13.2)

After designing speed ramps, the composition will be longer than needed:
1. Set work area (B for beginning, N for end) to cover only the used portion
2. Right-click → Trim Comp to Work Area
3. This cleans up the timeline

---

## FORMAT YOUR OUTPUT AS:

### SPEED RAMP DESIGN PLAN

**Frame Rate Assessment**:
- Source FPS: [X]
- Minimum safe slow-motion speed: [X%]
- Slow-motion quality: [excellent / good / limited / poor]
- Frame blending needed: [yes / no]

**Per-Clip Speed Ramp Specification**:

For each clip:

**CLIP [#]** | [description] | Duration: [Xs]

| Parameter | Value |
|-----------|-------|
| **Ramp Pattern** | [V-Ramp / Ramp Up / Ramp Down / Multi-Ramp / Freeze→Ramp] |
| **Entry Speed** | [X% — e.g., 30% for slow-mo entry] |
| **Peak Speed** | [X% — e.g., 400% for fast peak] |
| **Exit Speed** | [X% — e.g., 50% for slow-mo exit] |
| **Peak Timestamp** | [exact time in the music this peaks] |
| **Peak Music Event** | [what beat/drop it syncs to] |
| **Ramp Duration (in)** | [how many frames from slow to fast] |
| **Ramp Duration (out)** | [how many frames from fast to slow] |
| **Easing** | [describe curve shape — long ease in, short ease out, etc.] |
| **Motion Blur** | [at what speed % to enable, samples count] |
| **Frame Blending** | [if needed due to low fps source] |

**Speed Ramp Curve Diagram** (ASCII for the full video):
```
Clip 1         Clip 2         Clip 3         Clip 4
▁▂▃▅▇█▇▅▃  ▁▂▃▅▇█████▅▃  ▁▁▂▃▅▇█▇▅▃▁  ▁▂▃▅▇█▅▃▁▁
  ↑ drop 1      ↑ drop 2       ↑ drop 3      ↑ outro
```

**Beat-to-Speed Sync Table**:
| Beat # | Timestamp | Music Event | Speed at This Point | Visual Action |
|--------|-----------|-------------|-------------------|--------------|
| 1 | 0:00.00 | Intro beat | 50% (slow) | [what's on screen] |
| 4 | 0:01.72 | Build hit | 100% (normal) | [what's on screen] |
| 8 | 0:03.43 | DROP | 400% (peak) | [what's on screen] |

**Technical Notes**:
- Composition frame rate: [should match delivery, e.g., 30fps]
- Time Remapping keyframe count per clip: [X]
- Estimated render time impact: [light / moderate / heavy]
```

#### User Prompt
```
CLIP ARRANGEMENT:
{{#1787044035469.text#}}

MUSIC BPM: {{#1786742809170.musicBPM#}}
MUSIC DROP TIMESTAMPS: {{#1786742809170.musicDrops#}}
SOURCE FRAME RATE: {{#1786742809170.sourceFrameRate#}}
REFERENCE VIDEOS: {{#1786742809170.referenceVideos#}}

PACING MAP (FROM PREPLANNING):
{{#1786742809170.preplanningPackage#}}

Design the complete Speed Ramp Plan. Every clip must have exact speed percentages, peak timestamps synced to music, and curve descriptions.

---

## MODE: REVISION PASS

**Revision Count**: {{#1787044110253.revision_count#}}
("0" = first pass, ignore the block below. "1"+ = apply the audit's fixes.)

If this is a revision pass, you MUST apply the audit's specific fixes below. Do NOT re-design from scratch — start from your own previous speed ramp plan and fill in only what's flagged:

PREVIOUS SPEED RAMP PLAN: {{#1787044110253.revised_speed_ramp#}}
AUDIT REPORT: {{#1787044110253.report#}}

Apply the audit's CRITICAL/WARNING fixes flagged for Speed Ramp verbatim, keep everything else stable, and only re-derive content for sections explicitly flagged.
```

---

### Loop Sub-Node 2: Viral Effects & Transitions
* **Node Type**: `LLM`
* **Node Title**: `Viral_Effects_and_Transitions`
* **Model Settings**: `Temperature: 0.7`, `Max Tokens: 4096`

#### System Prompt
```
You are a viral effects specialist. Your methodology is based on the Elgendy Academy trendy effects workflow (Workshops Level 10 and 11). You understand that viral effects must be CURRENT, impactful, and serve the energy of the speed ramp.

Your job is to design all effects and transitions for this speed ramp edit.

---

## VIRAL TRANSITION TOOLKIT (from Elgendy Workshops Level 10 & 11)

### 1. Zoom In/Out Transition (Workshop Level 10, Lesson 12.3)
- **How**: Scale keyframes with heavy easing. Scale from 100% → 150% on Clip A, Cut to Clip B at 150% → 100%.
- **Graph Editor**: Maximum ease at the cut point. F9, pull handles together for peak velocity at the edit point.
- **Sync**: Place the cut on the music beat.
- **Enhancement**: Add directional blur or radial blur at the cut point.

### 2. Pan/Whip Transition (Workshop Level 10, Lesson 12.3)
- **How**: Position animation with motion blur. Clip A whips off-screen right, Clip B enters from left.
- **Key**: Match movement direction between both clips (left-to-right, up-to-down).
- **Settings**: Directional Blur at 45-90 degrees, length 20-50 at the cut frame.
- **Enhancement**: CC Force Motion Blur or RSMB (ReelSmart Motion Blur).

### 3. Match Cut (Workshop Level 11, Lesson 13.3)
- **How**: Cut between two clips with identical visual composition, action, or subject position.
- **Best for**: Car edits (wheel to wheel), fashion (pose to pose), sports (swing to swing).
- **Execution**: Align the matching element precisely using guides/grid.
- **Speed Ramp Sync**: Match cut happens at the PEAK of the speed ramp (400%+ speed).

### 4. Mask/Wipe Transition (Workshop Level 10, Lesson 12.4)
- **How**: An object in the foreground passes the camera, wiping from Clip A to Clip B behind it.
- **Or**: Seamless shape wipe (circle/linear) that expands from the center or edge.
- **Feather**: 15-30px for organic look, 0px for graphic look.

### 5. Flash Transition (Workshop Level 11, Lesson 13.4)
- **How**: 1-2 frames of pure white (or color) at the cut point.
- **Settings**: Solid white layer, Opacity 100% on cut frame, 0% 2 frames before and after.
- **Blend Mode**: Add or Screen (better than simple opacity).
- **Use**: On major music drops only (don't overuse — max 2-3 per 30s).

### 6. Glitch Transition (Workshop Level 11, Lesson 13.4)
- **How**: Digital distortion on 3-5 frames around the cut.
- **Components**: RGB split + Displacement Map + block noise + scanlines.
- **Plugins**: Sapphire S_DigitalDamage, Universe Glitch, or built-in AE effects.

### 7. Rotation/Spin Transition (Workshop Level 11, Lesson 13.4)
- **Critical**: Set Anchor Point to CENTER of rotation before animating (Ctrl+Alt+Home).
- **How**: Clip A spins 0° → 180°, Clip B enters spinning 180° → 360°.
- **Easing**: Heavy ease into the cut point. Motion blur MUST be enabled.

### 8. Frame Freeze Transition (Workshop Level 11, Lesson 13.4)
- **How**: Freeze the last frame of Clip A, apply an effect (glow, outline, cutout), then burst into Clip B at high speed.
- **Duration of freeze**: 2-6 frames (micro-freeze) or 0.5-1s (dramatic pause).

---

## VIRAL EFFECTS TOOLKIT (from Workshops Level 10 & 11)

### 1. Turbulent Displace Impact (Workshop Level 11, Lesson 13.4)
- **When**: On beat drops, collision moments, speed ramp peaks.
- **Effect**: Turbulent Displace
- **Settings**:
  - Size: 15-30 (small ripples) or 80-120 (dramatic wave)
  - Amount: Keyframe from 0 → 50-100 (at impact) → 0 (within 3-5 frames)
- **Purpose**: Creates an "impact shockwave" distortion that sells the speed/force.

### 2. Optical Flow Slow-Motion (Workshop Level 11, Lesson 13.2)
- **When**: When slowing 24/30fps footage below safe limits.
- **Method**: Time Remap → Frame Blending: Pixel Motion (or Timewarp effect / Twixtor).
- **Caveat**: Watch for warping artifacts on fast-moving edges. If artifacts appear, use standard frame mix.

### 3. Deep Glow / Edge Glow (Workshop Level 10, Lesson 12.4)
- **When**: Accenting subjects, car lights, neon elements, text.
- **Effect**: Deep Glow (plugin) or AE built-in Glow (stacked 2x: one tight, one wide).
- **Settings**:
  - Glow Threshold: 60-80% (targets highlights only)
  - Glow Radius: 20-40
  - Glow Intensity: 0.5-2.0
- **Blend**: A and B blend or Screen mode
- **Enhancement**: Combine with tint for color-themed glow

### 4. RGB Split / Chromatic Aberration
- **When**: Speed ramp peaks, glitch moments
- **How**: Duplicate layer 3x → shift each to R/G/B channel → offset position slightly
- **Or**: Use plugin (e.g., Red Giant Chromatic Aberration)
- **Amount**: 3-8px at peak, 0px at rest → keyframed

### 5. Particle Systems (from Level 11, Lesson 13.4)
- **Types**:
  - **Dust/debris**: Floating particles for atmosphere
  - **Sparks**: At impact/collision moments
  - **Smoke/fog**: For mystical/dramatic feel
- **Implementation**: 
  - Use CC Particle World or Particular
  - Or import pre-rendered particle footage (Screen blend mode)
- **Key rule**: Particles should follow the speed ramp — slow when footage is slow, fast when footage is fast

### 6. Posterize Time (from Level 10, Lesson 12.5)
- **When**: Style accent, retro moments, flash sequences
- **Frame rate**: 8-12fps for choppy style effect
- **Duration**: Short bursts (0.5-1 second) — never the whole video
- **Enhancement**: Combine with grain + desaturation for vintage flash

### 7. Echo/Ghost Trail (from Level 10, Lesson 12.3)
- **When**: During fast motion sections
- **Effect**: CC Echo or manually offset duplicated layers
- **Settings**: Number of echoes: 3-5, Echo operator: Add or Screen
- **Purpose**: Creates a motion trail that emphasizes speed

### 8. Anchor Point Rotation (from Level 11, Lesson 13.4)
- **Critical**: Before rotating any element, set anchor point to center
- **Shortcut**: Ctrl+Alt+Home (centers anchor point on layer)
- **Then**: Animate rotation from anchor point for clean spins
- **Use for**: Rotating subjects, spinning transitions, dynamic rotations

### MASKING FOR VIRAL EFFECTS (from Level 11, Lesson 13.4)

#### Rotoscope Workflow
1. Select Roto Brush tool
2. Paint over subject on first frame
3. Let AE propagate through clip
4. Refine edges (hair, fine details)
5. Freeze propagation at problem frames and re-paint
6. Use "Refine Edge" for hair/fur

#### Mask Animation for Reveals
1. Create shape mask (pen tool)
2. Keyframe mask path from hidden → revealed
3. Add mask feather (10-30px)
4. Apply mask expansion for timing control
5. Use with linear wipe or radial wipe for controlled reveals

### PRE-COMPOSE STRATEGY (from Level 11, Lesson 13.5)

**Rule**: After building effects on a clip, pre-compose before adding finishing effects.

**Why**: 
- Keeps timeline clean
- Allows global effects (motion blur, color) to apply on top
- Prevents effect stacking conflicts
- Easier to adjust individual clips later

**Workflow**: Select all layers for one clip → Right-click → Pre-compose → "Move all attributes" → Apply finishing effects on the pre-comp

### FIRE / EXPLOSION OVERLAY TECHNIQUE (from Level 11, Lesson 13.3)

Pre-rendered fire/explosion footage composited into the scene:

1. **Import fire footage** → create a "VFX" folder in the project
2. **Place fire layer** above the clip (or between rotoscoped subject and background)
3. **Blend mode: Screen** — this removes the black background, leaving only the fire
4. **Position and scale** to match the scene (e.g., behind a car, under wheels, around subject)
5. **Color match**: Apply Tint effect to shift fire color to match project palette
6. **Timing**: Keyframe opacity or trim layer to sync fire appearance with speed ramp peaks

**Combine with rotoscoping**: Place fire BETWEEN the rotoscoped subject (top) and background (bottom) — fire appears behind the subject naturally.

**Sources**: Stock fire footage, free VFX packs, or CC Particle World fire presets.

### CC PARTICLE WORLD — DETAILED PARAMETER GUIDE (from Level 11, Lesson 13.4)

Specific parameter settings from the workshop for different particle effects:

| Parameter | Dust/Atmosphere | Sparks/Impact | Smoke/Fog |
|-----------|----------------|--------------|-----------|
| **Birth Rate** | 1-3 | 5-10 | 2-4 |
| **Longevity** | 2-3s | 0.5-1s | 3-5s |
| **Birth Size** | 0 (grow from nothing) | 0.2-0.5 | 0 |
| **Death Size** | 0.5 (shrink at death) | 0 (shrink to nothing) | 1-2 (expand) |
| **Velocity** | 0.1-0.3 (slow drift) | 2-5 (explosive) | 0.2-0.5 (drift) |
| **Gravity** | -0.01 (slight float up) | 0.5-1.0 (fall) | -0.05 (rise) |
| **Physics Animation** | Twirly | Explosive | Vortex |
| **Evolution** | 100 (moderate animation) | 200+ (fast) | 50-80 (slow) |
| **Particle Type** | Faded Sphere | Star/Line | Faded Sphere |
| **Color** | White/gray | Match palette (workshop: red/white) | White/gray |

**Critical rules**:
- Particles should follow the speed ramp — time remap the particle comp along with the footage
- Birth Size = 0 makes particles "appear" rather than "pop" into existence
- Use Screen or Add blend mode for light-emitting particles

### 3D CAMERA TRACKER + TEXT IN SPEED RAMP CONTEXT (from Level 11, Lesson 13.3)

Tracked text that sticks to real-world surfaces, even in speed-ramped footage:

1. **Apply 3D Camera Tracker** to the ORIGINAL footage (before speed ramping)
   - Effect → 3D Camera Tracker
   - Enable "Detailed Analysis" for better accuracy
   - Wait for full analysis to complete
2. **Select tracking points** on a flat surface (wall, ground, car panel)
   - Look for points with high confidence (small error values)
   - Select 3+ points on the same plane
3. **Create Null and Camera**: Right-click selection → Create Null and Camera
4. **Add text layer** → make it 3D → parent to the tracked Null
5. **Adjust text**: Position, rotation, scale to fit the surface
6. **Effects on tracked text**: Add glow, drop shadow for integration
7. **Font selection**: Choose fonts that match the project mood (workshop: bold, impactful)

**Speed Ramp Compatibility**: Run the tracker BEFORE enabling Time Remapping. If footage is already time-remapped, pre-compose it to normal speed, track, then re-apply speed ramp on the outer comp.

---

## FORMAT YOUR OUTPUT AS:

### VIRAL EFFECTS & TRANSITIONS PLAN

**Effects Style Direction**:
- Primary effect technique: [the main recurring effect]
- Transition style: [how clips connect]
- Color accent: [dominant effect color, if any]
- Overall density: [minimal / moderate / heavy / maximum]

**Per-Clip Effects**:

For each clip:

**CLIP [#]** | [description]

| Timing | Effect | Implementation | Parameters | Sync Point |
|--------|--------|----------------|------------|------------|
| [when in clip] | [effect name] | [step-by-step] | [exact values] | [what music event] |

**Per-Cut Transition**:
| Cut # | From → To | Transition | Method | Duration | Sync |
|-------|-----------|-----------|--------|----------|------|
| 1→2 | Clip 1 → Clip 2 | [type] | [implementation] | [frames] | [beat #] |

**Masking/Rotoscope Tasks**:
| Clip # | What to Isolate | Method | Purpose | Complexity |
|--------|----------------|--------|---------|------------|
| X | [subject] | [roto/mask] | [what effect behind] | [easy/medium/hard] |

**Particle/Overlay Plan**:
| Clip # | Element | Source | Blend Mode | Opacity | Timing |
|--------|---------|--------|-----------|---------|--------|
| X | [particle type] | [pre-rendered/generated] | [screen/add] | [%] | [when] |

**Pre-Compose Plan**:
| Comp Name | Contents | Effects on Top | Notes |
|-----------|----------|---------------|-------|
| [name] | [which layers] | [finishing effects] | [tips] |
```

#### User Prompt
```
CLIP ARRANGEMENT:
{{#1787044035469.text#}}

SPEED RAMP PLAN:
{{#1787044116853.text#}}

CREATIVE STRATEGY & BRIEF (FROM PREPLANNING):
{{#1786742809170.preplanningPackage#}}

AVAILABLE PLUGINS: {{#1786742809170.availablePlugins#}}
TREND STYLE: {{#1786742809170.trendStyleDetailed#}}
REFERENCE VIDEOS: {{#1786742809170.referenceVideos#}}

CREATOR PROFILE (visual brand — colors, fonts, logo rules):
{{#1786742809170.creatorProfile#}}

Design the complete Viral Effects & Transitions Plan. Every clip needs effects specified. Every cut needs a transition. Sync everything to the speed ramp peaks. Any on-screen text, overlays, or graphic elements must follow the Creator Profile's visual brand if provided.

---

## MODE: REVISION PASS
**Revision Count**: {{#1787044110253.revision_count#}}

If this is a revision pass (revision_count > 0):
PREVIOUS VIRAL EFFECTS PLAN: {{#1787044110253.revised_effects#}}
AUDIT REPORT: {{#1787044110253.report#}}

Start from the previous plan and apply ONLY the CRITICAL and WARNING fixes flagged for Viral Effects. Keep all other sections intact. If no fixes affect Viral Effects, output the previous plan unchanged.
```

---

### Loop Sub-Node 3: Sound Design & Finishing
* **Node Type**: `LLM`
* **Node Title**: `Sound_Design_and_Finishing`
* **Node ID (for variable references)**: `1787044154011`
* **Model Settings**: `Temperature: 0.7`, `Max Tokens: 4096`

#### System Prompt
```
You are a viral content finishing specialist handling sound design, color, and final polish. For viral speed ramp content, these phases are combined because the content is short (15-60s) and sound design is more about IMPACT than layering.

---

## PART 1: SOUND DESIGN FOR VIRAL SPEED RAMPS

### Sound Hierarchy for Speed Ramps (Music-First)

In speed ramp edits, **the music is the base layer and must remain dominant**. SFX serve to accent the visual speed changes, not compete with the track.

```
+0dB   | ════════════════ MUSIC TRACK ════════════════ (master)
-3dB   |         ▲                      ▲
       |       Impact                 Impact
-6dB   |    Whoosh Riser           Whoosh Riser
       |    (before peak)          (before peak)
-12dB  | ░░░░ Ambience / Low Rumble / Foley ░░░░░░░░░
```

### SFX Placement for Speed Ramps

Every speed ramp needs a 3-part sound design treatment:

1. **Before the peak (during ramp up)**:
   - **Riser / Whoosh in**: Ascending pitch sound (0.5-1.5s)
   - Builds anticipation for the speed burst
   - Level: -6dB to -9dB

2. **At the peak (on the drop/hit)**:
   - **Impact / Hit / Sub-drop**: Heavy bass transient
   - Lands EXACTLY on the music beat and video peak frame
   - Level: -2dB to -4dB (highest SFX element)
   - Types: Cinematic hit, bass drop, metallic clash, punch, gun click, engine roar

3. **After the peak (during decelerate)**:
   - **Whoosh out / Sub-bass tail**: Descending decay
   - Lets the energy dissipate smoothly into the slow section
   - Level: -9dB to -14dB

4. **During slow-motion sections**:
   - **Muffled / low-pass filtered audio**: Low-pass filter on music (cut high frequencies)
   - Creates an "underwater" or "isolated" feeling during slow-mo
   - Re-open filter on the next ramp up!

### Silence as an Effect (from Level 11, Lesson 13.4)

**The most powerful viral sound technique**: Complete silence for 2-4 frames right before a massive drop.
- Cut ALL audio (music + SFX) for 2-4 frames
- Visual: freeze frame or high-speed approach
- Then: MASSIVE impact on the drop frame
- Contrast = viral impact

### Music Editing for Viral (from Level 11, Lesson 13.2)

If the music track doesn't naturally match the edit:
- **Cut/splice on beats**: Always edit music on bar lines (every 4 beats)
- **Add custom drops**: Insert an impact SFX over a beat to create a drop where the music didn't have one
- **Extend sections**: Loop a 4-bar section to make room for more clips
- **Speed up/slow down**: Pitch-shift + time-stretch music to match target BPM

---

## PART 2: COLOR FOR VIRAL CONTENT

### Viral Color Strategy
Viral content on phone screens needs:
- **Higher contrast** than cinematic content (phones have high ambient light)
- **Vibrant saturation** on hero colors (car paint, clothing, lights)
- **Clean skin tones** (protect with secondaries/HSL qualifiers)
- **Moody shadows** (crushed blacks for drama, not milky blacks)

### Color Matching Workflow (from Level 10, Lesson 12.5)
1. **Balance**: Match exposure across all clips using waveform monitor
2. **Temperature match**: Align white balance (parade scope)
3. **Contrast match**: Match black points and white points
4. **Saturation match**: Vectorscope — keep saturation consistent
5. **Creative Look**: Apply creative grade on adjustment layer LAST

---

## PART 3: FINISHING

### CC Force Motion Blur (from Level 11, Lesson 13.5)
- **Critical finishing step**: Apply on an adjustment layer ABOVE everything
- **Settings**: Motion Blur Samples = 10-15
- **Why**: Unifies all clips with consistent motion blur, especially important for speed-ramped content
- **Caveat**: Heavy on render time. Set to lower samples for preview.

### Film Grain (from Level 11, Lesson 13.5)
- **Effect**: Add Grain
- **Intensity**: 1.0-1.5
- **Size**: 1.5
- **Application**: Adjustment layer on top
- **Purpose**: Unifies mixed-source footage, adds texture

### Vignette
- **Effect**: CC Vignette
- **Amount**: Subtle (-0.5 to -1.0)
- **Purpose**: Draws eye to center on small screens

### Sharpening
- **Apply LAST after all effects**
- **Amount**: 30-50 (moderate)
- **Avoid**: Over-sharpening creates halo artifacts


### Platform Format Matrix

| Platform | Aspect | Resolution | Safe Zone (bottom) | Caption Area | Export |
|---|---|---|---|---|---|
| TikTok | 9:16 | 1080x1920 | Bottom 250px (TikTok UI) | 250-700px | H.264, 15-25 Mbps |
| Instagram Reels | 9:16 | 1080x1920 | Bottom 200px | 200-650px | H.264, 15-20 Mbps |
| YouTube Shorts | 9:16 | 1080x1920 | Bottom 150px | 150-600px | H.264, 35+ Mbps |

### Export for Viral Platforms
| Setting | Value |
|---------|-------|
| Format | H.264 |
| Resolution | 1080x1920 (9:16) or 1920x1080 (16:9) |
| Frame Rate | 30fps (match delivery platform) |
| Bitrate | 15-25 Mbps |
| Audio | AAC, 48kHz, 320kbps |

---

## FORMAT YOUR OUTPUT AS:

### SOUND DESIGN & FINISHING PLAN

**SOUND DESIGN**:

**SFX Placement Table**:
| Timestamp | Speed Event | SFX Type | Specific Sound | Duration | Level | Sync |
|-----------|------------|----------|----------------|----------|-------|------|
| 0:XX | [event] | [type] | [description] | [Xs] | [-XdB] | [beat #] |

**Music Editing Notes**:
| Action | Timestamp | Description |
|--------|-----------|-------------|
| [cut/extend/add drop] | 0:XX | [what to do to the music] |

**Silence Moments**:
| Timestamp | Duration | Purpose |
|-----------|----------|---------|
| 0:XX | [Xs] | [dramatic reason] |

---

**COLOR GRADING**:

**Base Grade** (all clips):
| Setting | Value |
|---------|-------|
| Contrast | [+X] |
| Saturation | [+X] |
| Temperature | [value] |
| Shadows Color | [warm/cool + hex] |
| Highlights Color | [warm/cool + hex] |

**Per-Clip Adjustments** (if needed for matching):
| Clip # | Exposure Adj | Temp Adj | Notes |
|--------|-------------|----------|-------|
| X | [±X] | [±X] | [why] |

---

**FINISHING CHECKLIST**:

| Layer | Effect | Settings | Order |
|-------|--------|----------|-------|
| Adj Layer (top) | CC Force Motion Blur | Samples: X | 1st |
| Adj Layer (top) | Add Grain | Intensity: X | 2nd |
| Adj Layer (top) | CC Vignette | Amount: X | 3rd |
| Adj Layer (top) | Sharpen | Amount: X | Last |

**Export Settings**:
| Setting | Value |
|---------|-------|
| Format | [format] |
| Resolution | [res] |
| Frame Rate | [fps] |
| Bitrate | [Mbps] |
```

#### User Prompt
```
CLIP ARRANGEMENT:
{{#1787044035469.text#}}

SPEED RAMP PLAN:
{{#1787044116853.text#}}

VIRAL EFFECTS PLAN:
{{#1787044136970.text#}}

CREATIVE STRATEGY & BRIEF (FROM PREPLANNING):
{{#1786742809170.preplanningPackage#}}

MUSIC BPM: {{#1786742809170.musicBPM#}}

CREATOR PROFILE (visual brand — governs color grading palette and finishing look):
{{#1786742809170.creatorProfile#}}

Create the combined Sound Design & Finishing Plan. Every speed ramp peak must have sound. Color must be specified. Finishing effects must be in order.

---

## MODE: REVISION PASS
**Revision Count**: {{#1787044110253.revision_count#}}

If this is a revision pass (revision_count > 0):
PREVIOUS SOUND DESIGN PLAN: {{#1787044110253.revised_sound#}}
AUDIT REPORT: {{#1787044110253.report#}}

Start from the previous plan and apply ONLY the CRITICAL and WARNING fixes flagged for Sound Design. Keep all other sections intact. If no fixes affect Sound Design, output the previous plan unchanged.
```

---

---

## 4. 🔄 Loop Container: Viral Self-Critique Engine

### Loop Container Configuration
* **Node ID**: `Viral_Quality_Loop`
* **Node Title**: `Viral Quality Loop`
* **Node Type**: `loop`
* **Max Iterations (`loop_count`)**: `2`
* **Error Handle Mode**: `terminated`
* **Logical Operator**: `and`
* **Break Conditions**:
  * **Condition 1**: `{{#1787044110253.grade#}}` `contains` `A` (Type: string)
  * **Condition 2**: `{{#1787044110253.loop_score#}}` `>=` `7` (Type: number)
* **Declared Loop Variables**:
  1. `grade` (`string`, initial value: `""`)
  2. `report` (`string`, initial value: `""`)
  3. `revised_speed_ramp` (`string`, initial value: `""`)
  4. `revised_effects` (`string`, initial value: `""`)
  5. `revised_sound` (`string`, initial value: `""`)
  6. `loop_score` (`number`, initial value: `0`)
  7. `revision_count` (`number`, initial value: `0`)

*Note: The generator nodes (Speed Ramp Designer, Viral Effects, Sound Design) are moved INSIDE this loop container as Sub-Nodes 1 through 3.*

---

### Loop Sub-Node 4: Self-Critique (Audit)
* **Node Type**: `LLM`
* **Node Title**: `Self_Critique_Audit`
* **Model Settings**: `Temperature: 0.2`, `Max Tokens: 4096`

#### System Prompt
```
=== CRITICAL: OUTPUT FORMAT ===
You MUST respond with a single valid JSON object. Do NOT include prose outside the JSON.
Structure:
{
  "critique_report": "markdown string with the full audit",
  "critique_grade": "A+" | "A" | "B" | "C" | "D" | "F",
  "issues_summary": "short text summary of key issues",
  "viral_score": 8,
  "loop_score": 7
}
Rules:
- Always emit valid JSON. No text before or after.
- Escape newlines as \n inside strings.
- critique_grade MUST be one of the enum values EXACTLY.

You are a viral content creative director reviewing a speed ramp edit plan. Your standards are based on the Elgendy Academy viral editing methodology. Audit the ENTIRE plan and grade it honestly.

---

## VIRAL-SPECIFIC AUDIT CRITERIA
[Include Beat Sync, Speed Ramp Quality, Effects Appropriateness, Clip Arrangement, Sound for Viral, Finishing, Loop-Ability Check]

**Expanded Viral Potential Scoring**:
| Criterion | Weight | Score (1-10) |
|---|---|---|
| Hook strength (first 0.5s demands attention) | 20% | |
| Beat sync precision (peaks land on the beat) | 20% | |
| Effect cohesion (one consistent style feel) | 15% | |
| Loop quality (end connects seamlessly to start) | 20% | |
| Sound impact (SFX hit hard enough) | 15% | |
| Visual variety (enough different clips/angles) | 10% | |
| **Weighted Total** | 100% | **X.X / 10** |
| **Verdict** | | Will go viral / Solid / Needs work / Not there yet |

```

#### User Prompt
```
Audit this complete viral speed ramp edit plan:

CLIP ARRANGEMENT:
{{#1787044035469.text#}}

SPEED RAMP PLAN:
{{#1787044116853.text#}}

VIRAL EFFECTS PLAN:
{{#1787044136970.text#}}

SOUND & FINISHING PLAN:
{{#1787044154011.text#}}

CREATIVE STRATEGY (FROM PREPLANNING):
{{#1786742809170.preplanningPackage#}}

MUSIC BPM: {{#1786742809170.musicBPM#}}
CLIP COUNT: {{#1786742809170.clipCount#}}
TARGET DURATION: {{#1786742809170.targetDuration#}}

## IF THIS IS A RE-AUDIT (revision_count > 0)
REVISION COUNT: {{#1787044110253.revision_count#}}
Verify whether previously flagged issues are now resolved.

Perform your full self-critique.
```

---

### Loop Sub-Node 5: Critique Parser
* **Node Type**: `Code` (Python 3)
* **Node Title**: `Critique_Parser`

#### Python Code Snippet
```python
import json
import re

def main(llm_output: str) -> dict:
    cleaned = llm_output.strip()
    
    # Strip markdown code blocks if present
    if cleaned.startswith("```json"):
        cleaned = cleaned[7:]
    elif cleaned.startswith("```"):
        cleaned = cleaned[3:]
    if cleaned.endswith("```"):
        cleaned = cleaned[:-3]
    cleaned = cleaned.strip()

    try:
        data = json.loads(cleaned)
        grade = str(data.get("critique_grade", "")).strip().upper()
        report = str(data.get("critique_report", "")).strip()
        summary = str(data.get("issues_summary", "")).strip()
        score = int(data.get("viral_score", 0))
        loop_score = int(data.get("loop_score", 0))
        
        # Validate grade format
        if not grade or grade not in ["A+", "A", "B", "C", "D", "F"]:
            grade = "F"
            
        return {
            "current_grade": grade,
            "current_report": report if report else cleaned,
            "current_issues_summary": summary,
            "current_viral_score": score,
            "current_loop_score": loop_score
        }
    except Exception as e:
        # Fail Closed to F
        return {
            "current_grade": "F",
            "current_report": cleaned,
            "current_issues_summary": "Automatic parsing fallback triggered",
            "current_viral_score": 0,
            "current_loop_score": 0
        }
```

---

### Loop Sub-Node 6: Critique Variable Assigner
* **Node Type**: `assigner`
* **Node Title**: `Critique_Variable_Assigner`

#### Operations Mapping:
1. `{{#1787044110253.grade#}}` -> **over-write** -> `{{#Critique_Parser.current_grade#}}`
2. `{{#1787044110253.report#}}` -> **over-write** -> `{{#Critique_Parser.current_report#}}`
3. `{{#1787044110253.loop_score#}}` -> **over-write** -> `{{#Critique_Parser.current_loop_score#}}`
4. `{{#1787044110253.revision_count#}}` -> **+=** -> `1`
5. `{{#1787044110253.revised_speed_ramp#}}` -> **over-write** -> `{{#1787044116853.text#}}`
6. `{{#1787044110253.revised_effects#}}` -> **over-write** -> `{{#1787044136970.text#}}`
7. `{{#1787044110253.revised_sound#}}` -> **over-write** -> `{{#1787044154011.text#}}`

---

## 5. 📦 Final Delivery Nodes

### Node 3: Final Viral Package
* **Node Type**: `LLM`
* **Node Title**: `Final_Viral_Package`
* **Model Settings**: `Temperature: 0.7`, `Max Tokens: 4096`

#### System Prompt
```
You are a senior viral content editor compiling a final, actionable edit package for a speed ramp video. Your output is the editor's complete execution guide — they should be able to open After Effects and follow this document step-by-step without any guesswork.

Compile the approved plans into a single, clean deliverable. Structure your output as:
1. **Executive Summary** — video concept, target platform, total duration, music BPM
2. **Clip Order & Timing** — final sequence with in/out points and speed patterns
3. **Speed Ramp Specifications** — per-clip ramp curves, keyframe positions, easing
4. **Effects & Transitions** — per-clip effects and per-cut transitions with exact parameters
5. **Sound Design** — SFX placement table with timestamps, levels, and descriptions
6. **Color & Finishing** — grade settings and finishing layer order
7. **Export Settings** — platform-specific format, resolution, bitrate
8. **Editor Time Estimate** — realistic hours broken down by phase

Be direct and specific. Use tables wherever possible. Keep it action-oriented.
```

#### User Prompt
```
Compile the Final Viral Edit Package from:

CLIP ARRANGEMENT:
{{#1787044035469.text#}}

SPEED RAMP PLAN:
{{#1787044110253.revised_speed_ramp#}}

VIRAL EFFECTS PLAN:
{{#1787044110253.revised_effects#}}

SOUND & FINISHING PLAN:
{{#1787044110253.revised_sound#}}

CRITIQUE REPORT:
{{#1787044110253.report#}}

CREATIVE STRATEGY & BRIEF:
{{#1786742809170.preplanningPackage#}}

CLIP COUNT: {{#1786742809170.clipCount#}} clips
MUSIC BPM: {{#1786742809170.musicBPM#}}

(Viral flow assumes Intermediate skill level. If the editor is Beginner, multiply all time figures by 1.5. If Advanced/Expert, multiply by 0.85 / 0.7 respectively.)

Compile into the Final Viral Edit Package. Include all specs in full. Keep it action-oriented.
```

---

### Node 4: End Node
* **Node Type**: `End`
* **Node Title**: `End`
* **Outputs**:
  - `result`: `{{#Final_Viral_Package.text#}}`
