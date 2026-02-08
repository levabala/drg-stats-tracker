# DRG Stats Tracker - Quick Start Guide

**Current Status:** UE4 Editor Launched ✅  
**Ready to Create:** Blueprints in UE4 Editor

---

## 🎯 What to Do Right Now

### 1. Verify UE4 Editor is Open
- **Look for:** Unreal Engine window title showing "FSD - Unreal Editor"
- **Check:** Content Browser panel is visible at bottom
- **Ignore:** "Compiling Shaders" progress (runs in background, ~30-60 min)

### 2. Create Folder Structure (3 minutes)

**In Content Browser:**
```
Right-click "Content" → New Folder → "ModStatsTracker"
Inside ModStatsTracker, create:
  - Blueprints/
  - Data/
  - UI/
```

### 3. Create Data Structures (5 minutes)

**A) Enemy Category Enum:**
```
Location: Content/ModStatsTracker/Data/
1. Right-click → Blueprints → Enumeration
2. Name: ST_EnemyCategory
3. Add 7 entries: Glyphid, Mactera, Naedocyte, QRonar, Robot, Dreadnought, Other
4. Save
```

**B) Player Data Struct:**
```
Location: Content/ModStatsTracker/Data/
1. Right-click → Blueprints → Structure
2. Name: ST_PlayerData
3. Add variables:
   - PlayerName (String)
   - TotalKills (Integer)
   - TotalDamage (Float)
   - KillsByCategory (Map: ST_EnemyCategory → Integer)
   - DamageByCategory (Map: ST_EnemyCategory → Float)
   - PlayerController (PlayerController reference)
4. Save
```

### 4. Create Entry Point Actors (10 minutes)

**A) InitCave:**
```
Location: Content/ModStatsTracker/Blueprints/
1. Right-click → Blueprint Class → Actor
2. Name: InitCave
3. Class Defaults: Replicates=True, Always Relevant=True
4. Event Graph:
   Event BeginPlay → Switch Has Authority → Spawn Actor (ST_Main)
5. Compile & Save
```

**B) InitSpacerig:**
```
Same as InitCave but minimal (just Event BeginPlay → Print String)
```

### 5. Create Core Tracker (20 minutes)

**ST_Main Actor:**
```
Location: Content/ModStatsTracker/Blueprints/
1. Right-click → Blueprint Class → Actor
2. Name: ST_Main
3. Class Defaults: Replicates=True, Always Relevant=True
4. Add Variables:
   - PlayerStats (Array<ST_PlayerData>, Replicated)
   - bIsTracking (Boolean, Replicated)
   - GameStateRef (FSDGameState reference)
5. Add Functions:
   - InitializePlayerStats()
   - CategorizeEnemy(Actor) → ST_EnemyCategory
   - AddKill(Actor, PlayerController)
   - AddDamage(Actor, PlayerController, Float)
   - DisplayStats() → String
6. Save
```

**⚠️ Event Hooks:** This is experimental - see full guide (docs/BLUEPRINT_GUIDE.md)

---

## 📖 Full Documentation

- **Step-by-step blueprints:** `docs/BLUEPRINT_GUIDE.md` (46KB guide just created)
- **Project overview:** `README.md`
- **Setup troubleshooting:** `docs/SETUP.md`
- **AI agent context:** `AGENTS.md`

---

## 🐛 Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| UE4 Editor not opening | Check Task Manager - may be loading in background |
| Can't find Content folder | Window → Content Browser |
| "Module missing" error | Click "Yes" to rebuild |
| Shaders taking forever | Normal! Continue working, they compile in background |
| Can't create blueprints | Ensure you're in correct folder, not a plugin folder |

---

## 📊 Implementation Progress Checklist

### Phase 1: Data Structures
- [ ] Create ModStatsTracker folder
- [ ] Create ST_EnemyCategory enum (7 categories)
- [ ] Create ST_PlayerData struct (6 fields)

### Phase 2: Entry Points
- [ ] Create InitCave actor
- [ ] Create InitSpacerig actor
- [ ] Test spawn in mission

### Phase 3: Core Tracker
- [ ] Create ST_Main actor
- [ ] Add variables (PlayerStats, bIsTracking, GameStateRef)
- [ ] Implement InitializePlayerStats function
- [ ] Implement CategorizeEnemy function
- [ ] Implement AddKill function
- [ ] Implement AddDamage function
- [ ] Implement DisplayStats function

### Phase 4: Event Hooks (EXPERIMENTAL)
- [ ] Research FSD event delegates in Header-Dumps
- [ ] Test binding to kill/damage events
- [ ] Fallback: Implement tick-based monitoring if needed

### Phase 5: Chat Integration
- [ ] Implement chat command detection
- [ ] Test /stats command
- [ ] Add mission-end auto-display

### Phase 6: Testing
- [ ] Test solo mission
- [ ] Test multiplayer
- [ ] Test all enemy categories
- [ ] Performance test (Hazard 5)

### Phase 7: Packaging
- [ ] Use DRGPacker to package mod
- [ ] Upload to mod.io
- [ ] Share in Discord #showcase

---

## 🎮 Testing in Game

**After creating blueprints:**
1. Press "Play" button in UE4 toolbar (PIE - Play In Editor)
2. Check Output Log for "Stats Tracker Spawned" message
3. If errors: Fix and iterate
4. Once working in PIE: Package for real game testing

**Note:** PIE may not fully simulate DRG multiplayer - final testing needs packaged mod

---

## 🔗 Useful Resources

- **DRG Modding Discord:** https://discord.gg/zQMKGTStfa
- **FSD-Template GitHub:** https://github.com/DRG-Modding/FSD-Template
- **Header Dumps:** `D:\projects\Header-Dumps\FSD.hpp` (C++ reference)
- **Blueprint Guide:** https://drg.mod.io/guides/how-to-blueprint-mod

---

## 💡 Pro Tips

1. **Save frequently** (Ctrl+S) - UE4 can crash
2. **Use Print String nodes** liberally for debugging
3. **Check Output Log** (Window → Developer Tools → Output Log) for errors
4. **Compile often** (green checkmark) to catch errors early
5. **Test incrementally** - don't build everything before first test
6. **Ask Discord** if stuck - community is very helpful

---

## 🚀 Next Steps After Blueprint Creation

Once all blueprints are created and tested in PIE:

1. **Package Mod:**
   - Download DRGPacker from Discord #learn-tools
   - Follow packaging guide
   - Test in standalone DRG game

2. **Iterate:**
   - Gather feedback from test missions
   - Fix bugs
   - Optimize performance

3. **Release:**
   - Upload to mod.io
   - Share in Discord
   - Rock and Stone! ⛏️

---

**Current Session Files:**
- `D:\projects\drg-stats-tracker\docs\BLUEPRINT_GUIDE.md` - Detailed implementation guide
- `D:\projects\FSD-Template\FSD.uproject` - Open in UE4 Editor (should be open now)
- `D:\projects\Header-Dumps\FSD.hpp` - Reference for FSD C++ classes

**Ready to start creating blueprints!** Open the BLUEPRINT_GUIDE.md for detailed node-by-node instructions.
