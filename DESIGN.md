DESIGN_DOC
game_name: arc-read
display_name: Arc Read

---

# ARC READ — Complete Design Document
### Round 15 | Mode: Initial Design

---

## The One Thing

Every other shooter asks you to aim at the target. Arc Read asks you to aim at a place the target hasn't reached yet — a place that exists only in your head. You are not shooting at ships. You are solving a moving equation under pressure, in real time, with your city's life as the stakes.

---

## IDENTITY

**Name:** Arc Read  
**Tagline:** *Draw where they will be.*

**What is the player:** A missile defense commander operating a single launch platform on the roof of the last inhabited city on a magenta-dark world. You have no guns. You have interceptors — arcing bolts that travel a fixed path you draw in the moment of launch. You don't aim. You predict.

**World feel (2 sentences):** The sky above the city glows deep magenta — not with sunset, but with the ambient radiation of everything that's already been lost. The enemy formations move in cold geometric silence, their trajectories clean and indifferent, like arithmetic that wants you dead.

**Emotional experience:** *The slow dread of certainty — and the electric release when your arc cuts through it.*

**Reference games and DNA:**

- **Missile Command (1980):** Same core premise — city defense, draw-to-fire, limited shots matter. Arc Read diverges by demanding lead angles rather than click-to-explode; there is no area-of-effect. Miss is a miss.
- **R-Type (1987):** Enemy formations announce themselves before attacking. The entire skill loop is committing to a read before you have confirmation. Enemies telegraph; players who wait lose. Arc Read encodes this as mechanic: the earlier you draw your arc, the more interceptors you conserve.
- **Thumper (2016):** Instant feedback, visceral misread punishment, meditative flow state when mastered. Arc Read borrows the rhythm of punishment → recalibration → flow.
- **Inside (2016):** No UI chrome explaining rules. The world teaches you through consequence. Arc Read's tutorial is the first miss — and the first hit 10 seconds later.

---

## VISUAL SPEC

### Color Palette (exact hex, no deviation)

| Role | Hex | Usage |
|---|---|---|
| Background | `#7A3060` | Sky, canvas fill — deep magenta, NOT near-black |
| Primary | `#F0EBD8` | City silhouette, HUD text, interceptor body, enemy outlines |
| Secondary | `#4DFFD6` | Arc trails, detonation flash, missile exhaust, hit sparks |
| Accent | `#FF6B35` | City health pip fill, damage flash on city, critical warning pulse |

### Bloom

- **Bloom strength:** 1.4  
- **Bloom threshold:** 0.35  
- **Bloom radius:** 0.6  
- Applied to: arc trails (`#4DFFD6`), detonation flash, missile exhaust, city windows  
- Implementation: Three.js `UnrealBloomPass` via `EffectComposer`

### Camera

- **View:** Top-down with 25° tilt — not isometric; the city reads as a flat silhouette at the bottom edge with slight perspective recession
- **Three.js camera position:** `camera.position.set(0, 520, 280)` with `camera.lookAt(0, 0, 0)`
- **FOV:** 50°  
- **Camera is static** — it never moves during gameplay. The entire battlefield is visible at all times.
- **Viewport:** 1280×720 reference resolution; scale proportionally. The city occupies the bottom 12% of screen height.

### City Silhouette (5 words)

**Jagged. Warm. Low. Dense. Fragile.**

- Procedurally generated once at game init from a fixed seed (`SEED = 42` for reproducibility)
- 18–24 buildings of varying heights (range: 28px–90px at 1280×720), clustered in 3 distinct urban "blocks" with gaps between
- Windows: small 4×4px rectangles, `#F0EBD8` at 60% opacity, randomly lit, slow 3s flicker cycle — gives the city life
- Color: `#F0EBD8` fill with a 2px `#F0EBD8` top-edge highlight
- City sits on a ground bar: 16px tall, full-width, `#F0EBD8` at 30% opacity

### Enemy Visual Design

