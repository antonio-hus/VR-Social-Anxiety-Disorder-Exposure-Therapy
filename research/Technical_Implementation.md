# VR EXPOSURE THERAPY - TECHNICAL IMPLEMENTATION (2-WEEK MVP)

## ARCHITECTURE OVERVIEW (ULTRA-SIMPLIFIED)

```
┌─────────────────────────────────────┐
│        UNITY VR APPLICATION         │
├─────────────────────────────────────┤
│                                     │
│  Scenes:                            │
│  ├─ Lobby.unity                     │
│  └─ ExposureScene.unity             │
│                                     │
│  Scripts:                           │
│  ├─ GameState.cs (static)           │
│  ├─ SessionController.cs            │
│  ├─ AudienceManager.cs              │
│  ├─ NPCBehavior.cs                  │
│  └─ SessionLogger.cs (optional)     │
│                                     │
│  Assets:                            │
│  ├─ 3D Models (classroom, NPCs)     │
│  ├─ Animations (idle)               │
│  ├─ Audio (room tone, murmurs)      │
│  └─ UI Prefabs (sliders, text)      │
│                                     │
│  Data:                              │
│  └─ sessions.json (local file)      │
│                                     │
└─────────────────────────────────────┘
```

---

## PROJECT FOLDER STRUCTURE

```
YourProjectName/
├── Assets/
│   ├── Scenes/
│   │   ├── Lobby.unity
│   │   └── ExposureScene.unity
│   ├── Scripts/
│   │   ├── GameState.cs
│   │   ├── SessionController.cs
│   │   ├── AudienceManager.cs
│   │   ├── NPCBehavior.cs
│   │   └── SessionLogger.cs
│   ├── Prefabs/
│   │   ├── NPC.prefab
│   │   ├── PreExposurePanel.prefab
│   │   ├── PostExposurePanel.prefab
│   │   └── ExposurePanel.prefab
│   ├── Models/
│   │   ├── Classroom.fbx (from Asset Store)
│   │   └── Humanoid.fbx (free model or Asset Store)
│   ├── Audio/
│   │   ├── room_tone.mp3
│   │   └── murmurs.mp3
│   ├── UI/
│   │   └── (Canvas prefabs)
│   └── Materials/
│       └── (skin, clothing for NPCs)
├── ProjectSettings/
├── Packages/
└── README.md
```

---

## STEP-BY-STEP SCENE SETUP

### SCENE 1: LOBBY.UNITY

**Hierarchy:**
```
Lobby (Scene)
├── XROrigin (XR Origin component)
│   ├── Camera (XR Camera)
│   └── LeftHand (Hand Controller)
│   └── RightHand (Hand Controller)
├── Canvas_Lobby (World-space Canvas)
│   ├── Text_Title ("VR PUBLIC SPEAKING EXPOSURE")
│   ├── Text_Description (2-3 lines explaining project)
│   ├── Button_Level1
│   ├── Button_Level2
│   ├── Button_Level3
│   └── Button_Quit
└── AudioSource (ambient)
```

**Canvas Setup:**
- **Canvas:** Set to "World Space"
- **Rect Transform:** Position in front of player (e.g., Z = 2)
- **Size:** 1024 x 768 (width x height)
- **Text font size:** 60–80pt (readable in VR)

**Button Layout:**
```
Title
(2 lines of description text)

[Level 1] [Level 2] [Level 3]

[Quit]
```

**Script on Canvas or Button Container:**
```csharp
public void OnLevel1Click() {
  GameState.SelectedLevel = 1;
  SceneManager.LoadScene("ExposureScene");
}

public void OnLevel2Click() {
  GameState.SelectedLevel = 2;
  SceneManager.LoadScene("ExposureScene");
}

public void OnLevel3Click() {
  GameState.SelectedLevel = 3;
  SceneManager.LoadScene("ExposureScene");
}

public void OnQuitClick() {
  #if UNITY_EDITOR
    EditorApplication.isPlaying = false;
  #endif
  Application.Quit();
}
```

---

### SCENE 2: EXPOSURESCENCE.UNITY

