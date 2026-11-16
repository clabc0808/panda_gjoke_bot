# Panda fit study — does everything actually fit at the locked size?

**Date:** 2026-11-16. **Question (his words):** with everything discussed — all the
microcontrollers, the K230 / Pi / electricity boards — can they actually FIT into the
robot at the locked size, with no surprises later? And where does the charging hole
go: the magnetic charging connector is preferred on the BACK; if that cannot work,
say so and give the fallback.

**Locked proportions used throughout (not re-opened):** total height ~35 cm; head
~15 cm across, hollow printed panels; trunk ~9–10 cm tall; legs 11–13 cm (under
15 cm whole); big flat feet; design mass ~2.2–2.4 kg; head contents budget ≤ 250 g.

**Method:** every dimension below is a real published figure with its source named,
or it is marked *(est.)* — an estimate for a part whose exact variant is unconfirmed.
Where two published figures disagree, both are given and the larger is used for
planning. Trunk reference size is the assembly report's estimate (~120 × 110 × 95 mm
outer); the head shell is taken as ~150 wide × ~115 tall × ~120 deep from the locked
proportions (350 total − ~120 legs − 95 trunk − ~25 neck stack ≈ 110–115 head).

---

## Resolutions added 2026-11-16 (his answers — these supersede the open items in §4–§7)

1. **Lidar is a Slamtec RPLIDAR C1** — 55.6 × 55.6 × 41.3 mm, 110 g, DTOF 360°,
   UART (manufacturer figures). §7's A1/LD06 classes no longer apply.
   **Placement: belly pod under the trunk** — scan plane ~8–10 cm off the floor,
   110 g carried low (helps the centre of mass). In-trunk still fails (stack +
   window), on-head still fails (+110 g over the head budget, +41 mm height).
2. **Charging port moves to the BACK OF THE HEAD** (his choice). The trunk back
   panel goes to the Sipeed mic array instead (§7's port-XOR-array conflict is
   resolved this way — both stay on the back, at different heights).
   Dock geometry this implies: a slim post standing ~100 mm behind the parked
   robot with a forward arm at head height (~280 mm) carrying its half — at
   trunk height there is nothing on the centreline for the mic array to strike,
   and the feet/heels never reach the post. The robot charges in a defined
   **park pose** (head yaw/pitch centred, slight back-tilt so the head's back
   panel rests toward the post; contact must not depend on servo torque alone).
   The 2 charge leads join the reserved neck cable channel down to the IP2326
   in the trunk (channel now ~12 wires — CAD must reserve it). INA226 charge
   current remains the dock-success signal.
3. **K230 cameras** — his board has 2 camera ports (never in doubt). The §5
   flag narrows to: 01Studio documents only the OV5647 for the expansion port,
   so a matched GC2093 pair is unproven on paper. Acceptable outcomes: ask
   01Studio, or run GC2093 + OV5647 mixed (fine for face tracking; matched
   matters mainly for precise stereo depth).
4. **Charging pair confirmed: the 2-point version** (~2 A class, not the
   4-pin ~4 A). Consequence is only speed: ~10 W into the IP2326, pack charge
   ≈1 A, **full charge ≈5–6 h** — fine for overnight docking, slow for a
   top-up. Dock and head-port design unchanged. A 4-pin pair is a drop-in
   upgrade later if faster charging is ever wanted.
5. **Sipeed array size verified** against Sipeed's own datasheet: the R6+1
   board really is **78.1 × 88.8 mm** — on the 120 × 95 mm trunk back panel
   it leaves ~3 mm spare top and bottom. It is the right part and the right
   number; the back-view render draws it too small.
6. **Lidar placement PARKED (his word, 2026-11-16)** — the C1 is owned and
   the belly pod works on paper, but he wants to think about it (the pod's
   look vs the on-head costs in §7). No placement is in the design until he
   decides; the current back-view render omits it.
7. **Mic array mounting settled**: external, at true size, as a plate on the
   trunk's back (his "glass hat") — no recessing into the panel. The R6+1's
   12-LED ring is part of its look.
8. **K230 identity RESOLVED (his description, 2026-11-16; closes audit
   finding F4)**: the board is the **full-size 01Studio CanMV K230** —
   85 × 56 × 15 mm, 31 g (01Studio wiki) — already owned. Built-in GC2093
   camera plus TWO expansion ports (CSI0/CSI1) on the board's longer sides.
   Consequences: the §2 head layout was drawn for the mini (60 × 43 mm,
   16 g) and must be redrawn around the bigger board — the 150 mm shell
   still swallows the footprint, and contents rise only ~15 g to ~235 g
   vs the 250 g budget. The camera problem gets easier: two documented
   expansion ports make a **matched OV5647 pair (15 cm FPC)** a fully
   on-paper stereo option, with the built-in GC2093 as a spare/third eye.

---

## 1. Dimension table — every part, real size, source named

### Head parts

