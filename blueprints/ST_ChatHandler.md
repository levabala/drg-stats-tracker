# ST_ChatHandler - Chat Command Handling

This document covers different approaches to detecting and handling the `/stats` chat command.

## Overview

DRG's chat system may or may not expose convenient events for Blueprint modding. This document covers multiple approaches from most elegant to most hacky.

---

## Approach 1: Direct Chat Event Binding (Ideal)

If FSD-Template exposes chat message events, this is the cleanest approach.

### Prerequisites

Check in FSD-Template for:
- `AFSDGameState` or `AFSDGameMode` chat-related delegates
- `UChatComponent` or similar
- Events like `OnChatMessageReceived`, `OnPlayerChat`, etc.

### Implementation

```
In ST_Main BeginPlay or SubscribeToEvents:
      │
      ▼
┌─────────────────────────────────────────┐
│ Get Chat Manager/Component              │
│ (from GameState or GameMode)            │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Bind Event to OnChatMessage             │
│                                         │
│ Event ──► Create Event ──► HandleChat   │
└─────────────────────────────────────────┘


Custom Event: HandleChat
Inputs: 
  - Sender (PlayerController or PlayerState)
  - Message (String)
      │
      ▼
┌─────────────────────────────────────────┐
│ Starts With: Message, "/stats"          │
└───────────┬─────────────────────────────┘
            │
            ▼
    ┌───────────────┐
    │    Branch     │
    └───────┬───────┘
            │
      ┌─────┴─────┐
    True        False
      │           │
      ▼           ▼
┌─────────────┐  (return)
│DisplayStats │
└─────────────┘
```

### Blueprint Nodes

- **Bind Event to [Delegate Name]**: Creates event binding
- **Starts With**: String comparison (case sensitive)
- **Custom Event**: Receives the bound event

---

## Approach 2: Timer-Based Chat Polling

If no direct events available, poll for chat messages periodically.

### Implementation

```
In SubscribeToEvents:
      │
      ▼
┌─────────────────────────────────────────┐
│ Set Timer by Event                      │
│                                         │
│ Time: 0.25 (4 checks per second)        │
│ Looping: True                           │
│ Event ──► CheckChatMessages             │
│                                         │
│ Return Value ──► Save as ChatTimer      │
└─────────────────────────────────────────┘


Custom Event: CheckChatMessages
      │
      ▼
┌─────────────────────────────────────────┐
│ Get Chat History/Recent Messages        │
│ (Method depends on what's available)    │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Compare to LastCheckedMessages          │
│ Find any new messages                   │
└───────────┬─────────────────────────────┘
            │
            ▼
      ┌──────────────┐
      │ For Each New │
      │   Message    │
      └──────┬───────┘
             │
      Loop Body
             │
             ▼
┌─────────────────────────────────────────┐
│ Starts With: Message, "/stats"          │
└───────────┬─────────────────────────────┘
            │
      ┌─────┴─────┐
    True        False
      │           │
      ▼           ▼
┌─────────────┐  (continue loop)
│DisplayStats │
└─────────────┘
             │
      [Loop Complete]
             │
             ▼
┌─────────────────────────────────────────┐
│ Update LastCheckedMessages              │
└─────────────────────────────────────────┘
```

### Additional Variables Needed

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `ChatTimer` | Timer Handle | None | Reference to cancel timer |
| `LastMessageCount` | Integer | 0 | Track last seen message count |
| `LastCheckedMessages` | Array of String | [] | Previously seen messages |

### Cleanup on Mission End

```
Custom Event: OnMissionComplete
      │
      ▼
┌─────────────────────────────────────────┐
│ Clear Timer by Handle: ChatTimer        │
└─────────────────────────────────────────┘
```

---

## Approach 3: Key Binding Instead of Chat

If chat access is too difficult, use a dedicated key instead.

### Implementation

```
In ST_Main BeginPlay:
      │
      ▼
┌─────────────────────────────────────────┐
│ Get Player Controller (Index 0)         │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Enable Input (this actor)               │
│ Player Controller ◄── Controller ref    │
└─────────────────────────────────────────┘


Input Event: [Key] (e.g., F8 or NumPad5)
      │
      ▼
┌─────────────────────────────────────────┐
│ Is Server (or Is Locally Controlled)    │
└───────────┬─────────────────────────────┘
            │
      ┌─────┴─────┐
    True        False
      │           │
      ▼           ▼
┌─────────────┐  (return - prevent spam)
│DisplayStats │
└─────────────┘
```

### Adding Key Input Event

1. In ST_Main Event Graph
2. Right-click → Input → Keyboard Events
3. Select a key (e.g., F8, Home, NumPad key)
4. This creates an input event node

### Considerations

**Pros**:
- 100% reliable, no chat parsing needed
- Works regardless of chat system access

**Cons**:
- Less intuitive than chat command
- May conflict with other mods/keybindings
- Need to document the keybind for users

---

## Approach 4: Hybrid - Key + Chat