- **Cruiser:** 28×14px elongated rhombus, `#F0EBD8` outline only (hollow), no fill, 2px stroke. Four small engine-glow dots at rear, `#4DFFD6`.
- **Fighter:** 16×10px arrowhead, `#F0EBD8` outline, 1.5px stroke. Fast rotation to match heading (rotation always matches velocity vector).
- **Bomber:** 36×20px wide delta shape, `#F0EBD8` outline, 2.5px stroke. Slow, deliberate. A subtle pulsing amber glow (`#FF6B35`, oscillating 0.6–1.0 opacity at 1.2Hz) beneath it marks it as the missile carrier before wave 5 players realize what it means.
- **Homing Missile:** 8×4px dart, `#4DFFD6` fill (missiles share the interceptor color to create momentary visual confusion — they are in the same space). Exhaust trail: 12px `#4DFFD6` at 40% opacity, fading.
- **Enemy death:** On hit, enemy explodes into 8 line-segments radiating outward, `#F0EBD8`, each 6–12px long, fading to 0 opacity over 300ms. No particle spray. Geometry only.

### Arc / Interceptor Visual Design

- **Arc trail while drawing:** Dashed line from city launch point to cursor. Dash pattern: 6px on, 4px off. Color: `#4DFFD6` at 50% opacity. Updates every frame.
- **Interceptor in flight:** 6×6px diamond (rotated square), `#4DFFD6` fill. Behind it: arc trail, solid, `#4DFFD6`, fading from 100% at head to 0% at tail over 200ms trail length. Bloom on.
- **Intercept detonation (enemy kill):** Flash: `#4DFFD6` circle expanding from 0 to 40px radius over 180ms, then fading to 0 over 80ms. Inner: `#F0EBD8` point flash at center, 60ms duration. No lingering smoke.
- **Mid-air missile kill (The Moment):** Same detonation as above, PLUS: the destroyed missile's exhaust trail abruptly terminates and the trail segments scatter (4 small `#4DFFD6` segments, 4px each, drift 20px outward and fade over 400ms). The visual difference from an enemy ship kill is subtle but readable.
- **Miss (interceptor reaches arc endpoint with no contact):** No explosion. The interceptor simply vanishes with a single-frame `#F0EBD8` pixel flash at endpoint. Quiet. Punishing.

---

## SOUND SPEC

### Music

**Tempo:** 92 BPM exactly. Everything is locked to this grid.

**Structure:**  
Two-layer composition that breathes with game state:

**Layer A — Drone (always present):**  
Low brass cluster chord (Bb2, F3, Ab3 — a flat-7 chord, no resolution). Sustained, bowing articulation. Reverb tail: 4 seconds. This never stops. It is the world's ambient dread. Volume: -12dBFS baseline.

**Layer B — Sparse High Synth:**  
Single-oscillator sine wave with slow triangle LFO (rate: 0.25Hz, depth: ±8 semitones). Plays arpeggiated notes against the drone — Bb4, Db5, Ab4, Eb5 — one note per beat, skipping beats 2 and 4 most of the time (only 30% chance to play on weak beats). Creates negative space that feels like held breath. Volume: -18dBFS baseline.

**State Responses:**

| Game State | Music Change |
|---|---|
| Menu / idle | Layer A at -12dB, Layer B at -18dB. Slow. Sparse. |
| Wave start | Layer B note density increases: 50% chance on all beats. Transition over 2 seconds. |
| Enemy within 60% of city (closing fast) | Layer A volume rises to -6dB. A third layer activates: a rhythmic low pulse (kick-like, 92 BPM, every beat), volume -20dB, slowly rising to -14dB over 4 seconds. |
| Interceptor in flight | No change. The music doesn't follow individual actions. |
| City hit (health lost) | Layer A pitch-bends down 2 semitones for 800ms then returns. Creates a stomach-drop feeling. |
| Wave cleared | All layers drop to near-silence (-30dB) over 1 second. Silence held for 1.5 seconds. Then a single bell-tone (Eb5, sine, 600ms decay) marks the transition. Layer A fades back in. |
| Final wave / climax | All three layers at maximum. Layer B density: 80% chance per beat. A fourth layer activates: high-pitched string tremolo (F5–Ab5, alternating, 16th notes at 92 BPM). |

**Tone.js implementation notes:**  
- Use `Tone.Player` for pre-rendered drone stems (WAV, looped).
- Use `Tone.Synth` with `oscillator.type = "sine"` for Layer B arpeggiation, scheduled via `Tone.Transport` at 92 BPM.
- Low pulse: `Tone.MembraneSynth` triggered on `Tone.Transport` every beat when active.
- All volume changes: `Tone.Volume` node with `.rampTo(targetDb, rampTimeSeconds)`.

