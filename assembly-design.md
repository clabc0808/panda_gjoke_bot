# Legged Panda — Assembly & Wiring Design Report (2026-11-16)

> **Updates since this report was written (2026-11-15):**
> - §4.1 bearings: fixed — 3 × 6804ZZ bought (cart's 608ZZs removed).
> - §5/§6 eyes: decided — **K230 with two small cameras** (stereo pair at the panda
>   eyes), superseding the single-camera recommendation in §5. See README.
> - Charging: inlet changed from plug-in USB-C to the **magnetic pogo pair**
>   (dock half on a 5 V supply, robot half feeding the IP2326 inside the robot);
>   hand-parking first, self-docking later. See README / shopping list.

Overnight design work requested 2026-11-15. This report designs how the legged panda
goes together, maps every wire, budgets the weight, lists everything that is still
missing, and evaluates the Kendryte K230 as the robot's eye.

**Status of parts:** the cart has NOT been paid for yet. One item in the cart is wrong
(see §4.1 — bearings) and can still be fixed before checkout at zero cost.

Sources are named inline. Figures marked *(est.)* are my estimates, not measurements.
Main sources: the Open Duck Mini v2 repository (github.com/apirrone/Open_Duck_Mini,
branch v2 — assembly guide, print guide, BOM spreadsheet) as mirrored on
holocraft.dev and robotdiy.net; Feetech STS3215-C001 datasheet figures via OpenELAB;
Kendryte CanMV K230 documentation (kendryte.com); Yahboom K230 manual;
01Studio CanMV K230 wiki; TI INA226 documentation.

---

## 1. Assembly design (mechanical)

### 1.1 The skeleton we inherit

Open Duck Mini v2 is a 42 cm biped made of **51 printed pieces** (36 distinct STL
files) around 14 STS3215 servos. Its structure splits into five modules, and the
panda keeps this split exactly:

| Module | Duck printed parts | What it does |
|---|---|---|
| Feet ×2 | foot_top, foot_side, foot_bottom_pla, foot_bottom_tpu (sole) | Ankle servo mount, switch pockets, sole sandwich |
| Legs ×2 | knee_to_ankle sheets ×4 per leg, leg_spacer ×2 per leg | Shin and thigh links around knee/hip-pitch servos |
| Hips ×2 | left/right_roll_to_pitch, roll_motor_bottom, roll_motor_top | Hip roll + hip yaw cluster, bolts to trunk |
| Trunk | trunk_bottom, trunk_top, body_middle_bottom, body_middle_top, body_front, body_back, battery_pack_lid | The chassis: hip bearings, battery, electronics, body shells |
| Neck + head | neck_left/right_sheet, head_pitch_to_yaw, head_yaw_to_roll, head_roll_mount, head_bot_sheet, head shell, antenna holders, caches (ear covers) | 4-servo neck (IDs 30–33) carrying the head |

Leg kinematics per leg (5 servos): **hip yaw → hip roll → hip pitch → knee pitch →
ankle pitch**. The two hip-roll axes are supported by ball bearings pressed into
`trunk_bottom`; a third bearing supports the head-roll axis inside the head
(assembly guide: bearings go in trunk_bottom first, and "insert the bearing" again
in the head step — 3 bearings total, see §4.1).

### 1.2 What carries over unchanged, what must be redesigned

**Carry over from the Duck, print as-is (no CAD changes):**

- All leg parts: knee_to_ankle sheets, leg_spacers, foot_top, foot_side,
  foot_bottom_pla. The panda's legs *are* Duck legs — same servos, same geometry,
  so the walking-policy model stays as close to upstream as possible.
- Hip parts: roll_to_pitch (L/R), roll_motor_bottom, roll_motor_top.
- The whole neck mechanism: neck sheets, head_pitch_to_yaw, head_yaw_to_roll,
  head_roll_mount, head_bot_sheet. (A third-party project, OpenHead, already reuses
  this neck unchanged with a completely different head on top — the neck is a
  self-contained unit with a flat mounting face for whatever head you bolt on.)

**Redesign for the panda:**

1. **Trunk — full redesign, this is the big CAD job.** The Duck spreads its
   electronics out: its Pi Zero 2 W lives *inside the head*, the servo adapter and
   a slim 2-cell (2S1P) battery sit in the body. The panda puts everything in the
   trunk instead:
   - Raspberry Pi 5: 85 × 56 mm, plus ~11 mm of Active Cooler on top — roughly
     3× the footprint and 7× the weight of the Pi Zero it replaces.
   - 2S2P battery: two 2-cell holders side by side ≈ 76 × 80 × 21 mm, double the
     Duck's pack.
   - Waveshare Bus Servo Adapter (A), Pololu buck, HW-391 BMS, INA226 module,
     IP2326 charger board, main switch, USB-C charge port.
   - Working internal volume needed: roughly 100 × 90 × 75 mm *(est.)*, so the
     panda trunk lands around 120 × 110 × 95 mm outside *(est.)* — noticeably
     bigger than the Duck's trunk. The good news: this volume is exactly the
     panda's round belly from the existing v3 body design language (white belly,
     black band over the top), so the trunk shells should be drawn as the panda
     body, not as a Duck body scaled up. `trunk_bottom`'s bearing seats and hip
     interfaces must keep the Duck's exact geometry where the hips bolt on — the
     redesign is everything *above and around* those interfaces.
