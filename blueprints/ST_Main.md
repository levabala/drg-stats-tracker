# ST_Main Blueprint

The main stats tracker actor - handles all event subscriptions, data tracking, and output.

## Blueprint Details

| Property | Value |
|----------|-------|
| **Name** | ST_Main |
| **Parent Class** | Actor |
| **Location** | Content/ModStatsTracker/ST_Main |

## Class Defaults

**Important**: Configure these in Class Defaults (click "Class Defaults" button):

| Setting | Value | Purpose |
|---------|-------|---------|
| Replicates | True | Enable network replication |
| Always Relevant | True | Always replicate to all clients |
| Net Load on Client | True | Load on clients |

## Components

None required (logic-only actor).

## Variables

### Core Variables

| Name | Type | Default | Replicated | Description |
|------|------|---------|------------|-------------|
| `PlayerStats` | Array of ST_PlayerData | [] | No | Stats for all players |
| `IsTrackingActive` | Boolean | True | No | Whether to track events |

### Reference Variables

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `GameStateRef` | AFSDGameState (Object Ref) | None | Game state reference |
| `GameModeRef` | AFSDGameMode (Object Ref) | None | Game mode reference |

### Timer Handle

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `ChatCheckTimer` | Timer Handle | None | For polling chat messages |

---

## Functions

### Function: InitializePlayerStats

Initializes stats for all current players.

```
┌─────────────────────────┐
│ InitializePlayerStats   │
│ (No inputs/outputs)     │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────────────┐
│ Get All Actors Of Class         │
│ Class: AFSDPlayerController     │
└───────────────┬─────────────────┘
                │
                ▼ (Array of Actors)
         ┌──────────────┐
         │ For Each Loop│
         └──────┬───────┘
                │
         Loop Body
                │
                ▼
    ┌────────────────────────────┐
    │ Cast to AFSDPlayerController│
    └──────────────┬─────────────┘
                   │
                   ▼
    ┌────────────────────────────┐
    │ Get Player State            │
    │ (from controller)           │
    └──────────────┬─────────────┘
                   │
                   ▼
    ┌────────────────────────────┐
    │ Get Player Name             │
    │ (from player state)         │
    └──────────────┬─────────────┘
                   │
                   ▼
    ┌────────────────────────────┐
    │ Make ST_PlayerData          │
    │ - PlayerName = (from above) │
    │ - PlayerController = Cast   │
    │ - TotalKills = 0            │
    │ - TotalDamage = 0.0         │
    │ - KillsByCategory = {}      │
    │ - DamageByCategory = {}     │
    └──────────────┬─────────────┘
                   │
                   ▼
    ┌────────────────────────────┐
    │ Add to PlayerStats array    │
    └────────────────────────────┘
```

---

### Function: SubscribeToEvents

Subscribes to game events for tracking.

```
┌─────────────────────────┐
│ SubscribeToEvents       │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Bind Event to On Player Logged In       │
│ (from GameModeRef)                      │
│                                         │
│ Event → Create Event → OnPlayerJoined   │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│ Set Timer by Event                      │
│                                         │
│ Time: 0.5                               │
│ Looping: True                           │
│ Event → Create Event → CheckForChatCmd  │
│                                         │
│ Return → Set ChatCheckTimer             │
└───────────────┬─────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────┐
│ [Subscribe to mission complete]         │
│ (Implementation depends on available    │
│  events in FSD-Template)                │
└─────────────────────────────────────────┘
```

**Note**: The exact event bindings depend on what's available in the FSD-Template. You may need to explore the header dumps to find the right events.

---

### Function: CategorizeEnemy

Determines the category of an enemy based on its class name.

```
┌─────────────────────────┐
│ CategorizeEnemy         │
│                         │
│ Input: EnemyActor       │
│   (Actor Reference)     │
│                         │
│ Output: Category        │
│   (ST_EnemyCategory)    │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Get Class               │
│ (from EnemyActor)       │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Get Display Name        │
│ (from Class)            │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ To Lower                │
│ (convert to lowercase)  │
└───────────┬─────────────┘
            │
            ▼
┌────────────────────────────────────────────────────┐
│ Contains "dreadnought" ─► Branch ─► True: Return   │
│                                      Dreadnought   │
│                              False ▼               │
│ Contains "glyphid" ─────► Branch ─► True: Return   │
│                                      Glyphid       │
│                              False ▼               │
│ Contains "mactera" ─────► Branch ─► True: Return   │
│                                      Mactera       │
│                              False ▼               │
│ Contains "naedocyte" ───► Branch ─► True: Return   │
│                                      Naedocyte     │
│                              False ▼               │
│ Contains "qronar" OR                               │
│ Contains "shellback" ───► Branch ─► True: Return   │
│                                      Qronar        │
│                              False ▼               │
│ Contains "patrolbot" OR                            │
│ Contains "shredder" OR                             │
│ Contains "turret" OR                               │
│ Contains "prospector" OR                           │
│ Contains "nemesis" OR                              │
│ Contains "caretaker" ───► Branch ─► True: Return   │
│                                      Robot         │
│                              False ▼               │
│                                      Return Other  │
└────────────────────────────────────────────────────┘
```

