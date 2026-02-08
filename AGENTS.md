# AGENTS.md - Project Context for AI Assistants

This file provides context for AI coding agents working on the DRG Stats Tracker project.

---

## PROJECT OVERVIEW

**Project Name:** DRG Stats Tracker  
**Type:** Deep Rock Galactic Mod (Unreal Engine 4 Blueprint)  
**Status:** Design/Documentation Phase → Implementation Phase  
**Platform:** Windows (UE4 4.27.2)  
**Language:** Unreal Engine Blueprints (visual scripting)

### Purpose
A mod for Deep Rock Galactic that tracks kills and damage statistics per player during missions. Stats are displayed via chat command (`/stats`) and automatically at mission end.

### Key Features
- Per-player kill and damage tracking
- Enemy categorization (Glyphid, Mactera, Robot, Dreadnought, etc.)
- `/stats` chat command for real-time statistics
- Auto-display on mission completion
- Host-only mod (multiplayer compatible)

---

## PROJECT STRUCTURE

```
D:\projects\
├── drg-stats-tracker/          # Documentation repository (THIS REPO)
│   ├── README.md               # Project overview
│   ├── AGENTS.md               # This file - AI context
│   ├── LICENSE                 # MIT License
│   ├── docs/
│   │   ├── SETUP.md           # Environment setup guide
│   │   ├── IMPLEMENTATION.md  # Blueprint implementation steps
│   │   └── ENEMY_CATEGORIES.md # Enemy type mappings
│   ├── blueprints/
│   │   ├── InitCave.md        # Mission entry point logic
│   │   ├── InitSpacerig.md    # Spacerig entry point logic
│   │   ├── ST_Main.md         # Main tracker actor
│   │   ├── ST_PlayerData.md   # Player data structure
│   │   └── ST_ChatHandler.md  # Chat command handling
│   ├── config/
│   │   ├── enemy_categories.json  # Enemy class mappings
│   │   └── chat_format.json       # Message templates
│   └── reference/
│       ├── fsd_classes.md     # FSD C++ class reference
│       └── events.md          # Game event hooks
│
├── FSD-Template/               # Unreal Engine 4 project (EXTERNAL)
│   ├── FSD.uproject           # UE4 project file
│   ├── Content/               # UE4 assets
│   │   └── ModStatsTracker/   # Mod blueprints (to be created)
│   └── Source/                # C++ class stubs
│
└── Header-Dumps/               # DRG C++ reference (EXTERNAL)
    └── FSD.hpp                # Game class definitions
```

---

## DEVELOPMENT ENVIRONMENT

### Software Requirements
- **Operating System:** Windows 10/11
- **Unreal Engine:** 4.27.2 (exact version required)
- **IDE:** Visual Studio 2019+ with C++ workloads
- **Version Control:** Git

### External Dependencies
- **FSD-Template:** `git clone https://github.com/DRG-Modding/FSD-Template.git`
- **Header-Dumps:** `git clone https://github.com/DRG-Modding/Header-Dumps.git`
- **DRG Modding Tools:** Available via DRG Modding Discord

### Setup Status
- ✅ Visual Studio 2019+ installed
- ✅ Unreal Engine 4.27.2 installed
- ✅ FSD-Template cloned to `D:\projects\FSD-Template`
- ✅ Header-Dumps cloned to `D:\projects\Header-Dumps`
- ⏳ FSD-Template build pending (manual step required)

---

## ARCHITECTURE

### Component Overview

| Component | Type | Purpose | Status |
|-----------|------|---------|--------|
| `InitCave` | Actor Blueprint | Spawns tracker on mission start | Design |
| `InitSpacerig` | Actor Blueprint | Minimal spacerig handler | Design |
| `ST_Main` | Actor Blueprint | Core stats tracking logic | Design |
| `ST_PlayerData` | Struct | Per-player statistics storage | Design |
| `ST_EnemyCategory` | Enum | Enemy type categorization | Design |
| `ST_ChatHandler` | Component | Chat command processing | Design |

### Data Flow

```
Mission Start (InitCave)
    ↓
Spawn ST_Main Actor (server-only)
    ↓
Initialize PlayerStats array
    ↓
Subscribe to game events:
    - OnEnemyKilled
    - OnDamageDealt
    - OnChatMessage
    - OnMissionComplete
    ↓
Track events → Update PlayerStats
    ↓
On /stats command or mission end:
    Format stats → Broadcast to chat
```

### Enemy Categories

- **Glyphid:** Grunt, Praetorian, Oppressor, Warden, Menace
- **Mactera:** Spawn, Grabber, Goo Bomber, Tri-Jaw
- **Naedocyte:** Shocker, Breeder, Hatchlings
- **Q'ronar:** Shellback, Youngling
- **Robot:** Patrol Bot, Shredder, Turrets
- **Dreadnought:** Dreadnought, Twins, Hiveguard
- **Other:** Cave Leech, Spitballer, Korlok, BET-C, etc.