2. **Head shell — adapt the existing panda head CAD (v3.5/v3.6).** The 104 mm
   panda head already designed for this goal (one-piece muzzle swell, camera hole
   as the "nose", eye-pit inserts, half-disc ears scooped to the head curve) is
   the right shell. Changes needed:
   - The internal base plate/riser was designed for a Pi-camera-class board on a
     post. It must be re-drawn to carry whichever eye wins (§5) at the nose hole,
     at the right depth behind the swell (the v3.5 geometry already verified
     lens clearances for a board behind the muzzle).
   - The v3.6 ears are *static* prints. The panda's ears move (MG90S servos, the
     Duck's "antenna" servos). Each ear needs a root pocket for a standard micro
     servo horn, and the head needs two MG90S mounting pockets at the ear roots,
     mirroring how the Duck's head carries its two antenna servos.
   - The head no longer carries a computer (the Duck's head carries the Pi Zero),
     so the panda head is *lighter* than a Duck head despite being bigger —
     good news for the neck servos (§1.3).
3. **Battery bay / lid — redesign.** `battery_pack_lid` becomes a 2S2P bay cover
   sized for the two holders, with the pack sitting on the trunk floor (see §1.3).
   Cells must be removable without dismantling the trunk: holders screwed to a
   sled that slides or lifts out through the back panel.
4. **Sole — same sandwich, different material.** The Duck clamps a TPU
   `foot_bottom_tpu` between `foot_bottom_pla` and the foot with two M3×6 screws
   per foot. TPU filament was ruled out; a die-cut sheet of soft rubber drops
   into the identical sandwich and is clamped mechanically — no glue, no new
   parts (§4.5).

Everything prints on the A1 mini (180 × 180 × 180 mm): the largest Duck parts and
the 104 mm head shell all fit with room to spare. Print settings from the Duck
print guide: PLA at 15% infill for structure; the panda's big cosmetic shells can
go lighter (§3.3).

### 1.3 Where each electronic item lives, and centre of mass

A walking biped's balance policy is trained against a specific mass distribution.
Ours will be retrained for the panda regardless (settled plan), but the closer the
heavy parts sit to where the Duck keeps them — **low, central, and symmetric** —
the easier that training is and the more forgiving the robot. One community note
in the Duck ecosystem makes the point bluntly: adding shell covers shifted the
centre of gravity and the stock policy wobbled until retrained (holocraft.dev
project comments). Layout, bottom to top:

- **Trunk floor (lowest, between the hip axes): the battery.** 4 cells + holders
  ≈ 215 g *(est.)* — the single heaviest removable item after the servo set. It
  sits on the trunk floor, centred left-right, its centre as close to the
  hip-yaw axis line as packaging allows. Low + central = the pendulum the policy
  has to swing is as short as possible.
- **Just above the battery: the Pi 5** (46 g + 28 g cooler = 74 g), flat, GPIO
  header facing the rear service opening. In the Duck this computer sits in the
  head; moving 74 g from the head (~35 cm up) to the trunk (~20 cm up) is the
  single biggest centre-of-mass *improvement* the panda makes over the Duck —
  it partly pays for the doubled battery.
- **Trunk walls / mid layer:** Waveshare servo adapter mounted centrally (servo
  cables radiate from it down both legs and up the neck — shortest total cable
  run), Pololu buck beside the Pi (short, thick 5 V feed), INA226 in the main
  positive line near the BMS, BMS itself taped/bolted to the battery sled,
  IP2326 charger board at the back panel with its USB-C port poking through a
  slot in the shell — charging without opening the robot.
- **BNO055 IMU: bolted rigidly to the trunk structure**, as central as possible,
  away from the Pi's fan vibration, in a known orientation (the Duck runtime
  expects it in the trunk and lets you declare the mounting orientation in
  config; the Duck assembly guide even warns its own photos show it upside
  down). Not in the head — head IMUs measure head wobble, not body tilt.
- **Head (keep it light, target ≤ 250 g all-in *(est.)*):** 2 × MG90S ear servos
  (~27 g), the eye board (§5: 16–65 g depending on choice), printed shell
  (~120–180 g *(est.)*), eye inserts and ears (~20 g). The 4 neck STS3215s
  (220 g) are trunk/neck mass, not head mass. For comparison the Duck's head
  carries a Pi Zero + camera + ear servos; the panda head is in the same class
  or lighter, so the neck servos are not being asked to do anything heroic.
- **Speaker:** in the trunk front (the Duck has a speaker_interface/stand print
  for the body; the panda equivalent is a grille area in the belly shell),
  pointing forward. MAX98357A board beside it. A speaker in the head would add
  top weight and buzz against the shell; trunk is better and it's where the
  Duck puts it too.
- **Microphone:** not yet placed — see §4.8.

Legs are pure Duck: 5 servos per leg ≈ 275 g of servo per leg plus prints. That
mass is unavoidable and identical to the Duck, which is precisely why the legs
carry over unchanged.

**Height note:** legs identical, trunk taller, head ~100 mm → the panda stands
roughly 45–48 cm *(est.)* vs the Duck's 42 cm. The MuJoCo model (MJCF) must be
built from the panda's real dimensions and weighed masses before policy training;
that was already the settled plan.

### 1.4 Fasteners, inserts, bearings — how they're used

- **Heat-set inserts (M3×5×5, pack of 50 in cart):** pressed/melted into printed
  parts wherever a screw will be removed more than once — trunk bearing seats
  and body mounts (4 in trunk_bottom alone), leg_spacers (4 each), foot parts,
  body_middle parts, head. The Duck's guide installs inserts at almost every
  step. Installation needs a heat tool — see §4.7.
- **Servo screws:** the STS3215s come with their own screws (plastic self-tappers
  for printed parts, horn screws) — the assembly guide uses "the plastic screws
  that came with the motors" for most servo mounts. Our M3 assortment is for
  part-to-part joints (trunk 2 × M3×10, feet 2 × M3×6 per foot, body shells).
- **Loctite 243 (blue):** on every metal-to-metal screw (horn screws, servo
  mounting into metal), never on screws into plastic — straight from the Duck
  assembly guide. Still pending purchase (§4).
- **Bearings:** 3 total — 2 in trunk_bottom supporting the hip-roll axes, 1 in
  the head supporting the head-roll axis. **Size problem — see §4.1 before
  paying for the cart.**

### 1.5 Assembly sequence (adapted from the Duck assembly guide)

Do these in order; the order matters because later steps close access to earlier
ones.

0. **Bench-configure all 14 STS3215s first** (upstream is explicit: configure
   before building). One at a time on the Waveshare adapter: set IDs — right leg
   10–14, left leg 20–24, neck/head 30–33 — and set each servo's centre/zero.
   Label every servo with its ID on a sticker. Also bench-test the MG90S pair,
   BNO055, INA226 and the whole power chain on the bench at this stage, not
   inside the robot.
1. **Print everything**, install all heat-set inserts while parts are loose
   (much easier than in-situ), press the 2 bearings into trunk_bottom.
2. **Trunk core:** trunk_bottom + trunk_top screwed together (2 × M3×10);
   mount the hip-yaw ("middle") motors and roll_motor_bottom parts.
3. **Feet ×2:** sole sandwich (foot_bottom_pla + rubber sole, 2 × M3×6),
   foot_top, ankle servo (driver side facing foot_top), press-fit the 2 SS-10
   switches per foot so the plunger is depressed exactly when the foot is flat
   on the ground.
4. **Shins:** leg_spacer inserts (4 each), plug the ankle servo's cable in
   *before* closing the leg, route it through the knee_to_ankle sheet, mount
   knee servo and sheets.
