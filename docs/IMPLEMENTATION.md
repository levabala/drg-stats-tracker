# Implementation Guide

This guide provides step-by-step instructions for implementing the Stats Tracker mod in Unreal Engine 4.

## Overview

We'll create these blueprints:

| Blueprint | Type | Purpose |
|-----------|------|---------|
| `InitCave` | Actor | Entry point for missions |
| `InitSpacerig` | Actor | Entry point for spacerig |
| `ST_Main` | Actor | Main stats tracking logic |
| `ST_PlayerData` | Struct | Per-player statistics data |
| `ST_EnemyCategory` | Enum | Enemy category definitions |

## Prerequisites

- Completed [SETUP.md](SETUP.md)
- FSD-Template project open in UE4 4.27.2
- `Content/ModStatsTracker` folder created

---

## Phase 1: Create Basic Structures

### Step 1.1: Create Enemy Category Enum

1. In Content Browser, navigate to `ModStatsTracker`
2. Right-click > Blueprints > Enumeration
3. Name it `ST_EnemyCategory`
4. Add these enumerators:
   - `Glyphid`
   - `Mactera`
   - `Naedocyte`
   - `Qronar` (can't use apostrophe)
   - `Robot`
   - `Dreadnought`
   - `Other`

### Step 1.2: Create Player Data Structure

1. Right-click > Blueprints > Structure
2. Name it `ST_PlayerData`
3. Add these variables:

| Name | Type | Description |
|------|------|-------------|
| `PlayerName` | String | Player's display name |
| `PlayerController` | FSDPlayerController (Object Reference) | Reference to player |
| `TotalKills` | Integer | Total kill count |
| `TotalDamage` | Float | Total damage dealt |
| `KillsByCategory` | Map (ST_EnemyCategory → Integer) | Kills per enemy type |
| `DamageByCategory` | Map (ST_EnemyCategory → Float) | Damage per enemy type |

---

## Phase 2: Create Entry Points

### Step 2.1: Create InitCave

1. Right-click > Blueprint Class > Actor
2. Name it `InitCave`
3. Open the blueprint
4. In Event Graph, implement:

```
Event BeginPlay
    │
    ├─► Is Server? ─── True ──► Spawn Actor from Class
    │                              Class: ST_Main
    │                              Spawn Transform: (0,0,0)
    │                              Collision: Always Spawn
    │                              │
    │                              └─► [Store reference if needed]
    │
    └─── False ──► [Do nothing - clients don't spawn tracker]
```

**Blueprint nodes to use:**
- `Event BeginPlay`
- `Is Server` (pure function)
- `Branch`
- `Spawn Actor from Class`

### Step 2.2: Create InitSpacerig

1. Right-click > Blueprint Class > Actor
2. Name it `InitSpacerig`
3. Open the blueprint
4. Keep it minimal for now:

```
Event BeginPlay
    │
    └─► [Optional: Print "Stats Tracker Loaded" for debugging]
```

The spacerig version can be expanded later to show last mission stats.

---

## Phase 3: Create Main Tracker Actor

### Step 3.1: Create ST_Main Blueprint

1. Right-click > Blueprint Class > Actor
2. Name it `ST_Main`
3. Configure Class Defaults:
   - `Replicates`: True
   - `Always Relevant`: True

### Step 3.2: Add Variables

Add these variables to ST_Main:

| Name | Type | Default | Replicated |
|------|------|---------|------------|
| `PlayerStats` | Array of ST_PlayerData | Empty | No |
| `GameState` | AFSDGameState (Object Ref) | None | No |
| `GameMode` | AFSDGameMode (Object Ref) | None | No |
| `IsTrackingActive` | Boolean | True | No |

### Step 3.3: Implement BeginPlay

```
Event BeginPlay
    │
    ├─► Get Game State ──► Cast to AFSDGameState ──► Set GameState variable
    │
    ├─► Get Game Mode ──► Cast to AFSDGameMode ──► Set GameMode variable
    │
    ├─► [Call] Initialize Player Stats
    │
    ├─► [Call] Subscribe to Events
    │
    └─► Print String: "Stats Tracker Active" (for debugging)
```

### Step 3.4: Create Initialize Player Stats Function

Create a function called `InitializePlayerStats`:

```
Function: InitializePlayerStats
    │
    ├─► Get All Actors of Class: AFSDPlayerController
    │       │
    │       └─► For Each Loop
    │               │
    │               ├─► Create ST_PlayerData struct
    │               │       - PlayerController = Current element
    │               │       - PlayerName = Get Player Name from controller
    │               │       - TotalKills = 0
    │               │       - TotalDamage = 0.0
    │               │       - KillsByCategory = Empty Map
    │               │       - DamageByCategory = Empty Map
    │               │
    │               └─► Add to PlayerStats array
    │
    └─► [Handle late joiners - see Subscribe to Events]
```

### Step 3.5: Create Subscribe to Events Function

```
Function: SubscribeToEvents
    │
    ├─► Bind Event to OnPlayerLoggedIn (from GameMode)
    │       └─► Create Event → OnPlayerJoined
    │
    ├─► [For mission end - need to find correct event]
    │   └─► Bind Event to OnMissionComplete or similar
    │           └─► Create Event → OnMissionEnded
    │
    └─► [For chat - implementation depends on available hooks]
```

### Step 3.6: Create Kill Tracking Logic

This is the core of the mod. We need to hook into enemy death events.

**Option A: Using Enemy Death Delegate (if available)**
```
For each enemy spawned:
    │
    └─► Bind to OnDeath delegate
            │
            └─► On Death:
                    ├─► Get Killer (instigator)
                    ├─► Get Enemy Class
                    ├─► Categorize Enemy
                    ├─► Find Player in PlayerStats
                    └─► Increment kill count
```

**Option B: Using Damage Events**
```
Hook into damage system:
    │
    └─► On Damage Dealt:
            ├─► Get Damage Causer (player)
            ├─► Get Damage Target (enemy)
            ├─► Get Damage Amount
            ├─► Categorize Enemy
            ├─► Find Player in PlayerStats
            ├─► Add to damage total
            │
            └─► If enemy health <= 0:
                    └─► Increment kill count
```

### Step 3.7: Create Categorize Enemy Function

```
Function: CategorizeEnemy
Input: EnemyActor (Actor Reference)
Output: ST_EnemyCategory

    │
    ├─► Get Class of EnemyActor
    │
    ├─► Get Class Name as String
    │
    └─► Switch on String (contains check):
            │
            ├─► Contains "Glyphid" → Return Glyphid
            ├─► Contains "Mactera" → Return Mactera
            ├─► Contains "Naedocyte" → Return Naedocyte
            ├─► Contains "Qronar" or "Shellback" → Return Qronar
            ├─► Contains "Patrol" or "Shredder" or "Turret" → Return Robot
            ├─► Contains "Dreadnought" → Return Dreadnought
            └─► Default → Return Other
```

See [ENEMY_CATEGORIES.md](ENEMY_CATEGORIES.md) for complete class name mappings.

### Step 3.8: Create Add Kill Function

```
Function: AddKill
Inputs: 
    - PlayerController (AFSDPlayerController)
    - Category (ST_EnemyCategory)

    │
    ├─► Find player in PlayerStats array (by PlayerController)
    │
    ├─► If found:
    │       ├─► Increment TotalKills
    │       │
    │       ├─► Get current kills for Category from KillsByCategory map
    │       │       └─► If not found, default to 0
    │       │
    │       └─► Set KillsByCategory[Category] = current + 1
    │
    └─► Update PlayerStats array with modified entry
```

### Step 3.9: Create Add Damage Function

```
Function: AddDamage
Inputs:
    - PlayerController (AFSDPlayerController)
    - Category (ST_EnemyCategory)
    - DamageAmount (Float)

    │
    ├─► Find player in PlayerStats array (by PlayerController)
    │
    ├─► If found:
    │       ├─► TotalDamage += DamageAmount
    │       │
    │       ├─► Get current damage for Category from DamageByCategory map
    │       │       └─► If not found, default to 0.0
    │       │
    │       └─► Set DamageByCategory[Category] = current + DamageAmount
    │
    └─► Update PlayerStats array with modified entry
```

---

## Phase 4: Chat Command Handling

### Step 4.1: Detect Chat Messages

This requires hooking into the chat system. Options:

**Option A: Tick-based chat polling**
```
Event Tick (or Timer)
    │
    └─► Check for new chat messages
            │
            └─► If message starts with "/stats":
                    └─► Call DisplayStats
```

**Option B: Chat delegate binding (if available)**
```
Bind to OnChatMessage delegate
    │
    └─► On Message Received:
            ├─► Get Message Text
            │
            └─► If starts with "/stats":
                    └─► Call DisplayStats
```

### Step 4.2: Create Display Stats Function

```
Function: DisplayStats
    │
    ├─► Create header: "=== MISSION STATS ==="
    │
    ├─► For Each player in PlayerStats:
    │       │
    │       ├─► Format line 1: "[PlayerName]: X kills | Y dmg"
    │       │
    │       ├─► Format line 2: "  Category1: N | Category2: M | ..."
    │       │       └─► Only include categories with kills > 0
    │       │
    │       └─► Add to output string
    │
    ├─► Add footer: "====================="
    │
    └─► Broadcast to all players (Server Say or similar)
```

### Step 4.3: Format Numbers

```
Function: FormatNumber
Input: Number (Float)
Output: String

    │
    ├─► If Number >= 1000:
    │       └─► Format with commas: "1,234"
    │
    └─► Else:
            └─► Convert to String directly
```

---

## Phase 5: Mission End Trigger

### Step 5.1: Hook Mission Complete

```
Bind to AFSDGameMode.OnMissionComplete (or similar event)
    │
    └─► On Mission Complete:
            │
            ├─► Set IsTrackingActive = False
            │
            ├─► Delay 2 seconds (let dust settle)
            │
            └─► Call DisplayStats
```

---

## Phase 6: Packaging

### Step 6.1: Prepare for Cooking

1. Delete any test/debug blueprints
2. Remove any Print String nodes (optional but cleaner)
3. Compile all blueprints (no errors)
4. Save all

### Step 6.2: Cook Content

1. In UE4: File > Cook Content for Windows

Or use DRGPacker scripts:

1. Create folder structure:
   ```
   Input/Content/ModStatsTracker/
   ```
2. Copy your `.uasset` and `.uexp` files there
3. Drag `Input` folder onto `_Repack.bat`

### Step 6.3: Test Locally

1. Rename output to `StatsTracker_P.pak`
2. Copy to: `Steam\steamapps\common\Deep Rock Galactic\FSD\Content\Paks\`
3. Launch DRG and test in a mission
4. Type `/stats` in chat
5. Complete mission and verify auto-display

### Step 6.4: Upload to mod.io

1. Remove `_P` suffix: `StatsTracker.pak`
2. Go to [drg.mod.io](https://drg.mod.io)
3. Click "+" to add mod
4. Fill in details:
   - Name: Stats Tracker
   - Description: Track kills and damage per player
   - Category: Gameplay
   - Tags: Approved/Verified as appropriate
5. Upload .pak file
6. Submit for review

---

## Testing Checklist

- [ ] Mod loads without errors
- [ ] Stats track for host player
- [ ] Stats track for all players (multiplayer)
- [ ] `/stats` command works
- [ ] Stats display on mission end
- [ ] Enemy categories are correct
- [ ] No crashes or performance issues
- [ ] Works in different mission types

---

## Common Issues

### Stats not tracking
- Ensure ST_Main is spawned (check Is Server)
- Verify event bindings are working
- Add debug Print String nodes

### Chat command not detected
- Check chat message format
- Verify case sensitivity
- Try alternative detection methods

### Wrong enemy categories
- Check class name matching
- Add debug output for enemy class names
- Update category mapping

### Multiplayer issues
- Ensure replication settings are correct
- Test with actual multiplayer (not just multiple clients)
- Check host vs client logic

---

## Next Steps

- Add persistence (save stats between sessions)
- Add more detailed enemy tracking
- Create settings UI with ModHub
- Add graphs/visualizations
- Track additional stats (resources, revives, etc.)