### SFX (6)

**1. Arc draw / drag (continuous while dragging):**  
Name: `arc-draw`  
A soft, rising electrical hum that increases in pitch as the arc extends longer. Starts at 200Hz, rises to 600Hz maximum as arc length increases from 0 to full screen diagonal.  
Tone.js: `Tone.Oscillator` (type: "sawtooth"), frequency mapped to arc length, gain -24dB, playing while pointer is down.

**2. Interceptor launch (on pointerup):**  
Name: `launch-thwip`  
A short percussive attack: 8ms noise burst (filtered through a highpass at 2kHz), followed immediately by a falling frequency sweep (600Hz → 200Hz over 80ms). Feels like a bolt leaving a rail. Total duration: 88ms.  
Tone.js: `Tone.NoiseSynth` (type: "white") gated for 8ms, followed by `Tone.Synth` with `envelope: { attack: 0, decay: 0.08, sustain: 0, release: 0 }` and a `Tone.FrequencyShifter` automating pitch down over 80ms.

**3. Enemy ship destroyed (interceptor hits ship):**  
Name: `ship-kill`  
A metallic crack: transient noise burst (full spectrum, 15ms) with a low "thud" component (80Hz sine, 60ms decay). Feels solid. Satisfying.  
Tone.js: `Tone.NoiseSynth` + simultaneous `Tone.Synth` at 80Hz with fast decay.

**4. Mid-air missile kill — THE SIGNATURE SOUND:**  
Name: `missile-kill-thwip-crack`  
The sound that defines the game. Two components in exact sequence:  
- **Thwip:** A rising frequency sweep (200Hz → 1400Hz over 40ms) with bright sawtooth character. This is the interceptor connecting.  
- **CRACK:** Immediately after, a sharp transient: white noise burst (5ms) with no tail, followed by a high-frequency ring (3200Hz sine, decaying over 120ms from -6dB to silence).  
Total feel: a whipcrack in open sky. Tight. Precise. Distinct from `ship-kill` — higher pitch, shorter, more fragile sounding.  
Tone.js: `Tone.Synth` sweep + `Tone.NoiseSynth` + `Tone.Synth` at 3200Hz, all triggered in sequence with 40ms offset between sweep-end and crack.

**5. City takes damage (enemy reaches city):**  
Name: `city-hit`  
A heavy, low impact: subby boom (50Hz sine, 300ms decay) with a mid crunch (500Hz filtered noise, 150ms decay). Played simultaneously. Followed 200ms later by a short distant rumble (filtered noise, 400ms, panned slightly).  
This sound should feel bad. Not game-over bad — consequence bad. The player should wince.  
Tone.js: Two `Tone.Synth` instances + `Tone.NoiseSynth` with `Tone.Filter`, triggered with staggered timing.

**6. Wave clear / wave incoming:**  
Name: `wave-clear` / `wave-start`  
- `wave-clear`: Single bell tone (Eb5, sine wave, attack 0ms, decay 600ms). Pure. Quiet relief.  
- `wave-start`: Three ascending stabs (Bb3, F4, Bb4) at 92 BPM intervals (each 652ms apart), all low brass character (`Tone.FMSynth` with high modulation index). Announces the next threat.

---

## MECHANIC SPEC

### Core Loop

1. Enemy formation enters screen from top or sides, moving on declared paths.
2. Player studies trajectory — enemies do not change direction mid-approach (wave 1–3); they may curve (wave 4–5).
3. Player places pointer at city launch platform (anywhere along the city bar counts as the origin), drags outward to the predicted intercept position, releases.
4. Interceptor launches and travels the arc at fixed speed. If interceptor position intersects a valid target within hit radius at any point along the arc: detonation.
5. If interceptor reaches arc endpoint without contact: it vanishes. Interceptor expended.
6. If enemy reaches the city bottom edge: city takes damage.
7. Clear all enemies (or all enemies that will reach the city — some can be ignored if they pass harmlessly): wave ends.

### Input (exact)

