# Blueprint Implementation Guide - DRG Stats Tracker

**Status:** UE4 Project Ready ✅  
**Last Updated:** 2026-02-08  
**Prerequisites:** UE4 Editor open with FSD-Template project loaded

---

## ⚠️ IMPORTANT: Wait for UE4 to Fully Load

Before creating blueprints, verify:
- ✅ UE4 Editor window is open
- ✅ Content Browser is visible (bottom panel)
- ✅ No "Compiling Shaders" blocking dialog (shaders can compile in background)
- ✅ Output Log shows no critical errors (Window → Developer Tools → Output Log)

**Shader compilation will continue in background (~30-60 min first time) - this is NORMAL and won't block blueprint creation.**

---

## Phase 1: Project Setup & Data Structures

### Step 1.1: Create Folder Structure

**Location:** Content Browser → Content root folder

1. Right-click "Content" folder
2. Select "New Folder"
3. Name: `ModStatsTracker`
4. Inside ModStatsTracker, create 3 subfolders:
   - `Blueprints` (actors and components)
   - `Data` (structs, enums, data tables)
   - `UI` (reserved for future widget blueprints)

**Result:** 
```
Content/
└── ModStatsTracker/
    ├── Blueprints/
    ├── Data/
    └── UI/
```

---

### Step 1.2: Create Enemy Category Enum

**Location:** Content/ModStatsTracker/Data/

**Steps:**
1. Right-click in "Data" folder
2. Blueprints → Enumeration
3. Name: `ST_EnemyCategory`
4. Double-click to open
5. Click "New" 7 times to add enumerators:

| Index | Display Name | Internal Name | Description |
|-------|-------------|---------------|-------------|
| 0 | Glyphid | Glyphid | Grunts, Praetorians, Oppressors, Wardens, Menaces |
| 1 | Mactera | Mactera | Spawns, Grabbers, Goo Bombers, Tri-Jaws |
| 2 | Naedocyte | Naedocyte | Shockers, Breeders, Hatchlings |
| 3 | Q'ronar | QRonar | Shellbacks, Younglings |
| 4 | Robot | Robot | Patrol Bots, Shredders, Turrets |
| 5 | Dreadnought | Dreadnought | Dreadnought, Twins, Hiveguard |
| 6 | Other | Other | Cave Leeches, Spitballers, Korlok, BET-C, etc. |

6. Add descriptions to each (optional but recommended)
7. Save (Ctrl+S)
8. Close

**File Created:** `ST_EnemyCategory.uasset`

---

### Step 1.3: Create Player Data Struct

**Location:** Content/ModStatsTracker/Data/

**Steps:**
1. Right-click in "Data" folder
2. Blueprints → Structure
3. Name: `ST_PlayerData`
4. Double-click to open
5. Click "New Variable" 6 times:

| Variable Name | Type | Default Value | Description |
|--------------|------|---------------|-------------|
| PlayerName | String | "" | Player's display name |
| TotalKills | Integer | 0 | Total enemy kills |
| TotalDamage | Float | 0.0 | Total damage dealt |
| KillsByCategory | Map (Key: ST_EnemyCategory, Value: Integer) | Empty | Kills per enemy type |
| DamageByCategory | Map (Key: ST_EnemyCategory, Value: Float) | Empty | Damage per enemy type |
| PlayerController | Object Reference (PlayerController) | None | Reference to player |

**Variable Type Details:**

**For KillsByCategory:**
- Click type dropdown → Map
- Key Type: ST_EnemyCategory (search for it)
- Value Type: Integer

**For DamageByCategory:**
- Click type dropdown → Map
- Key Type: ST_EnemyCategory
- Value Type: Float

**For PlayerController:**
- Click type dropdown → Object Reference
- Search: "PlayerController"
- Select: PlayerController (class reference)

6. Save (Ctrl+S)
7. Close

**File Created:** `ST_PlayerData.uasset`