---

## IMPLEMENTATION PHASES

### Phase 1: Basic Structures ⏳
- [ ] Create `ST_EnemyCategory` enum
- [ ] Create `ST_PlayerData` struct
- [ ] Set up `ModStatsTracker` folder in FSD-Template

### Phase 2: Entry Points ⏳
- [ ] Implement `InitCave` blueprint
- [ ] Implement `InitSpacerig` blueprint
- [ ] Test spawn logic

### Phase 3: Core Tracker ⏳
- [ ] Create `ST_Main` actor
- [ ] Implement `InitializePlayerStats` function
- [ ] Implement `SubscribeToEvents` function
- [ ] Implement `CategorizeEnemy` function
- [ ] Implement `AddKill` function
- [ ] Implement `AddDamage` function

### Phase 4: Chat Integration ⏳
- [ ] Implement chat message detection
- [ ] Create `DisplayStats` function
- [ ] Format output with categories

### Phase 5: Testing ⏳
- [ ] Test in single-player
- [ ] Test in multiplayer
- [ ] Verify all enemy categories
- [ ] Performance testing

### Phase 6: Packaging ⏳
- [ ] Cook content for Windows
- [ ] Package with DRGPacker
- [ ] Upload to mod.io

---

## TECHNICAL NOTES

### Blueprint Implementation Notes
- All tracking runs **server-only** (use `Is Server` checks)
- Replication settings: `Replicates: True`, `Always Relevant: True`
- Event hooks may require experimentation (FSD API not fully documented)
- Chat command detection options: polling vs delegate binding

### Known Challenges
1. **Event Hooking:** FSD API documentation is incomplete
   - May need to experiment with delegates
   - Alternative: poll/tick-based approaches
   
2. **Enemy Classification:** Class name matching may require refinement
   - Use `Contains` checks on class name strings
   - See `config/enemy_categories.json` for mappings

3. **Multiplayer Replication:** Host-only mod with broadcast
   - Stats calculated on server
   - Results broadcast to all clients via chat

4. **Performance:** Minimize overhead
   - Use efficient data structures (Maps for category lookups)
   - Avoid per-tick operations where possible

---

## CODING STANDARDS

### File Naming
- Blueprint prefix: `ST_` (Stats Tracker)
- Enums: `ST_<Name>` (e.g., `ST_EnemyCategory`)
- Structs: `ST_<Name>Data` (e.g., `ST_PlayerData`)
- Functions: PascalCase (e.g., `InitializePlayerStats`)

### Blueprint Organization
- Keep Event Graph clean and commented
- Use functions for reusable logic
- Name variables descriptively
- Add tooltips to public variables

### Documentation
- Update blueprint .md files when implementation differs from design
- Document any workarounds or API discoveries
- Note performance considerations

---

## RESOURCES

