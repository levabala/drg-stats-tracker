# Enemy Categories

This document maps DRG enemy class names to their categories for the stats tracker.

## Category Overview

| Category | Description |
|----------|-------------|
| Glyphid | Main bug enemies - grunts, guards, praetorians, etc. |
| Mactera | Flying enemies - spawns, grabbers, bombers |
| Naedocyte | Jellyfish-like enemies - shockers, breeders |
| Q'ronar | Armored rolling enemies - shellbacks |
| Robot | Rival tech enemies - patrol bots, turrets |
| Dreadnought | Boss enemies - elimination targets |
| Other | Everything else - leeches, spitballers, plants |

---

## Glyphid Category

Main bug faction - most common enemies.

| Enemy Name | Class Name Pattern | Notes |
|------------|-------------------|-------|
| Glyphid Grunt | `Glyphid_Grunt` | Basic enemy |
| Glyphid Grunt Guard | `Glyphid_Grunt_Guard` | Armored variant |
| Glyphid Grunt Slasher | `Glyphid_Grunt_Slasher` | Fast variant |
| Glyphid Praetorian | `Glyphid_Praetorian` | Heavy armored |
| Glyphid Oppressor | `Glyphid_Oppressor` | Very heavy armored |
| Glyphid Warden | `Glyphid_Warden` | Buffs nearby enemies |
| Glyphid Menace | `Glyphid_Menace` | Burrowing ranged |
| Glyphid Web Spitter | `Glyphid_WebSpitter` | Ranged web attack |
| Glyphid Acid Spitter | `Glyphid_AcidSpitter` | Ranged acid attack |
| Glyphid Exploder | `Glyphid_Exploder` | Suicide bomber |
| Glyphid Bulk Detonator | `Glyphid_Bulk` | Large exploder |
| Glyphid Crassus Detonator | `Glyphid_Crassus` | Gold-dropping bulk |
| Glyphid Swarmer | `Glyphid_Swarmer` | Tiny swarm enemy |
| Glyphid Brood Nexus | `Glyphid_BroodNexus` | Spawns swarmers |
| Glyphid Stingtail | `Glyphid_Stingtail` | Season 3+ enemy |

**Detection Pattern**: Class name contains `Glyphid`

---

## Mactera Category

Flying enemies - aerial threats.

| Enemy Name | Class Name Pattern | Notes |
|------------|-------------------|-------|
| Mactera Spawn | `Mactera_Spawn` | Basic flyer |
| Mactera Grabber | `Mactera_Grabber` | Grabs and carries players |
| Mactera Goo Bomber | `Mactera_GooBomber` | Drops goo |
| Mactera Brundle | `Mactera_Brundle` | Tanky flyer |
| Mactera Tri-Jaw | `Mactera_TriJaw` | Shoots projectiles |
| Mactera Brundles Swarm | `Mactera_Swarm` | Small swarm variant |

**Detection Pattern**: Class name contains `Mactera`

---

## Naedocyte Category

Jellyfish-like floating enemies.

| Enemy Name | Class Name Pattern | Notes |
|------------|-------------------|-------|
| Naedocyte Shocker | `Naedocyte_Shocker` | Electric attack |
| Naedocyte Breeder | `Naedocyte_Breeder` | Spawns hatchlings |
| Naedocyte Hatchling | `Naedocyte_Hatchling` | Small spawned enemy |
| Naedocyte Roe | `Naedocyte_Roe` | Egg form |

**Detection Pattern**: Class name contains `Naedocyte`

---

## Q'ronar Category

Armored rolling enemies (Shellbacks).

| Enemy Name | Class Name Pattern | Notes |
|------------|-------------------|-------|
| Q'ronar Shellback | `Qronar_Shellback` or `Shellback` | Adult roller |
| Q'ronar Youngling | `Qronar_Youngling` | Small roller |

**Detection Pattern**: Class name contains `Qronar` or `Shellback`

---

## Robot Category

Rival tech enemies from Season 1+.

| Enemy Name | Class Name Pattern | Notes |
|------------|-------------------|-------|
| Patrol Bot | `PatrolBot` | Flying scout |
| Shredder | `Shredder` | Small aggressive drone |
| Burst Turret | `BurstTurret` | Stationary turret |
| Sniper Turret | `SniperTurret` | Long-range turret |
| Repulsion Turret | `RepulsionTurret` | Pushback turret |
| Prospector Drone | `Prospector` | Data cell carrier |
| Nemesis | `Nemesis` | Large robot hunter |
| Caretaker | `Caretaker` | Industrial Sabotage boss |

