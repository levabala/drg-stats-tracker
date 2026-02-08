# InitSpacerig Blueprint

The spacerig entry point - minimal for now, can be expanded for persistent features.

## Blueprint Details

| Property | Value |
|----------|-------|
| **Name** | InitSpacerig |
| **Parent Class** | Actor |
| **Location** | Content/ModStatsTracker/InitSpacerig |

## Class Defaults

No special settings needed.

## Components

None required.

## Variables

None required for basic implementation.

## Event Graph

### Basic Implementation (Minimal)

For now, we keep the spacerig entry point minimal since stats tracking only matters during missions.

```
┌─────────────────┐
│ Event BeginPlay │
└────────┬────────┘
         │
         ▼
   ┌─────────────┐
   │Print String │  (Debug only - remove for release)
   │             │
   │Text: "Stats │
   │Tracker Ready"│
   │             │
   │Duration: 3.0│
   └─────────────┘
```

## Future Expansion Ideas

The spacerig blueprint could be expanded to:

### 1. Show Last Mission Stats

```
Event BeginPlay
    │
    ▼
Is Server ──► Branch
                │
              True
                │
                ▼
    Load Last Mission Stats from Save
                │
                ▼
    If stats exist:
        │
        ▼
    Display in chat or create UI widget
```

### 2. Persistent Stats Terminal

Create an interactive terminal in the spacerig:

```
Event BeginPlay
    │
    ▼
Spawn Stats Terminal Actor
    │
    ▼
On Interact:
    │
    ▼
Show Stats UI Widget
```

### 3. Career Stats Tracking

```
Event BeginPlay
    │
    ▼
Load Career Stats from Save
    │
    ▼
Update UI or send to server
```

## Why Minimal?

1. **Mission Focus**: Stats tracking is mission-specific
2. **Performance**: No reason to run complex logic in spacerig
3. **Simplicity**: Keep initial implementation focused
4. **Extensibility**: Easy to add features later

## Node-by-Node Instructions (Minimal Version)

### 1. Event BeginPlay
- Already exists in new Actor blueprints

### 2. Print String (Debug Only)
- Drag from BeginPlay execution pin
- Search for "Print String"
- Set In String: "Stats Tracker Ready"
- Set Duration: 3.0
- **Remove this for release builds**

## Testing

1. Load into spacerig
2. Check for debug message (if using Print String)
3. Verify no errors in log

## Complete Blueprint

```
[Event BeginPlay] ──► [Print String: "Stats Tracker Ready"]
```

That's it for the basic version!

## Related Files

- [InitCave.md](InitCave.md) - Mission entry point (main logic)
- [ST_Main.md](ST_Main.md) - Main tracker actor
