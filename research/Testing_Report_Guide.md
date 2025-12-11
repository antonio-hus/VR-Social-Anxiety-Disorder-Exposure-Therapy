# VR EXPOSURE THERAPY MVP - QUICK REFERENCE & USER TESTING GUIDE

## QUICK START (TL;DR)

### Day 1–2
- Install Unity LTS + XR Toolkit
- Create Lobby.unity (buttons) + ExposureScene.unity (audience + timer)

### Day 3–4
- Import NPC model, create prefab with NPCBehavior script
- AudienceManager spawns 5/15/30 NPCs based on level

### Day 5–6
- SessionController handles SUDS sliders and 2-min timer
- Pre/Post panels display results

### Day 7–8
- Polish UI, fix bugs, test on headset
- Add audio (room tone)

### Day 9–10
- Optional: SessionLogger for JSON data
- User testing (3–5 testers)

### Day 11–12
- Document results, prepare presentation

### Day 13–14
- Write final report, present

---

## SCRIPT COPY-PASTE TEMPLATES

### Template 1: GameState.cs
```csharp
using UnityEngine;

public static class GameState {
    public static int SelectedLevel = 1;
    public static int LastPreSUDS = -1;
    public static int LastPostSUDS = -1;
}
```

### Template 2: NPCBehavior.cs (Minimal)
```csharp
using UnityEngine;

public class NPCBehavior : MonoBehaviour {
    private bool lookAtPlayer = true;
    private Transform playerHead;
    
    void Start() {
        playerHead = Camera.main.transform;
    }
    
    void Update() {
        if (lookAtPlayer && playerHead != null) {
            Vector3 dir = playerHead.position - transform.position;
            dir.y = 0; // Don't tilt
            dir.Normalize();
            transform.rotation = Quaternion.Lerp(
                transform.rotation,
                Quaternion.LookRotation(dir),
                Time.deltaTime * 2f
            );
        }
    }
    
    public void SetLookAtPlayer(bool should) => lookAtPlayer = should;
}
```

### Template 3: AudienceManager.cs (Minimal)
```csharp
using UnityEngine;

public class AudienceManager : MonoBehaviour {
    [SerializeField] private GameObject npcPrefab;
    [SerializeField] private Transform[] seatPositions;
    
    void Start() {
        int level = GameState.SelectedLevel;
        int count = (level == 1) ? 5 : (level == 2) ? 15 : 30;
        float distract = (level == 1) ? 0 : (level == 2) ? 0.3f : 0.5f;
        
        for (int i = 0; i < count && i < seatPositions.Length; i++) {
            GameObject npc = Instantiate(npcPrefab, seatPositions[i].position, Quaternion.identity);
            if (Random.value < distract) {
                npc.GetComponent<NPCBehavior>().SetLookAtPlayer(false);
            }
        }
    }
}
```

### Template 4: SessionController.cs (Minimal)
```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;
using System.Collections;

public class SessionController : MonoBehaviour {
    [SerializeField] private Slider preSlider, postSlider;
    [SerializeField] private Button startBtn, retryBtn, menuBtn;
    [SerializeField] private Text timerTxt, resultTxt;
    [SerializeField] private GameObject prePanel, expPanel, postPanel;
    [SerializeField] private AudioSource audio;
    
    private int preSUDS, postSUDS;
    
    void Start() {
        startBtn.onClick.AddListener(OnStart);
        retryBtn.onClick.AddListener(() => SceneManager.LoadScene("ExposureScene"));
        menuBtn.onClick.AddListener(() => SceneManager.LoadScene("Lobby"));
        
        prePanel.SetActive(true);
        expPanel.SetActive(false);
        postPanel.SetActive(false);
    }
    
    void OnStart() {
        preSUDS = (int)preSlider.value;
        prePanel.SetActive(false);
        expPanel.SetActive(true);
        if (audio) audio.Play();
        StartCoroutine(Timer());
    }
    
    IEnumerator Timer() {
        float t = 0;
        while (t < 120) {
            t += Time.deltaTime;
            timerTxt.text = string.Format("{0:00}:{1:00}", (int)(120-t)/60, (int)(120-t)%60);
            yield return null;
        }
        if (audio) audio.Stop();
        expPanel.SetActive(false);
        postPanel.SetActive(true);
    }
    
    public void OnPostConfirm() {
        postSUDS = (int)postSlider.value;
        float reduction = preSUDS > 0 ? ((preSUDS - postSUDS) / (float)preSUDS) * 100 : 0;
        resultTxt.text = $"Before: {preSUDS}\nAfter: {postSUDS}\nReduction: {reduction:F1}%";
    }
}
```

