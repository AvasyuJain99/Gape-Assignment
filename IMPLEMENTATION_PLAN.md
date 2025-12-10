# Multiplayer Arena Shooter - Implementation Plan

## 📋 Overview
This document outlines the complete plan of action and game flow for building a lightweight multiplayer arena shooter prototype using Unity, Mirror networking, Addressables, and Asset Bundles.

---

## 🎯 Architecture Decisions

### Networking Framework
- **Choice: Mirror Networking** (already installed)
- **Reason**: Free, open-source, well-documented, good for learning multiplayer fundamentals
- **Transport**: Use KCP or Telepathy (included with Mirror)

### Project Structure
```
Assets/
├── Scripts/
│   ├── Networking/
│   │   ├── NetworkManager.cs (custom manager)
│   │   ├── PlayerNetworkController.cs
│   │   └── NetworkGameManager.cs
│   ├── Gameplay/
│   │   ├── PlayerController.cs
│   │   ├── PlayerHealth.cs
│   │   ├── PlayerAbility.cs
│   │   ├── Projectile.cs
│   │   └── RespawnSystem.cs
│   ├── Addressables/
│   │   ├── AddressableLoader.cs
│   │   └── AddressableManager.cs
│   ├── AssetBundles/
│   │   ├── AssetBundleLoader.cs
│   │   └── AssetBundleManager.cs
│   ├── UI/
│   │   ├── MainMenuUI.cs
│   │   ├── GameplayUI.cs
│   │   ├── VirtualJoystick.cs
│   │   ├── AbilityButton.cs
│   │   └── ErrorPopupUI.cs
│   └── Utils/
│       ├── ObjectPool.cs
│       └── PlayerPrefsManager.cs
├── Prefabs/
│   ├── Player/
│   ├── Projectiles/
│   └── Props/
├── Scenes/
│   ├── MainMenu.unity
│   └── Arena.unity
└── AddressableAssets/
    └── (Addressable assets)
```

---

## 🎮 Game Flow

### Phase 1: Main Menu
1. **App Launch**
   - Load MainMenu scene
   - Generate/Retrieve PlayerPrefs user ID
   - Initialize Addressables system
   - Initialize Asset Bundle system

2. **Play Button Click**
   - Show loading indicator
   - Attempt to join existing room via Mirror
   - If no room exists → create new room
   - If join fails → show error popup with Retry button
   - On success → Load Arena scene

### Phase 2: Arena Initialization
1. **Scene Load**
   - NetworkManager spawns player prefab
   - Load Addressable assets (player model/projectile/prop)
   - Download Asset Bundle from CDN
   - Instantiate loaded assets in scene
   - Initialize UI (virtual joystick, ability button, health bar)

2. **Player Setup**
   - Assign local player controls
   - Disable Update-heavy scripts on remote players
   - Set player name from PlayerPrefs
   - Initialize health system
   - Initialize ability cooldown

### Phase 3: Gameplay Loop
1. **Movement**
   - Virtual joystick input → local player movement
   - Check movement threshold (e.g., 0.1 units)
   - If moved > threshold → sync position to network
   - Network sync frequency: every 100ms (not every frame)

2. **Combat**
   - Ability button pressed → trigger attack
   - Check cooldown (if applicable)
   - Spawn projectile from object pool
   - Projectile travels and detects hits
   - On hit → apply damage to target
   - Sync health updates to network

3. **Health & Respawn**
   - Health decreases on damage
   - Health bar updates in real-time
   - At 0 HP → disable player
   - Wait 2 seconds → respawn at spawn point
   - Reset health to max

### Phase 4: Error Handling
- **Addressables Load Failure** → Show popup with Retry
- **Asset Bundle Download Failure** → Show popup with Retry
- **Network Join Failure** → Show popup with Retry
- **Retry Button** → Re-attempt the failed operation

---

## 📝 Implementation Phases

### Phase 1: Core Setup & Networking (Priority: HIGH)
**Goal**: Get basic multiplayer connection working

1. **Setup NetworkManager**
   - Create custom NetworkManager extending Mirror's NetworkManager
   - Configure room/matchmaking (auto-join or create)
   - Set up player spawning
   - Generate PlayerPrefs user ID