**Detection Pattern**: Class name contains `PatrolBot`, `Shredder`, `Turret`, `Prospector`, `Nemesis`, or `Caretaker`

---

## Dreadnought Category

Boss enemies - elimination mission targets.

| Enemy Name | Class Name Pattern | Notes |
|------------|-------------------|-------|
| Glyphid Dreadnought | `Dreadnought_Classic` or `Dreadnought_Normal` | Standard boss |
| Dreadnought Twins | `Dreadnought_Twin` | Split into two |
| Dreadnought Hiveguard | `Dreadnought_Hiveguard` | Sentinel-protected |
| Glyphid Dreadnought (Arbalest) | `Dreadnought_Arbalest` | Season variant |

**Detection Pattern**: Class name contains `Dreadnought`

---

## Other Category

Everything that doesn't fit above categories.

| Enemy Name | Class Name Pattern | Notes |
|------------|-------------------|-------|
| Cave Leech | `CaveLeech` | Ceiling grabber |
| Spitball Infector | `Spitballer` | Stationary plant |
| Korlok Tyrant Weed | `Korlok` | Boss plant |
| Korlok Healing Pod | `Korlok_Heal` | Heals tyrant weed |
| Korlok Sprout | `Korlok_Sprout` | Attacks from weed |
| BET-C | `BETC` | Corrupted robot |
| Loot Bug | `LootBug` | Friendly bug (don't kill!) |
| Huuli Hoarder | `Huuli` | Treasure bug |
| Fester Flea | `FesterFlea` | Jumpy bug |
| Stabber Vine | `StabberVine` | Environment hazard |
| Deeptora Honeycomb | `DeeptoraHoneycomb` | Fungus Bogs hazard |
| Carnivorous Larva | `CarnivorousLarva` | Hollow Bough enemy |
| Rockpox Larvae | `RockpoxLarvae` | Season 4 enemies |
| Lithophage Meteor | `Lithophage` | Rockpox source |

**Detection Pattern**: Default fallback for anything not matched above

---

## Implementation: Categorize Enemy Function

Pseudocode for categorizing an enemy by class name:

```
Function CategorizeEnemy(EnemyActor):
    className = GetClassName(EnemyActor)
    classNameLower = ToLowerCase(className)
    
    // Check in priority order (most specific first)
    
    if contains(classNameLower, "dreadnought"):
        return Dreadnought
    
    if contains(classNameLower, "glyphid"):
        return Glyphid
    
    if contains(classNameLower, "mactera"):
        return Mactera
    
    if contains(classNameLower, "naedocyte"):
        return Naedocyte
    
    if contains(classNameLower, "qronar") or contains(classNameLower, "shellback"):
        return Qronar
    
    if contains(classNameLower, "patrolbot") or 
       contains(classNameLower, "shredder") or
       contains(classNameLower, "turret") or
       contains(classNameLower, "prospector") or
       contains(classNameLower, "nemesis") or
       contains(classNameLower, "caretaker"):
        return Robot
    
    // Default fallback
    return Other
```

---

## Blueprint Implementation

In UE4 Blueprint, use this pattern:

1. Get the enemy actor's class: `Get Class`
2. Get the class name: `Get Display Name` or `Get Name`
3. Convert to lowercase: `To Lower`
4. Use `Contains` nodes to check for patterns
5. Chain with `Branch` nodes in priority order
6. Return the matching `ST_EnemyCategory` enum value

---

## Notes

### Class Name Variations
- Actual class names may vary from display names
- Use FModel or UAssetGUI to find exact class names in game files
- Some enemies have multiple variants (normal, hardened, etc.)

### Season Content
- Robots added in Season 1
- New enemies may be added in future seasons
- Update this list as new content releases

### Finding Class Names
1. Extract game files with DRGPacker
2. Navigate to `FSD\Content\Enemies\`
3. Open enemy blueprints in FModel
4. Note the exact class names used

### Testing
- Add debug output to log enemy class names during gameplay
- Build a mapping of all encountered enemies
- Refine detection patterns based on actual data