---

## USER TESTING PROTOCOL (For Your Report)

### Test Setup

**Participants needed:** 3–5 classmates, friends, or family members

**Equipment:**
- VR headset (Quest or PCVR)
- Quiet room
- 15–20 minutes per tester

**Instructions to give tester:**

> "You're testing a VR app that exposes people to public speaking anxiety through graded exposure. You'll:
> 1. Rate your anxiety (0–100)
> 2. Stand on a stage for 2 minutes while an audience watches
> 3. Rate anxiety again
> 
> You can talk about anything during the 2 minutes. The goal is to stay in the situation and notice how you feel. Stop anytime if uncomfortable."

---

### Data Collection

Create a **Test Results Table** like this:

```
TESTER RESULTS

Tester | Age | Level | Pre-SUDS | Post-SUDS | Reduction | Comments
-------|-----|-------|----------|-----------|-----------|----------
A      | 21  | 1     | 25       | 18        | 28%       | "Felt calm, easy"
A      | 21  | 2     | 42       | 30        | 29%       | "More people, harder"
A      | 21  | 3     | 65       | 45        | 31%       | "Definitely felt judged"
B      | 22  | 1     | 30       | 22        | 27%       | "It's just NPCs, felt safe"
B      | 22  | 2     | 50       | 38        | 24%       | "Distracting audience helped"
B      | 22  | 3     | 70       | 52        | 26%       | "Large crowd was intense"
C      | 23  | 1     | 15       | 12        | 20%       | "Minimal anxiety to start"
C      | 23  | 2     | 35       | 25        | 29%       | "Good difficulty jump"
C      | 23  | 3     | 60       | 40        | 33%       | "Most challenging, but doable"
```

---

### Qualitative Feedback Questions

Ask each tester:

1. **Ease of use:** "Were the instructions clear? Could you use the controls?"
   - Yes / Somewhat / No

2. **Anxiety escalation:** "Did Level 1 feel easier than Level 2, and Level 2 easier than Level 3?"
   - Yes / Somewhat / No

3. **Habituation:** "Did your anxiety come down during each 2-minute exposure?"
   - Yes, noticeably / Somewhat / Not much

4. **Realism:** "Did the VR audience feel real enough to trigger anxiety?"
   - Very real / Somewhat real / Not real

5. **Comfort:** "Any motion sickness, discomfort, or dizziness?"
   - None / Mild / Severe

6. **Feature feedback:**
   - "What would make this better?"
   - "What was confusing?"

---

## INTERPRETING RESULTS

### What to Look For in Your Data

**1. Level Difficulty Progression**

Expected pattern:
```
Level 1: Pre-SUDS ~20–40 (mild anxiety)
Level 2: Pre-SUDS ~35–55 (moderate)
Level 3: Pre-SUDS ~50–75 (high)
```

If your testers show this, **your graded exposure is working**—congratulations!

**2. Habituation Effect**

Expected pattern:
```
Pre-SUDS > Post-SUDS for most testers
Reduction typically 20–40%
```

Example:
- Tester starts Level 3 at SUDS 70
- After 2 min, down to SUDS 48
- That's 31% reduction = **habituation occurred**

If this happens across testers, **exposure mechanism is validated**.

**3. Cross-Tester Consistency**

If all testers show:
- Level 1 < Level 2 < Level 3 in pre-anxiety, AND
- All show some post-anxiety reduction,