- **Origin:** `pointerdown` anywhere in the bottom 15% of the viewport (the "city zone") initiates a draw. The arc origin snaps to the nearest city block center (the tallest building in the nearest cluster). This means players don't need pixel-perfect clicks on the city — the "launch zone" is generous.
- **Drag:** While pointer is held, a dashed preview arc renders from origin to current pointer position.
- **Release:** `pointerup` fires the interceptor. The interceptor travels from the origin along the arc path to the endpoint, at fixed speed. The arc is a quadratic Bézier curve: start = city origin, end = pointer release position, control point = midpoint of start→end offset 80px perpendicular (leftward of direction of travel, creating a consistent leftward bow to all arcs). This gives arcs a visual character and means the flight path is always slightly curved — not a straight line.
- **Multi-draw:** Player can draw a new arc while a previous interceptor is in flight. Interceptors are the constraint — see ammo system.
- **No right-click, no keyboard input required** (keyboard shortcuts are additive, not required): Spacebar does nothing. This is a single-input game.

### Interceptor Ammo System

- Player has a reserve of interceptors displayed as pips in the HUD (bottom-right corner).
- **Starting reserve:** 6 interceptors per wave.
- **Replenishment:** At wave start, reserve refills to 6. Unused interceptors do NOT carry over (no hoarding).
- **Maximum simultaneous in-flight:** 3. If 3 are already in flight, new draws are blocked (arc preview turns `#FF6B35` and shakes — 3px horizontal shake at 20Hz for 200ms — indicating locked out).
- If reserve reaches 0 and active interceptors reach 0: player cannot fire. Any remaining enemies reach the city. This is a designed failure mode, not a softlock.

### Interceptor Speed

- **Fixed flight speed:** 520 px/s at 1280×720 reference resolution. This is constant — no upgrades, no variation. The player's only variable is where they aim.
- Scale proportionally with viewport: `speed_px_per_s = 520 * (viewportWidth / 1280)`.

### Enemy Speed Classes (exact, at 1280×720)

| Class | Name | Speed (px/s) | Visual | Behavior |
|---|---|---|---|---|
| Slow | Cruiser | 72 px/s | 28×14px rhombus | Straight lines only (waves 1–4). Wide arcing turns in wave 5. |
| Medium | Fighter | 148 px/s | 16×10px arrowhead | Straight lines, slight random drift: ±8px lateral wander every 1.2s |
| Fast | Interceptor-class | 230 px/s | 12×8px dart | Straight lines only. Very fast. Difficult to lead correctly. |
| Missile | Homing Missile | 180 px/s | 8×4px dart | Wave 5+ only. Tracks city after launch. Turns at max 90°/s. |

- All speeds scale with viewport: `speed = base_speed * (viewportWidth / 1280)`
- Enemy entry paths are declared at wave start — the programmer should log entry angle, speed, and path type at wave-init so a determined designer can verify them.

### Hit Detection

- **Hit radius:** 18px (center-to-center distance between interceptor and target at any frame). If distance ≤ 18px: hit.
- Hit check runs every frame (60fps) for each active interceptor against each active enemy and missile.
- Interceptor position is computed by parametric travel along its Bézier arc: `t` advances at `(speed * deltaTime) / arcLength` per frame.
- An interceptor detonates on first contact. It cannot chain-kill.
- **City damage:** An enemy is considered "reaching the city" when its Y-position (in screen space) reaches the top edge of the city silhouette. At that moment, the enemy is removed and the city loses 1 health point. No damage radius — this is contact only.

### City Health

- **Starting health:** 5 (displayed as 5 `#F0EBD8` window-light pips in HUD, bottom-left)
- Each pip represents one building cluster. When a pip is lost, that building cluster's windows go dark permanently (visual feedback tied to damage).
- At 1 health remaining, all remaining windows flicker rapidly (10Hz on/off, `#FF6B35` tint). The city is dying.
- At 0 health: GAME OVER immediately. No last-ditch wave. City collapses animation: buildings fall inward over 1.5 seconds (rotate inward around base pivot, gravity-style, using keyframe animation), then fade to black. The score appears in white on black after 2 seconds.

### Score