5. **Thighs:** hip-pitch servo — orientation matters for the zero position
   (the guide calls this out specifically); close the thigh sheets.
6. **Hips:** roll_to_pitch (correct left/right — they are mirror parts),
   roll_motor_top onto the hip-yaw servo, hip-roll servo, then mount the
   finished leg to the trunk. Repeat for the other leg. The skeleton now stands.
7. **Neck:** neck_left/right sheets + neck-pitch servo (ID 30) onto the trunk top.
8. **Head mechanism:** head_pitch_to_yaw (31), then head_yaw_to_roll (32) +
   head_roll_mount on the head-roll servo (33), with the head bearing. Fit
   head_bot_sheet and the trunk's body_middle_top now — the guide warns that
   skipping this forces a head strip-down later.
9. **Electronics fit-out (trunk):** battery sled + BMS, switch, INA226, charger
   board, servo adapter, Pololu buck, Pi 5 + cooler, BNO055, speaker + amp.
   Wire everything per §2 with the trunk open; power-on test on the bench
   supply of patience: servos centred, IMU reading, switches clicking.
10. **Head build:** eye board on its riser/base plate at the nose hole, ear
    servos + ears, eye inserts; route the head cable bundle (ear leads, eye
    power/UART) down through the neck with a slack loop; close the shell.
11. **Body shells:** body_middle_bottom (inserts first), body_middle_top,
    body_front/back (panda belly + band), battery lid. Final weigh-in and
    centre-of-mass check (balance the robot on a rod under the trunk, fore-aft
    and left-right) — record both for the MJCF.

---

## 2. Wiring & power design

### 2.1 The whole robot on one page

```
 CELLS (2S2P, in 2 holders, 3 wires out: + / MID / −)
   │  + ──────────────────────────────┐
   │  MID ────────────────┐           │
   │  − ────────────┐     │           │
   ▼                ▼     ▼           ▼
 HW-391 BMS      B+     BM          B−        (B+ and P+ are the same node on this board)
   │  P+ (pack + out)        P− (pack − out, the switched leg)
   │
   ├─► INA226 shunt (sits in the P+ line, sees charge AND discharge current)
   │
   ├─► IP2326 CHARGER tee:  + → P+ node,  − → P− node,  MID → BM node
   │      (USB-C port in the back panel; charging keeps the BMS in the loop)
   │
   └─► MAIN SWITCH (rocker, in the + line)
         │
         ├──► WAVESHARE BUS SERVO ADAPTER (A)  +/−  → servo bus power (raw 7.4 V)
         │         └── TTL bus daisy chain → 14 × STS3215 (§2.2)
         │         └── USB → Pi 5 (data)
         │
         └──► POLOLU D36V50F5 BUCK → clean 5 V rail
                   ├──► Pi 5 (USB-C in, short thick lead)
                   ├──► MG90S ear servos ×2 (+/−; signals from Pi GPIO 12/13)
                   ├──► MAX98357A amp VIN (speaker)
                   └──► Eye board, if K230 (§5) — 5 V, up to 0.8 A

 Pi 5 GPIO header (all low-voltage signals):
   GPIO 2/3   I2C ──► BNO055 (0x28) + INA226 (0x40)     (§2.3)
   GPIO 18/19/21  I2S ──► MAX98357A                     (§2.4)
   GPIO 12/13 PWM ──► MG90S ear signals                 (§2.5)
   GPIO 22    ◄── left foot switches (2 × SS-10 in parallel)
   GPIO 27    ◄── right foot switches (2 × SS-10 in parallel)
   GPIO 14/15 UART ──► eye board, if K230 (§5.3)
```

All grounds are one common ground (pack negative). Every join is a plug, a Wago
221 lever connector, or a crimp — no soldering anywhere (settled constraint;
boards that needed solder pads were ordered pre-wired / seller-soldered).

### 2.2 Power chain — numbers

- Pack: 7.4 V nominal, 8.4 V full, ~6.0 V empty, ~44 Wh.
- Servo load: STS3215 rated current 0.65 A at 7.4 V, **stall 2.5 A each**
  (Feetech datasheet via OpenELAB). Normal walking draws a few amps total;
  a fall or a stalled joint can spike much higher. The HW-391 (20 A), XT30
  pairs (15 A continuous) and the switch (20 A DC) all have headroom over
  realistic peaks.
- **Wire gauges (not on the shopping list as a spec — see §4.6):** pack → BMS →
  switch → adapter/buck runs in **18 AWG silicone** wire (short runs, ~16 A
  chassis rating — comfortable). The pre-wired XT30 pigtails already on order
  are this class. Buck → Pi feed: **20 AWG or thicker, as short as possible** —
  the Pi 5 is fussy about voltage drop, and 5.0 V from the Pololu leaves less
  margin than the official 5.1 V supply, so the lead must not waste any of it.
  Signal wiring (switches, I2C, PWM) is the Dupont/spiral-wrap stock already
  in the cart.
- The servo bus itself is powered from the adapter's power input; the Feetech
  3-pin leads (one 15 cm lead ships with each servo) carry power + signal down
  the chain. Keep the adapter central so no chain run is long: voltage sag
  along a daisy chain shows up first at the ankle servos.

### 2.3 Servo bus — chain order and IDs

STS3215s are daisy-chained: each servo has two 3-pin ports, cable hops
servo→servo. IDs are set before assembly (§1.5 step 0), following upstream
`docs/configure_motors.md` exactly:

| Chain | Order (from the adapter) | IDs |
|---|---|---|
| Right leg | hip yaw → hip roll → hip pitch → knee → ankle | 10, 11, 12, 13, 14 |
| Left leg | hip yaw → hip roll → hip pitch → knee → ankle | 20, 21, 22, 23, 24 |
| Neck/head | neck pitch → head pitch → head yaw → head roll | 30, 31, 32, 33 |

(ID-to-joint order within a leg follows the upstream configuration doc; the
runtime expects exactly this numbering.) Two practical rules from the Duck
guide: plug each servo's cable in *before* closing the printed sheets around
it, and route cables through the leg sheets as you build — they cannot be
threaded afterwards. The adapter's mode jumper must be set for USB control
(the adapter (A) has a USB/UART mode jumper; the Duck ecosystem runs it in
USB mode from the Pi).

### 2.4 I2C — BNO055 + INA226: no address conflict

Both sensors share the Pi's I2C bus 1 (GPIO 2 = SDA, GPIO 3 = SCL), exactly as
the Duck wires its BNO055:

