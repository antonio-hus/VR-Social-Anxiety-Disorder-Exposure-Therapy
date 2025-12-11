# VR EXPOSURE THERAPY FOR SOCIAL ANXIETY - 2-WEEK SCHOOL PROJECT MVP

## PART 1: RESEARCH FOUNDATION (SIMPLIFIED)

### What is Social Anxiety Disorder (SAD)?

Social Anxiety Disorder is an intense, persistent fear of being judged or embarrassed in social situations. It's more than shyness—people with SAD experience:

- **Physical symptoms:** racing heart, trembling, sweating, blushing
- **Cognitive symptoms:** fear of negative evaluation, catastrophic thoughts, self-consciousness
- **Behavioral symptoms:** avoidance of social situations, reliance on "safety behaviors"
- **Impact:** affects work, school, relationships, and overall quality of life

**Prevalence:** About 7–13% of the population experiences SAD at some point; about 2–3% have clinically diagnosed SAD.

**Key insight for treatment:** SAD is maintained by an **avoidance cycle**:
```
Social Trigger → Anxiety → Avoidance/Escape → Temporary relief 
→ Fear strengthens → Next situation feels even scarier
```

### How Exposure Therapy Works

Exposure therapy breaks the avoidance cycle by:

1. **Habituation** – Your body gets used to the anxiety; the nervous system naturally calms down over time.
2. **Extinction Learning** – Your brain learns that the feared outcome (being judged, embarrassing yourself) doesn't actually happen.
3. **Self-Efficacy** – "I survived this; I can handle it."

**Three key principles:**

- **Graduated exposure:** Start with easier situations, progress to harder ones.
- **Sustained exposure:** Stay in the situation long enough for anxiety to come down (typically 20–30 minutes).
- **Response prevention:** Don't escape or use "safety behaviors" (like looking at your phone or hiding).

### Why VR is Effective for Exposure Therapy

**Advantages:**

- **Safety:** No real social danger, completely controlled environment.
- **Reproducibility:** Same scenario can be run multiple times for practice.
- **Customization:** Difficulty can be adjusted (audience size, reactions, judgment level).
- **Measurement:** Easy to track anxiety before and after, notice habituation.
- **Accessibility:** No need to find real audience, no real-world logistics.
- **Low cost to scale:** Once built, can be used by many people.

**Research evidence:** Studies show VR exposure therapy produces **large effect sizes** (improvement similar to in-person therapy) with **low dropout rates** (people stick with it because it feels safer and more controllable).[web:68][web:90]

**Key mechanism:** When people use VR, their brain responds as if the situation is real—anxiety genuinely rises, which allows habituation to occur. This is called "presence" (sense of "being there").

### Realistic MVP Scope

For a 2-week school project focusing on **public speaking anxiety**:

- **1 VR scene** (classroom/lecture hall).
- **3 difficulty levels** (5 audience members → 15 → 30).
- **Simple exposure flow:** rate anxiety before, stay for ~2 minutes, rate after.
- **Local data only:** save anxiety ratings to a file or memory for your report.
- **No backend, no login, no cloud.**

This matches **self-guided VR exposure therapy research**, where people do simple exposures without a therapist present and still see significant anxiety reduction.[web:68]

---

## PART 2: MVP DESIGN SPECIFICATION

### Core Concept

**"VR Public Speaking Exposure Trainer"**

A self-guided VR app where the user:

1. Stands on a stage in front of a virtual audience.
2. Chooses a difficulty level (Level 1: small audience → Level 3: large audience).
3. Rates their anxiety (0–100) before the exposure.
4. Talks/presents for ~2 minutes while the audience watches.
5. Rates their anxiety after.
6. Sees the before/after values to understand habituation.
7. Can retry or go back to menu.

### Features (Realistic for 2 weeks)

#### Must-Have (MVP Core)