| Part | L × W × H (mm) | Weight | Source |
|---|---|---|---|
| 01Studio CanMV K230 mini (bare board) | 60.3 × 42.7 × 17.6 | 16 g | 01Studio wiki / product spec (17.6 includes the on-board camera stack standing proud of the PCB) |
| K230 mini in 01Studio acrylic case | 64.3 × 46.7 × ~25 *(case adds ~8)* | 40 g | 01Studio wiki — **not used**; the board goes bare into our own shell with its stick-on heatsink (~5 g, *est.*) |
| Camera module, GC2093 (the mini's standard camera) | module total length 17; lens barrel ~Ø10 *(est.)* | ~3 g *(est.)* | 01Studio wiki ("Size 17 mm (total length)", 24P interface, 70° FOV; 140° version exists) |
| Camera module, OV5647 (the mini's documented expansion camera) | module on 22P-0.5 mm FPC, cable lengths 6 / 15 / 30 cm | ~3 g *(est.)* | 01Studio wiki |
| GC2093 extension FPC | 24P, **15 cm only** | — | 01Studio wiki |
| MG90S ear servo × 2 | 22.8 × 12.2 × 28.5 | 13.4 g each | TowerPro-class listings |
| STS3215-C001 servo (× 14 in the robot; 4 are neck/head) | 45.2 × 24.7 × 35 | 55 ± 1 g each | Feetech datasheet / OpenELAB |
| 6804ZZ bearing (head, × 1 of the 3 bought) | Ø32 × 7 (20 mm bore) | ~19 g *(est.)* | Bearing catalogue dims; weight estimated |
| Waterproof 4 Ω 3 W cavity speaker (voice set) | ~31 × 28 × ~10 *(est.)* | ~18 g *(est.)* | **Estimate for the class** (2831/3128 cavity-box listings). The exact ¥35.9-set unit is unmeasured — measure it when it arrives; the common class is 31 × 28 mm footprint, 9–11 mm deep |
| MAX98357A amp breakout | 19.4 × 17.8 × 3.0 | ~1.2 g | Adafruit datasheet (generic purple modules are the same class) |

### Trunk parts

| Part | L × W × H (mm) | Weight | Source |
|---|---|---|---|
| Raspberry Pi 5 | 85 × 56 × ~17 (to top of port stacks) | 46 g | Raspberry Pi spec / retailer listings |
| Pi 5 Active Cooler | sits over the SoC; planning envelope for Pi + cooler **85 × 56 × ~28** | 28 g | Retailer listings; combined height *(est.)* — the cooler's fan top is the highest point over the board's centre |
| 2-cell 18650 series holder × 2 (4 cells, 2S2P) | 77.9 × 40.1 × 21.4 each | 15.5 g each (empty) | DFRobot FIT0538 listing; generic holders are the same class (~78 × 40 × 21) |
| 18650 cell × 4 | Ø18.6 × 65.2 each | ~47 g each *(est., 30 A-class cells)* | Standard 18650 dims |
| HW-391 BMS (2S 20 A balanced) | 48 × 20 × ~4 | ~5 g *(est.)* | Multiple listings agree on 48 × 20 |
| Pololu D36V50F5 buck (#4091) | 25.4 × 25.4 × 9.5 | 7.0 g | Pololu |
| IP2326 charger module (15 W USB-C variant) | 32 × 18 × ~6 | ~4 g *(est.)* | ElectroPeak / iFuture listings (31.2–32 × 18 × 5–6.3). **A larger 50 × 30 mm PD variant exists — do not buy that one (see §5.8)** |
| Waveshare Bus Servo Adapter (A), SKU 25514 | 42 × 33 × ~10 | 16 g | Waveshare docs / Botland; height *(est.)* from Cirkit-class listings |
| INA226 module (CJMCU-226 class, already owned) | 27 × 20 × ~8 with pin headers (bare PCB ~3; terminal-block variants up to 14 tall) | ~5 g *(est.)* | OpenELAB / module manual (20 × 27 mm); height varies by variant — his exact unit unmeasured |
| BNO055 breakout, Adafruit 4646 STEMMA QT | ~25.6 × 22.7 × 4.6 | ~3 g | Adafruit — 4646's own fab print not published in the sources checked; figures are the identical STEMMA QT form factor of sibling 4754 (25.6 × 22.7 × 4.6 mm, 2.5 g). Treat as *(near-certain estimate)* |
| Rocker switch, KCD1 class (2-pin, 2-position) | body 21 × 15; **panel cutout 19.2 × 13.2**; ~25–28 deep behind the panel incl. terminals | ~5 g *(est.)* | KCD1 listings (cutout 19.2 × 13.2 mm; overall depth 24–28 mm) |
| Magnetic pogo charging pair, robot half | ~28 × 12 × ~8 *(est.)* | ~10 g *(est.)* | **Estimate for the pre-wired 4-pin class** — his exact pair's brand/size unconfirmed; measure on arrival. Fit conclusions below hold for anything in the 25–35 mm wide class |

### Legs / feet

| Part | L × W × H (mm) | Weight | Source |
|---|---|---|---|
| Omron SS-10 foot switch × 4 (pin plunger) | 19.8 × 10.2 × 6.4; operating position 8.4 ± 0.5; operating force 150 gf | 1.6 g each | Omron SS-series datasheet |

### §7 parts (lidar & mic array — evaluated in §7)

| Part | L × W × H (mm) | Weight | Source |
|---|---|---|---|
| Slamtec RPLIDAR A1 (A1M8) | 96.8 × 70.3 × 55 (98.5 wide overall) | 170 g (sensor; 248 g gross kit weight in some listings) | Slamtec datasheet via Waveshare/OpenELAB; Adafruit (170 g) |
| LDROBOT LD06 | 38.6 × 38.6 × 33.3 | 42 g | LDROBOT specs via multiple retailers |
| LDROBOT LD19 (= D300 kit sensor) | sensor 38 × 38 × 34.3; with D300 mounting plate 54 × 46.3 × 35 | 45–47 g | LDROBOT specs via multiple retailers |
| Sipeed R6+1 mic array (7 × MSM261S4030H0 + LED ring) | **78.1 × 88.8** board | ~38 g gross (Seeed listing; bare board ~30 g, *est.*) | Sipeed wiki (size); Seeed (weight) |

---

## 2. Head layout — what goes where

Shell assumed ~150 wide × ~115 tall × ~120 deep, 3 mm printed panels → interior
≈ 144 × 109 × 114 mm. That is a lot of room for what has to live in it; the head's
real constraints are **weight** and the **neck stack at the bottom centre**, not volume.

Where the neck/head servos live (Open Duck Mini v2 mechanism, per the assembly
report §1.5): servo **30 (neck pitch)** is mounted on the **trunk top** — its body
belongs to the trunk. Servos **31 (head pitch), 32 (head yaw), 33 (head roll)** stack
in the neck/head-base mechanism, closed over by the head's bottom sheet. So of the
four, only 32 and 33 sit fully inside the head's lower mechanism, 31 straddles the
neck, and 30 stays in the trunk. The CAD must reserve a ~55 × 45 mm column at the
bottom centre of the head for the 31–33 stack plus the 6804ZZ head bearing (Ø32).

```
HEAD — front view (interior, ~144 wide × ~109 tall), not to exact scale
┌──────────────────────────────────────────────────────────────┐
│  [MG90S ear]                                    [MG90S ear]  │  ears: servos inside top
│   top-left                                        top-right  │  corners, shafts pointing
│                                                                │  out sideways; hollow ear
│        ┌──────────────────────────────────┐                  │  shells reserve ~Ø10 mm
│        │   K230 mini  60.3 × 42.7         │   crown vents    │  for a future mic each
│        │   vertical, on face frame        │   (slots above)  │
│        └──────────────────────────────────┘                  │
│   ( ● ) eye L        face frame        eye R ( ● )           │  eye centres ±35 mm →
│    GC2093/OV5647  ← FPC ≤ ~50 mm →  K230 CSI ports           │  **stereo baseline 70 mm**
│                                                                │
│        ┌────────────────────┐                                  │
│        │ speaker 31 × 28    │  behind the smiling mouth slot   │
│        └────────────────────┘                                  │
│              ┌─────────────────────┐                           │
│              │ neck stack 31·32·33 │  bottom centre, ~55 × 45  │
│              │ + 6804ZZ bearing    │                           │
│              └─────────────────────┘                           │
└──────────────────────────────────────────────────────────────┘
```

- **K230 mini** mounts vertically on the inner face frame, service/USB side down.
  60.3 wide × 42.7 tall × 17.6 deep fits the face zone with ~40 mm to spare on every
  side. Stick-on heatsink on the SoC (see thermal note below).
- **Eyes / stereo baseline:** camera modules sit in the black eye patches with lens
  centres at **±35 mm → 70 mm baseline** (close to human eye spacing ~63 mm; a good
  stereo baseline at this size). FPC run from each module to the K230's CSI ports is
  ~40–60 mm including bend allowance. The board's standard GC2093 tail is too short
  for this, so remote mounting needs the **15 cm GC2093 extension FPC** (the only
  length sold) or the OV5647's **15 cm** option — the 6 cm OV5647 cable is too short
  once bends are counted. Excess ribbon folds once, gently (keep bend radius ≥ 5 mm).
  Because the K230 lives *in* the head, these ribbons never cross a moving joint —
  no flex-fatigue issue. **But see §5.1: which second camera the mini actually
  accepts is not yet proven — read that before buying anything.**
- **Speaker** (31 × 28 × ~10 *est.*) sits directly behind the smiling mouth slot,
  lower-centre face. The MAX98357A amp does **not** go in the head: it moves to the
  trunk mezzanine (§3), next to the Pi whose I2S pins drive it, and a single 2-core
  speaker lead runs up the neck. Short I2S, dumb speaker wires across the joint —
  the right way round.
- **Ear servos:** MG90S bodies inside the top side walls (22.8 long fits the ~109 mm
  interior height zone easily), shafts out through the side panels. The hollow ear
  shells keep ~Ø10 mm of free space each — the parked mics-in-ears idea stays
  possible with no redesign.
- **Cable bundle up the neck:** servo bus (3 wires) + K230 power/UART (4 wires) +
  speaker pair (2) ≈ 10 thin wires, bundle Ø ~8–10 mm. The neck stack must keep a
  channel at least that big clear of the roll bearing — a CAD check, not a blocker
  (§5.10).

**Head weight vs the 250 g budget** (contents = shell + electronics, the way the
earlier report defined the budget):

| Item | g |
|---|---|
| Printed shell panels + face frame + ears | ~118 *(est., swing ±30 — the risk item)* |
| K230 mini + heatsink | 21 |
| 2 camera modules + FPCs | ~8 *(est.)* |
| 2 × MG90S | 26.8 |
| Speaker | ~18 *(est.)* |
| 6804ZZ head bearing | ~19 *(est.)* |
| Cables / screws / misc | ~10 *(est.)* |
| **Total contents** | **≈ 220 g — inside the 250 g budget, ~30 g margin** |

**The definitional trap (no surprise later, so stated plainly):** the 250 g budget
counts *contents*. Servos 32 + 33 (110 g) physically ride in the head base, so the
mass the head-pitch servo (31) actually lifts is ≈ 220 + 110 = **~330 g**, and the
neck-pitch servo (30) sees ~385 g including servo 31. Against the STS3215's
30 kg·cm stall torque that is still trivial (~1.5 kg·cm worst-case static at a 4 cm
offset) — but if the printed shell comes out 30 g heavy, the *contents* budget is
what breaks first. Weigh the shell before final assembly.

**K230 thermal:** the K230 draws roughly 2–3 W under AI load *(est. from board
class)* inside a sealed plastic head. The stick-on heatsink is **mandatory**, and
the crown needs vent slots above the board (shown in the sketch). 01Studio sells an
official cooling-fan accessory for this board (5 V, fits the top cover) — reserve
~10 mm and ~10 g for it above the board; only fit it if throttling shows up in
testing.

---

## 3. Trunk layout — the stack, with heights added up

Trunk outer ~120 (W) × 110 (D) × 95 (H); 3 mm walls/floor/top → **interior
≈ 114 × 104 × 89 mm**. Vertical budget: 89 mm. The plan, bottom to top:

```
TRUNK — side view (depth 104 →), heights in mm from the floor
 ┌────────────────────────────────────────────────────┐  89  top panel
 │        neck servo 30 hangs from top, centre:       │  89–54  servo 30 zone
 │        body 45.2 × 24.7 × 35, bottom edge at 54    │        (over the REAR strip)
 │  Pi 5 + Active Cooler   85 × 56, top at ~56        │  56
 │  (front of trunk)       power mezzanine (rear):    │  40  mezzanine board tops
 │                         servo adapter 42 × 33,     │
 │                         INA226, buck, MAX98357A    │
 ├────────────────────────────────────────────────────┤  26  mid plate
 │  battery holders ×2     │ rear power strip:        │  24  holder tops
 │  78 deep × 80 across,   │ HW-391 + IP2326 + Wagos  │
 │  pushed left            │ (full width × 26 deep)   │
 └────────────────────────────────────────────────────┘  0   floor / bottom panel
   BNO055: screwed rigid to the mid plate, front-right corner (see below).
   Rocker switch: right side panel, low, intruding into the floor-level
   right channel (see §5.7).  Charge port: back panel, low centre (§4).
```

**Layer sums (the check that matters):**
- Floor layer: holders 21.4 + clearance = **24 mm**. Batteries are the lowest,
  heaviest thing in the robot (4 cells ≈ 188 g + holders 31 g ≈ **220 g at the
  very bottom, centred**) — centre of mass stays low, as planned.
- Mid plate at 26. Pi layer: 26 + ~28 (Pi + cooler) = **top at ~56**.
- Neck servo 30 hangs from the top panel: its body reaches down to 89 − 35 = **54**.
  That is 2 mm *below* the Pi stack's top (56) — **if the Pi sat at the trunk's
  centre, they would clash.** The fix is layout, not size: the Pi sits at the
  **front**, the servo hangs over the **rear strip**, where the mezzanine boards
  are only ~14 mm tall (tops at ~40) — 14 mm of clear air under the servo. The
  Pi's SoC/cooler must likewise stay out of the servo's footprint; with the Pi at
  the front, it does. (This is tight spot §5.3.)
- **Pi airflow path:** the Active Cooler pulls air down onto the SoC and pushes it
  out sideways. Path: intake slots low in both side panels → air rises past the
  mid-plate edges → cooler → exhaust slots high on the **front** panel and top
  edge of the back panel. Keep ≥ 10 mm of clear plenum above the cooler fan.
  (If the §7 mic array is fitted to the back, it covers the back exhaust — the
  front/top exhaust must then carry it alone; noted in §7.)
- **Power mezzanine (rear strip, Pi level):** Bus Servo Adapter (42 × 33), INA226
  (27 × 20), D36V50F5 buck (25.4 × 25.4), MAX98357A (19.4 × 17.8) — combined
  footprint ≈ a 100 × 40 mm zone; the rear strip offers 114 × 48. Fits with the
  servo-bus and power cables dressed along the strip's back edge.
- **Floor-level rear power strip (behind the holders, 114 × 26 × 24):** HW-391
  (48 × 20 × 4), IP2326 (32 × 18 × 6) mounted against the back panel next to the
  charge port, Wago 221 junctions for the battery/switch/buck runs. All the fat
  power wiring lives in this one strip — battery → BMS → switch → junctions →
  buck (5 V to Pi/K230/adapter) and raw 7.4 V to the servo adapter.
- **BNO055:** must be rigid, central-ish, and fixed in orientation. It screws to
  the **mid plate at the front-right corner** — rigid plate, ~50 mm from the neck
  servo's motor and clear of the buck's inductor on the mezzanine. (Magnetometer
  + nearby motors is a calibration nuisance, not a fit problem; keep the 50 mm.)
- **Battery access:** the report's earlier idea of a battery sled out of the
  **back** panel is **dropped** (the back panel is spoken for — §4/§5.2). Holders
  are fixed to the floor plate; access is by removing the **bottom panel**
  (6 screws, robot inverted on the bench). Cells are open-top drop-in, so this is
  a 2-minute job, and it is rare — charging is by the magnetic port, not by
  swapping cells.
- **microSD:** the Pi's card slot ends up facing the mid plate. Fit the card
  before the trunk is closed; it is not reachable afterwards without opening the
  bottom. Assembly-order note, not a fit failure.

**Legs & feet (brief, as asked):** each foot's sandwich has ~14 mm of vertical
room; the SS-10 is 10.2 mm tall — it fits with the plunger poking through the
sole plate, its tip ~1.5 mm proud of the hard sole line so it triggers before the
3 mm neoprene sole (report §2.11) has fully compressed. Operating force is
150 gf per switch, two per foot (heel + toe) = 300 gf to trigger both; a
~2.3 kg robot puts ~575 gf on each foot standing, ~1.15 kg in single support —
comfortable margin. Two thin wires per switch run up the leg's rear channel
alongside the servo bus cable and enter the trunk through the floor — no new
holes in the locked geometry, no surprises.

---

## 4. Charging port study — magnetic pair on the BACK of the trunk

**His preference:** the robot half of the pre-wired magnetic pogo pair on the back
of the trunk; a cable feeds the other half on a dock post; the halves touch and it
charges. No plugging anything in, ever.

**Verdict: WORKS, with conditions** (listed at the end of this section).

**Position & panel space.** Port centre on the back panel, low: ~30 mm above the
trunk floor, i.e. **~150 mm off the ground** (legs ~120 + 30). The pair's robot
half (~28 × 12 × ~8 mm, *est.* for the class) needs a ~30 × 15 mm recess — the
back panel offers 120 × 95 mm and, after the §5.2 clear-out (no battery door, no
USB-C slot), the low-centre zone is free. It sits **recessed, face flush** with
the panel: a printed pocket from inside, the half held by its flange screws (or a
printed clamp ring), so nothing protrudes to snag and the magnets meet face-to-face.

**Internal lead run.** The pair is pre-wired (2 tails). The IP2326 sits on the
back panel's inside face in the floor-level rear power strip (§3), ~40 mm from
the port — the tails run straight to the IP2326's input pads' pre-wired leads via
Wagos. Shortest possible run, no soldering, nothing crossing the trunk. The
IP2326's own USB-C port then faces the panel interior: the old plan for a
through-shell USB-C bench-charging slot is **dropped** (§5.2) — bench fallback is
USB-C into the IP2326 with the back/bottom panel off, which is fine for a
fallback you use twice a year.

**The parked pose — the one real geometry risk is the heels.** The robot charges
standing, backed against the post. Its big flat feet are centred under the legs,
so the heels reach roughly **20–30 mm behind the back panel's plane** *(est. from
the Duck foot proportions)*. If the dock were a flat wall, the heels would touch
it ~25 mm before the magnetic halves meet — no contact, no charge. The fix is in
the dock, not the robot: the post is **narrow (≤ 60 mm wide)** and carries the
cable-side half on a short arm that reaches **~35 mm forward of the post face**,
slipping between the two feet to meet the port. The feet straddle it; nothing
else on the robot's back (panel is flat, port flush) interferes. When the robot
sits down (Duck rest pose drops the trunk back and down), the port lifts/tilts
away from the post and contact breaks — so it **charges standing**, servos
holding pose. If power fades mid-charge it may slump and stop charging; the
INA226 sees charge current start/stop, so the software *knows* docking worked —
a free success signal, no extra sensor.

**The dock side.** A plain vertical post (printed or a small stand) with the
cable half on its arm at exactly **150 mm** height, fed by a 5 V supply. Two
supply details from the earlier decision stand: a USB-A brick (always-on 5 V)
works directly; a USB-C brick + cut cable will **not** switch on (no CC
handshake) — use USB-A, or a USB-C cable with the trigger handled. Supply rating
above 3 A is fine; voltage must be exactly 5 V. Magnetic self-alignment forgives
roughly ±5 mm *(est., class-typical)* of parking error.

**Self-docking later.** The K230's cameras face forward, so while reversing the
last stretch the robot is blind to the dock. The standard solution, noted not
solved: drive *toward* the dock facing it, centre on a marker (AprilTag/ArUco
card on the post) with the K230, stop at a measured standoff, turn 180°, then
reverse the final ~30 cm open-loop on the walking policy and let the magnets
pull the last few millimetres in. INA226 charge-current confirms success; if no
current, shuffle and retry. Nothing about the port position blocks this.

**Conditions for "works":**
1. Battery access moves to the bottom panel (done in §3) — the back panel is
   then free for the port.
2. The dock post is narrow (≤ 60 mm) with its half on a ~35 mm arm, at 150 mm
   height — heels cleared by design, not luck.
3. Charging pose is standing; a slump breaks contact (detectable via INA226).
4. The port and the §7 mic array cannot share the back panel — if the array is
   fitted, the port moves (see fallback).

**Fallback (if the back ever stops working — e.g. the mic array takes the
panel):** the **bottom face of the trunk** + a flat dock **plate on the floor**.
The robot half sits flush in the bottom panel; standing on the plate = charging.
No post, no heel problem, works even in the sit pose's footprint. It costs a
floor plate instead of a post and picks up floor dust on the contacts — which is
why the back stays first choice.

---

## 5. Surprises & tight spots — the adversarial list

Everything here is within ~5 mm, needs a specific part variant, contradicts an
earlier assumption, or would otherwise surface at CAD/assembly time. Fixes given
for each.

**5.1 The second camera is NOT proven — connector and sensor mismatch.**
The mini's standard camera is a GC2093 on a **24P** interface. 01Studio's own
docs describe the mini's expansion path as **one** extra camera on CSI0, and the
expansion camera they document/sell for it is the **OV5647 (22P-0.5 mm FPC)** —
a different sensor (5 MP vs 2 MP) on a different connector. A matched
GC2093 + GC2093 stereo pair is therefore **not guaranteed to work** on the mini.
Options, in order: (a) ask 01Studio directly whether a second GC2093 runs on the
mini's CSI0 — one email settles it; (b) run a matched **OV5647 pair** if the
board accepts one in the primary slot too; (c) accept a mismatched pair —
independent face/person detection per eye still works (that's what the K230 is
*for* in this design); only true depth-from-stereo calibration suffers. Also:
whichever sensor, buy the **15 cm** FPC — the 6 cm cable cannot reach the eye
positions with bend allowance. **Do not order the second camera until (a) is
answered.**

**5.2 The back panel cannot hold everything that was promised to it.**
Four claims collided on one 120 × 95 mm panel: the assembly report's battery
sled door (§1.2 of that report), its USB-C charging slot (§2.8 there), the
magnetic port (§4), and now the §7 mic array (88.8 mm tall — nearly the whole
panel height by itself). Resolution adopted in this study: battery door →
**bottom panel**; USB-C through-slot → **dropped** (bench fallback with a panel
off); back panel carries the **port**, or the **array**, not both (§7).

**5.3 Neck servo 30 vs Pi + cooler: 2 mm overlap on paper.**
Servo 30's body hangs to 54 mm; a centred Pi + cooler tops out at ~56 mm. Fixed
by layout (Pi front, servo over the low rear mezzanine) — §3. The CAD must keep
the cooler's fan inside the Pi's front zone; if the Pi ever moves back, the
clash returns.

**5.4 The 250 g head budget counts contents, not what the neck lifts.**
Contents ≈ 220 g ✓, but ~330 g sits above the head-pitch axis once servos 32/33
are counted (§2). No action needed — torque margin is large — but nobody should
quote "the head is 220 g" to the walking-policy work later. Shell weight is the
swing item (±30 g): weigh it when printed.

**5.5 Lidar: the big unit fits nowhere; the small unit fits only under the trunk.**
Full verdicts in §7. Headlines: an RPLIDAR A1-class unit (96.8 × 70.3 × 55 mm,
170 g) fits in **no** position on this robot. An LD06/LD19-class unit fits
**inside** the trunk only by growing the trunk ~30 mm (breaks the locked
height); it fits **as locked** only in a belly pod under the trunk. If the lidar
he owns turns out to be A1-sized, the honest answer is: not on this robot.

**5.6 Mic array and charge port are mutually exclusive on the back.**
Array 88.8 mm tall + port ~15 mm > 95 mm panel. Both-on-the-back is arithmetically
impossible at the locked trunk height (§7). Choosing the array moves the port to
the bottom face + floor plate (§4 fallback).

**5.7 Rocker switch intrusion: ~4–6 mm clearance.**
A KCD1 body is ~25–28 mm deep behind the panel with terminals. In the right
side panel at floor level it intrudes into the 34 mm side channel beside the
battery holder, leaving ~4–6 mm to the holder. Fits — with **right-angle push-on
terminals**, and the switch placed at the channel's front end where the holder's
wiring doesn't run. If his switch turns out to be a deeper illuminated variant,
it moves to the front-bottom edge of the side panel instead.

**5.8 IP2326 variant trap.**
The common module is 32 × 18 mm; a PD variant at **50 × 30 mm** also exists and
would **not** fit the rear power strip (26 mm deep). Buy/list the small one
explicitly.

**5.9 Pogo pair pin count = charge speed.**
Fit is identical, but the 2-pin version of these pairs carries ~2 A (slow
charge, ~5–6 h) vs the 4-pin ~4 A (full IP2326 speed, ~3 h) *(class-typical
figures)*. Which one he actually bought is still unconfirmed — worth one look at
the order before the dock is designed around a charge-time assumption.

**5.10 Neck cable channel must be reserved in CAD.**
~10 wires (Ø ~8–10 mm bundle) cross the neck (§2). The Duck neck mechanism
wasn't designed for a bundle this size alongside its bearing; the head/trunk CAD
must keep a dedicated channel clear of the roll axis. Not a blocker — a
"remember it now, not at assembly" item.

**5.11 K230 in a sealed head will cook without its heatsink + vents.**
§2 covers it: heatsink mandatory, crown vents, fan accessory space reserved.
Cheap insurance, zero proportion cost.

**5.12 INA226 height depends on his exact unit.**
Bare-header variants are ~8 mm; terminal-block variants reach ~14 mm. Both fit
the mezzanine (tops ~40 mm either way), but the CAD should use 14 mm until his
unit is measured. Same story for the speaker (§1): the layout holds for the
31 × 28 class; if his voice-set speaker is the bigger 30 × 70 enclosed type it
goes to the trunk front instead, behind a belly grille — the mouth slot then
becomes purely decorative. One measurement on arrival settles both.

---

## 6. Verdict summary

**The base robot FITS as locked — with the listed changes, none of which moves a
locked proportion:**

1. Battery access moves from the back panel to the **bottom panel** (§5.2).
2. The through-shell USB-C charging slot is **dropped**; bench charging is
   panel-off (§4).
3. The Pi sits at the **front** of the trunk, the neck servo over the low rear
   mezzanine (§5.3).
4. Cameras use **15 cm FPCs**, and the second camera is **not ordered** until
   01Studio confirms what the mini's expansion CSI accepts (§5.1).
5. Small-variant parts specified where a big sibling exists: IP2326 32 × 18 mm
   (§5.8), right-angle terminals on the rocker switch (§5.7).
6. K230 gets its heatsink + crown vents (§5.11).

**Head:** contents ≈ 220 g vs the 250 g budget ✓ (~330 g above the pitch axis
with servos — fine, but quoted correctly from now on). **Trunk:** the stack
sums to 89 mm of 89 mm used *by zone*, with every overlap resolved by layout,
not by growth. **Charging port on the back: works, with the §4 conditions**
(narrow dock post on a 35 mm arm at 150 mm; charges standing).

**Optional extras change the answer:**
- **Lidar:** fits as locked **only** as an LD06/LD19-class unit in a belly pod
  (§7). In-trunk costs +30 mm of trunk height (~38 cm total); on-head costs
  +33 mm and the head budget. An A1-class unit fits nowhere.
- **Sipeed mic array on the back:** fits the panel **only if the charge port
  moves** to the bottom face + floor dock plate (§7). Both-on-the-back is
  impossible at 95 mm of trunk height; making room for both means trunk
  +~15 mm (~36.5 cm total).

Nothing in the base build forces a proportion change. The two "measure on
arrival" items (speaker, INA226 variant) and one "ask the vendor" item (second
camera) are the only open unknowns, and each has a fallback that fits.

---

## 7. Lidar & mic array

*Flagged up front, as asked: the lidar verdict depends on which unit he actually
owns. The settled spec class from earlier planning is "UART, 5 V, 2D" — that
describes both the huge and the tiny units below, and they get opposite answers.*

### 7.1 Lidar — three candidate units

| Unit | Size (mm) | Weight | Interface |
|---|---|---|---|
| Slamtec RPLIDAR A1 | 96.8 × 70.3 × 55 | 170 g | UART (via USB adapter) |
| LDROBOT LD19 / D300 | 38 × 38 × 34.3 (54 × 46.3 with kit plate) | 45–47 g | UART 230400 |
| LDROBOT LD06 | 38.6 × 38.6 × 33.3 | 42 g | UART 230400 |

**Position A — inside the trunk.** A lidar needs an unobstructed 360° view at its
scan height: in a closed trunk that means a **window band** — the shell becomes
an upper and lower half joined by pillars, with a clear ~20 mm band at the
lidar's eye line. The LD06/LD19 footprint (38.6²) fits the trunk's floor plan
easily; the problem is **height**. The trunk's layers are already fully spoken
for (§3): a dedicated lidar layer with nothing else above the scan line costs
~30 mm the 89 mm interior does not have — the Pi layer and the neck-servo
overhang collide (§5.3's arithmetic gets worse, not better). Sharing the Pi's
layer instead (lidar front, Pi rear) blocks ~120° of the scan with the Pi itself
and puts the Pi's cooler under the servo — not a real 360° lidar and a thermal
clash. The A1 is worse: at 96.8 mm wide it eats the whole layer, and at 55 mm
tall most of the trunk. **Verdict: doesn't fit as locked.** It fits only with
the trunk grown ~30 mm → total height ~38 cm.

**Position B — on top of the head.** Mechanically easy (a 38.6 mm puck between
the ears; ear-servo sweep clears it), and the scan view is perfect. The costs
are exactly where this robot can least afford them: **+33 mm height → ~38.3 cm
total** (locked ~35 cm broken), and **+42–47 g at the very top** — head contents
220 + 42 = **~262 g, over the 250 g budget**, all of it raising the centre of
mass and the neck servos' load. For the A1 (170 g, 55 mm tall) it isn't even a
conversation. **Verdict: doesn't work within the locked proportions.**

**Position C — belly pod under the trunk (the third position worth naming).**
The LD06/LD19 (kit plate removed) hangs in a small printed pod under the trunk
floor, scan plane **~100 mm off the ground**, between the legs. Costs: **+42–47 g
at the lowest, most central point of the robot** (the best place mass can go),
**+0 mm height**, one printed pod. Compromises, honestly: the four legs block
narrow blind wedges (~15–20° each) in the scan; at 100 mm it sees furniture
legs and skirting, not tabletop overhangs; it does not replace the cliff
sensors (already planned — a 2D lidar can't see a stair edge below its plane);
and the sit pose needs a clearance check so the pod doesn't strike the floor
when the trunk drops. **Verdict: works as locked, with those compromises.**

*(Fourth position, one line: a chest band on the trunk's front face gives only
a forward ~180–240° scan past the arms/belly — a proximity sensor's job, not a
mapping lidar's.)*

**Recommendation:** if his unit is **LD06/LD19 class → belly pod (Position C)**,
cost 42–47 g, 0 mm, one pod, blind wedges at the legs. If it's an **RPLIDAR A1
→ it does not go on this robot** in any position; leave lidar off the panda (or
use the A1 on a fixed base station for room mapping instead). Confirming which
unit he owns is a one-line question that flips this whole section.

### 7.2 Sipeed R6+1 mic array on the trunk back panel

The board is **78.1 × 88.8 mm** (Sipeed wiki), ~38 g gross. The back panel is
120 × 95 mm outer, ~114 × 89 usable inside the frame.

**Size vs panel:** width is fine (78.1 vs 114 — ~18 mm spare each side).
**Height is the problem: 88.8 mm of array on 89 mm of usable panel — it covers
the back essentially edge to edge.** Mounted proud on standoffs (the practical
way — the 7 mics and LED ring face outward, no drilling a 7-hole pattern into
the panel), it becomes a "backpack" plate standing ~5 mm off the surface.

**Coexistence with the back charge port: impossible as locked.** Array (88.8) +
port zone (~15) = ~104 mm > 95 mm panel. There is no "port low, array above"
arrangement — the array alone is the whole panel height. The pair of options:

```
BACK PANEL — option 1: PORT only          BACK PANEL — option 2: ARRAY fitted
┌────────────────────────┐ 120 wide       ┌────────────────────────┐
│                        │                │ ┌────────────────────┐ │
│      (free — vents     │  95 tall       │ │  Sipeed R6+1       │ │
│       top edge only)   │                │ │  78.1 × 88.8       │ │
│                        │                │ │  (proud ~5 mm)     │ │
│      ┌──────────┐      │                │ └────────────────────┘ │
│      │ POGO PORT│      │                │  port → BOTTOM face +  │
│      └──────────┘      │                │  floor dock plate (§4) │
└────────────────────────┘                └────────────────────────┘
```

**Cable run:** the array connects by a 10-way ribbon (2×5 IDC) or 10P FPC. From
the back panel to the Pi's header is ~15 cm inside the trunk — an easy run
through a slot in the panel's top edge. One non-space flag: the array's mics are
I2S devices and the Pi's single I2S bus already drives the MAX98357A speaker
amp — sharing that bus is an electrical/software question for build time, not a
fit problem. Also: fitted proud on the back, the array **covers the back
exhaust vents** (§3) — exhaust must then go front/top only.

**The acoustic reality, one honest line:** on the back, the array hears the room
*behind* the robot best — someone talking to its face is off-axis and shell-
muffled, so face-to-face wake-word reliability will be noticeably worse than a
front or ear position.

**Verdict:** the array **fits the back panel physically, but only by evicting
the charge port** to the bottom face + floor dock plate (§4 fallback — which,
to be fair, also deletes the heel-strike condition entirely). It does **not**
cancel the parked mics-in-ears idea; it's an either/or with the back port, and
acoustically it's the weakest of the three mic positions. If he wants both the
array *and* a back port, the price is trunk +~15 mm (~36.5 cm total) — his call,
not recommended.