Try chat detection, fall back to key binding.

```
// Try to bind to chat events
If chat event binding succeeds:
    Use chat-based detection
Else:
    Enable key binding as fallback
    Print message: "Press [Key] to show stats"
```

---

## Broadcasting the Response

Once `/stats` (or key) is detected, we need to show output to all players.

### Option A: Server Say (if available)

```
┌─────────────────────────────────────────┐
│ Get Game Mode                           │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Server Say / Broadcast Message          │
│                                         │
│ Message ◄── Formatted stats string      │
└─────────────────────────────────────────┘
```

### Option B: Say for Each Player

```
┌─────────────────────────────────────────┐
│ Get All Player Controllers              │
└───────────┬─────────────────────────────┘
            │
            ▼
      ┌──────────────┐
      │ For Each     │
      │ Controller   │
      └──────┬───────┘
             │
      Loop Body
             │
             ▼
┌─────────────────────────────────────────┐
│ Client Message / Say                    │
│                                         │
│ Target ◄── Current controller           │
│ Message ◄── Formatted stats string      │
└─────────────────────────────────────────┘
```

### Option C: Print to All (Screen Message)

Less elegant but works:

```
┌─────────────────────────────────────────┐
│ Get All Player Controllers              │
└───────────┬─────────────────────────────┘
            │
            ▼
      ┌──────────────┐
      │ For Each     │
      │ Controller   │
      └──────┬───────┘
             │
      Loop Body
             │
             ▼
┌─────────────────────────────────────────┐
│ Client Travel / Client Message          │
│ or                                      │
│ Add On Screen Debug Message             │
│ (with longer duration)                  │
└─────────────────────────────────────────┘
```

---

## Message Formatting

### Breaking Into Lines

Chat messages may have character limits. Split long messages:

```
Function: BroadcastStats
      │
      ▼
┌─────────────────────────────────────────┐
│ Build header: "=== MISSION STATS ==="   │
│ Broadcast header                        │
└───────────┬─────────────────────────────┘
            │
            ▼
      ┌──────────────┐
      │ For Each     │
      │ PlayerStats  │
      └──────┬───────┘
             │
      Loop Body
             │
             ▼
┌─────────────────────────────────────────┐
│ Format: "[Name]: X kills | Y dmg"       │
│ Broadcast line                          │
└───────────┬─────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│ Format: "  Cat1: N | Cat2: M"           │
│ Broadcast line                          │
└─────────────────────────────────────────┘
             │
      [Loop Complete]
             │
             ▼
┌─────────────────────────────────────────┐
│ Broadcast footer: "===================" │
└─────────────────────────────────────────┘
```

### Delay Between Messages

Prevent message spam/overflow:

```
For Each PlayerStats:
    │
    ├─► Broadcast Line 1
    │
    ├─► Delay 0.1 seconds
    │
    ├─► Broadcast Line 2
    │
    └─► Delay 0.1 seconds
```

---

## Investigating Available Chat Functions

### Where to Look in FSD-Template

1. **FSD.hpp** in header dumps - search for:
   - `Chat`
   - `Message`
   - `Say`
   - `Broadcast`

2. **Classes to investigate**:
   - `AFSDGameMode`
   - `AFSDGameState`
   - `AFSDPlayerController`
   - `UFSDChatManager` (if exists)

### Common UE4 Chat Functions

These may or may not be exposed in FSD:

```cpp
// In PlayerController
void ClientMessage(const FString& S, FName Type, float MsgLifeTime);
void Say(const FString& Msg);

// In GameMode
void Broadcast(AActor* Sender, const FString& Msg, FName Type);
void BroadcastLocalized(AActor* Sender, TSubclassOf<ULocalMessage> Message, ...);
```

### Debugging Chat Access

Add this to BeginPlay for investigation:

```
Print all functions/properties available on GameMode
Print all functions/properties available on PlayerController

// In Blueprint, you can drag off a reference and see available nodes
```

---

## Fallback: No Chat at All

If chat system is completely inaccessible:

1. Use key binding only
2. Display stats as screen overlay widget
3. Or: Create a floating 3D text in the game world

```
On Key Press:
    │
    ▼
┌─────────────────────────────────────────┐
│ Create Widget: StatsDisplayWidget       │
│ Add to Viewport                         │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Delay 5 seconds                         │
└───────────┬─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│ Remove Widget from Parent               │
└─────────────────────────────────────────┘
```

---

## Summary

| Approach | Reliability | Elegance | Difficulty |
|----------|-------------|----------|------------|
| Chat Event Binding | High (if available) | High | Low |
| Timer Polling | Medium | Medium | Medium |
| Key Binding | High | Medium | Low |
| Widget Display | High | Medium | Medium |

**Recommendation**: Try chat event binding first. If not available, use key binding with optional chat polling as enhancement.

---

## Related Files

- [ST_Main.md](ST_Main.md) - Main actor where this is implemented
- [../reference/fsd_classes.md](../reference/fsd_classes.md) - Check for chat-related classes