---

### Function: AddKill

Records a kill for a player.

```
┌─────────────────────────┐
│ AddKill                 │
│                         │
│ Inputs:                 │
│ - PlayerController      │
│ - Category              │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Find in PlayerStats array               │
│ (where PlayerController matches)        │
└───────────┬─────────────────────────────┘
            │
            ▼
    ┌───────────────┐
    │ Branch        │
    │ (Was found?)  │
    └───────┬───────┘
            │
      ┌─────┴─────┐
    True        False
      │           │
      ▼           ▼
┌──────────────┐  (do nothing)
│Break struct  │
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────┐
│ TotalKills = TotalKills + 1      │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│ Find Category in KillsByCategory │
│ If not found, default to 0       │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│ Set KillsByCategory[Category]    │
│ = CurrentValue + 1               │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│ Make new ST_PlayerData with      │
│ updated values                   │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│ Set PlayerStats[Index] =         │
│ Updated struct                   │
└──────────────────────────────────┘
```

---

### Function: AddDamage

Records damage dealt by a player.

```
┌─────────────────────────┐
│ AddDamage               │
│                         │
│ Inputs:                 │
│ - PlayerController      │
│ - Category              │
│ - DamageAmount (Float)  │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Find in PlayerStats array               │
│ (where PlayerController matches)        │
└───────────┬─────────────────────────────┘
            │
            ▼
    ┌───────────────┐
    │ Branch        │
    │ (Was found?)  │
    └───────┬───────┘
            │
      ┌─────┴─────┐
    True        False
      │           │
      ▼           ▼
┌──────────────┐  (do nothing)
│Break struct  │
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────┐
│ TotalDamage = TotalDamage +      │
│               DamageAmount       │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│ Find Category in DamageByCategory│
│ If not found, default to 0.0     │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│ Set DamageByCategory[Category]   │
│ = CurrentValue + DamageAmount    │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│ Update PlayerStats array         │
└──────────────────────────────────┘
```

---

### Function: DisplayStats

Formats and broadcasts stats to all players.

```
┌─────────────────────────┐
│ DisplayStats            │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Create String: "=== MISSION STATS ===" │
└───────────┬─────────────────────────────┘
            │
            ▼
     ┌──────────────┐
     │ For Each Loop│ (PlayerStats array)
     └──────┬───────┘
            │
     Loop Body
            │
            ▼
┌─────────────────────────────────────────┐
│ Format Line 1:                          │
│ "[PlayerName]: X kills | Y dmg"         │
│                                         │
│ Append to output string                 │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Format Line 2:                          │
│ "  Cat1: N | Cat2: M | ..."             │
│                                         │
│ For each category in KillsByCategory:   │
│   If kills > 0:                         │
│     Append "CategoryName: Count"        │
│                                         │
│ Append to output string                 │
└───────────┬─────────────────────────────┘
            │
            ▼
     [Loop Complete]
            │
            ▼
┌─────────────────────────────────────────┐
│ Append "====================="          │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Broadcast Message                       │
│ (Use Server Say or similar function)    │
│                                         │
│ May need to send multiple messages      │
│ if output is too long                   │
└─────────────────────────────────────────┘
```

---

### Function: FormatNumber

Formats a number with commas for readability.

```
┌─────────────────────────┐
│ FormatNumber            │
│                         │
│ Input: Value (Float)    │
│ Output: String          │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Round to Integer (Floor or Round)       │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Convert to String                       │
│ (Integer → String)                      │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ [Optional: Add commas for thousands]    │
│                                         │
│ If Length > 3:                          │
│   Insert comma at appropriate position  │
│                                         │
│ This is complex in Blueprint - may      │
│ skip for simplicity                     │
└───────────┬─────────────────────────────┘
            │
            ▼
         [Return]
```

---

## Event Graph