- Enemy ship destroyed: +100 points
- Homing missile destroyed mid-flight: +250 points (distinct higher value — reinforces The Moment)
- Wave completed with interceptors remaining: +50 per unused interceptor
- Wave completed without city taking any damage: +500 bonus
- No time bonus. No combo multiplier. This is not a score game — score exists for bragging rights only.
- Score displayed top-center, `#F0EBD8`, small (18px), no animation.

### Missile Sub-Layer (Wave 5+)

- Bombers (slow class enemies) begin launching homing missiles when they enter the lower half of the screen (Y > 50% viewport height).
- Each Bomber launches 1 missile after a 1.2-second wind-up (Bomber pulses amber `#FF6B35` glow during wind-up — this is the tell).
- Missile launches from Bomber's center and immediately homes toward city center (not toward a specific building — toward the exact center of the city bar).
- Missile can be intercepted by the player's interceptors. Hit radius same as enemy hit (18px).
- If missile reaches city: 1 health damage (same as enemy contact).
- Missile has no HP — one hit kills it.
- A Bomber that is destroyed before launching does not fire its missile.
- Wave 5 maximum: 2 Bombers on screen at once, so maximum 2 active missiles simultaneously.

---

## LEVEL DESIGN — 5 WAVES

### Wave 1 — The Lesson

**What's new:** The game begins. Cruisers only. Slow. Straight paths. The player will likely miss their first arc (aimed at ship position, not ahead of it). They will adjust. They will hit. The aha happens here.

**Parameters:**
- 4 Cruisers total, entering in sequence (one every 3 seconds)
- Entry: top edge only, equally spaced horizontally (25%, 40%, 55%, 75% of viewport width), moving straight down
- Speed: 72 px/s
- Interceptors: 6
- City health: 5
- No missiles
- Any Cruiser not intercepted before reaching Y=85% viewport changes course: it circles back up and exits screen (does NOT damage city). This prevents a first-time player from losing health on wave 1 before they understand the mechanic. Only on wave 1 — this safety behavior does not repeat.

**Designer note:** Wave 1 is a tutorial that never calls itself a tutorial. The Cruiser's slow speed gives 8+ seconds of lead time. The player has time to stare, think, draw wrong, miss, redraw, hit. The satisfaction arrives on its own.

---

### Wave 2 — Speed is Knowledge

**What's new:** Fighters introduced. Players immediately discover that the lead distance calibrated for Cruisers wildly overshoots Fighters. The calibration problem is made explicit.

**Parameters:**
- 3 Cruisers + 4 Fighters, entering in mixed sequence
- Cruisers enter top edge. Fighters enter top-left and top-right corners, angling toward city center at 20° inward.
- Cruiser speed: 72 px/s
- Fighter speed: 148 px/s
- Entry interval: one enemy every 2.2 seconds
- Interceptors: 6
- A Fighter left alone will reach the city in ~4 seconds from mid-screen. This is the first wave where timing pressure is real.

**Designer note:** The mix of speeds is intentional. A player who tries to use one "default" arc length will fail consistently. Pattern recognition begins here: the arrowhead (Fighter) needs a shorter lead than the rhombus (Cruiser).

---

### Wave 3 — The Formation

**What's new:** Enemies enter in coordinated formation rather than sequentially. Players see a "block" of threats and must triage — which do I intercept first? The ammo limit (6) becomes meaningful when 7 enemies enter at once.

**Parameters:**
- Formation A (enters at wave start, 0s): 3 Fighters in a V-formation, tips forward, spacing 60px between each. Moving straight down at 148 px/s.
- Formation B (enters at 4s): 2 Cruisers side-by-side, 80px apart, moving straight down at 72 px/s.
- Formation C (enters at 7s): 2 Fighters, from left and right edges respectively, converging at city center. Speed: 148 px/s.
- Total enemies: 7. Interceptors: 6. The math means the player cannot shoot everything — one enemy will likely slip through unless they are fast and accurate.
- The 7th enemy (design intent: one of the Formation C Fighters) should be the one that reaches the city if the player panics. 1 health damage is survivable. It's a lesson in triage, not a punishment.

---

### Wave 4 — The Drift

**What's new:** Fast Interceptor-class enemies (230 px/s). Fighters now have lateral drift. The battlefield is no longer a set of simple straight-line problems — the player must predict a slightly non-linear future.