### Official Documentation
- [DRG Modding Discord](https://discord.gg/zQMKGTStfa) - Primary community resource
- [FSD-Template GitHub](https://github.com/DRG-Modding/FSD-Template)
- [Header Dumps GitHub](https://github.com/DRG-Modding/Header-Dumps)
- [Blueprint Modding Guide](https://drg.mod.io/guides/how-to-blueprint-mod)

### Useful Channels (Discord)
- `#mod-questions` - Technical help
- `#learn-tools` - Modding tool downloads
- `#blueprint-modding` - Blueprint-specific discussion
- `#showcase` - See other mods for inspiration

---

## AI AGENT WORKFLOW RULES

### 🎯 MANDATORY TODO ITEMS

**ALWAYS add these TODO items at the END of every task list:**

1. **UPDATE AGENTS.MD WITH NEW FINDINGS** (priority: medium)
   - Add any new discoveries about the FSD API
   - Document workarounds or solutions found
   - Update implementation status
   - Note any architectural changes

2. **COMMIT&PUSH** (priority: medium)
   - Commit all changes with descriptive message
   - Push to remote repository
   - Ensures work is never lost

### Task Management
- Use TodoWrite tool for all multi-step tasks
- Mark todos as `in_progress` before starting
- Mark as `completed` immediately after finishing (don't batch)
- Only one task `in_progress` at a time

### Git Workflow
- Commit after completing significant work units
- Use descriptive commit messages (present tense)
- Format: `<action>: <description>`
  - Examples: `add: InitCave blueprint implementation`
  - Examples: `update: enemy category mappings`
  - Examples: `fix: multiplayer replication issue`

### Documentation Updates
- Update relevant .md files when implementation diverges from design
- Keep AGENTS.md current with latest findings
- Add new sections for discovered patterns or APIs

### Communication
- Be concise and technical
- Provide file paths with line numbers when relevant
- Explain "why" for non-obvious decisions
- Flag blockers or uncertainties clearly

---

## CURRENT STATUS

**Last Updated:** 2026-02-08 21:15 UTC

### Completed
- ✅ Project repository structure created
- ✅ Complete documentation written
- ✅ AGENTS.md created with workflow rules
- ✅ Visual Studio 2019+ installed (user completed)
- ✅ Unreal Engine 4.27.2 installed at `D:\unreal\UE_4.27` ✅
- ✅ FSD-Template cloned to `D:\projects\FSD-Template`
- ✅ Header-Dumps cloned to `D:\projects\Header-Dumps`
- ✅ Visual Studio project files generated (`FSD.sln` created)
- ✅ UE4 Editor launched with FSD-Template project
- ✅ Comprehensive blueprint implementation guide created (`docs/BLUEPRINT_GUIDE.md`)
- ✅ Quick reference card created (`QUICKSTART.md`)

### In Progress
- ⏳ UE4 Editor compiling shaders in background (~30-60 min expected)
- ⏳ User creating blueprints manually in UE4 Editor

### Next Steps (User Action Required - Blueprint Creation in UE4)
**Follow `QUICKSTART.md` or `docs/BLUEPRINT_GUIDE.md`:**

1. **Verify UE4 Editor opened** (should be running now)
2. **Create folder structure** in Content Browser:
   - `Content/ModStatsTracker/Blueprints/`
   - `Content/ModStatsTracker/Data/`
   - `Content/ModStatsTracker/UI/`
3. **Create data structures:**
   - `ST_EnemyCategory` enum (7 categories)
   - `ST_PlayerData` struct (6 fields)
4. **Create entry point actors:**
   - `InitCave` blueprint
   - `InitSpacerig` blueprint
5. **Create core tracker:**
   - `ST_Main` blueprint with functions
6. **Implement event hooks** (experimental - requires testing)
7. **Test in Play-In-Editor (PIE)**

### Blockers
- None - awaiting user to manually create blueprints in UE4 Editor

### Recent Findings (2026-02-08 Session 2)

**Environment Setup Discoveries:**
1. **Pre-compiled Binaries:** FSD-Template includes pre-built DLLs (`UE4Editor-FSD.dll`, `UE4Editor-FSDEngine.dll`)
   - No compilation prompts needed on first open
   - Project ready to use immediately after shader compilation
   
2. **UE4 Installation Path:** Located at `D:\unreal\UE_4.27` (not default Epic Games location)
   - Registry key confirmed: `HKLM\SOFTWARE\EpicGames\Unreal Engine\4.27`
   
3. **VS Project Generation:** Successfully automated via UnrealBuildTool.exe
   - Command: `UnrealBuildTool.exe -projectfiles -project="..." -game -engine -progress`
   - Generated `FSD.sln` + IntelliSense data
   
4. **Visual Studio Error Was Misleading:** User attempted to open .uproject in VS directly
   - Correct workflow: Open .uproject in UE4 first, then use generated .sln for C++ editing
   - For blueprint-only mods, VS not required for daily workflow

5. **Blueprint Implementation Must Be Manual:** Cannot automate blueprint creation via CLI
   - UE4 blueprints are binary `.uasset` files requiring editor GUI
   - Created comprehensive guides instead: `BLUEPRINT_GUIDE.md` (46KB detailed walkthrough)

**FSD API Research Needed:**
- Event delegates for kill/damage tracking are undocumented
- Will require experimentation or community consultation
- Fallback: Tick-based health monitoring (less efficient but functional)

**Documentation Created This Session:**
- `docs/BLUEPRINT_GUIDE.md` - Complete node-by-node implementation instructions
- `QUICKSTART.md` - Quick reference for immediate blueprint creation
- Updated `AGENTS.md` - This section

---

## VERSION HISTORY

### v0.1.0 - Design Phase (2026-02-08)
- Initial repository structure
- Complete documentation suite
- Blueprint pseudocode and logic
- Enemy category mappings
- Setup instructions

---

## NOTES FOR FUTURE AGENTS

### When Starting Work
1. Read this file first (you're doing it right!)
2. Check "CURRENT STATUS" section for latest state
3. Review relevant docs in `docs/` and `blueprints/`
4. Verify environment setup (see DEVELOPMENT ENVIRONMENT)

### When Making Changes
1. Update implementation status in this file
2. Document any FSD API discoveries
3. Note workarounds or alternate approaches
4. Commit changes with clear messages

### When Encountering Issues
1. Check `docs/SETUP.md` troubleshooting section
2. Search Header-Dumps for relevant classes/functions
3. Consult DRG Modding Discord if blocked
4. Document the issue and solution here

### Blueprint-Specific Tips
- FSD classes are in `AFSDGameState`, `AFSDGameMode`, etc.
- Player controller: `AFSDPlayerController`
- Enemy base class: investigate in Header-Dumps
- Event delegates: may be named `On<EventName>`

---

**Rock and Stone!** ⛏️