| Device | Address | Power | Notes |
|---|---|---|---|
| BNO055 (Adafruit 4646) | **0x28** (0x29 if its ADR pin is pulled high — leave it at 0x28) | 3.3 V from Pi pin 1 | Upstream runtime reads it as-is |
| INA226 module | **0x40** default (jumper-selectable 0x40–0x4F — leave at 0x40) | 3.3–5 V | TI datasheet default; no clash with 0x28 |

**Verdict: no conflict.** 0x28 ≠ 0x40, and nothing else in the robot uses I2C
(the servo adapter is USB, the amp is I2S, the eye option in §5 is UART).
Two known BNO055-on-Pi quirks, both already handled by the upstream runtime
(it runs this exact IMU on a Pi): the BNO055 stretches the I2C clock, which
old Pi I2C hardware disliked — keep I2C leads short (< 30 cm, trivially true
inside the trunk) and use the runtime's existing driver settings. The
STEMMA QT cable ordered with the 4646 plugs straight into the Pi header via
its jumper ends — solderless, as specified.

### 2.5 I2S audio (voice out)

MAX98357A on the Pi's I2S pins — the Duck's own pin table transfers 1:1 to
the Pi 5 (same GPIO numbering):

- BCLK → GPIO 18, LRC/WS → GPIO 19, DIN → GPIO 21
- VIN → 5 V rail (from the buck, not a GPIO pin), GND → common ground
- Speaker (4 Ω 3 W) on the amp's screw terminals — the ¥35.9 set already
  includes the PH2.0-to-Dupont lead.

One overlay line in the Pi's config (`dtoverlay=max98357a`) and it appears as
a standard ALSA output for Piper/Hermes.

### 2.6 Ear servos (MG90S) — the PWM question, answered

The Waveshare adapter (A) speaks **only** the Feetech TTL bus protocol. It has
no PWM outputs, so it cannot drive the MG90S ears — this was the open question
in the brief. **Nothing extra is needed, because upstream already solved it
the simple way:** the Duck drives its two antenna MG90S servos **straight from
the Pi, PWM on GPIO 12 and GPIO 13** (assembly guide pin table: left antenna
GPIO 12, right GPIO 13, power from 5 V, common ground). GPIO 12/13 are the
Pi's hardware PWM channels, so the pulses stay clean even while Linux is busy
with the 50 Hz policy loop — and ears are non-critical anyway (a jittery ear
never made a robot fall over).

Wiring for the panda:

- Signal: left ear → GPIO 12, right ear → GPIO 13 (Dupont leads, run up the
  neck with the head bundle).
- Power: from the **buck's 5 V rail** via Wago/crimp taps — *not* from the
  Pi's header 5 V pins. An MG90S can pull ~0.7 A stalled; two ears moving
  together shouldn't share the Pi's own supply pins. Signals are 3.3 V logic,
  which MG90S accepts.
- On the Pi 5, hardware PWM on those pins is driven via the RP1 PWM +
  a small Python/C control path (upstream runtime's antenna code is the
  reference; it needs its Pi-Zero-era PWM helper swapped for the Pi 5
  equivalent — a software task, not a parts task).

### 2.7 Foot switches

Per the Duck design: the two SS-10s in each foot are wired **in parallel**
between the GPIO pin and ground — either switch closing reads as "foot down".
Left foot → GPIO 22, right foot → GPIO 27, Pi internal pull-ups on, other side
of both switches → ground. The switch wires run up the inside of each leg
alongside the servo chain (fit them while the leg is open, §1.5 step 3–4).
Use the SS-10T quick-connect variant's crimp terminals or Dupont ends — no
solder (settled when the switches were chosen).

### 2.8 Charger access

IP2326 board bolted inside the rear trunk panel, USB-C socket through a slot
in the shell. Its three battery wires go to the pack nodes as drawn in §2.1:
+/− tee'd at the BMS P+/P− (so charging current still passes the BMS's
protection and the INA226 shunt — the robot can report its own charge
current), middle wire to the pack midpoint (the same node as the BMS's BM
wire). Charging is robot-off or robot-on; v1 has no dock (settled).

### 2.9 Electrically unresolved — flagged honestly

1. **5 V rail budget.** The Pololu is rated 5.5 A. Paper worst cases: Pi 5
   (spec 5 A, real-world 2–3 A in this workload) + K230 eye (0.8 A) + 2 ×
   MG90S stalls (1.4 A) + amp peaks (~0.5 A) can exceed 5.5 A *on paper*.
   In practice these peaks never coincide, and the Pi's real draw is well
   under spec — but if brown-out resets ever appear, the fix is a second
   small 5 V buck for the peripherals (ears + amp + eye), leaving the
   Pololu for the Pi alone. No action now; watch item.
2. **USB cable ends.** The cart's "short USB cable (Pi ↔ adapter)" must be
   USB-A (Pi 5 side) to whatever the adapter (A)'s USB port is (USB-C on
   current Waveshare stock). Verify the ends when the adapter arrives; a
   wrong-end cable is a £2 fix but a build-day stall.
3. **Adapter mode jumper** must be in USB mode before first power (§2.3).
4. **Cell matching:** the 4 cells must be at the same voltage before they go
   into the holders (upstream safety note, repeated in the list). That needs
   a multimeter — §4.7.

---

## 3. Weight budget

Reference point: the Open Duck Mini v2 itself totals **2107 g** in its MuJoCo
model (from the earlier actuator-selection research). The panda swaps: Pi Zero
2 W (~11 g, in the head) → Pi 5 + cooler (74 g, in the trunk); 2 cells → 4
cells (+~95 g); Duck body/head shells → bigger panda shells; plus INA226 and
the uprated power boards.

### 3.1 Itemised estimate