**Parameters:**
- 2 Interceptor-class enemies (straight paths, 230 px/s) — these are the new threat, introduced in pairs so the player sees them clearly
- 3 Fighters with drift (±8px wander every 1.2s) — the drift is subtle, but players who drew a "memorized" arc from wave 2 will notice their timing is slightly off
- 2 Cruisers (reliable slow targets — these are the "breathers" the player can confidently intercept)
- Entry timing: 2 Interceptor-class first (0s), then Fighters (4s), then Cruisers (8s)
- Interceptors: 6
- The Interceptor-class ships are the speed test: at 230 px/s from the top edge, they reach the city in ~4 seconds. At 520 px/s interceptor speed, a player has a very narrow window to catch them with a long lead arc.

**Designer note:** Wave 4 is the hard reset on confidence built in waves 1–3. Players who mastered slow-class lead angles will be humbled by fast-class. This is intentional. The game is asking: can you recalibrate?

---

### Wave 5 — The Missile Layer

**What's new:** Homing missiles. The game becomes two simultaneous games — the slow-moving ship threat layer and the fast reactive missile threat layer. The first mid-air missile kill happens here.

**Parameters:**
- 2 Bombers enter at 0s, from top-left and top-right corners, angling toward city at 25° inward, speed 72 px/s
- Each Bomber launches 1 missile at Y > 50% viewport (approximately 3.5 seconds after entry at 72 px/s)
- 1.2s wind-up before missile launch (amber pulse on Bomber)
- 2 Fighters enter at 5s from top edge, 148 px/s, straight path
- 1 Interceptor-class enters at 9s from top-left corner, 230 px/s
- Missiles: 180 px/s, homing toward city center, turning at max 90°/s
- Interceptors: 6 (critical — player must decide: spend interceptors on ships to prevent missiles being launched, or let Bombers fire and intercept the missiles mid-air)
- **City health carries over from waves 1–4.** If the player survived perfect, they have 5 health. If they took damage, they're playing wave 5 with a depleted buffer.

**Designer note:** The two Bombers are the strategic choice point. An advanced player (10x) destroys both Bombers before they reach 50% screen height — no missiles launched. A mid-level player (5x) destroys one Bomber, intercepts one missile. A new player (1x) panics at the missiles and draws reactive arcs — sometimes succeeds, sometimes the city takes a hit. All three skill levels can complete wave 5. The ceiling is making it look effortless.

---

## THE MOMENT

**Wave 5. A Bomber has entered. It's pulsing amber. The player didn't intercept it in time.**

The missile launches. A small cyan dart, trailing light, curving toward the city. The player has drawn all their intercept arcs thinking about Fighters. Now something is homing — not traveling in a straight line — and it's 2 seconds from the city.

**The player draws an arc. They aim not at the missile's position but where it will be when the interceptor arrives.**

This is the second time the player has had to predict a future. The first time (wave 1) the target was a slow rhombus moving in a straight line. This time the target is a small dart curving through space.

The interceptor launches. Travels its arc. The missile curves.

**They meet.**

The `thwip-CRACK` fires. The detonation bloom expands in the open sky — no ship behind it, no building nearby. Just a burst of cyan light in empty magenta air, 200px above the city.

**Nothing about this looks different from destroying a ship. But it feels completely different.** The player just proved they can predict a curve. They just learned they are playing a harder game than they thought. They just became, without any UI telling them so, a better player than they were 30 seconds ago.

The four scattered missile trail segments drift and fade. The silence after the CRACK is 400ms of held breath before the next threat demands attention.

This moment is why the missile's color (`#4DFFD6`) matches the interceptor's color. When they meet in the sky, it's two pieces of the same light colliding. The game is giving you its only aesthetic mercy: the things you destroy look like the things you make.

---

## EMOTIONAL ARC

### First 30 Seconds (The Aha)

Player sees a Cruiser descending. They tap the city, drag to the Cruiser's current position, release. The interceptor launches, travels its arc, arrives at the Cruiser's previous position. Miss. The Cruiser continues downward.

The player waits 2 seconds — the miss is quiet, no dramatic feedback, just the Cruiser still moving — then draws again. This time they drag further ahead, to where the Cruiser will be. Release.

The interceptor meets the Cruiser. Detonation. The `ship-kill` sound fires.