---

## Phase 2: Entry Point Actors

### Step 2.1: Create InitCave Blueprint

**Location:** Content/ModStatsTracker/Blueprints/

**Steps:**
1. Right-click in "Blueprints" folder
2. Blueprint Class → Actor
3. Name: `InitCave`
4. Double-click to open

**Class Settings:**
1. Click "Class Defaults" button (top toolbar)
2. In Details panel:
   - Replication → Replicates: ✅ True
   - Replication → Always Relevant: ✅ True

**Event Graph:**
1. Already has "Event BeginPlay" node
2. Drag from "Event BeginPlay" execution pin
3. Add node: "Switch Has Authority" (or "Is Server")
   - This ensures only server spawns the tracker
4. From "Authority" pin (or "True" branch):
   - Add node: "Spawn Actor from Class"
   - Class: ST_Main (we'll create this next)
   - Spawn Transform: Make Transform (Location 0,0,0)
   - Collision Handling: Always Spawn
5. (Optional Debug) Add "Print String" node:
   - In String: "Stats Tracker Spawned"
   - Duration: 5.0

**Node Layout:**
```
[Event BeginPlay] → [Switch Has Authority]
                           ↓ Authority
                     [Spawn Actor ST_Main] → [Print String (Debug)]
```

6. Compile (Green checkmark button)
7. Save
8. Close

**File Created:** `InitCave.uasset`

---

### Step 2.2: Create InitSpacerig Blueprint

**Location:** Content/ModStatsTracker/Blueprints/

**Steps:**
1. Right-click in "Blueprints" folder
2. Blueprint Class → Actor
3. Name: `InitSpacerig`
4. Double-click to open

**Event Graph:**
1. "Event BeginPlay" node
2. (Optional Debug) Add "Print String":
   - In String: "In Space Rig - Stats Disabled"
   - Duration: 3.0

**Purpose:** Minimal handler - we don't track stats in space rig, only in missions.

3. Compile
4. Save
5. Close

**File Created:** `InitSpacerig.uasset`

---

## Phase 3: Core Tracker Actor (ST_Main)

### Step 3.1: Create ST_Main Blueprint

**Location:** Content/ModStatsTracker/Blueprints/

**Steps:**
1. Right-click in "Blueprints" folder
2. Blueprint Class → Actor
3. Name: `ST_Main`
4. Double-click to open

**Class Settings:**
1. Click "Class Defaults"
2. Replication → Replicates: ✅ True
3. Replication → Always Relevant: ✅ True

---

### Step 3.2: Add Variables to ST_Main

Click "+ Variable" button in My Blueprint panel:

| Variable Name | Type | Replication | Default | Description |
|--------------|------|-------------|---------|-------------|
| PlayerStats | Array<ST_PlayerData> | Replicated | Empty | Stats for all players |
| bIsTracking | Boolean | Replicated | False | Whether tracking is active |
| GameStateRef | Object Reference (FSDGameState) | None | None | Cached game state |

**Set Replication:**
- Select variable
- Details panel → Replication dropdown
- Choose "Replicated" for PlayerStats and bIsTracking

---

### Step 3.3: Initialize Function

**Create Function:**
1. In "My Blueprint" panel → Functions section
2. Click "+Function"
3. Name: `InitializePlayerStats`

**Function Graph:**

**Purpose:** Populate PlayerStats array with all current players

**Nodes:**
```
1. Get Game State → Cast to FSDGameState
   ↓
2. Get Player Array (from FSDGameState)
   ↓
3. ForEachLoop (iterate over players)
   ↓ Loop Body
4. Create new ST_PlayerData struct
   - Set PlayerName (get from PlayerState)
   - Set PlayerController (loop variable)
   - Set TotalKills = 0
   - Set TotalDamage = 0.0
   - Initialize KillsByCategory map (empty)
   - Initialize DamageByCategory map (empty)
   ↓
5. Add to PlayerStats array
   ↓
6. (Loop Complete) Set bIsTracking = True
```

**Implementation Notes:**
- Use "Get Game State" node → "Cast to FSDGameState"
- If cast fails, log error and abort
- For map initialization, use "Clear" node on empty map

---

### Step 3.4: Enemy Categorization Function

**Create Function:**
1. Functions → +Function
2. Name: `CategorizeEnemy`

**Inputs:**
- EnemyActor (Actor Object Reference)

**Outputs:**
- Category (ST_EnemyCategory)

**Function Logic:**

**Purpose:** Determine enemy category from class name

**Approach:** Get class name as string, use Contains checks

**Pseudocode:**
```
Get Enemy Class → Get Object Name → Convert to String

If Contains "Grunt" OR "Praetorian" OR "Oppressor" OR "Warden" OR "Menace"
   → Return Glyphid

Else If Contains "Mactera" OR "Spawn" OR "Grabber" OR "Bomber" OR "TriJaw"
   → Return Mactera

Else If Contains "Naedo" OR "Shocker" OR "Breeder" OR "Hatchling"
   → Return Naedocyte

Else If Contains "Shellback" OR "Youngling" OR "Qronar"
   → Return QRonar

Else If Contains "Robot" OR "Patrol" OR "Shredder" OR "Turret" OR "Nemesis"
   → Return Robot

Else If Contains "Dread" OR "Twin" OR "Hiveguard"
   → Return Dreadnought

Else
   → Return Other
```

**Blueprint Implementation:**
- Use "Branch" nodes for each category
- Get Class → Get Display Name → String Contains
- Use OR gates to combine multiple name checks
- Final else returns "Other"

**Testing Needed:** Enemy class names may differ - check Header-Dumps or test in-game

---

### Step 3.5: Add Kill Function

**Create Function:**
1. Functions → +Function
2. Name: `AddKill`

**Inputs:**
- EnemyActor (Actor)
- KillerController (PlayerController)

**Function Logic:**
```
1. Call CategorizeEnemy(EnemyActor) → Category
   ↓
2. Find player in PlayerStats array (match KillerController)
   ↓
3. Increment TotalKills by 1
   ↓
4. Get KillsByCategory map
   ↓
5. Check if Category exists in map
   - If exists: Increment value by 1
   - If not: Add key with value 1
   ↓
6. Update PlayerStats array (Set Array Elem)
   ↓
7. (Optional) Print String (Debug): "Kill recorded: [Category]"
```

**Map Update Pattern:**
```
[Get Map] → [Find Key]
   ↓ Found?
   Yes: [Get Value] → [Add 1] → [Set Key]
   No:  [Add Key with Value 1]
```

---

### Step 3.6: Add Damage Function

**Create Function:**
1. Functions → +Function
2. Name: `AddDamage`

**Inputs:**
- EnemyActor (Actor)
- DamagerController (PlayerController)
- DamageAmount (Float)

**Function Logic:**
```
1. Call CategorizeEnemy(EnemyActor) → Category
   ↓
2. Find player in PlayerStats array
   ↓
3. Add DamageAmount to TotalDamage
   ↓
4. Get DamageByCategory map
   ↓
5. Check if Category exists
   - If exists: Add DamageAmount to existing value
   - If not: Add key with value = DamageAmount
   ↓
6. Update PlayerStats array
```

**Same map update pattern as AddKill, but with float addition instead of increment**

---

### Step 3.7: Display Stats Function

**Create Function:**
1. Functions → +Function
2. Name: `DisplayStats`

**Outputs:**
- FormattedStats (String)

**Function Logic:**

**Purpose:** Format stats into readable text for chat display

**Format Example:**
```
==========================================
MISSION STATISTICS
==========================================
Player: Driller Dave
  Total: 127 kills, 8542 damage
  Glyphid: 89 kills (6234 dmg)
  Mactera: 23 kills (1456 dmg)
  Dreadnought: 1 kill (856 dmg)
------------------------------------------
Player: Scout Sarah
  Total: 98 kills, 7123 damage
  ...
==========================================
```

**Implementation:**
```
1. Start with header string
   ↓
2. ForEachLoop over PlayerStats array
   ↓ For each player:
3. Append player name
4. Append total kills/damage
5. ForEachLoop over KillsByCategory map
   ↓ For each category with kills > 0:
6. Append "  [Category]: [kills] kills ([damage] dmg)"
7. Append separator line
   ↓
8. After all players, append footer
   ↓
9. Return formatted string
```

**String Building:**
- Use "Append" node repeatedly
- Format float: "Round" node or "Format Text"
- Only show categories with kills > 0

**Broadcast to Chat:**
- After building string, call FSDGameState → BroadcastChatMessage
- Or PlayerController → SendChatMessage (test which works)

---

## Phase 4: Event Hooking (EXPERIMENTAL)

**⚠️ WARNING:** This is the most uncertain part - FSD API is not fully documented.

### Approach 1: Try Event Dispatchers (Preferred)

**Steps:**
1. In ST_Main Event Graph
2. Event BeginPlay:
   - Get Game State → Cast to FSDGameState
   - Search for delegate bindings:
     - "Bind Event to OnPawnKilled" (if exists)
     - "Bind Event to OnDamageDealt" (if exists)
3. Create Custom Events:
   - Event name: `OnEnemyKilled`
   - Inputs: Actor (enemy), PlayerController (killer)
   - Implementation: Call AddKill function
   
   - Event name: `OnEnemyDamaged`
   - Inputs: Actor (enemy), PlayerController (damager), Float (amount)
   - Implementation: Call AddDamage function

**Test in-game:** Kill an enemy, check if event fires (use Print String)

**If events don't exist:** Proceed to Approach 2

---

### Approach 2: Tick-Based Health Monitoring (Fallback)

**Steps:**
1. Enable "Event Tick" in ST_Main
2. Every tick (or every 0.1 seconds using timer):
   - Get all enemies in level (Get All Actors of Class)
   - For each enemy:
     - Check current health vs last known health
     - If health decreased: Calculate damage, call AddDamage
     - If health = 0 and was alive last tick: Call AddKill

**Data Structure Needed:**
- Add variable: `TrackedEnemies` (Map: Actor → Float, stores last known health)

**Performance Warning:** This is expensive - optimize:
- Use timer (0.1s interval) instead of every tick
- Only track enemies within certain radius
- Remove dead enemies from tracking map

---

### Approach 3: Research & Community Help

**Before implementing fallback:**
1. Check Header-Dumps for event delegates:
   - File: `D:\projects\Header-Dumps\FSD.hpp`
   - Search for: "OnKilled", "OnDeath", "OnDamage", "Delegate"
2. Ask in DRG Modding Discord:
   - Channel: #mod-questions
   - Question: "How to detect enemy kills/damage in blueprint mod?"
   - Other modders likely have solved this

---

## Phase 5: Chat Command Detection

### Option 1: Chat Event Binding

**If FSDGameState has chat delegate:**
```
Event BeginPlay:
   Get Game State → Cast to FSDGameState
   Bind Event to OnChatMessageReceived
   
Custom Event: OnChatReceived
   Input: String (message), PlayerController (sender)
   
   If message == "/stats":
      Call DisplayStats
      Broadcast result to chat
```

### Option 2: Input Action Binding

**If no chat event:**
```
Bind to player input action (slash key)
Check if typed text == "stats"
Trigger DisplayStats
```

### Option 3: Console Command

**Register custom console command:**
```
In ST_Main Constructor:
   Register Console Command "stats" → DisplayStats
```

**Test which method works in FSD modding environment**

---

## Phase 6: Mission End Auto-Display

**Trigger stats display automatically when mission completes**

**Implementation:**
```
Event BeginPlay:
   Get Game State → Bind to OnMissionComplete delegate
   
Custom Event: OnMissionComplete
   Wait 2 seconds (for mission end animations)
   Call DisplayStats
   Broadcast to chat
```

**Alternative:** Bind to extraction success event or victory screen trigger

---

## Phase 7: Testing Checklist

### Test 1: Spawn Verification
- [ ] Start Hazard 1 solo mission
- [ ] Check Output Log for "Stats Tracker Spawned" message
- [ ] Verify no errors in log
- [ ] If tracker doesn't spawn: Check InitCave is placed in level

### Test 2: Kill Tracking
- [ ] Kill 5 grunts
- [ ] Type /stats (or trigger command)
- [ ] Verify output shows 5 kills under Glyphid
- [ ] Kill 1 mactera
- [ ] Check stats show both categories

### Test 3: Damage Tracking
- [ ] Partially damage enemy (don't kill)
- [ ] Check stats (damage should update even without kill)
- [ ] Verify damage value is reasonable

### Test 4: Multiple Players
- [ ] Host game with 1 friend
- [ ] Both players kill enemies
- [ ] Check /stats shows both players separately
- [ ] Verify other player can see stats

### Test 5: All Enemy Categories
- [ ] Test one enemy from each category:
   - Glyphid: Grunt
   - Mactera: Spawn
   - Naedocyte: Breeder
   - Q'ronar: Shellback
   - Robot: Patrol Bot
   - Dreadnought: Dreadnought
   - Other: Cave Leech
- [ ] Verify each categorizes correctly
- [ ] Document any miscategorizations

### Test 6: Performance
- [ ] Run Hazard 5 mission (high enemy count)
- [ ] Monitor FPS (Ctrl+Shift+H to show FPS)
- [ ] Check for lag spikes when updating stats
- [ ] If performance issues: Optimize update frequency

### Test 7: Edge Cases
- [ ] Mission with 0 kills (stats should show gracefully)
- [ ] Mission with 1000+ kills (verify no overflow)
- [ ] Late join player (should track from join time)
- [ ] Player disconnect mid-mission (should persist stats)

---

## Troubleshooting

### Issue: Tracker doesn't spawn
**Solution:**
1. Check InitCave is in the level (search in World Outliner)
2. Verify "Switch Has Authority" or "Is Server" check is correct
3. Add Print String before spawn to confirm event fires

### Issue: Stats always show 0
**Solution:**
1. Event hooks not working - check bindings
2. Add Print String in AddKill/AddDamage to verify they're called
3. Test with tick-based monitoring as fallback

### Issue: Wrong enemy categories
**Solution:**
1. Print enemy class name to log
2. Update CategorizeEnemy string checks
3. Check Header-Dumps for exact class names

### Issue: Chat command doesn't work
**Solution:**
1. Test different chat detection methods (see Phase 5 options)
2. Ask community for working chat hook example
3. Fallback: Use console command instead

### Issue: Multiplayer stats don't replicate
**Solution:**
1. Verify ST_Main has Replicates=True
2. Check PlayerStats array is set to Replicated
3. Test with 2 PCs (not solo + join)

---

## Next Steps After Blueprint Creation

1. **Package Mod:**
   - Use DRGPacker tool (from DRG Modding Discord)
   - Upload to mod.io
   
2. **Optimization:**
   - Profile performance in high-enemy missions
   - Reduce update frequency if needed
   - Cache frequently accessed references

3. **Feature Additions:**
   - Add UI widget (visual stats display)
   - Track additional metrics (accuracy, deaths, resources)
   - Add configuration file for custom categories

4. **Community Release:**
   - Post in #showcase Discord channel
   - Gather feedback
   - Iterate based on player reports

---

**Rock and Stone!** ⛏️
