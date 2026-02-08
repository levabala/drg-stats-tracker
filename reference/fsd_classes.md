# FSD Classes Reference

Key classes from Deep Rock Galactic that are useful for the Stats Tracker mod.

> **Note**: These are based on the FSD-Template and header dumps. Exact names and availability may vary by game version.

---

## Core Game Classes

### AFSDGameMode

The main game mode class - controls game flow and rules.

```cpp
class AFSDGameMode : public AGameModeBase
{
    // Delegates
    FOnPlayerLoggedIn OnPlayerLoggedIn;
    FOnPlayerLoggedOut OnPlayerLoggedOut;
    
    // Functions
    void Broadcast(AActor* Sender, const FString& Msg, FName Type);
    TArray<AFSDPlayerController*> GetAllPlayerControllers();
    
    // Mission state
    bool IsMissionComplete();
    void EndMission(bool Success);
}
```

**Useful for**:
- Detecting player join/leave
- Broadcasting messages
- Getting all players
- Mission completion detection

---

### AFSDGameState

The game state - replicated to all clients.

```cpp
class AFSDGameState : public AGameStateBase
{
    // Mission info
    FString MissionName;
    float MissionTime;
    
    // Players
    TArray<APlayerState*> PlayerArray;
    
    // Events
    FOnMissionStarted OnMissionStarted;
    FOnMissionEnded OnMissionEnded;
}
```

**Useful for**:
- Getting mission information
- Accessing all player states
- Mission start/end events

---

### AFSDPlayerController

The player controller - one per player.

```cpp
class AFSDPlayerController : public APlayerController
{
    // Functions
    void ClientMessage(const FString& S, FName Type, float MsgLifeTime);
    void Say(const FString& Msg);
    
    // References
    AFSDPlayerState* GetFSDPlayerState();
    APlayerCharacter* GetPlayerCharacter();
    
    // Delegates
    FOnPlayerDeath OnPlayerDeath;
}
```

**Useful for**:
- Sending messages to specific player
- Getting player state/character
- Player death events

---

### AFSDPlayerState

Player state - contains player data.

```cpp
class AFSDPlayerState : public APlayerState
{
    // Player info
    FString PlayerName;
    int32 PlayerId;
    
    // Stats
    int32 Kills;
    int32 Deaths;
    int32 Revives;
    float DamageDealt;
    
    // Character class
    EPlayerCharacterType CharacterType;
}
```

**Useful for**:
- Getting player name
- Potentially accessing built-in kill/damage stats
- Character class (Driller, Scout, etc.)

---

## Enemy Classes

### AEnemyBase / AFSDEnemy

Base class for all enemies.

```cpp
class AEnemyBase : public APawn
{
    // Health
    UHealthComponent* HealthComponent;
    
    // Delegates
    FOnEnemyDeath OnDeath;
    FOnDamageTaken OnDamageTaken;
    
    // Functions
    float GetHealth();
    float GetMaxHealth();
    bool IsDead();
    
    // Info
    FString EnemyName;
    EEnemyType EnemyType;
}
```

**Useful for**:
- Hooking death events
- Getting enemy type
- Health tracking

---

### UHealthComponent

Health management component.

```cpp
class UHealthComponent : public UActorComponent
{
    // Properties
    float Health;
    float MaxHealth;
    
    // Delegates
    FOnHealthChanged OnHealthChanged;
    FOnDeath OnDeath;
    FOnDamageReceived OnDamageReceived;
    
    // Functions
    void TakeDamage(float Damage, AActor* DamageCauser, AController* Instigator);
}
```

**Useful for**:
- Damage tracking
- Death detection
- Getting damage source

---

## Damage System

### UDamageComponent

Damage dealing component (on weapons/players).

```cpp
class UDamageComponent : public UActorComponent
{
    // Properties
    float Damage;
    EDamageType DamageType;
    
    // Delegates
    FOnDamageDealt OnDamageDealt;
    FOnKill OnKill;
    
    // Functions
    void DealDamage(AActor* Target, float Amount);
}
```

**Alternative**: Check if weapons have damage components or if there's a central damage manager.

---

### FDamageData

Damage event data structure.

```cpp
struct FDamageData
{
    float DamageAmount;
    AActor* DamageCauser;
    AController* InstigatorController;
    AActor* DamagedActor;
    EDamageType DamageType;
    FVector HitLocation;
}
```

---

## Chat System

### UChatManager (if exists)

```cpp
class UChatManager : public UObject
{
    // Functions
    void SendChatMessage(const FString& Message, AFSDPlayerController* Sender);
    void BroadcastMessage(const FString& Message);
    
    // Delegates
    FOnChatMessageReceived OnChatMessageReceived;
    
    // History
    TArray<FChatMessage> ChatHistory;
}
```

### FChatMessage

```cpp
struct FChatMessage
{
    FString Message;
    FString SenderName;
    AFSDPlayerController* Sender;
    float Timestamp;
}
```

---

## Player Character

### APlayerCharacter

The actual player pawn in the game.

```cpp
class APlayerCharacter : public ACharacter
{
    // Health
    UHealthComponent* HealthComponent;
    
    // Weapons
    TArray<AWeapon*> Weapons;
    AWeapon* CurrentWeapon;
    
    // Info
    EPlayerCharacterType CharacterType;
    FString CharacterName;
    
    // Delegates
    FOnWeaponFired OnWeaponFired;
    FOnKilledEnemy OnKilledEnemy;
}
```

**Note**: `OnKilledEnemy` delegate may be ideal for tracking kills if available.

---

## Enums

### EPlayerCharacterType

```cpp
enum class EPlayerCharacterType : uint8
{
    Driller,
    Engineer,
    Gunner,
    Scout
}
```

### EEnemyType

```cpp
enum class EEnemyType : uint8
{
    Grunt,
    Guard,
    Slasher,
    Praetorian,
    Oppressor,
    // ... many more
}
```

---

## Finding Classes in Header Dumps

### Search Tips

1. Open `FSD.hpp` in your code editor
2. Use Ctrl+F to search
3. Try these search terms:
   - `OnDeath`
   - `OnKill`
   - `OnDamage`
   - `OnMission`
   - `Chat`
   - `Message`
   - `Broadcast`

### Important Notes

- Not all classes may be exposed to Blueprint
- Some functions are C++ only
- Check if functions are marked `BlueprintCallable`
- Delegates may need special binding

---

## Using Classes in Blueprint

### Casting

To use a specific class:

1. Get a generic reference (e.g., Actor)
2. Use "Cast to AFSDPlayerController"
3. If cast succeeds, you can access class-specific functions

### Getting References

```
Get Game Mode → Cast to AFSDGameMode → Access functions
Get Game State → Cast to AFSDGameState → Access functions
Get Player Controller → Cast to AFSDPlayerController → Access functions
```

---

## Verifying Availability

Before implementing, verify in UE4:

1. Open FSD-Template project
2. Create a test blueprint
3. Try to add nodes for the classes/functions you need
4. If they appear in the node search, they're available

---

## Related Files

- [events.md](events.md) - Specific events to use
- [Header Dumps](https://github.com/DRG-Modding/Header-Dumps) - Full C++ class definitions
