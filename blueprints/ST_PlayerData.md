# ST_PlayerData Structure

The data structure that holds per-player statistics.

## Structure Details

| Property | Value |
|----------|-------|
| **Name** | ST_PlayerData |
| **Type** | Structure (UserDefinedStruct) |
| **Location** | Content/ModStatsTracker/ST_PlayerData |

## How to Create in UE4

1. In Content Browser, navigate to `ModStatsTracker` folder
2. Right-click in empty space
3. Select **Blueprints → Structure**
4. Name it `ST_PlayerData`
5. Double-click to open the Structure Editor

## Structure Members

Add these variables in order:

### 1. PlayerName

| Property | Value |
|----------|-------|
| **Name** | PlayerName |
| **Type** | String |
| **Default Value** | "" (empty string) |
| **Description** | Display name of the player |

### 2. PlayerController

| Property | Value |
|----------|-------|
| **Name** | PlayerController |
| **Type** | AFSDPlayerController (Object Reference) |
| **Default Value** | None |
| **Description** | Reference to the player's controller |

**Note**: To set the type:
1. Click the type dropdown
2. Search for "AFSDPlayerController"
3. Select "Object Reference" variant

### 3. TotalKills

| Property | Value |
|----------|-------|
| **Name** | TotalKills |
| **Type** | Integer |
| **Default Value** | 0 |
| **Description** | Total number of enemies killed |

### 4. TotalDamage

| Property | Value |
|----------|-------|
| **Name** | TotalDamage |
| **Type** | Float |
| **Default Value** | 0.0 |
| **Description** | Total damage dealt to enemies |

### 5. KillsByCategory

| Property | Value |
|----------|-------|
| **Name** | KillsByCategory |
| **Type** | Map (ST_EnemyCategory → Integer) |
| **Default Value** | {} (empty map) |
| **Description** | Kill count per enemy category |

**How to set Map type**:
1. Click type dropdown
2. Select "Map"
3. Set Key type: `ST_EnemyCategory` (your enum)
4. Set Value type: `Integer`

### 6. DamageByCategory

| Property | Value |
|----------|-------|
| **Name** | DamageByCategory |
| **Type** | Map (ST_EnemyCategory → Float) |
| **Default Value** | {} (empty map) |
| **Description** | Damage dealt per enemy category |

**How to set Map type**:
1. Click type dropdown
2. Select "Map"
3. Set Key type: `ST_EnemyCategory` (your enum)
4. Set Value type: `Float`

---

## Structure Preview

After adding all members, your structure should look like this:

```
ST_PlayerData
├── PlayerName: String = ""
├── PlayerController: AFSDPlayerController (Object Ref) = None
├── TotalKills: Integer = 0
├── TotalDamage: Float = 0.0
├── KillsByCategory: Map<ST_EnemyCategory, Integer> = {}
└── DamageByCategory: Map<ST_EnemyCategory, Float> = {}
```

---

## Usage in Blueprints

### Creating a New Instance

```
┌────────────────────────────────┐
│ Make ST_PlayerData             │
│                                │
│ PlayerName ◄── [String value]  │
│ PlayerController ◄── [Ref]     │
│ TotalKills ◄── 0               │
│ TotalDamage ◄── 0.0            │
│ KillsByCategory ◄── [Empty]    │
│ DamageByCategory ◄── [Empty]   │
│                                │
│ Return Value ──►               │
└────────────────────────────────┘
```

### Breaking Apart (Reading Values)

```
┌────────────────────────────────┐
│ Break ST_PlayerData            │
│                                │
│ Input ◄── [ST_PlayerData]      │
│                                │
│ PlayerName ──►                 │
│ PlayerController ──►           │
│ TotalKills ──►                 │
│ TotalDamage ──►                │
│ KillsByCategory ──►            │
│ DamageByCategory ──►           │
└────────────────────────────────┘
```

### Modifying a Value

To update a single field while keeping others:

1. Break the struct to get all values
2. Modify the value you need
3. Make a new struct with updated value
4. Replace in array

```
[Get from Array]
      │
      ▼
[Break ST_PlayerData] ─► Get all values
      │
      ▼
[Modify TotalKills] ─► TotalKills + 1
      │
      ▼
[Make ST_PlayerData] ─► Use modified value + unchanged values
      │
      ▼
[Set Array Element] ─► Replace at same index
```

### Working with Maps

**Adding to KillsByCategory**:

```
[Break struct] ─► Get KillsByCategory
      │
      ▼
[Find] ─► Look for Category key
      │
      ├─► Found: Get current value
      │
      └─► Not Found: Use 0
      │
      ▼
[Add] ─► Put Category → (CurrentValue + 1)
      │
      ▼
[Make struct with updated map]
```

**Getting from Map**:

```
[KillsByCategory] ──► [Find] ◄── Category (Key)
                          │
                          ├─► Found: Return Value (Integer)
                          │
                          └─► Not Found: Return 0 (default)
```

---

## Example: Recording a Kill

Full flow for adding a kill to a player's stats:

```
Input: PlayerController, EnemyCategory
            │
            ▼
┌─────────────────────────────────────────┐
│ Find player index in PlayerStats array │
│ (Loop through, compare PlayerController)│
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Get element at index                    │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Break ST_PlayerData                     │
│ - Get TotalKills                        │
│ - Get KillsByCategory                   │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ NewTotalKills = TotalKills + 1          │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Find EnemyCategory in KillsByCategory   │
│ CurrentCategoryKills = Found ? Value : 0│
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Add to Map:                             │
│ EnemyCategory → CurrentCategoryKills + 1│
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Make ST_PlayerData:                     │
│ - PlayerName = (unchanged)              │
│ - PlayerController = (unchanged)        │
│ - TotalKills = NewTotalKills            │
│ - TotalDamage = (unchanged)             │
│ - KillsByCategory = (updated map)       │
│ - DamageByCategory = (unchanged)        │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Set Array Element:                      │
│ PlayerStats[index] = NewPlayerData      │
└─────────────────────────────────────────┘
```

---

## Related Types

### ST_EnemyCategory Enum

Must be created before this struct (used in Maps):

```
ST_EnemyCategory (Enum)
├── Glyphid
├── Mactera
├── Naedocyte
├── Qronar
├── Robot
├── Dreadnought
└── Other
```

See [ST_Main.md](ST_Main.md) for how to create the enum.

---

## Notes

### Why Use a Struct?

1. **Organization**: Groups related data together
2. **Arrays**: Can have an array of player stats
3. **Passing Data**: Easy to pass as function parameter
4. **Modification**: Can break apart and rebuild

### Maps vs Arrays for Categories

Using Maps (ST_EnemyCategory → Integer) instead of Arrays:

**Pros**:
- Direct lookup by category
- No need to maintain index positions
- Easy to check if category exists

**Cons**:
- Slightly more complex Blueprint nodes
- Need to handle "not found" case

### Default Values

All numeric defaults are 0/0.0 so new players start with clean stats.

Maps default to empty `{}` so categories are only added when first kill/damage occurs.

---

## Related Files

- [ST_Main.md](ST_Main.md) - Uses this structure
- [../docs/ENEMY_CATEGORIES.md](../docs/ENEMY_CATEGORIES.md) - Category definitions