**The understanding is instantaneous.** Not "I learned to lead targets" — that cognitive framing comes later. In this moment it's just: *oh. further. that's it.*

The player has mastered the fundamental mechanic in under 30 seconds. The rest of the game is complexity layered onto that single insight.

### After 2 Minutes (The Groove)

The player is no longer looking at enemy positions. They are looking at trajectories — invisible lines extending from each enemy's current position through the battlefield.

They see a Fighter entering top-left. Without consciously deciding, they draw an arc that meets the Fighter at 40% screen height, 1.8 seconds from now. Correct. They were drawing before they finished the thought.

The battlefield has transformed. What was a set of positions is now a set of futures — a dynamic diagram the player reads and writes simultaneously. Enemy ships are not in a place. They are *arriving* at a place, and the player is filling those future places with interceptors before the enemies get there.

This is the groove. It is quiet and meditative and requires real-time spatial calculation that the player stops thinking about because it has become instinct.

### Near Win / Climax (Three Seconds Left)

Wave 5. The player has 1 interceptor remaining. Three threats visible:

- **Left:** An Interceptor-class ship, 230 px/s, from top-left corner, 3 seconds from city edge.
- **Center:** A homing missile, curving toward city center, 2.8 seconds out.
- **Right:** A Fighter, 148 px/s, drifting slightly, 4 seconds from city edge.

Two of these threats will reach the city if unaddressed. The player has one shot.

The Fighter (right) has the most time — it's the lowest priority. The real question is: missile or ship?

**The player who reads this correctly:** The Interceptor-class ship cannot change direction — it's on a fixed path. But the missile is homing. If the player destroys the ship, the missile curves and hits the city. If the player destroys the missile, the ship hits the city — but the city has 2 health remaining. It survives.

**The 10x player already saw this.** They had 2 interceptors 5 seconds ago and burned one on the Bomber before it launched. They have 1 interceptor and zero missiles to worry about. They are shooting the Interceptor-class ship from above midscreen, pre-clearing it before it becomes a time-pressure problem.

**The 1x player is here right now.** Three seconds. One arc. The magenta sky is full of moving futures, and only one of them is the right one.

---

## THIS GAME'S IDENTITY IN ONE LINE

*You don't aim at the enemy — you aim at where the enemy will be when your arc arrives, and you earn the right to know the difference.*

---

## START SCREEN

### Idle Animation

The title screen is the game, slowed to a dreamlike pace.

- 8 enemy ships (mix of Cruisers and Fighters, ratio 5:3) descend in two loose formations from the top of the screen toward a city silhouette at the bottom. Speed: 30% of normal gameplay speed. They loop — when they reach 90% screen height, they fade out and re-enter from the top.
- Ghost arc trails: Faint dashed arcs (`#4DFFD6` at 15% opacity) connect the city to points along the enemy paths. These are "suggested" intercept points — they look like someone drew them and they half-succeeded. They appear and fade over 4-second cycles, non-interactive.
- One interceptor (full opacity `#4DFFD6`) travels along one of these ghost arcs every ~7 seconds, visually demonstrating the mechanic without text. It "misses" — arrives just behind an enemy — and the arc fades. 3 seconds later, another interceptor travels a slightly longer arc, "hits" a Cruiser mid-screen. Detonation bloom fires. City briefly lights up. This is the aha, running on loop, silent.
- The city silhouette at idle has all windows lit, slow flicker. It is alive. It is what the player is protecting.

### SVG Title Spec — Option A (Glowing Title) + Option C (Decorative Corner Brackets)

**Option A — Glowing Title:**
- Text: "ARC READ" — all caps, two words with a space (not hyphenated on start screen)
- Font: Monospace geometric — use `font-family: 'Courier New', monospace` or embed a subset of `SpaceMono-Regular` if available. Fallback: system monospace.
- Letter-spacing: 0.25em
- Font size: 7vw (scales with viewport)
- Color: `#F0EBD8`
- SVG filter applied to text group: Gaussian blur (`stdDeviation="3"`) layered behind the sharp text, same color — creates a soft glow halo effect without Three.js post-processing.
- A second, larger blur pass (`stdDeviation="8"`, opacity 0.4) for the bloom radius.
- The title subtly pulses: opacity oscillates between 0.92 and 1.0 at 0.4Hz (sine easing). Not distracting — just breathing.
- Positioned: 38% from top of viewport, centered horizontally.