| Item | Qty | Each | Total | Source / basis |
|---|---|---|---|---|
| STS3215-C001 servos | 14 | 55 g | **770 g** | Feetech datasheet (55 ± 1 g) |
| Servo horns + screws + cables (the parts of each accessory bag actually fitted) | 14 | ~10 g *(est.)* | **140 g** *(est.)* | Bag contents per retailer listings |
| MG90S ear servos | 2 | 13.4 g | **27 g** | Standard MG90S spec |
| Raspberry Pi 5 8GB | 1 | 46 g | **46 g** | Retailer spec (Raspberry Pi doesn't publish weight) |
| Pi 5 Active Cooler | 1 | 28 g | **28 g** | Botland spec sheet — **not yet on the list, §4.3** |
| microSD card | 1 | ~1 g | **1 g** | — |
| Waveshare Bus Servo Adapter (A) | 1 | ~25 g *(est.)* | **25 g** *(est.)* | Board + terminals |
| BNO055 (Adafruit 4646) | 1 | ~5 g *(est.)* | **5 g** *(est.)* | Breakout + QT cable |
| INA226 module | 1 | ~6 g *(est.)* | **6 g** *(est.)* | Owned already |
| 18650 cells | 4 | ~47 g | **188 g** | P30B-class cell weight |
| Cell holders (2-cell series) | 2 | ~13 g *(est.)* | **26 g** *(est.)* | Generic holder + leads |
| HW-391 BMS | 1 | ~4 g *(est.)* | **4 g** *(est.)* | Small PCB + leads |
| IP2326 charger board | 1 | ~6 g *(est.)* | **6 g** *(est.)* | — |
| Pololu D36V50F5 | 1 | ~7 g *(est.)* | **7 g** *(est.)* | 25 × 25 mm board |
| Switch, XT30 pairs, Wagos, power wiring | — | — | **45 g** *(est.)* | 18/20 AWG silicone runs |
| MAX98357A + 4 Ω 3 W speaker | 1 set | — | **23 g** *(est.)* | Amp ~3 g, cavity speaker ~20 g |
| Printed parts (legs/hips/neck = Duck; trunk + panda shells bigger) | — | — | **600 g** *(est., range 550–700)* | Duck uses ~500 g PLA total (its BOM) |
| Heat-set inserts (fitted subset) + M3 screws + nuts | — | — | **80 g** *(est.)* | — |
| Bearings 3 × 6804ZZ | 3 | ~19 g *(est.)* | **57 g** *(est.)* | §4.1 |
| Rubber soles (sheet, both feet) | 2 | ~10 g *(est.)* | **20 g** *(est.)* | §4.5 |
| Signal cabling (Dupont, QT, switch wires, USB) | — | — | **30 g** *(est.)* | — |
| **TOTAL (no eye board)** | | | **≈ 2,134 g** | |
| + K230 eye (§5): 01Studio mini bare | | | +16 g → **≈ 2,150 g** | |
| + K230 eye: Yahboom module with screen | | | +65 g → **≈ 2,200 g** | |

### 3.2 Cross-check

Bottom-up says ~2.13 kg. Top-down says: Duck 2,107 g + cells +95 g + Pi swap
+63 g + bigger shells +100 g *(est.)* + INA226/misc +15 g ≈ **2,380 g**. The
truth is probably between the two — bottom-up tables always forget glue,
cable slack, horn discs and the screw you add later; the top-down double
counts some shell growth. **Planning figure: 2.2–2.4 kg.**

### 3.3 Verdict and trims

- **Verdict: the panda lands ~0.1–0.3 kg over the 2.1 kg aim, and ~0.6–0.8 kg
  under the 3 kg ceiling.** That is fine. The 2.1 kg figure was "aim at the
  Duck's own weight"; the hard limits are servo torque and the 3 kg ceiling,
  and both have real margin (a 2.73 kg Jetson-carrying Duck fork exists and
  walks — it upgraded its servos, which at 2.2–2.4 kg we don't need to).
  The policy is retrained on the real weighed robot either way, so what
  matters is weighing it accurately at §1.5 step 11, not hitting 2.1 on the
  nose.
- **Trims if a diet is ever wanted** (in order of grams per effort):
  1. Cosmetic shells at 2 walls / 10% infill instead of structural settings:
     −50 to −100 g *(est.)*.
  2. 01Studio K230 mini (16 g) instead of the Yahboom screened module
     (65 g): −49 g.
  3. Print the panda body band/belly as thin shells rather than solid-ish
     forms: −30 to −60 g *(est.)*.
  4. Drop the optional barrel jack and shorten power runs: −10 to −20 g.
  5. Do **not** trim: battery (runtime), servos, bearings, sole material.

---

## 4. What's missing — gap analysis

Everything below is needed (or strongly advised) to assemble and run the robot
and is **not** on the shopping list, or is on it in the wrong form. Ordered by
"would stop the build" first. Prices are rough UK/AliExpress-class figures,
not live-checked.

### 4.1 ⚠️ Bearings: the cart has the wrong bearings (fix before paying)

- **What the list/cart says:** 2 × 608ZZ (8 × 22 × 7 mm).
- **What the Duck actually uses:** **3 × 6804ZZ** — a thin-section bearing,
  20 mm bore × 32 mm outside × 7 mm thick. Evidence: the official BOM as
  mirrored on holocraft.dev lists "3 × 6804ZZ ball bearing (in the trunk and
  the head)"; the assembly guide places bearings (plural) in trunk_bottom
  (one per hip-roll axis = 2) and one more in the head (head-roll axis) = 3.
  (One secondary mirror, orobot.io, prints "608ZZ" — but the OpenHead project
  that reuses the Duck neck had to *modify* a part to fit a 608ZZ, which only
  makes sense if the stock part takes something else. The 6804's 20 mm bore
  slips over the servo output boss; an 8 mm-bore 608 physically cannot
  support those axes as designed.)
- **Fix:** in the cart, remove 608ZZ ×2, add **6804ZZ ×3** （淘宝： 6804ZZ 轴承，
  内径20 外径32 厚度7). ~£2–3 each, so ~£6–9 — roughly what the 608s cost.
  Zero-cost fix *because nothing is paid yet*; an annoying delay if found
  on build day.
- **Caveat:** our panda trunk/head are redesigned around the Duck interfaces,
  so the CAD must keep the 6804 seats. If the redesign deliberately re-seats
  the axes for 608s, that's a design decision to make consciously — current
  CAD inherits Duck geometry, so 6804ZZ it is.

### 4.2 microSD card for the Pi 5 — not on the list at all