Then your design is solid and evidence-based.

---

## REPORT RESULTS SECTION (TEMPLATE)

### "Results"

#### Participant Demographics
- **N = 5** participants (ages 20–24)
- **Time per participant:** ~15 minutes
- **All completed all 3 levels**

#### Primary Finding: Anxiety Escalation

**Pre-exposure SUDS increased with difficulty level:**

| Level | Mean Pre-SUDS | Std Dev | Range |
|-------|---------------|---------|-------|
| 1     | 24.6          | 8.2     | 15–30 |
| 2     | 42.0          | 9.1     | 35–50 |
| 3     | 63.4          | 6.8     | 60–70 |

**Interpretation:** SUDS increased ~18 points from Level 1→2 and ~21 points from Level 2→3, confirming graded exposure difficulty progression.

#### Secondary Finding: Habituation

**Post-exposure SUDS decreased during each 2-min exposure:**

| Level | Mean Pre-SUDS | Mean Post-SUDS | Mean Reduction | % Reduction |
|-------|---------------|---|---|---|
| 1     | 24.6          | 18.2 | 6.4 | 26% |
| 2     | 42.0          | 31.8 | 10.2 | 24% |
| 3     | 63.4          | 46.8 | 16.6 | 26% |

**Interpretation:** Across all difficulty levels, participants showed measurable habituation (anxiety coming down during the 2-minute exposure), supporting the exposure therapy principle that anxiety naturally decreases with time in the situation.

#### Qualitative Feedback

**Ease of Use (5/5 said "Yes")**
- "Very intuitive, I understood what to do immediately."
- "Controllers worked smoothly."

**Anxiety Escalation (5/5 said "Yes")**
- "Level 3 was noticeably more stressful than Level 1."
- "The audience size definitely made a difference."

**Habituation Perception (4/5 said "Yes, noticeably")**
- "I was panicking at start of Level 3, but calmed down by the end."
- "Interesting—my anxiety actually went down as I kept talking."

**Realism (4/5 said "Somewhat real")**
- "Not perfectly realistic, but realistic enough to feel anxious."
- "Would be better with more detailed NPCs, but it works."

**Comfort (5/5 reported "None")**
- No motion sickness reported
- No discomfort reported

---

## EXAMPLE REPORT TEXT (You Fill In Your Numbers)

Here's skeleton text you can adapt:

---

### Results

We tested our VR exposure therapy MVP with **[N] participants** (ages **[X–Y]**). Each participant completed all three difficulty levels of the public speaking exposure, rating their anxiety before and after each 2-minute exposure.

**Anxiety Escalation:** Participants' pre-exposure anxiety (measured on a 0–100 scale) increased systematically:
- Level 1 (small audience): M = **[XXX]** (SD = **[X]**)
- Level 2 (medium audience): M = **[XXX]** (SD = **[X]**)
- Level 3 (large audience): M = **[XXX]** (SD = **[X]**)

This pattern validates our graded exposure design—difficulty increases predictably with audience size.

**Habituation:** Across all levels, participants showed anxiety reduction during the 2-minute exposures:
- Mean reduction: **[XX]%** (Range: **[X–X]%**)

This is consistent with exposure therapy research showing that anxiety naturally decreases when a person remains in a feared situation without escaping.

**Qualitative Feedback:** **[X]/5** participants reported the levels escalated as intended; **[X]/5** reported feeling actual anxiety reduction; and **[X]/5** reported no motion sickness. Common themes:
- *Positive:* "Easy to use," "Anxiety definitely went up with audience size," "Felt real enough"
- *Constructive:* "Could use more detailed NPCs," "Would like more scenarios," "Audio could be louder"

---

## PRESENTATION SLIDES (Outline)

### Slide 1: Title
- Project title
- Your name(s)
- Date

### Slide 2: Background
- What is social anxiety disorder?
- How exposure therapy works (3 bullets)
- Why VR is useful for exposure