**Hierarchy:**
```
ExposureScene (Scene)
├── XROrigin (on stage)
│   ├── Camera
│   ├── LeftHand
│   └── RightHand
├── Environment
│   ├── Classroom (3D model from Asset Store)
│   ├── Stage/Podium (simple cube or model)
│   └── Lighting
├── AudienceParent (empty GameObject)
│   └── NPC instances spawn here
├── Canvas_PreExposure
│   ├── Panel_SUDS
│   │   ├── Text "How anxious do you feel? (0-100)"
│   │   ├── Slider_PreSUDS (0-100)
│   │   ├── Text_SliderValue (displays current value)
│   │   └── Button_Confirm
│   └── ...
├── Canvas_Exposure
│   ├── Text_Timer (MM:SS format)
│   └── Text_Instruction ("Try to talk for the next 2 minutes...")
├── Canvas_PostExposure
│   ├── Panel_Results
│   │   ├── Text_ResultsHeader
│   │   ├── Slider_PostSUDS
│   │   ├── Text_SliderValue
│   │   ├── Text_Habituation ("Before: X | After: Y | Reduction: Z%")
│   │   ├── Button_Retry
│   │   └── Button_Menu
│   └── ...
├── SessionController (script)
├── AudienceManager (script)
└── AudioSource (ambient room tone + murmurs)
```

**XROrigin Position:**
- Set Y = ~1.7 (eye height)
- Set Z = -2 (on stage, facing audience)

**Audience Seating:**
- Create ~30 empty GameObjects in rows (e.g., at Z = 0 to -5, X = -3 to +3)
- Assign each one as a seat position in AudienceManager

**Lighting:**
- Add a Directional Light (sunlight) angled toward stage
- Add ambient lighting so scene is visible
- Optional: spotlight on stage

---

## DETAILED SCRIPT IMPLEMENTATIONS

### 1. GameState.cs (Global State Manager)

```csharp
using UnityEngine;

/// <summary>
/// Global state that persists across scenes.
/// </summary>
public static class GameState {
    public static int SelectedLevel = 1; // 1, 2, or 3
    public static int LastPreSUDS = -1;
    public static int LastPostSUDS = -1;
}
```

**Usage:**
- Read `GameState.SelectedLevel` in `AudienceManager.Start()` to know how many NPCs to spawn
- Write `GameState.LastPreSUDS` and `GameState.LastPostSUDS` in `SessionController` for logging

---

### 2. AudienceManager.cs (Spawn & Control Audience)

```csharp
using UnityEngine;

public class AudienceManager : MonoBehaviour {
    [SerializeField] private GameObject npcPrefab;
    [SerializeField] private Transform[] seatPositions; // Assign 30+ positions in inspector
    
    private void Start() {
        int level = GameState.SelectedLevel;
        SpawnAudience(level);
    }
    
    private void SpawnAudience(int level) {
        int count = 5;
        float distractionChance = 0f;
        
        switch (level) {
            case 1:
                count = 5;
                distractionChance = 0f; // All look at player
                break;
            case 2:
                count = 15;
                distractionChance = 0.3f; // 30% distracted
                break;
            case 3:
                count = 30;
                distractionChance = 0.5f; // 50% distracted
                break;
        }
        
        for (int i = 0; i < count && i < seatPositions.Length; i++) {
            Vector3 seatPos = seatPositions[i].position;
            Quaternion seatRot = seatPositions[i].rotation;
            
            GameObject npc = Instantiate(npcPrefab, seatPos, seatRot, transform);
            NPCBehavior behavior = npc.GetComponent<NPCBehavior>();
            
            // Randomly decide if this NPC looks at player
            if (Random.value < distractionChance) {
                behavior.SetLookAtPlayer(false);
            } else {
                behavior.SetLookAtPlayer(true);
            }
        }
    }
}
```

**In Inspector:**
- Assign NPC prefab
- Create 30+ empty GameObjects as seat positions, assign them to the `seatPositions` array

---

### 3. NPCBehavior.cs (Individual NPC Logic)