### Event BeginPlay

```
┌─────────────────┐
│ Event BeginPlay │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│ Get Game State ──► Cast to AFSDGameState│
│              ──► Set GameStateRef       │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Get Game Mode ──► Cast to AFSDGameMode  │
│             ──► Set GameModeRef         │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Call: InitializePlayerStats             │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Call: SubscribeToEvents                 │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Print String: "Stats Tracker Active"   │
│ (Debug - remove for release)            │
└─────────────────────────────────────────┘
```

### Custom Event: OnPlayerJoined

```
┌─────────────────────────────────┐
│ Custom Event: OnPlayerJoined    │
│                                 │
│ Input: NewPlayer                │
│   (AFSDPlayerController)        │
└───────────┬─────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Create ST_PlayerData for new player     │
│ (same as in InitializePlayerStats)      │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Add to PlayerStats array                │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Print String: "Tracking: [PlayerName]"  │
│ (Debug)                                 │
└─────────────────────────────────────────┘
```

### Custom Event: CheckForChatCmd

Called by timer to check for /stats command.

```
┌─────────────────────────────────┐
│ Custom Event: CheckForChatCmd   │
└───────────┬─────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ [Get recent chat messages]              │
│                                         │
│ Implementation depends on available     │
│ chat system hooks in FSD-Template       │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ For each new message:                   │
│   │                                     │
│   ▼                                     │
│ Does message start with "/stats"?       │
│   │                                     │
│   ├─► True ──► Call DisplayStats        │
│   │                                     │
│   └─► False ──► (continue)              │
└─────────────────────────────────────────┘
```

### Custom Event: OnMissionComplete

```
┌─────────────────────────────────┐
│ Custom Event: OnMissionComplete │
└───────────┬─────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Set IsTrackingActive = False            │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Clear Timer: ChatCheckTimer             │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Delay: 2.0 seconds                      │
│ (Let mission end sequence play)         │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Call: DisplayStats                      │
└─────────────────────────────────────────┘
```

---

## Hooking into Kill/Damage Events

This is the most complex part and depends on what's available in FSD-Template.

### Option A: Hook Enemy Death Events

If enemies have OnDeath delegates:

```
For each enemy spawned:
    │
    ▼
Bind to Enemy.OnDeath
    │
    ▼
On Death:
    ├─► Get Damage Instigator (killer)
    ├─► Get Instigator Controller
    ├─► Cast to AFSDPlayerController
    ├─► Get Enemy Class
    ├─► Call CategorizeEnemy
    └─► Call AddKill(Controller, Category)
```

### Option B: Hook Damage Component

If there's a central damage system:

```
Bind to UDamageComponent.OnDamageDealt (or similar)
    │
    ▼
On Damage:
    ├─► Get Damage Causer
    ├─► Get Damage Target
    ├─► Get Damage Amount
    ├─► If Target is Enemy:
    │       ├─► Get Controller from Causer
    │       ├─► Call CategorizeEnemy(Target)
    │       ├─► Call AddDamage(Controller, Category, Amount)
    │       │
    │       └─► If Target.Health <= 0:
    │               └─► Call AddKill(Controller, Category)
    └─► (continue)
```

### Option C: Tick-Based Enemy Health Monitoring

Less elegant but may work if events aren't available:

```
Event Tick (or slower timer):
    │
    ▼
Get All Enemies
    │
    ▼
For Each Enemy:
    │
    ├─► Check if health changed since last tick
    │       └─► If decreased: Record damage to last attacker
    │
    └─► Check if newly dead
            └─► If yes: Record kill to last attacker
```

---

## Notes

### Performance Considerations

- Use timers instead of Tick where possible
- Minimize per-frame operations
- Clear subscriptions on mission end

### Debugging Tips

1. Add Print String nodes liberally during development
2. Log enemy class names to verify categorization
3. Test with multiple players to verify tracking

### Known Challenges

1. **Finding the right events**: FSD-Template may not expose all needed delegates
2. **Chat system access**: May need creative solutions for /stats detection
3. **Damage attribution**: Ensuring kills are attributed to correct player

---

## Related Files

- [ST_PlayerData.md](ST_PlayerData.md) - Player data structure
- [ST_ChatHandler.md](ST_ChatHandler.md) - Chat command handling details
- [InitCave.md](InitCave.md) - Entry point that spawns this actor
- [../reference/fsd_classes.md](../reference/fsd_classes.md) - FSD class reference
- [../reference/events.md](../reference/events.md) - Available game events