2. **Player Network Controller**
   - Create NetworkBehaviour for player
   - Implement position syncing with threshold (0.1 units)
   - Implement health syncing
   - Network sync rate: 100ms intervals

3. **Basic Player Movement**
   - Simple character controller
   - Test with keyboard input first (mobile later)

**Deliverable**: Two players can connect and see each other move

---

### Phase 2: Gameplay Systems (Priority: HIGH)
**Goal**: Core combat and health mechanics

1. **Health System**
   - Health component with max health (e.g., 100)
   - Network synced health value
   - Health bar UI

2. **Ability System**
   - One ability: Projectile attack (easiest to implement)
   - Ability cooldown (e.g., 1 second)
   - Cooldown synced to network

3. **Projectile System**
   - Projectile prefab with collider
   - Damage on hit
   - Destroy after lifetime or hit

4. **Respawn System**
   - Spawn points in arena
   - 2-second delay before respawn
   - Reset health on respawn

**Deliverable**: Players can attack, take damage, and respawn

---

### Phase 3: Addressables Integration (Priority: MEDIUM)
**Goal**: Async asset loading

1. **Setup Addressables**
   - Install Addressables package (if not installed)
   - Create Addressable Groups
   - Mark one prefab as Addressable (e.g., player model or projectile)

2. **Addressable Loader**
   - Load asset asynchronously on scene start
   - Show loading indicator
   - Instantiate when loaded
   - Error handling with retry

**Deliverable**: Asset loads from Addressables before appearing in game

---

### Phase 4: Asset Bundle System (Priority: MEDIUM)
**Goal**: Runtime download and load from CDN

1. **Create Asset Bundle**
   - Select one model/prop
   - Build Asset Bundle
   - Upload to Google Drive/Dropbox/GitHub RAW

2. **Asset Bundle Loader**
   - Download using UnityWebRequest
   - Load from bytes
   - Instantiate in scene
   - Error handling with retry

**Deliverable**: Asset Bundle downloads and appears in game

---

### Phase 5: Mobile Controls (Priority: MEDIUM)
**Goal**: Touch input for mobile

1. **Virtual Joystick**
   - Create UI joystick component
   - Touch input handling
   - Output direction vector
   - Connect to player movement

2. **Ability Button**
   - On-screen button
   - Touch/click to trigger ability
   - Visual feedback (pressed state)

**Deliverable**: Game playable on mobile with touch controls

---

### Phase 6: Optimization (Priority: MEDIUM)
**Goal**: Performance improvements

1. **Disable Remote Player Updates**
   - Only local player processes input
   - Remote players only receive network updates
   - Disable Update() on remote player controllers

2. **Object Pooling**
   - Create ObjectPool class
   - Pool projectiles (e.g., 20 projectiles)
   - Reuse instead of instantiate/destroy

3. **Texture Optimization**
   - Select one texture
   - Change import setting to ETC2 (Android) or ASTC (iOS)
   - Reduce texture size if needed

4. **Network Sync Frequency**
   - Already implemented in Phase 1 (100ms intervals)
   - Verify it's working correctly

**Deliverable**: Optimized performance for mobile

---

### Phase 7: Error Handling & Polish (Priority: LOW)
**Goal**: Robust error handling

1. **Error Popup System**
   - Create error popup UI prefab
   - Show on Addressables failure
   - Show on Asset Bundle failure
   - Show on Network join failure
   - Retry button functionality

2. **UI Polish**
   - Player name display
   - Health bar/text
   - Basic styling (functional, not pretty)

**Deliverable**: All error cases handled gracefully

---

## 🔧 Technical Specifications

### Network Sync Threshold
- **Position Sync**: Only when player moves > 0.1 units
- **Sync Frequency**: Every 100ms (10 times per second)
- **Synced Variables**: Position, Health, Ability Cooldown

### Player Stats
- **Max Health**: 100
- **Respawn Delay**: 2 seconds
- **Ability Cooldown**: 1 second (if applicable)
- **Projectile Speed**: 10 units/second
- **Projectile Damage**: 25

### Optimization Settings
- **Object Pool Size**: 20 projectiles
- **Network Update Rate**: 10 Hz (100ms)
- **Texture Format**: ETC2 (Android) / ASTC (iOS)

