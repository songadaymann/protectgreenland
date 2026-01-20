# Protect Greenland! - Tower Defense Game

## Concept
A Greenland-themed tower defense game where you defend downtown Nuuk against waves of Trump heads trying to reach the Inuksuk at the center. Place Tupilaqs (Inuit spirit figures) on roads to throw bones at the invaders. Collect falling snowflakes for Snow Power (SP) to build more defenses.

---

## Current Assets

### Map Layers
- `assets/nuuk_map.png` - Satellite imagery base map (1920x1080)
- `assets/roads.png` - Purple road overlay (visual only)
- `assets/spawn-and-waypoints.png` - Waypoint system for pathfinding:
  - Red dots = spawn points
  - Green dot = center target (Inuksuk)
  - Purple dots = normal waypoints
  - Yellow dots = fork waypoints (random path choice)

### Intro/UI
- `assets/introvideo_optimized.mp4` - Intro zoom video (4.3MB)
- `assets/intromusic.mp3` - Intro screen music (2.3MB)
- `assets/tutorial-music.mp3` - Tutorial music (loops during tutorial)
- `assets/20ProtectGreenland.mp3` - Main gameplay music (loops during waves)
- `assets/Averia_Libre/` - Custom font for UI text

### Tupilaq (Tower)
- `assets/Tupilaks/Tupilak-humanoid/idle1-3.png` - Idle animation (3 frames)
- `assets/Tupilaks/Tupilak-humanoid/throw1-2.png` - Throwing animation (2 frames)
- `assets/Tupilaks/Tupilak-humanoid/bone.png` - Bone projectile
- Base sprite size: 427x278
- Current scale: 0.18 (~77x50 in-game)

### Center Target
- `assets/Inuksuk.png` - Snowy Inuksuk stone landmark
- Current size: 100x100
- Has health bar (uses "lives" variable, starts at 10)

### Enemies (Trump Heads)
- `assets/trump/regular1-5.png` - Normal state (5 frames for animation)
- `assets/trump/hit1-3.png` - Hit state (damage feedback)
- Current size: 40x40

### Snowflakes
- `assets/snowflakes/snowflakes-1-5.png` - 5 varieties
- Used for: falling SP collectibles

---

## Game Flow

### Start Screen
- Shows first frame of intro video as background
- "PROTECT GREENLAND" title with icy glow effect
- Pulsing "Click / Tap to Start" text
- Intro music loops in background

### Intro Sequence
- On click: intro video plays with sound
- At 7 seconds: fade to black
- Video audio continues while game starts

### Tutorial (before Wave 1)
1. "Click on the snowflake!" - Player clicks falling snowflake
2. "Snowflakes give you Snow Power!" - Click to continue
3. "Tap here to increase snowfall!" - Arrow points to More Snow item, wait for purchase
4. "Now tap on a road to place a defender!" - Wait for Tupilaq placement
5. "That's a Tupilaq! He's protecting Greenland." - Click to continue
6. "A Tupilaq costs 50 SP" - Arrow points to Tupilaq item, click to continue
7. "PROTECT GREENLAND!" - Click to start game (+60 SP bonus awarded)

### Gameplay
- 5 waves of enemies
- Defend the Inuksuk at center

### Music Flow
1. **Intro screen**: `intromusic.mp3` loops
2. **Intro video**: Video audio plays, intro music stops
3. **Tutorial**: After intro video ends, `tutorial-music.mp3` starts (seamless looping via Web Audio API, lower volume)
4. **Game start**: Tutorial music finishes current loop, then `20ProtectGreenland.mp3` starts and loops

### End Screens
- **Win:** "YOU HAVE PROTECTED NUUK, THE CAPITAL OF GREENLAND! PROTECT THE WORLD FROM THE UNITED STATES!"
- **Lose:** "NUUK, THE CAPITAL OF GREENLAND HAS FALLEN. PROTECT THE WORLD FROM THE UNITED STATES!"

---

## Wave System (Time-Based)

Each wave lasts **60 seconds** (3600 frames). The max number of enemies *on screen at once* ramps up over the wave duration.

| Wave | Max On Screen | Speed (range)    | HP  | Pathfinding |
|------|---------------|------------------|-----|-------------|
| 1    | 3             | 0.08 - 0.10      | 3   | Always shortest path |
| 2    | 9             | 0.12 - 0.14      | 3   | Always shortest path |
| 3    | 14            | 0.17 - 0.19      | 4   | Always shortest path |
| 4    | 17            | 0.21 - 0.23      | 5   | Always shortest path |
| 5    | 20            | 0.25 - 0.27      | 6   | Always shortest path |