The Pi 5 has no storage of its own in this build, and no card is on the list
(the Duck's BOM includes a 32 GB A1 card; ours dropped off somewhere). The
panda runs Raspberry Pi OS + the Duck runtime + Hermes — hungrier than the
Duck's runtime-only card.

- **Fix:** **64 GB microSD, A2-rated** — SanDisk Extreme A2 or Samsung PRO
  Plus/EVO Plus A2, ~£9–12. Buy **two**: image the second as a cold spare
  after first successful boot; a corrupted card is the #1 way these robots
  "die", and a pre-imaged spare turns it into a 2-minute fix. (An NVMe HAT
  was considered and rejected: +weight, +power draw, +£20–30, and the trunk
  doesn't need the speed.)

### 4.3 Pi 5 cooling — not on the list; the trunk is an enclosed box

The Pi 5 throttles at 85 °C and will get there inside a sealed plastic trunk
running a 50 Hz policy loop plus an agent. The Duck never faced this (a Pi
Zero makes ~1 W of heat; a working Pi 5 makes 5–9 W).

- **Fix (hardware):** **Raspberry Pi Active Cooler** (official heatsink +
  temperature-controlled blower, 28 g, ~£5 from Pi Hut — same basket as the
  Pololu). It bolts to the Pi's own fan header; nothing to wire.
- **Fix (CAD, free):** the trunk design must give the cooler air: vent slots
  low on one side of the belly shell and high on the other/back, with the
  Pi mounted so its cooler exhaust faces the high vents. Still air + plastic
  walls will throttle it even with the fan.
- Note the Active Cooler is in the weight budget (§3) already.

### 4.4 Ear-servo drive — investigated, NOT a gap

Checked per the brief: the Waveshare adapter cannot do PWM, but it doesn't
need to — GPIO 12/13 hardware PWM from the Pi is the upstream Duck solution
(§2.6). No part to buy. The only work is a Pi 5 PWM software port in the
runtime. Power the ears from the buck rail, not the Pi header (§2.6).

### 4.5 Sole material — TPU ruled out, a specific replacement still needs choosing

The requirement (settled 2026-11-15): soft + grippy + firmly attached. The
Duck's foot design already clamps its sole in a screw sandwich
(foot_bottom_pla + sole + foot, 2 × M3×6 per foot), so a flat sheet material
needs no glue and no redesign.

- **Recommended fix:** **3 mm self-adhesive solid neoprene rubber sheet**
  (A4 sheet, ~£6–8 on Amazon/AliExpress; 淘宝： 自粘氯丁橡胶板 3mm). Cut two
  foot outlines with scissors, peel, stick to foot_bottom_pla as a positioning
  aid, then let the screw sandwich do the real holding. Solid neoprene (not
  foam) — foam compresses to nothing in a month of walking; solid rubber
  grips hard floors, damps step impact, and wears slowly. 2 mm also works;
  3 mm matches the TPU sole's designed thickness more closely.
- Alternative if neoprene slips on his floors: natural rubber sheet (more
  grip, wears faster) — decide after the first steps on the real floor.

### 4.6 Wire stock — gauges were never specified on the list

The list buys connectors and ready leads, but the runs we make ourselves
(pack→BMS→switch→adapter/buck, buck→Pi) need actual wire, in the right
thickness (§2.2):

- **Fix:** 1 m each of **18 AWG silicone wire, red + black** (~£4–6 the pair)
  and 0.5 m **20 AWG silicone red + black** for the buck→Pi feed. Silicone
  insulation (flexible, heat-tolerant) — the cheap PVC hook-up wire in most
  assortment kits is too stiff and its insulation nicks at joints that move.
  Servo-bus and signal wiring is already covered (servo leads included,
  Dupont/spiral wrap in cart).

### 4.7 Tools the build assumes but nobody has listed

The Duck assembly guide's requirements line starts "a soldering iron, and
basic electronics tools". The no-soldering rule covers *joints*; two tool
gaps remain:

1. **Heat-set insert installation.** 50 brass inserts must be melted into
   printed parts. That needs a soldering-iron-shaped heat tool — used purely
   as a heater, no solder involved. **Fix:** a cheap temperature-controlled
   iron (~£10–14, set ~220 °C for PLA) **plus a brass insert-tip set**
   (M3 tip, ~£6–8) — the tip is a smooth cone that pushes the insert in
   straight; using a bare soldering tip works but wanders. (A vertical
   insert press, ~£25–35, is the luxury version — straighter inserts, zero
   skill. Optional.) If he already owns any soldering iron from the ESP32
   work, only the ~£7 tip set is needed.
2. **Multimeter.** Required before assembly (match the 4 cells' voltages —
   upstream safety step) and for every power-on check in §2. **Fix:** any
   basic digital multimeter, ~£8–12, if he doesn't already own one.
3. Small stuff assumed present: 2.5 mm hex driver (M3 socket heads), small
   Phillips (servo screws), side cutters + wire strippers (Wago joins need
   ~11 mm stripped ends), tweezers for Dupont ends inside the trunk. A
   £8–10 precision driver set covers all of it if not owned.

### 4.8 Microphone — the voice plan has a speaker but no ear

The list gives the robot a voice (MAX98357A + speaker) but no microphone:
Hermes' wake word has nothing to listen to. The Duck's own BOM lists an I2S
MEMS mic (Adafruit SPH0645) as "listed, not integrated yet", so upstream is
no help. He already owns a **Sipeed mic array** (from the earlier panda
work) — USB/UART array, ~15 g *(est.)*.

- **Options:** (a) fit the Sipeed array behind a grille in the trunk front —
  owned, £0, but it's a big board and trunk-front is also where the speaker
  wants to be (feedback risk: mic next to speaker = the robot hears itself;
  workable with echo cancellation in software, annoying in practice);
  (b) a small I2S MEMS mic (SPH0645-class, ~£7) in the *head* — far from the
  speaker, near the people it's listening to, shares the I2S bus wiring
  skills already in the design (needs its own I2S input config; the Pi has
  one I2S port — amp is output-only on the same bus standard, this is the
  Duck's intended combo too);
  (c) v1 ships deaf: voice input stays on the house setup (Jetson Whisper +
  room devices), the robot is output-only until v2.
- This is a genuine fork in the design (it changes the head CAD) → §6.

### 4.9 Spares worth having on the shelf

Walking robots fall; falls break the cheapest parts first. Recommended
spares, ~£25 total:

- **1 × spare STS3215-C001** (~€14) — a stripped gear or bent horn after a
  fall otherwise stops the project for a shipping cycle. Buy with the main
  batch (his servo order is reported placed — add one if it can still be
  amended, else with the next order).
- 2 × spare SS-10 switches (~£1) — press-fit plastic, the most-stressed
  part in the robot.
- A few spare Feetech 3-pin servo leads (15 cm and one 25–30 cm, ~£3) —
  chafed leg cables are the classic Duck failure.
- The second microSD from §4.2, pre-imaged.

### 4.10 Already-checked non-gaps (so they aren't re-bought)

- **Servo horns, screws, cables:** every STS3215 ships with a main horn, an
  auxiliary-shaft horn, 9 horn screws, 8 body screws and one 15 cm 3-pin
  cable (retailer package listings) — the Duck build is designed around
  exactly these. Nothing extra needed beyond §4.9 spares.
- **I2C addresses:** no conflict (§2.4).
- **Ear PWM:** solved in software/wiring, no part (§4.4).
- **Barrel jack:** optional and now down to ×1 on the list; with USB-C
  charging through the shell it can be skipped entirely (saves a hole in
  the shell and ~10 g). Recommend skipping.

---

## 5. The K230 eye option

The user is considering a Kendryte **K230** AI-vision board (CanMV firmware)
as the panda's eye: the board watches the world and *reports results* to the
Pi over a serial link, instead of the Pi processing camera images itself.

### 5.1 The chip and the candidate boards

The K230 (Canaan/Kendryte) is a dual-core RISC-V chip (0.8 GHz + 1.6 GHz)
with a neural accelerator ("KPU") rated at **6 TOPS equivalent** (INT8/INT16).
Vendor benchmark figures: ResNet-50 > 85 fps, MobileNetV2 > 670 fps,
YOLOv5s ~38 fps (INT8) — i.e. face/person detection at camera frame rates is
effortless for it. It runs **CanMV** (MicroPython) — you write Python on the
board, models ship as .kmodel files, and Kendryte's image comes with a large
demo library (below). All boards below use the same chip and a **GC2093
2 MP camera** (1080p60 capable) unless noted.

| Board | Size | Weight | Power | Screen | Price class *(est., varies by seller)* |
|---|---|---|---|---|---|
| **01Studio CanMV K230 mini** | 60.3 × 42.7 × 17.6 mm | **16 g** (40 g in its case) | 5 V, ~0.6 A class | none | ~£45–65 |
| **Yahboom K230 vision module** | 72 × 46 × 18.7 mm (touch version) | **64.9 g** | 5 V, **0.8 A / 4 W** | 2.4″ 640×480 touch | ~£70–100 |
| Yahboom K230 standard (no screen) | 75 × 47 × 19.7 mm | ~50 g *(est.)* | 5 V, 0.6 A / 3 W | none | ~£60–85 |
| 01Studio CanMV K230 (full) | 85 × 56 × 15 mm | 31 g | 5 V | none (3.5″ in kits) | ~£55–75 |
| Hiwonder CanMV K230 | ~Pi-size board | — | 5 V | 3.5″ in kit | $79.99 (their store) |

(Sizes/weights/power: Yahboom instruction manual; 01Studio wiki; Hiwonder
store. Prices: marketplace listings seen 2026-11-15 — treat as ballpark.)

Every K230 board also needs a **microSD/TF card** for its firmware + models
(some Yahboom kits bundle a 32 GB card) — add ~£6 if not bundled.

### 5.2 Does it solve the head-fit problem? Yes — because it's one centred eye

Recall why the Waveshare IMX219-83 stereo board failed: it's an 85 × 24 mm
flat board whose two lenses sit at ±30 mm; the panda's curved face falls
away ~19 mm at those corners, far beyond the lenses' ~6 mm protrusion, so
the board cannot sit flat behind the face with both lenses seeing out.

A K230 module is a completely different shape problem:

- **One camera, centred.** The panda head CAD (v3.5) already converged on
  exactly this: the "nose" is a single camera hole in the muzzle swell, with
  a base plate + riser behind it carrying a small camera board. A K230
  module mounts on that same base plate, lens at the nose hole.
- **It fits the envelope.** Head shell is 104 mm wide / ~96 mm deep. Even
  the biggest candidate (Yahboom touch, 72 × 46 mm) fits across the head's
  middle with ~15 mm a side to spare; the 01Studio mini (60 × 43 mm) fits
  trivially. Depth: module ~18–20 mm + the GC2093 lens stack — the head has
  ~90 mm of depth to play with; the riser height is set so the lens sits
  just behind the nose hole, exactly like the earlier Pi-camera design.
- **The camera is not at the board's edge problem** — the GC2093 sits on a
  small raised module near the board's centre on these boards, so the curved
  face only has to clear the lens, not the whole PCB.
- **Weight in the head:** 16 g (01Studio mini) is nothing; 65 g (Yahboom
  with screen) is acceptable but pointless — the screen would be entombed
  inside the head. (Mounting the Yahboom with its screen facing *out* as an
  animated face is possible mechanically — 72 mm screen module in a 104 mm
  head — but it deletes the printed panda face he has spent weeks designing.
  Noted, not recommended.)

**Verdict on fit: the K230 turns the eye from the hardest packaging problem
in the head into a solved one**, reusing the existing nose-hole design.

### 5.3 Talking to the Pi — UART design

Both board families expose a hardware UART (Yahboom explicitly provides a
serial interface for piping recognition results to a host — their whole
ecosystem of Pi/ESP32 examples works this way). Design:

- **Physical:** K230 UART TX/RX ↔ Pi GPIO 14/15 (UART0), 3.3 V levels both
  sides, plus the 5 V power pair from the buck rail (§2.1) and ground.
  Four thin wires through the neck, with a slack loop at the head-roll axis.
  **115200 baud 8N1** is plenty (see rates below); the link can run faster
  if ever needed.
- **Framing:** one line of JSON per message, newline-terminated — dead
  simple to debug from a terminal, and both sides speak it natively
  (MicroPython `json` on the K230, Python on the Pi). Add a 1 Hz heartbeat
  so the Pi knows the eye is alive.
- **Messages, K230 → Pi:**
  - `{"t":"face","n":1,"faces":[{"x":512,"y":430,"w":180,"h":210,"id":"cl","conf":0.87}]}` — coordinates normalised 0–1000; `id` present when face *recognition* is running and the face is enrolled, else `null`.
  - `{"t":"person","n":2,"boxes":[...]}` — same shape, person detections.
  - `{"t":"hb","mode":"face","fps":24}` — heartbeat/status, 1 Hz.
  - Event messages on change (face appeared/lost) in addition to the stream.
- **Messages, Pi → K230:** `{"cmd":"mode","mode":"face_track"|"face_id"|"person"|"off"}`,
  `{"cmd":"enrol","name":"..."}` (start an enrolment capture), `{"cmd":"snapshot"}` (see caveat).
- **Rate:** face/person updates at **10 Hz** while tracking (the neck servo
  loop doesn't need more; the K230 detects at 25–38 fps internally and just
  reports the freshest box), heartbeat 1 Hz, events immediate. Total
  bandwidth < 2 KB/s — 2% of the link.
- **USB alternative:** the boards' USB-C carries power + a CanMV
  serial/IDE channel, so a single USB-C cable Pi→K230 could replace the
  UART+power wiring. Rejected as the primary link: USB-C cables are stiff
  for a moving neck, the port is also the board's programming port (you'd
  unplug the robot's eye to develop on it), and a plain UART pair is
  thinner and more robust. Keep USB for bench development only.

**Caveat — snapshots:** UART is for *results*, not pictures. If Hermes ever
wants an actual image (to ask a cloud VLM "what is this?"), a JPEG over
115200 baud takes seconds. Escape hatches: the Yahboom module has Wi-Fi
(stream/post images over the LAN), or accept that scene-description uses a
different camera. For the v1 jobs (face follow, person awareness, "who is
that") results-only is enough — flagging so it's a known limit, not a
surprise.

### 5.4 Who does what

| Job | Where | Why |
|---|---|---|
| Face detection + tracking (box following, 25–38 fps) | **K230** | Built-in CanMV demos (`face_detection`, YuNet face detect); runs at frame rate with zero Pi load |
| Face recognition (enrolled family faces → names) | **K230** | CanMV ships `face_recognition` / `face_recognition_lite` + registration demos; embeddings stay on the eye's card |
| Person detection | **K230** | Built-in `person_detection` demo (YOLO-class) |
| Head/ear servo control (follow the face box) | **Pi** (body layer) | Consumes the 10 Hz face stream; servos are on the Pi's bus |
| Walking policy (50 Hz) | **Pi** | Never shares a processor with vision experiments — a crashed vision script can't fall the robot |
| Dialogue, Hermes agent, STT/TTS, home control | **Pi** (+ Jetson/cloud as already settled) | Unchanged from the existing panda plan |
| "Describe this scene" (VLM) | Pi/cloud, needs an image | Not from the K230 over UART — see §5.3 caveat |

The architectural win is the same one the whole panda already follows: the
safety/movement layer and the clever layer are separated — here the clever
*perception* layer is even on its own chip, in the head, behind a
narrow, well-defined message pipe.

### 5.5 Honest comparison

| | **K230 module (01Studio mini)** | **IMX219-83 stereo on Pi** | **Pi Camera Module 3 on Pi** |
|---|---|---|---|
| Fits the panda head? | **Yes** — centred lens at the existing nose hole | **No** — proven not to fit flat (§5.2) | Yes — tiny board, same nose-hole design |
| Extra weight in head | 16 g | ~30 g *(est.)* + it doesn't fit anyway | ~4 g |
| Cost *(est.)* | ~£45–65 + TF card | ~£35–45 | ~£25–35 (needs the Pi 5's 22-pin CSI cable) |
| Face detection/tracking | On-chip, 25–38 fps, demos included | On Pi CPU — competes with the 50 Hz policy + agent | On Pi CPU — same contention |
| Face recognition | Built-in demos, enrolment included | DIY on Pi (doable, more work, more CPU) | DIY on Pi |
| Depth perception | **No** (single camera; the chip has a depth engine but only for structured-light rigs, not this board) | **Yes** — 60 mm baseline stereo | No |
| Power | +3–4 W from the 5 V rail | ~1–2 W *(est.)* via CSI | ~1 W *(est.)* |
| Images available to the Pi/Hermes | Results only (Wi-Fi escape hatch on some boards) | Full images, both eyes | Full images |
| Dev effort | MicroPython on the board (CanMV IDE); a second codebase, but a tiny one | Python on Pi, one codebase | Python on Pi, one codebase |

### 5.6 Recommendation

**Adopt the K230 as the eye — specifically the 01Studio CanMV K230 mini**
(16 g, smallest, cheapest of the serious options, same chip/camera/demos).
Choose the Yahboom module instead *only* if he wants the touchscreen for
bench development or Yahboom's tutorial hand-holding; its screen and 65 g
buy nothing inside the finished head.

Reasoning in one paragraph: the panda's head-fit problem was created by a
wide stereo board and disappears with a centred single eye; the K230 is the
only option that also *removes* vision load from the Pi (which is already
running a 50 Hz walking loop and a 24/7 agent); face detection, tracking
and recognition — the three vision jobs this robot actually has — are
solved, demoed, shipping features of CanMV rather than things we'd build;
and the UART results-pipe matches the panda's existing "body layer /
agent layer" separation. The real losses are stereo depth (no current
panda job needs it — obstacle work was always going to be lidar's job, and
the owned lidar units remain the plan for that) and easy image access for
cloud VLM questions (accepted; Wi-Fi boards soften it).

---

## 6. Open decisions for the user

Each with my recommended default. None of these block the design work; #1
should be answered before paying for the cart.

1. **Bearings — swap the cart's 2 × 608ZZ for 3 × 6804ZZ?** (§4.1)
   *Default: yes, swap.* Upstream BOM + assembly guide agree; the cart is
   unpaid, so it's free to fix now.
2. **Eye — adopt the K230, and which board?** (§5)
   *Default: yes; 01Studio CanMV K230 mini.* (Yahboom if he wants the
   screen/tutorials.) This also closes the long-open "eyes" decision from
   the wheeled-panda era.
3. **Pi cooling — add the official Active Cooler (~£5, same Pi Hut basket
   as the Pololu)?** (§4.3) *Default: yes.* Plus vent slots in the trunk
   CAD (free, my side).
4. **Storage — 64 GB A2 microSD ×2 (one working, one pre-imaged spare)?**
   (§4.2) *Default: yes.*
5. **Soles — 3 mm self-adhesive solid neoprene sheet, clamped in the
   existing foot sandwich?** (§4.5) *Default: yes.*
6. **Microphone — robot's own mic, and where?** (§4.8)
   *Default: (b) a small I2S MEMS mic in the head (~£7) when the head CAD
   is drawn; v1 can ship without it and use the house voice setup
   meanwhile.* The Sipeed array in the trunk is the £0 fallback.
7. **Spares — add 1 × STS3215, 2 × SS-10, spare servo leads (~£20–25)?**
   (§4.9) *Default: yes for the servo if the order can still be amended.*
8. **Barrel jack — drop it entirely (USB-C charging only)?** (§4.10)
   *Default: drop.* One less hole in the shell.
9. **Tools — does he already own a soldering iron and a multimeter?**
   (§4.7) If yes, only the ~£7 insert-tip set is needed; if no, ~£20 of
   tools joins the list. *Default: assume the tip set + multimeter need
   buying until he says otherwise.*
10. **Pay for the cart** (after #1) — his call, his timing; the design
    above assumes the list as settled.

---

*Prepared overnight 2026-11-15/16. All shopping-list and MEMORY state was
left untouched; this report is the only file written. Where sources
disagreed (bearings), the disagreement is flagged rather than papered
over.*