---

## 🎯 Testing Checklist

### Networking
- [ ] Two players can connect to same room
- [ ] Player movement syncs correctly
- [ ] Health syncs across network
- [ ] Auto-join or create room works

### Gameplay
- [ ] Player can move
- [ ] Ability fires projectile
- [ ] Projectile deals damage
- [ ] Health decreases on damage
- [ ] Player respawns after 2 seconds at 0 HP

### Addressables
- [ ] Asset loads asynchronously
- [ ] Asset appears only after loading
- [ ] Error handling works

### Asset Bundles
- [ ] Asset Bundle downloads from CDN
- [ ] Asset Bundle loads from bytes
- [ ] Asset appears in scene
- [ ] Error handling works

### Mobile Controls
- [ ] Virtual joystick controls movement
- [ ] Ability button triggers attack
- [ ] Works on mobile device/emulator

### Optimization
- [ ] Remote players don't process input
- [ ] Projectiles use object pooling
- [ ] Texture uses ETC/ASTC format
- [ ] Network sync is throttled to 100ms

### Error Handling
- [ ] Addressables error shows popup
- [ ] Asset Bundle error shows popup
- [ ] Network error shows popup
- [ ] Retry button works

---

## 📦 Dependencies & Packages

### Required Packages
- ✅ Mirror Networking (already installed)
- ⚠️ Addressables (need to install via Package Manager)
- ✅ UnityWebRequest (built-in)
- ✅ Asset Bundle system (built-in)

### Installation Commands
```
Window → Package Manager → Add package by name → com.unity.addressables
```

---

## 🚀 Quick Start Implementation Order

1. **Day 1**: Phase 1 (Networking) + Phase 2 (Gameplay)
2. **Day 2**: Phase 3 (Addressables) + Phase 4 (Asset Bundles)
3. **Day 3**: Phase 5 (Mobile Controls) + Phase 6 (Optimization)
4. **Day 4**: Phase 7 (Error Handling) + Testing & Bug Fixes

---

## 📝 Notes for Implementation

### Mirror Networking Tips
- Use `[SyncVar]` for simple variables (health, cooldown)
- Use `[Command]` for client-to-server calls (ability trigger)
- Use `[ClientRpc]` for server-to-client calls (damage, respawn)
- Use `isLocalPlayer` to differentiate local vs remote players

### Addressables Tips
- Use `Addressables.LoadAssetAsync<GameObject>()`
- Wait for completion before instantiating
- Always check for errors

### Asset Bundle Tips
- Use `UnityWebRequest.Get()` for download
- Use `AssetBundle.LoadFromMemory()` for loading
- Always dispose of AssetBundle after use

### Mobile Optimization Tips
- Test on actual device, not just editor
- Use Profiler to check performance
- Monitor network bandwidth usage

---

## 🎬 Demo Video Checklist

Make sure to show in demo video:
1. ✅ Play button → auto-join/create room
2. ✅ Two players in arena
3. ✅ Movement with virtual joystick
4. ✅ Ability attack (projectile)
5. ✅ Health bar updates
6. ✅ Respawn after death
7. ✅ Addressable asset loading (show loading state)
8. ✅ Asset Bundle downloading and appearing
9. ✅ Error popup with retry (simulate failure)
10. ✅ Mobile controls working

---

## ❓ Common Questions & Solutions

**Q: How to test multiplayer locally?**
A: Build two instances, run one as host, other as client. Or use Mirror's LAN discovery.

**Q: What if Addressables takes too long?**
A: Show loading indicator, allow gameplay to continue with placeholder if needed.

**Q: Asset Bundle URL not working?**
A: Use GitHub RAW URL format: `https://raw.githubusercontent.com/user/repo/branch/file`

**Q: Network sync too laggy?**
A: Reduce sync frequency or increase movement threshold.

---

## 📚 Resources

- Mirror Networking Docs: https://mirror-networking.com/docs/
- Unity Addressables: https://docs.unity3d.com/Packages/com.unity.addressables@latest
- Unity Asset Bundles: https://docs.unity3d.com/Manual/AssetBundlesIntro.html

---

**Last Updated**: [Current Date]
**Status**: Planning Phase
