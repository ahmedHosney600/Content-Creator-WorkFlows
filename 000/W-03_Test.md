Listed directory 000
Viewed W-02%20Test:1-44
Searched for "Sample Input"
Viewed W-03_Production_Planning_Filming_Guide.md:1103-1133

To test the **W-03_Production_Planning_Filming_Guide** workflow, you can use the built-in **Sample Input & Verification Checklist** detailed in Section 8 of your guide.

Here is the step-by-step process:

### 1. Provide Sample Input
Run the workflow in your environment (like Dify) and paste the following JSON payload into the Start Node fields (or fill the corresponding form fields manually):

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

### 2. Run the Verification Checklist
Once the workflow processes the input, review the output at each stage against this checklist to ensure all nodes are functioning correctly:

- [ ] **Shot List Extractor**: Did it return a table with Shot #, type tags (A-Roll / B-Roll / AI-gen), and priority labels? Is the phone screen shot flagged as an AI-gen candidate?
- [ ] **AI Footage Prompt Writer**: Does it provide a full prompt for the phone screen shot? Are there no faces in the prompt (to avoid common AI failures)? Does it suggest Runway Gen-3 as specified?
- [ ] **Equipment Planner**: Are the three tiers clearly labeled? Does it recommend the Sony A6400 + 35mm in the Standard tier? Is the camera settings table present with specific numbers (ISO, shutter, aperture) for each shot type?
- [ ] **Lighting Planner**: Does it return a specific position diagram and a table with K values for each light? Is it specific to a home studio and the available LED panels (not generic)?
- [ ] **Performance Director**: Is there section-by-section direction with specific energy levels? Does it mention adjustments for a solo shoot (no 2nd camera operator)?
- [ ] **Schedule Builder**: Is it a realistic 4-hour timeline? Is A-Roll scheduled before B-Roll? Is a contingency buffer present? Does it include a post-shoot checklist?
- [ ] **Quality Loop**: Did a Grade A exit the loop successfully? If it returned Grade B/C, did the revision pass address specific gaps (e.g., adjusting timing if the schedule was too optimistic)?
- [ ] **Final Package**: Is the Quick Reference Card on page 1? Is the Integration Data block present at the end and formatted for the W-04/W-05 workflows?

You can also run a test using the actual output from your **W-02_Script_Writer** run to test the continuity between workflows.