# Game Events Reference

Events and delegates available in DRG for hooking into game systems.

---

## Priority Events for Stats Tracker

These are the most important events to find and hook:

### 1. Kill/Death Events (Critical)

| Event | Source | Purpose |
|-------|--------|---------|
| `OnEnemyKilled` | Player/Weapon | When player kills enemy |
| `OnDeath` | Enemy/HealthComponent | When enemy dies |
| `OnKill` | DamageComponent | When damage kills target |

**What we need**:
- Who killed (player reference)
- What was killed (enemy reference)
- Optionally: how much damage dealt

### 2. Damage Events (Important)

| Event | Source | Purpose |
|-------|--------|---------|
| `OnDamageDealt` | DamageComponent/Weapon | When damage is dealt |
| `OnDamageReceived` | HealthComponent | When actor takes damage |
| `OnHealthChanged` | HealthComponent | Any health change |

**What we need**:
- Who caused damage (player reference)
- Who received damage (enemy reference)
- Damage amount

### 3. Player Events (Important)

| Event | Source | Purpose |
|-------|--------|---------|
| `OnPlayerLoggedIn` | GameMode | New player joins |
| `OnPlayerLoggedOut` | GameMode | Player leaves |
| `OnPlayerSpawned` | GameState | Player spawns in |

**What we need**:
- Player controller reference
- Player name

### 4. Mission Events (Required)

| Event | Source | Purpose |
|-------|--------|---------|
| `OnMissionComplete` | GameMode/GameState | Mission ends |
| `OnMissionStarted` | GameMode/GameState | Mission begins |
| `OnMissionEnded` | GameMode/GameState | Any mission end |

**What we need**:
- When mission completes (to auto-display stats)

### 5. Chat Events (Nice to Have)

| Event | Source | Purpose |
|-------|--------|---------|
| `OnChatMessage` | ChatManager | New chat message |
| `OnPlayerSay` | PlayerController | Player sends message |

**What we need**:
- Message text
- Sender reference

---

## How to Bind Events in Blueprint

### Delegate Binding Pattern

```
┌─────────────────────────────────────────┐
│ Get Reference to Object with Delegate  │
│ (e.g., Get Game Mode)                   │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Cast to specific class                  │
│ (e.g., Cast to AFSDGameMode)            │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Drag off cast result                    │
│ Search for "Bind Event to..."          │
│ Select the delegate you want           │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Create Custom Event                     │
│ Wire to Event pin of Bind node         │
└─────────────────────────────────────────┘
```

### Event Signature

The custom event must match the delegate's signature:

```
Delegate: OnPlayerLoggedIn(AFSDPlayerController* Player)
                 │
                 ▼
Custom Event: OnPlayerJoined
    Input: Player (AFSDPlayerController Reference)
```

---

## Finding Events in FSD-Template

### Search in Header Dumps

Open `FSD.hpp` and search for:

```
// Common delegate naming patterns
DECLARE_DYNAMIC_MULTICAST_DELEGATE
FOn*
*Delegate
OnPlayer*
OnEnemy*
OnDamage*
OnMission*
```

### Check Blueprint Availability

In UE4, after casting to a class:
1. Drag off the reference
2. Type "Bind Event" in search
3. Available delegates will appear

If a delegate doesn't appear, it may be:
- C++ only (not exposed to Blueprint)
- Internal use only
- Named differently

---

## Event Implementation Examples

### Example 1: Bind to Player Join

```
Event BeginPlay
    │
    ▼
Get Game Mode
    │
    ▼
Cast to AFSDGameMode
    │
    ▼
Bind Event to On Player Logged In
    │
    └─► Custom Event: OnNewPlayer
        Input: NewPlayer (AFSDPlayerController)
            │
            ▼
        [Add player to tracking list]
```

### Example 2: Bind to Enemy Death

If binding to each enemy:

```
On Enemy Spawned (or Get All Enemies):
    │
    ▼
For Each Enemy:
    │
    ▼
Get Health Component
    │
    ▼
Bind Event to On Death
    │
    └─► Custom Event: OnEnemyDied
        Inputs: 
          - DeadEnemy (reference to enemy)
          - Killer (instigator controller)
            │
            ▼
        Cast Killer to AFSDPlayerController
            │
            ▼
        [Record kill for player]
```

### Example 3: Mission Complete

```
Event BeginPlay
    │
    ▼
Get Game Mode (or Game State)
    │
    ▼
Cast to AFSDGameMode
    │
    ▼
Bind Event to On Mission Complete
    │
    └─► Custom Event: OnMissionEnded
        Input: Success (Boolean)
            │
            ▼
        Delay 2 seconds
            │
            ▼
        Display Stats
```

---

## Alternative Approaches if Events Unavailable

### Tick-Based Polling

If no delegates available:

```
Set Timer by Event (repeating)
    │
    └─► Custom Event: CheckGameState
            │
            ▼
        Get All Enemies
            │
            ▼
        Compare to last known state
            │
            ▼
        Detect changes (deaths, damage)
            │
            ▼
        Update stats accordingly
```

### Interface Binding

Create custom interface for damage tracking:

1. Create Blueprint Interface with functions
2. Implement in custom weapon/damage actor
3. Call interface when dealing damage

### Actor Component Approach

Create a component that:
1. Attaches to player characters
2. Intercepts damage dealing
3. Reports to central tracker

---

## Events by Class

### AFSDGameMode

| Event | Parameters | When Fired |
|-------|------------|------------|
| `OnPlayerLoggedIn` | PlayerController | Player joins |
| `OnPlayerLoggedOut` | PlayerController | Player leaves |
| `OnMissionStarted` | None | Mission begins |
| `OnMissionComplete` | Success (bool) | Mission ends |

### AFSDPlayerController

| Event | Parameters | When Fired |
|-------|------------|------------|
| `OnPossess` | Pawn | Controller takes pawn |
| `OnUnPossess` | Pawn | Controller releases pawn |
| `OnDeath` | None | Player dies |

### APlayerCharacter

| Event | Parameters | When Fired |
|-------|------------|------------|
| `OnKilledEnemy` | Enemy | Player kills enemy |
| `OnDamagedEnemy` | Enemy, Damage | Player damages enemy |
| `OnWeaponFired` | Weapon | Weapon is used |

### UHealthComponent

| Event | Parameters | When Fired |
|-------|------------|------------|
| `OnDeath` | None | Health reaches 0 |
| `OnDamageReceived` | Amount, Instigator | Damage taken |
| `OnHealthChanged` | NewHealth, OldHealth | Any health change |

---

## Debugging Events

### Print on Event Fire

Add Print String nodes to confirm events are firing:

```
Custom Event: OnEnemyDied
    │
    ▼
Print String: "Enemy killed!"
    │
    ▼
[Rest of logic]
```

### Log Event Parameters

```
Custom Event: OnDamageDealt
    Inputs: Target, Amount, Instigator
        │
        ▼
    Append Strings:
        "Damage: " + Amount + " to " + Target.Name
        │
        ▼
    Print String (above result)
```

### Check Event Binding

After binding, the "Bind Event" node's output will be valid. Store and check:

```
Bind Event to OnDeath
    │
    └─► Return Value ──► Is Valid? ──► Branch
                                          │
                                    ┌─────┴─────┐
                                  True        False
                                    │           │
                                    ▼           ▼
                              "Bound!"    "Failed to bind"
```

---

## Related Files

- [fsd_classes.md](fsd_classes.md) - Class definitions
- [../blueprints/ST_Main.md](../blueprints/ST_Main.md) - Implementation using these events
- [Header Dumps](https://github.com/DRG-Modding/Header-Dumps) - Full event definitions
