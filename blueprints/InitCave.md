# InitCave Blueprint

The mission entry point - spawns the main stats tracker when a mission begins.

## Blueprint Details

| Property | Value |
|----------|-------|
| **Name** | InitCave |
| **Parent Class** | Actor |
| **Location** | Content/ModStatsTracker/InitCave |

## Class Defaults

No special class defaults needed. This is a simple spawner.

## Components

None required.

## Variables

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `TrackerRef` | ST_Main (Object Reference) | None | Optional: Store reference to spawned tracker |

## Event Graph

### Event BeginPlay

This is the main (and only) logic for InitCave.

```
┌─────────────────┐
│ Event BeginPlay │
└────────┬────────┘
         │
         ▼
    ┌─────────┐
    │Is Server│ (Pure function - returns Boolean)
    └────┬────┘
         │
         ▼
   ┌───────────┐
   │  Branch   │
   └─────┬─────┘
         │
    ┌────┴────┐
    │         │
  True      False
    │         │
    ▼         ▼
┌────────────────────────┐    ┌──────────┐
│ Spawn Actor from Class │    │ (nothing)│
│                        │    └──────────┘
│ Class: ST_Main         │
│                        │
│ Spawn Transform:       │
│   Location: (0, 0, 0)  │
│   Rotation: (0, 0, 0)  │
│   Scale: (1, 1, 1)     │
│                        │
│ Collision Handling:    │
│   Always Spawn, Ignore │
│   Collisions           │
└───────────┬────────────┘
            │
            ▼
      ┌───────────┐
      │Return Value│ ──► Set TrackerRef (optional)
      └───────────┘
```

## Node-by-Node Instructions

### 1. Event BeginPlay
- Already exists in new Actor blueprints
- Right-click → Add Event → Event BeginPlay (if missing)

### 2. Is Server
- Drag from BeginPlay execution pin
- Search for "Is Server"
- This is a **pure function** (no execution pins)
- Returns True if this instance is the server/host

### 3. Branch
- Drag from BeginPlay execution pin  
- Search for "Branch"
- Connect Is Server output to Condition input

### 4. Spawn Actor from Class
- Drag from True execution pin of Branch
- Search for "Spawn Actor from Class"
- Configure:
  - **Class**: Click dropdown → Select ST_Main
  - **Spawn Transform**: Right-click the pin → Split Struct Pin
    - Location: 0, 0, 0
    - Rotation: 0, 0, 0
    - Scale: 1, 1, 1
  - **Collision Handling Override**: Always Spawn, Ignore Collisions

### 5. (Optional) Store Reference
- Drag from Return Value of Spawn Actor
- Search for "Set TrackerRef"
- Useful if you need to reference the tracker later

## Why Server-Only Spawn?

The tracker only needs to exist once per mission, controlled by the host:

1. **No Duplicates**: Without the server check, each client would spawn their own tracker
2. **Authority**: Only the server can reliably track all players' actions
3. **Replication**: The spawned ST_Main actor replicates to clients automatically

## Complete Blueprint Screenshot (Pseudocode)

```
[Event BeginPlay] ─────► [Is Server] ─► [Branch]
                                            │
                                      ┌─────┴─────┐
                                    True        False
                                      │           │
                                      ▼           ▼
                          [Spawn Actor from Class]  (end)
                                      │
                                      ▼
                            [Set TrackerRef] (optional)
```

## Testing

1. Add a **Print String** node after spawn:
   - Text: "Stats Tracker Spawned!"
   - Duration: 5.0
2. Package and test
3. Should only see message once (not per player)

## Common Mistakes

### Forgetting Server Check
```
❌ Wrong:
Event BeginPlay ──► Spawn Actor ──► ...

✓ Correct:
Event BeginPlay ──► Is Server ──► Branch ──► (True) ──► Spawn Actor
```

### Wrong Collision Handling
```
❌ Wrong: "Do Not Spawn" or "Adjust If Possible"
✓ Correct: "Always Spawn, Ignore Collisions"
```

The tracker has no physical presence, so collision doesn't matter.

### Spawning Wrong Class
Make sure you select **ST_Main** (your custom tracker), not a built-in class.

## Related Files

- [ST_Main.md](ST_Main.md) - The actor being spawned
- [InitSpacerig.md](InitSpacerig.md) - Spacerig entry point