```csharp
using UnityEngine;

public class NPCBehavior : MonoBehaviour {
    private bool lookAtPlayer = true;
    private Transform playerHead;
    private Animator animator;
    
    private void Start() {
        // Find the VR camera (player's head)
        playerHead = Camera.main.transform;
        animator = GetComponent<Animator>();
        
        // Play idle animation if available
        if (animator != null) {
            animator.SetBool("IsIdle", true);
        }
    }
    
    private void Update() {
        if (lookAtPlayer && playerHead != null) {
            LookAt(playerHead.position);
        }
    }
    
    private void LookAt(Vector3 targetPos) {
        Vector3 direction = targetPos - transform.position;
        
        // Only look in horizontal plane (don't tilt head excessively)
        direction.y = 0;
        direction.Normalize();
        
        Quaternion lookRotation = Quaternion.LookRotation(direction);
        
        // Smooth rotation
        transform.rotation = Quaternion.Lerp(
            transform.rotation,
            lookRotation,
            Time.deltaTime * 2f // Smooth factor
        );
    }
    
    public void SetLookAtPlayer(bool shouldLook) {
        lookAtPlayer = shouldLook;
    }
}
```

**Animator Setup (if using):**
- Create a simple Animator with "Idle" state
- No transitions needed for MVP

---

### 4. SessionController.cs (Flow & UI Management)

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;
using System.Collections;

public class SessionController : MonoBehaviour {
    [Header("Pre-Exposure UI")]
    [SerializeField] private GameObject preExposurePanel;
    [SerializeField] private Slider preSUDSSlider;
    [SerializeField] private Text preSUDSValueText;
    [SerializeField] private Button startButton;
    
    [Header("Exposure UI")]
    [SerializeField] private GameObject exposurePanel;
    [SerializeField] private Text timerText;
    [SerializeField] private Text instructionText;
    
    [Header("Post-Exposure UI")]
    [SerializeField] private GameObject postExposurePanel;
    [SerializeField] private Slider postSUDSSlider;
    [SerializeField] private Text postSUDSValueText;
    [SerializeField] private Text habitationText;
    [SerializeField] private Button retryButton;
    [SerializeField] private Button menuButton;
    
    [Header("Audio")]
    [SerializeField] private AudioSource ambientAudio;
    
    private int preSUDS = 0;
    private int postSUDS = 0;
    private float exposureDuration = 120f; // 2 minutes
    private bool exposureInProgress = false;
    
    private void Start() {
        // Hook up button events
        startButton.onClick.AddListener(OnStartExposure);
        retryButton.onClick.AddListener(OnRetry);
        menuButton.onClick.AddListener(OnBackToMenu);
        
        // Hook up slider value changes
        preSUDSSlider.onValueChanged.AddListener(OnPreSUDSChange);
        postSUDSSlider.onValueChanged.AddListener(OnPostSUDSChange);
        
        // Show initial UI
        ShowPreExposurePanel();
    }
    
    private void OnPreSUDSChange(float value) {
        preSUDSValueText.text = ((int)value).ToString();
    }
    
    private void OnPostSUDSChange(float value) {
        postSUDSValueText.text = ((int)value).ToString();
    }
    
    private void ShowPreExposurePanel() {
        preExposurePanel.SetActive(true);
        exposurePanel.SetActive(false);
        postExposurePanel.SetActive(false);
    }
    
    private void OnStartExposure() {
        preSUDS = (int)preSUDSSlider.value;
        GameState.LastPreSUDS = preSUDS;
        
        preExposurePanel.SetActive(false);
        exposurePanel.SetActive(true);
        
        // Start audio
        if (ambientAudio != null) {
            ambientAudio.Play();
        }
        
        StartCoroutine(ExposureTimer());
    }
    
    private IEnumerator ExposureTimer() {
        exposureInProgress = true;
        float elapsed = 0f;
        
        while (elapsed < exposureDuration) {
            elapsed += Time.deltaTime;
            float remaining = exposureDuration - elapsed;
            
            int minutes = (int)remaining / 60;
            int seconds = (int)remaining % 60;
            timerText.text = string.Format("{0:00}:{1:00}", minutes, seconds);
            
            yield return null;
        }
        
        exposureInProgress = false;
        
        // Stop audio
        if (ambientAudio != null) {
            ambientAudio.Stop();
        }
        
        // Transition to post-exposure
        exposurePanel.SetActive(false);
        ShowPostExposurePanel();
    }
    
    private void ShowPostExposurePanel() {
        postExposurePanel.SetActive(true);
        
        // Clear post-SUDS slider
        postSUDSSlider.value = 50; // Default middle value
    }
    