**Tagline below title:**
- Text: "Draw where they will be."
- Font: same monospace, 1.8vw, letter-spacing: 0.15em
- Color: `#F0EBD8` at 60% opacity
- No glow. Plain.
- Positioned: 8px below title baseline.

**Start prompt:**
- Text: "CLICK TO BEGIN"
- Color: `#4DFFD6`
- Font: same monospace, 1.4vw, letter-spacing: 0.2em
- Blink: on/off at 1Hz (CSS animation or SVG animate element — 500ms on, 500ms off, no easing — sharp blink)
- Positioned: 65% from top of viewport, centered.

**Option C — Decorative Corner Brackets:**
- Four corners of the viewport, inset 24px from each edge
- Each corner: an L-shaped bracket drawn in SVG lines, `#F0EBD8` at 35% opacity, stroke-width 1.5px
- Bracket arm length: 32px (horizontal) × 32px (vertical)
- The brackets do NOT pulse or animate independently — they are static structural decoration.
- On hover over the start button / click prompt area: brackets briefly animate inward 4px and back over 200ms (ease-in-out). This is the only interactive feedback on the start screen beyond the click.
- At game start (click detected): brackets animate inward rapidly (4px over 80ms), hold for 200ms, then the entire screen cuts to black over 300ms and the game begins.

### Start Screen HUD Elements (non-animated)

- Top-right: Version number `v0.1` in `#F0EBD8` at 20% opacity, 12px monospace.
- No other chrome. No "high score." No "settings." This is the first communication that the game does not hand-hold.

---

## HUD DESIGN (In-Game)

The HUD is minimal. It never occludes the battlefield.

**Health (bottom-left):**
- Label: "CITY" in 11px monospace, `#F0EBD8` at 50% opacity
- 5 pips in a row, each 10px × 10px square, 6px gap between
- Active pip: `#F0EBD8` fill. Lost pip: `#7A3060` fill with `#F0EBD8` 30% opacity border. Dead pip looks like a dark hole.
- At 1 health remaining: pips pulse `#FF6B35` at 2Hz.

**Interceptors (bottom-right):**
- Label: "ARCS" in 11px monospace, `#F0EBD8` at 50% opacity
- Up to 6 pips, each 10px × 10px diamond shape, 6px gap
- Active: `#4DFFD6` fill. Expended: `#4DFFD6` at 15% opacity outline only.
- When a new arc is drawn, the rightmost active pip animates: scale from 1.0 to 0.0 over 150ms (squeeze-vanish). Satisfying pip spend animation.

**Wave counter (top-left):**
- "WAVE 01" → "WAVE 05" in 13px monospace, `#F0EBD8` at 70% opacity
- Updates at wave start with a brief 1-frame flash to full opacity then fades to 70%.

**Score (top-center):**
- Raw number, 18px monospace, `#F0EBD8`
- On score increase: number briefly scales to 1.15× over 100ms then returns to 1.0× over 100ms (number-pop feedback).

**All HUD elements are in screen-space**, not world-space. They do not tilt with the camera. They live in an HTML overlay layer above the Three.js canvas.

---

## WIN / LOSE CONDITION

**Win:** Survive all 5 waves with city health > 0.  
**Win screen:** The city silhouettes all windows fully lit. "CITY SURVIVED" in `#4DFFD6`, large centered text. Score displayed below. "WAVE CLEAR: 5/5". A single bell tone (`wave-clear` SFX, played twice with 600ms gap). After 3 seconds, a "PLAY AGAIN" prompt appears in `#F0EBD8` blinking at 1Hz.

**Lose:** City health reaches 0.  
**Lose screen:** Building collapse animation (1.5s). Fade to black (0.3s). "CITY LOST" in `#F0EBD8` at 50% opacity (dim — not dramatic, just final). Score. "WAVE: X/5". After 2 seconds, "TRY AGAIN" prompt in `#4DFFD6`, blinking 1Hz.  
No game-over music sting. The drone Layer A continues at -24dB (barely audible) — the world keeps existing even when you lose.

---

END_DESIGN_DOC
