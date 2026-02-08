# DRG Stats Tracker

A Deep Rock Galactic mod that tracks kills and damage statistics per player during missions.

## Features

- **Per-Player Stats**: Track kills and damage dealt by each player
- **Enemy Categories**: Statistics broken down by major enemy types (Glyphid, Mactera, Robot, etc.)
- **Chat Command**: Type `/stats` to view current mission statistics
- **Auto-Display**: Automatically shows stats when mission completes
- **Multiplayer Support**: Works as host-only mod (other players see results but don't need the mod)

## Example Output

```
=== MISSION STATS ===
[Karl]: 45 kills | 12,340 dmg
  Glyphid: 30 | Mactera: 10 | Robot: 5
[Driller]: 38 kills | 9,870 dmg
  Glyphid: 25 | Naedocyte: 8 | Other: 5
[Scout]: 52 kills | 8,450 dmg
  Glyphid: 40 | Mactera: 12
[Gunner]: 67 kills | 18,920 dmg
  Glyphid: 45 | Robot: 15 | Dreadnought: 7
=====================
```

## Requirements

- Deep Rock Galactic (Steam version)
- Mod.io account (created automatically through DRG)

## Installation

1. Subscribe to the mod on [mod.io](https://mod.io/g/drg)
2. Enable the mod in DRG's Modding menu
3. Host a mission to track stats

## Usage

- **During Mission**: Type `/stats` in chat to see current statistics
- **Mission End**: Stats are automatically displayed when the mission completes

## Development Status

This mod is currently in the **design phase**. The repository contains:

- Complete implementation documentation
- Blueprint pseudocode and logic
- Enemy category mappings
- Setup and build instructions

See [docs/SETUP.md](docs/SETUP.md) to get started with development.

## Project Structure

```
drg-stats-tracker/
├── README.md                 # This file
├── docs/
│   ├── SETUP.md             # UE4 development environment setup
│   ├── IMPLEMENTATION.md    # Step-by-step implementation guide
│   └── ENEMY_CATEGORIES.md  # Enemy type to category mapping
├── blueprints/
│   ├── InitCave.md          # Mission entry point logic
│   ├── InitSpacerig.md      # Spacerig entry point logic
│   ├── ST_Main.md           # Main stats tracker actor
│   ├── ST_PlayerData.md     # Player data structure
│   └── ST_ChatHandler.md    # Chat command handling
├── config/
│   ├── enemy_categories.json # Enemy class mappings
│   └── chat_format.json     # Output message templates
└── reference/
    ├── fsd_classes.md       # Relevant FSD C++ classes
    └── events.md            # Game events to hook
```

## Technical Overview

### Architecture

```
Mission Start (InitCave)
        │
        ▼
  Spawn ST_Main Actor
        │
        ├── Subscribe to OnEnemyKilled
        ├── Subscribe to OnDamageDealt
        ├── Subscribe to OnChatMessage
        └── Subscribe to OnMissionComplete
        │
        ▼
  Track Events → Update Stats Map
        │
        ▼
  On /stats or Mission End → Format & Broadcast
```

### Key Components

| Component | Purpose |
|-----------|---------|
| `InitCave` | Entry point - spawns tracker when mission loads |
| `InitSpacerig` | Minimal - could show last mission stats |
| `ST_Main` | Core logic - event handling, data storage, output |
| `ST_PlayerData` | Structure for per-player statistics |

### Enemy Categories

| Category | Examples |
|----------|----------|
| Glyphid | Grunt, Praetorian, Oppressor, Warden, Menace |
| Mactera | Spawn, Grabber, Goo Bomber, Tri-Jaw |
| Naedocyte | Shocker, Breeder, Hatchlings |
| Q'ronar | Shellback, Youngling |
| Robot | Patrol Bot, Shredder, Burst Turret, Sniper Turret |
| Dreadnought | Dreadnought, Twins, Hiveguard |
| Other | Cave Leech, Spitballer, Korlok, BET-C, etc. |

## Contributing

This project is open for contributions! Areas that need work:

1. Testing with FSD-Template
2. Verifying event hooks work correctly
3. Multiplayer replication testing
4. UI/formatting improvements

## Resources

- [DRG Modding Discord](https://discord.gg/zQMKGTStfa)
- [DRG Basic Modding Guide](https://mod.io/g/drg/r/drg-basic-modding-guide)
- [Blueprint Modding Guide](https://drg.mod.io/guides/how-to-blueprint-mod)
- [FSD-Template Project](https://github.com/DRG-Modding/FSD-Template)
- [Header Dumps](https://github.com/DRG-Modding/Header-Dumps)

## License

MIT License - See LICENSE file for details.

---

**Rock and Stone!**