    public void OnPostSUDSConfirm() {
        postSUDS = (int)postSUDSSlider.value;
        GameState.LastPostSUDS = postSUDS;
        
        // Calculate habituation
        int reduction = preSUDS - postSUDS;
        float percentReduction = 0f;
        
        if (preSUDS > 0) {
            percentReduction = (reduction / (float)preSUDS) * 100f;
        }
        
        // Update result text
        habitationText.text = string.Format(
            "Before: {0}\nAfter: {1}\nHabituation: {2:F1}%",
            preSUDS, postSUDS, percentReduction
        );
        
        // Optional: Log session
        if (SessionLogger.Instance != null) {
            SessionLogger.Instance.LogSession(GameState.SelectedLevel, preSUDS, postSUDS);
        }
    }
    
    private void OnRetry() {
        SceneManager.LoadScene("ExposureScene");
    }
    
    private void OnBackToMenu() {
        SceneManager.LoadScene("Lobby");
    }
}
```

**In Inspector:**
- Drag all UI elements into the corresponding fields
- Drag AudioSource into `ambientAudio`

---

### 5. SessionLogger.cs (Optional Data Logging)

```csharp
using UnityEngine;
using System.Collections.Generic;
using System.IO;

[System.Serializable]
public class SessionData {
    public int level;
    public int preSUDS;
    public int postSUDS;
    public long timestampMS;
}

[System.Serializable]
public class SessionList {
    public List<SessionData> sessions = new List<SessionData>();
}

public class SessionLogger : MonoBehaviour {
    public static SessionLogger Instance { get; private set; }
    
    private SessionList sessionList = new SessionList();
    private string filePath;
    
    private void Awake() {
        if (Instance != null && Instance != this) {
            Destroy(gameObject);
            return;
        }
        
        Instance = this;
        DontDestroyOnLoad(gameObject);
        
        filePath = Path.Combine(Application.persistentDataPath, "sessions.json");
        LoadSessions();
    }
    
    public void LogSession(int level, int preSUDS, int postSUDS) {
        SessionData data = new SessionData {
            level = level,
            preSUDS = preSUDS,
            postSUDS = postSUDS,
            timestampMS = System.DateTime.Now.Ticks
        };
        
        sessionList.sessions.Add(data);
        SaveSessions();
        
        Debug.Log($"Session logged: Level {level}, Pre:{preSUDS}, Post:{postSUDS}");
    }
    
    private void SaveSessions() {
        string json = JsonUtility.ToJson(sessionList, true);
        File.WriteAllText(filePath, json);
        Debug.Log($"Sessions saved to {filePath}");
    }
    
    private void LoadSessions() {
        if (File.Exists(filePath)) {
            string json = File.ReadAllText(filePath);
            sessionList = JsonUtility.FromJson<SessionList>(json);
            Debug.Log($"Loaded {sessionList.sessions.Count} previous sessions");
        }
    }
}
```

**Setup:**
- Create an empty GameObject in ExposureScene
- Add SessionLogger script to it
- Check "DontDestroyOnLoad" in code

---

## NPC PREFAB SETUP

**Steps:**

1. Import humanoid model (free from Sketchfab or Asset Store)
2. Create prefab:
   - Drag model into Prefabs folder
   - Select it
   - Drag model GameObject into that prefab folder (create prefab)
3. In Prefab inspector:
   - Add `NPCBehavior` script component
   - Optionally add Animator component if model has animations
4. Save prefab

**NPC Prefab Structure:**
```
NPC (GameObject)
├── Mesh (SkinnedMeshRenderer)
├── Animator (if using animations)
├── NPCBehavior (script)
└── (no collider needed for MVP)
```

---

## SEAT POSITIONS SETUP

**How to create 30 seat positions:**

```csharp
// In ExposureScene, create this script to generate seat positions automatically
using UnityEngine;

public class SeatGenerator : MonoBehaviour {
    public int rowCount = 6;
    public int seatsPerRow = 5;
    public float rowSpacing = 1.5f;
    public float seatSpacing = 1.5f;
    