1. **Lobby Scene**
   - Title and brief explanation (write it yourself, don't copy long sources)
   - 3 buttons: "Level 1 / Level 2 / Level 3 / Quit"
   - Short disclaimer: "This is for educational purposes. Stop if you feel unwell."

2. **Exposure Scene (Reused for all 3 levels)**
   - Simple classroom or lecture hall (Unity Asset Store)
   - Stage with podium or speaker area
   - Audience of varying size based on selected level

3. **Audience NPCs (Simple)**
   - 1–2 basic humanoid models from Asset Store (or free models)
   - Duplicated to fill seats
   - Simple idle animations (breathing, head movement)
   - Attention behavior:
     - **Level 1:** All facing player, neutral/friendly
     - **Level 2:** 70% face player, 30% glance around or fidget
     - **Level 3:** 50% face player, 20% look at phones, 30% turned away

4. **Anxiety Rating UI**
   - **Pre-exposure panel:** "How anxious do you feel? (0–100)"
     - Slider controlled by thumbstick or A/X button
     - Confirm button
   - **During exposure:** Timer countdown (2 minutes)
   - **Post-exposure panel:** "How anxious do you feel now? (0–100)"
     - Slider
     - Display: "Before: X | After: Y | Habituation: (X-Y)%"
     - Buttons: "Retry Level" / "Back to Menu"

5. **Simple Ambient Audio**
   - Soft room tone (very quiet for Level 1)
   - Light murmurs for Level 2–3
   - Can be just a single looped audio file

#### Nice-to-Have (If Time Permits)

- Second environment (boardroom instead of lecture hall)
- Save SUDS ratings to JSON file for report generation
- Vignette effect or comfort settings
- Simple instruction panel in VR on first run
- Pre/post session questionnaire (text panel)

#### NOT included (Out of scope)

- Cloud backend
- User login/authentication
- Therapist dashboard
- Biofeedback integration (wearables)
- Speech recognition or analysis
- Multiple conversation trees or dialogue
- Customization UI for scenario parameters
- Mobile version

---

### Level Details

#### Level 1: Small Audience (SUDS ~20–30)

| Aspect | Details |
|--------|---------|
| **Audience size** | 5 people in front rows |
| **Audience behavior** | All sitting, all looking at you, neutral friendly faces |
| **Ambient noise** | Very quiet, maybe soft breathing |
| **Duration** | 2 minutes |
| **Purpose** | Get comfortable being watched in a safe context |
| **Clinical rationale** | Mild but real exposure; builds confidence |

#### Level 2: Medium Audience (SUDS ~40–50)

| Aspect | Details |
|--------|---------|
| **Audience size** | ~15 people spread across rows |
| **Audience behavior** | 70% watching you, 30% glance away, fidget, shift in seat |
| **Ambient noise** | Light murmurs, quiet shuffling |
| **Duration** | 2 minutes |
| **Purpose** | More realistic—people don't all stare constantly |
| **Clinical rationale** | Moderate exposure; tests ability to continue despite inattention |

#### Level 3: Large Audience (SUDS ~60–70)

| Aspect | Details |
|--------|---------|
| **Audience size** | ~30 people filling most visible rows |
| **Audience behavior** | 50% watching, 20% looking at phones/notes, 30% facing away |
| **Ambient noise** | Louder murmurs, coughing, chair shifting |
| **Duration** | 2 minutes |
| **Purpose** | High challenge; tests performance under "real" scrutiny |
| **Clinical rationale** | High exposure to feared outcome (being ignored/judged) |

---

## PART 3: UNITY IMPLEMENTATION PLAN (2-WEEK TIMELINE)

### Days 1–2: Setup & Scene

**What to do:**
- Install **Unity LTS 2022** (or 2023)
- Install **XR Interaction Toolkit** and **OpenXR** plugin
- Set build target: Quest 2/3 or SteamVR
- Create folder structure:
  ```
  Assets/
  ├── Scenes/
  │   ├── Lobby.unity
  │   └── ExposureScene.unity
  ├── Scripts/
  ├── Prefabs/
  ├── Models/
  ├── Audio/
  └── UI/
  ```
- Create **Lobby scene:**
  - XR Origin (with controllers)
  - Canvas (world-space) with:
    - Title text
    - Brief explanation (2–3 sentences)
    - Three buttons for Level 1/2/3
    - Quit button
- Test that you can see the scene in headset

**Deliverable:** Lobby loads and displays buttons.

---

### Days 3–4: Audience & NPC System

**What to do:**
- Import simple classroom/lecture hall from Asset Store (search "classroom VR" or "lecture hall")
- Import or find 1–2 free humanoid models (e.g., from Sketchfab or Asset Store freebies)
- Create **NPC prefab:**
  ```
  NPCPrefab
  ├─ Mesh (humanoid model)
  ├─ Animator (with simple idle animation)
  └─ LookAtPlayer script (makes NPC look toward player)
  ```
- Create **AudienceManager.cs script:**
  ```csharp
  public enum Level { L1, L2, L3 }
  public Level currentLevel;
  
  void Start() {
    SpawnAudience(currentLevel);
  }
  
  void SpawnAudience(Level level) {
    int count = 0;
    float distraction = 0f; // % of NPCs not looking at player
    
    if (level == Level.L1) {
      count = 5; distraction = 0f;
    } else if (level == Level.L2) {
      count = 15; distraction = 0.3f;
    } else if (level == Level.L3) {
      count = 30; distraction = 0.5f;
    }
    
    for (int i = 0; i < count; i++) {
      Vector3 seatPos = GetSeatPosition(i, count);
      GameObject npc = Instantiate(npcPrefab, seatPos, Quaternion.identity);
      
      // Some NPCs look away
      if (Random.value < distraction) {
        npc.GetComponent<LookAtPlayer>().enabled = false;
      }
    }
  }
  ```
- Create seat positions (hard-code ~30 positions in rows in front of stage)
- Test spawning different audience sizes

**Deliverable:** Can load ExposureScene and see 5/15/30 NPCs sitting and looking at player.

---

### Days 5–6: UI & Flow

**What to do:**
- Create **SessionController.cs** to manage exposure flow:
  ```csharp
  public class SessionController : MonoBehaviour {
    public enum Phase { PreExposure, Exposure, PostExposure }
    
    [SerializeField] private Canvas preExposurePanel;
    [SerializeField] private Canvas exposurePanel;
    [SerializeField] private Canvas postExposurePanel;
    
    private int preSUDS = -1;
    private int postSUDS = -1;
    private float exposureDuration = 120f; // 2 minutes
    
    void Start() {
      ShowPreExposurePanel();
    }
    
    void ShowPreExposurePanel() {
      // Show slider (0–100)
      // "How anxious do you feel?"
      // Confirm button → StartExposure()
    }
    
    void StartExposure(int sudsRating) {
      preSUDS = sudsRating;
      preExposurePanel.enabled = false;
      exposurePanel.enabled = true;
      StartCoroutine(ExposureTimer());
    }
    
    IEnumerator ExposureTimer() {
      float elapsed = 0f;
      while (elapsed < exposureDuration) {
        elapsed += Time.deltaTime;
        UpdateTimerDisplay(exposureDuration - elapsed);
        yield return null;
      }
      ShowPostExposurePanel();
    }
    
    void ShowPostExposurePanel() {
      exposurePanel.enabled = false;
      postExposurePanel.enabled = true;
      // Show slider (0–100)
      // Display: "Before: X | After: Y | Reduction: Z%"
      // Buttons: Retry / Menu
    }
  }
  ```
- Create **SUDS slider UI:**
  - World-space Canvas with Slider component (0–100)
  - Text showing current value
  - Confirm button (A button / trigger)
- Create **timer display:**
  - Text showing MM:SS countdown
  - Corner of screen or center
- Create **post-exposure panel:**
  - Show before/after SUDS
  - Calculate and show percentage reduction: `(before - after) / before * 100`
  - Buttons: "Retry Level", "Back to Menu"
- Hook up button presses to load scenes or reset

**Deliverable:** Can do a full cycle: rate anxiety → 2-min timer → see results.

---

### Days 7–8: Polish & Scene Transitions

**What to do:**
- Fix UI layout and make it readable in VR
  - Ensure text is large enough (~60–80pt)
  - Position UI at comfortable viewing distance (~1.5m in front of player)
  - Test button hit zones
- Add **audio:**
  - Import ambient room tone (search "free room tone audio" on Freesound)
  - Play it during exposure
  - For Level 2/3, add very quiet murmurs (can be same audio but layered or just described to grader)
- Create **SceneManager.cs:**
  ```csharp
  public static class GameState {
    public static int SelectedLevel = 1;
  }
  ```
  - On Level button click in Lobby: `GameState.SelectedLevel = levelNumber; SceneManager.LoadScene("ExposureScene");`
  - In ExposureScene, read `GameState.SelectedLevel` and spawn appropriate audience
- Add **fade/transition:**
  - Simple fade-to-black between scenes (optional but nice)
- Test on actual headset (VR discomfort check)
  - Confirm no frame rate drops
  - Check for dizziness/discomfort

**Deliverable:** Smooth flow from Lobby → Level selection → Exposure → Results → Back to Lobby.

---

### Days 9–10: Data Logging (Optional)

**What to do (if time permits):**
- Create **SessionData.cs:**
  ```csharp
  [System.Serializable]
  public class SessionData {
    public int level;
    public int preSUDS;
    public int postSUDS;
    public long timestampMS;
  }
  
  public static class SessionLogger {
    public static List<SessionData> sessions = new List<SessionData>();
    
    public static void LogSession(int level, int preSUDS, int postSUDS) {
      sessions.Add(new SessionData {
        level = level,
        preSUDS = preSUDS,
        postSUDS = postSUDS,
        timestampMS = System.DateTime.Now.Ticks
      });
      
      SaveToFile();
    }
    
    static void SaveToFile() {
      string json = JsonUtility.ToJson(new SessionList { data = sessions }, true);
      System.IO.File.WriteAllText("sessions.json", json);
    }
  }
  ```
- Call `SessionLogger.LogSession(...)` at end of each exposure
- Retrieve and display in report later

**Deliverable:** `sessions.json` file with session history.

---

### Days 11–12: User Testing & Refinement

**What to do:**
- Have 3–5 classmates/friends test the app
- Collect feedback:
  - Is it easy to understand?
  - Do they feel anxiety increase from Level 1 → 3?
  - Any motion sickness or discomfort?
  - Any UI/button confusion?
- Note their anxiety ratings:
  - Does Level 1 have lower SUDS than Level 3?
  - Do most people see habituation (pre > post)?
- Fix critical bugs:
  - UI not showing properly
  - Audio not playing
  - Scene transitions broken
- Polish visuals:
  - Ensure stage/podium area is clear
  - Good lighting (no dark areas)
  - NPCs don't clip through each other

**Deliverable:** Working build with feedback notes.

---

### Days 13–14: Documentation & Presentation

**What to do:**
- **Write report covering:**
  1. Background (SAD, exposure therapy, why VR)
  2. Your MVP design (3 levels, 2-min exposures, SUDS rating)
  3. Implementation (what you built, challenges)
  4. User testing results (5 test users' anxiety ratings, feedback)
  5. Results discussion (did anxiety increase with level difficulty? did people habituate?)
  6. Future work (what would you add with more time)
- **Create slides** showing:
  - Screenshot of each level (audience sizes)
  - SUDS ratings from test users (table or graph)
  - Architecture diagram (Lobby → Level selection → Exposure → Results)
  - Demo video (if possible)
- **Prepare demo:**
  - Have a short live demo, or
  - Show pre-recorded video of someone using it

**Deliverable:** Written report (3–5 pages) + presentation slides.

---

## PART 4: C# SCRIPT TEMPLATES

### Minimal AudienceManager.cs

```csharp
using UnityEngine;

public class AudienceManager : MonoBehaviour {
  public enum DifficultyLevel { Level1 = 1, Level2 = 2, Level3 = 3 }
  
  [SerializeField] private GameObject npcPrefab;
  [SerializeField] private Transform[] seatPositions; // Assign in inspector
  
  void Start() {
    DifficultyLevel level = (DifficultyLevel)GameState.SelectedLevel;
    SpawnAudience(level);
  }
  
  void SpawnAudience(DifficultyLevel level) {
    int count = 5;
    float distractionChance = 0f;
    
    if (level == DifficultyLevel.Level1) {
      count = 5;
      distractionChance = 0f;
    } else if (level == DifficultyLevel.Level2) {
      count = 15;
      distractionChance = 0.3f;
    } else if (level == DifficultyLevel.Level3) {
      count = 30;
      distractionChance = 0.5f;
    }
    
    for (int i = 0; i < count && i < seatPositions.Length; i++) {
      GameObject npc = Instantiate(npcPrefab, seatPositions[i].position, Quaternion.identity);
      
      if (Random.value < distractionChance) {
        // Disable look-at behavior for this NPC
        npc.GetComponent<NPCBehavior>().lookAtPlayer = false;
      }
    }
  }
}
```

### Minimal SessionController.cs

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class SessionController : MonoBehaviour {
  [SerializeField] private GameObject preExposurePanel;
  [SerializeField] private Slider preSUDSSlider;
  [SerializeField] private Button startButton;
  
  [SerializeField] private GameObject exposurePanel;
  [SerializeField] private Text timerText;
  
  [SerializeField] private GameObject postExposurePanel;
  [SerializeField] private Slider postSUDSSlider;
  [SerializeField] private Text resultText;
  [SerializeField] private Button retryButton;
  [SerializeField] private Button menuButton;
  
  private int preSUDS = 0;
  private int postSUDS = 0;
  private float exposureDuration = 120f; // 2 minutes
  
  void Start() {
    startButton.onClick.AddListener(OnStartExposure);
    retryButton.onClick.AddListener(OnRetry);
    menuButton.onClick.AddListener(OnBackToMenu);
    
    ShowPreExposurePanel();
  }
  
  void ShowPreExposurePanel() {
    preExposurePanel.SetActive(true);
    exposurePanel.SetActive(false);
    postExposurePanel.SetActive(false);
  }
  
  void OnStartExposure() {
    preSUDS = (int)preSUDSSlider.value;
    preExposurePanel.SetActive(false);
    exposurePanel.SetActive(true);
    StartCoroutine(ExposureTimer());
  }
  
  IEnumerator ExposureTimer() {
    float elapsed = 0f;
    while (elapsed < exposureDuration) {
      elapsed += Time.deltaTime;
      float remaining = exposureDuration - elapsed;
      int minutes = (int)remaining / 60;
      int seconds = (int)remaining % 60;
      timerText.text = string.Format("{0:00}:{1:00}", minutes, seconds);
      yield return null;
    }
    
    exposurePanel.SetActive(false);
    ShowPostExposurePanel();
  }
  
  void ShowPostExposurePanel() {
    postExposurePanel.SetActive(true);
  }
  
  public void OnPostSUDSConfirm() {
    postSUDS = (int)postSUDSSlider.value;
    int reduction = preSUDS - postSUDS;
    float percentReduction = (preSUDS > 0) ? (reduction / (float)preSUDS) * 100f : 0f;
    
    resultText.text = string.Format(
      "Before: {0}\nAfter: {1}\nHabituation: {2:F1}%",
      preSUDS, postSUDS, percentReduction
    );
  }
  
  void OnRetry() {
    UnityEngine.SceneManagement.SceneManager.LoadScene("ExposureScene");
  }
  
  void OnBackToMenu() {
    UnityEngine.SceneManagement.SceneManager.LoadScene("Lobby");
  }
}
```

### Minimal NPCBehavior.cs

```csharp
using UnityEngine;

public class NPCBehavior : MonoBehaviour {
  public bool lookAtPlayer = true;
  private Transform playerHead;
  
  void Start() {
    // Find player (XR Origin camera)
    playerHead = Camera.main.transform;
  }
  
  void Update() {
    if (lookAtPlayer && playerHead != null) {
      // Simple look-at
      Vector3 direction = playerHead.position - transform.position;
      Quaternion lookRotation = Quaternion.LookRotation(direction);
      transform.rotation = Quaternion.Lerp(transform.rotation, lookRotation, Time.deltaTime * 2f);
    }
  }
}
```

### GameState.cs (Static Manager)

```csharp
using UnityEngine;

public static class GameState {
  public static int SelectedLevel = 1;
}
```

---

## PART 5: REPORT STRUCTURE (FOR GRADING)

Write a 3–5 page report:

### Section 1: Background (0.5–1 page)

- What is social anxiety disorder?
- How does exposure therapy work?
- Why VR is useful for exposure therapy
- *Reference 2–3 papers you found* (e.g., self-guided VR SAD studies)

### Section 2: MVP Design (1 page)

- Overview of your app concept
- Why you chose public speaking as the core scenario
- 3 difficulty levels and what changes at each
- SUDS rating system and habituation concept

### Section 3: Implementation (1 page)

- What you built (Lobby scene, ExposureScene, NPC system, UI)
- Key Unity components and scripts
- Challenges faced and how you solved them
- Timeline (how you used your 2 weeks)

### Section 4: User Testing & Results (0.5–1 page)

- How many testers? (3–5 is fine)
- Table of their anxiety ratings:
  ```
  Tester | Level | Pre-SUDS | Post-SUDS | Reduction
  -------|-------|----------|-----------|----------
  A      | 1     | 25       | 18        | 28%
  A      | 2     | 40       | 30        | 25%
  A      | 3     | 65       | 45        | 31%
  ...
  ```
- Did Level 1 < Level 2 < Level 3 in anxiety? (validate graded exposure)
- Did people show habituation (pre > post)? (validate exposure mechanism)
- Qualitative feedback (was it easy to use? any bugs?)

### Section 5: Discussion & Future Work (0.5–1 page)

- What worked well
- What would you improve with more time (therapist portal, more scenarios, better graphics, etc.)
- Limitations of the MVP (only public speaking, no real dialogue, simple NPCs)
- How this relates to published VR exposure therapy research

---

## PART 6: KEY RESEARCH REFERENCES

When writing your report, cite:

1. **Self-guided VR exposure therapy for SAD:**
   - Wallinger et al. (2022) – showed self-guided VR VRET is feasible and effective
   - Study Protocol: simple exposures (no therapist needed) work for anxiety reduction
   
2. **Public speaking anxiety VR:**
   - PublicVR (2024) and similar studies show even short VR presentations elicit real anxiety
   - Simple audience scaling (5 → 30 people) is used in research studies
   
3. **Exposure principles:**
   - Standard exposure therapy guidelines emphasize graded, sustained, response-prevented exposure
   - SUDS rating before/after is standard clinical measurement

**How to cite:**
```
[1] Wallinger, X. et al. (2022). "Self-guided virtual reality exposure therapy 
    for social anxiety disorder: A study protocol for RCT." Journal Citation.
    
[2] [PublicVR citation or similar public speaking VR study from 2024]

[3] [General SAD/exposure therapy guideline, e.g., American Psychiatric Association]
```

---

## SUMMARY: MVP Scope

| Feature | Included | Notes |
|---------|----------|-------|
| 1 VR scene | ✅ | Classroom/auditorium |
| 3 difficulty levels | ✅ | 5 / 15 / 30 audience |
| SUDS rating UI | ✅ | Pre/post sliders |
| 2-min exposure timer | ✅ | Simple countdown |
| Audience NPCs | ✅ | Simple models, idle animations |
| Attention behavior | ✅ | % look at player varies by level |
| Ambient audio | ✅ | Room tone + murmurs |
| Habituation feedback | ✅ | Show % reduction |
| Lobby with buttons | ✅ | Level selection |
| Local data logging | ✅ Optional | Save to JSON for report |
| **NOT included:** | | |
| Backend/cloud | ❌ | Out of scope |
| Login/auth | ❌ | No user accounts |
| Therapist dashboard | ❌ | Single-user app |
| Multiple scenarios | ❌ | Only public speaking |
| Speech analysis | ❌ | Just timed exposure |
| Advanced graphics | ❌ | Asset Store models OK |
| Dialogue/interaction | ❌ | Silent audience only |

---

## FINAL CHECKLIST (Before Submission)

- [ ] Unity project builds and runs on target headset
- [ ] Lobby scene loads with 3 buttons
- [ ] Level buttons transition to ExposureScene
- [ ] SUDS pre-exposure panel works (slider + confirm)
- [ ] Audience spawns correctly (5/15/30 by level)
- [ ] Timer counts down correctly (2 minutes)
- [ ] SUDS post-exposure panel works
- [ ] Results show before/after and % reduction
- [ ] Can retry or go back to menu
- [ ] No major frame rate drops or crashes
- [ ] Audio plays (room tone)
- [ ] Report written (3–5 pages)
- [ ] Presentation slides ready
- [ ] Have demo ready (live or video)

---

**Good luck! This is a realistic, achievable MVP for 2 weeks. Focus on functionality over fancy graphics. Your grader will be impressed by the solid exposure therapy mechanism and user testing, not by perfect 3D assets.**