**How it works:**
- Wave timer counts up to 60 seconds
- Current max enemies = ramps from 1 to wave max over the duration
- When an enemy dies, a new one spawns (up to current max)
- After 60 seconds, spawning stops
- Wave ends when all remaining enemies are cleared

**Speed formula:**
- Base speed: `0.08 + (wave - 1) * 0.0425`
- Random variance: `+0.0 to +0.02`

**Pathfinding:**
- Always takes the shortest path toward the center

---

## Game Settings

### Economy
| Setting | Value |
|---------|-------|
| Starting SP | 100 |
| Tutorial completion bonus | +60 SP |
| Tupilaq cost | 50 SP |
| More Snow cost | 20 SP |
| Snowflake collect | +10 SP |
| Kill reward | None |
| Wave completion bonus | None |

### Snow Rate
| Setting | Value |
|---------|-------|
| Starting rate | 0.006 (spawn chance per frame) |
| Increase per purchase | +0.001 |

### Tupilaq (Tower) Stats
| Setting | Value |
|---------|-------|
| HP | 3 |
| Range | 150 px |
| Fire rate | 300 frames (~5 sec) |
| Scale | 0.18 (~77x50 px) |

### Bone Projectile
| Setting | Value |
|---------|-------|
| Speed | 1.5 |
| Size | 40 px |
| Damage | 1 HP |

### Enemy Behavior
| Setting | Value |
|---------|-------|
| Attack rate | 60 frames (1 damage per attack) |
| Attack range | 40 px (stops and attacks Tupilaq) |
| Starting lives (Inuksuk HP) | 10 |

### Falling Snowflakes
| Setting | Value |
|---------|-------|
| Fall speed | 0.2 - 1.0 |
| Size range | 10 - 70 px |

---

## Pathfinding (Waypoint-Based)

Enemies follow a **waypoint graph** system:

1. Waypoints detected from `spawn-and-waypoints.png`:
   - Red = spawn points
   - Green = center target
   - Purple = normal waypoints
   - Yellow = fork waypoints

2. Auto-connection: waypoints within 40px are connected

3. Enemy navigation:
   - Start at a spawn point
   - Move directly between connected waypoints
   - Track previous waypoint to prevent backtracking
   - Always take the shortest path to center

4. Tupilaq placement: allowed within 50px of any waypoint

---

## Controls

- **Click shop item** - Select Tupilaq for placement
- **Click near waypoint** - Place selected Tupilaq (costs 50 SP)
- **Click "More Snow"** - Increase snowfall rate (costs 20 SP)
- **Click falling snowflake** - Collect +10 SP

---

## Shop UI

- Horizontal panel in top-left corner
- Two square items (100x100 px) side by side:
  - Left: Tupilaq
  - Right: More Snow
- **Fill meter effect**: Items fill with blue from bottom as you accumulate SP
- Grey overlay + dimmed sprite when can't afford
- Icy color scheme (dark blue, light blue accents)
- Snow Power and Wave display below shop (Averia Libre font with icy glow)

---

## Display

- Game resolution: 1920 x 1080
- Responsive scaling with black letterbox bars
- Roads overlay at 70% opacity
- Health bars on Tupilaqs (green) and Inuksuk
- Custom font: Averia Libre

---

## TODO / Ideas

- [x] Start screen
- [x] Intro music
- [x] Intro video
- [x] Waypoint-based pathfinding
- [x] Tutorial system
- [ ] Sound effects (gameplay)
- [ ] Mobile touch controls optimization
- [ ] Different Trump variants (faster, more HP, etc.)
- [ ] Boss waves
- [ ] More Tupilaq types
- [ ] Upgradeable Tupilaqs

---

## File Structure

```
greenland/
├── index.html              # Main game file
├── PROGRESS.md             # This file
├── assets/
│   ├── nuuk_map.png
│   ├── roads.png
│   ├── spawn-and-waypoints.png
│   ├── Inuksuk.png
│   ├── introvideo_optimized.mp4
│   ├── intromusic.mp3
│   ├── tutorial-music.mp3
│   ├── 20ProtectGreenland.mp3
│   ├── Averia_Libre/
│   │   ├── AveriaLibre-Regular.ttf
│   │   └── AveriaLibre-Bold.ttf
│   ├── Tupilaks/
│   │   └── Tupilak-humanoid/
│   │       ├── idle1.png, idle2.png, idle3.png
│   │       ├── throw1.png, throw2.png
│   │       └── bone.png
│   ├── trump/
│   │   ├── regular1-5.png
│   │   └── hit1-3.png
│   └── snowflakes/
│       └── snowflakes-1-5.png
```