    public void GenerateSeats() {
        for (int row = 0; row < rowCount; row++) {
            for (int seat = 0; seat < seatsPerRow; seat++) {
                float x = (seat - seatsPerRow / 2f) * seatSpacing;
                float z = -row * rowSpacing;
                
                GameObject seatGO = new GameObject($"Seat_{row}_{seat}");
                seatGO.transform.parent = transform;
                seatGO.transform.position = new Vector3(x, 0, z);
            }
        }
    }
}
```

**Alternatively (manual):**
- In Editor, create empty GameObjects in a grid
- Position them in rows facing the stage
- Assign all to `seatPositions` array in AudienceManager

---

## UI PREFAB TEMPLATES

### Pre-Exposure Panel Prefab

```
Canvas (World Space)
├── Panel (Image, dark semi-transparent background)
│   ├── Text "Rate your anxiety (0-100)"
│   ├── Slider (PreSUDS)
│   │   ├── Background
│   │   ├── Fill
│   │   └── Handle
│   ├── Text (displays value, e.g., "50")
│   ├── Text (instruction: "Click confirm to start")
│   └── Button (Confirm)
```

**Slider setup:**
- Min Value: 0
- Max Value: 100
- Whole Numbers: ON

### Post-Exposure Panel Prefab

Similar to pre-exposure, but add:
- **Text (Results Header):** "Your Results"
- **Text (Habituation):** "Before: X | After: Y | Reduction: Z%"
- **Button (Retry)**
- **Button (Back to Menu)**

---

## BUILD & TEST CHECKLIST

### Before Building

- [ ] All scenes added to Build Settings (File → Build Settings)
- [ ] Scenes in correct order: Lobby (0), ExposureScene (1)
- [ ] XR Plugin installed (Windows > TextMesh Pro > Import TMP Essentials if needed)
- [ ] Quest/SteamVR target selected in Build Settings

### Building for Quest

1. Switch platform: File → Build Settings → Android → Switch Platform
2. Player Settings:
   - Company Name: Your name
   - Product Name: Your app name
   - Package Name: com.yourname.vrexposure
3. Build: File → Build and Run

### Building for PCVR

1. Keep PC/Windows platform
2. Build: File → Build and Run (requires SteamVR running)

### In Headset Testing

- [ ] Can move around with controllers
- [ ] UI is readable (text size OK)
- [ ] Buttons respond to press
- [ ] Each level spawns correct # of NPCs
- [ ] NPCs look at you (Level 1) or look away (Levels 2/3)
- [ ] Timer counts down correctly
- [ ] Audio plays during exposure
- [ ] Pre/post SUDS sliders work
- [ ] Results show habituation % correctly
- [ ] No crashes or major lag

---

## COMMON ISSUES & SOLUTIONS

| Issue | Solution |
|-------|----------|
| "XR Plugin not found" | Install XR Interaction Toolkit from Package Manager |
| Sliders not responding | Ensure GraphicRaycaster is on Canvas |
| NPCs not spawning | Check seatPositions array is populated in inspector |
| Audio plays but very quiet | Adjust AudioSource volume in inspector |
| Scene is too dark | Add Ambient Light (Window → Rendering → Lighting) |
| Controller buttons don't work | Check XRI EventSystem is in scene |
| Low frame rate | Reduce NPC count, lower graphics settings |

---

## SCENE PERSISTENCE & DATA

If using SessionLogger:

**File location:**
- **Android (Quest):** `/sdcard/Android/data/com.yourname.vrexposure/files/sessions.json`
- **PC:** `%APPDATA%/LocalLow/YourCompanyName/VRExposure/sessions.json`

**Accessing for report:**
- Build APK, run on Quest
- After test users complete sessions, connect Quest to PC via USB
- Use Android File Transfer or adb to pull `sessions.json`
- Open in text editor to see results

---

## NEXT STEPS (AFTER MVP)

If you had more time (not in scope):

1. **More scenarios:** Job interview, group meeting, restaurant
2. **NPC dialogue:** Simple pre-recorded responses
3. **Better animations:** Walking, talking, reacting to user
4. **Speech analysis:** Detect user speaking via microphone
5. **Therapist controls:** Live adjustment UI (probably out of reach for 2 weeks)
6. **Real 3D models:** Better looking classroom and avatars
7. **Multiuser:** Friends joining as audience (very advanced)

---

**This is a solid, achievable implementation for 2 weeks. Good luck!**