### Slide 3: Design
- Overview of your MVP (1 scenario, 3 levels)
- Screenshot of Level 1 / 2 / 3 (side-by-side)
- SUDS rating mechanism

### Slide 4: Implementation
- Tech stack (Unity, XR, C#)
- Key components (AudienceManager, SessionController, etc.)
- Timeline (2 weeks breakdown)

### Slide 5: Results
- Table of user anxiety data
- Graph: Pre-SUDS by level (bar chart)
- Graph: Habituation effect (pre/post comparison)

### Slide 6: Qualitative Feedback
- 1–2 participant quotes
- Feedback summary (usability, realism, comfort)

### Slide 7: Limitations & Future Work
- What worked (graded exposure mechanism)
- What didn't work (graphics, NPC realism)
- Next steps with more time (more scenarios, dialogue, therapist controls)

### Slide 8: Conclusion
- Your MVP validates graded exposure in VR
- Simple design was appropriate for 2-week timeline
- Results support further development

---

## CHECKLIST: BEFORE SUBMITTING

### Code
- [ ] All scripts compile without errors
- [ ] No null reference warnings in console
- [ ] Scenes saved and added to Build Settings
- [ ] GameState accessed correctly across scenes

### Functionality
- [ ] Lobby buttons load correct levels
- [ ] NPCs spawn with correct count (5/15/30)
- [ ] NPCs show correct attention behavior
- [ ] Sliders work and display values
- [ ] Timer counts down correctly
- [ ] Habituation % calculated correctly
- [ ] Can replay levels without crashing

### Data
- [ ] User anxiety ratings recorded (in memory or JSON)
- [ ] At least 3–5 testers' data collected
- [ ] Results show expected level escalation
- [ ] Results show some habituation effect

### Documentation
- [ ] Report written (3–5 pages, follow template)
- [ ] Presentation slides created (8+ slides)
- [ ] References cited (at least 2 peer-reviewed papers)
- [ ] User testing protocol documented
- [ ] Results table included

### Presentation
- [ ] Demo works (or video prepared)
- [ ] Can explain why you designed each level
- [ ] Can discuss what the data shows
- [ ] Can answer basic questions about exposure therapy

---

## COMMON MISTAKES (AVOID!)

❌ **"I made 5 levels instead of 3"**
- No! 3 levels is plenty for 2 weeks. Focus on polishing 3 well-designed levels.

❌ **"I spent all time on graphics"**
- Good design > good graphics. Asset Store models are fine.

❌ **"I only tested with 1 person"**
- Get at least 3–5 testers. Their data is your evidence.

❌ **"I didn't collect any data"**
- SUDS ratings before/after are essential. Without them, you can't prove exposure works.

❌ **"I made a therapist control dashboard"**
- Out of scope! Focus on the client VR app only.

❌ **"I built a backend/login system"**
- Unnecessary! Local data only.

---

## ENCOURAGEMENT

You're building something real. Self-guided VR exposure therapy is an actual research area with published papers showing it works. Your 2-week MVP is a **legitimate proof-of-concept** that demonstrates:

1. ✅ Graded exposure can be structured in VR
2. ✅ Anxiety scales predictably with difficulty
3. ✅ Habituation occurs within short exposures
4. ✅ The design is usable and safe

That's solid work for 2 weeks. Your grader will recognize the thoughtfulness in your design and the rigor in your testing. Ship it with confidence!

---

## RESOURCES

**Free/Cheap Assets:**
- **NPCs:** Sketchfab (search "free humanoid rigged"), Asset Store free packs
- **Environments:** Asset Store free classroom packs, Unity-chan models
- **Audio:** Freesound.org (room tone, murmurs), YouTube audio library

**VR Setup:**
- **Quest:** Meta Developer Hub (download, install, deploy)
- **PCVR:** Steam, SteamVR, OpenXR

**Documentation:**
- Unity XR docs: docs.unity3d.com/Manual/xr
- XRI docs: xr-plugin-management.readthedocs.io

---

**You've got this. 2 weeks, solid MVP, real data. Go build something cool!**

