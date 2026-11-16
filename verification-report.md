# Panda design — verification report (overnight audit, 2026-11-16)

**What this is:** a front-to-back audit of the whole legged-panda design record,
done in three passes — (1) every document cross-read against every other and
against the decisions locked on 2026-11-16, (2) every load-bearing number
recomputed from scratch, (3) a systems sanity check of power, data, thermal,
mechanical interfaces and failure modes.

**Documents audited:** repo `README.md`, `shopping-list.md`,
`assembly-design.md`, `fit-study.md` (including its "Resolutions added
2026-11-16" note, which supersedes older text below it), the goal-files copy
of the shopping list, and `GOAL.md`.

**How to use it:** findings are numbered **F1, F2, …** — reply by number.
Severity: **BLOCKER** = must be settled before CAD or before the step it gates;
**SHOULD-FIX** = wrong or stale on paper, cheap to fix now, annoying later;
**NOTE** = small, recorded so it isn't a surprise. *This report changes
nothing in the design documents — every fix below is a proposal for the
parent agent / user to apply.*

**Scoreboard: 31 findings — 6 blockers, 17 should-fix, 8 notes.**

---

## Pass 1 — Consistency findings

### BLOCKERS

**F1 — The two design documents give irreconcilable robot heights and opposite
leg instructions.**
`assembly-design.md` §1.2 says the legs "carry over unchanged" from the Duck
(same geometry), the trunk gets *taller*, the head stays ~100 mm — and its
§1.3 height note concludes the panda stands **~45–48 cm**. The locked design
(README look section, fit-study) is **~35 cm total with legs of 11–13 cm**.
Both cannot be true. The fit-study's stack (legs ~120 + trunk 95 + neck ~25 +
head ~110 = 350 mm) is the locked one, but nobody has yet drawn the leg that
is actually 120 mm — whether the Duck leg *is* ~120 mm floor-to-trunk or
needs shortening is not established anywhere in the record.
*Fix: declare assembly §1.2/§1.3 superseded on proportions; before CAD, one
dimensioned stack drawing (floor → hip axis → trunk top → head top) that the
CAD, the MJCF and this audit's numbers all reference. CAD gate.*

**F2 — The charger-to-BMS wiring is drawn two different ways, and one of them
contradicts the corrected design record.**
`assembly-design.md` §2.1/§2.8 tee the IP2326's +/− across the BMS **P+/P−**
output nodes and its middle wire to BM. The settled record from 2026-11-15
(user-flagged correction) is that the charger's three battery wires go to
**the pack itself** (+, middle, −). On the HW-391 the positive is common
(B+ = P+), so the + side agrees either way — but the negative does not:
P− is the BMS's *switched* leg, B− is the raw cell negative. Which one the
IP2326's BAT− must see decides whether charging works through a tripped BMS
and whether the balance lead is referenced correctly. The documents never
reconcile this.
*Fix: settle it from the HW-391 and IP2326 board documentation, redraw the
§2.1 power diagram with B+/B−/BM/P+/P− labelled at every tee, and only then
wire anything. Power-build gate.*

**F3 — The neck cable bundle is undercounted in the fit study; the CAD channel
would be sized too small.**
Fit-study §2 counts "servo bus (3) + K230 power/UART (4) + speaker pair (2)
≈ 10 wires", and its resolutions note says the 2 charge leads bring it to
"~12". The ear servos are missing from that count. Full enumeration (Pass 3b):
servo bus 3, K230 5 V/GND/TX/RX 4, ear servos 4 (one shared 5 V/GND pair +
2 signals — 6 if each ear gets a full 3-wire home run), speaker pair 2,
charge leads 2 → **15 conductors (17 worst case)**, a bundle of Ø ~11–14 mm,
not ~8–10 mm.
*Fix: reserve a Ø 14 mm clear channel in the neck CAD (clear of the roll
bearing), or add a small head power-distribution point so grounds/5 V are
shared deliberately and the count drops to a documented 12. CAD gate.*

**F4 — The K230's identity and status are three different stories.**
README look section and the fit study design around the **01Studio mini**
(60.3 × 42.7 mm). README status says "K230 board + 2 cameras" is **still to
buy**. The user says "**my** K230 has 2 camera ports" — which suggests a
board already in hand, model unstated. The full-size 01Studio K230 is
85 × 56 mm (assembly §5.1) — it would not fit the fit-study's head layout as
drawn, and its camera arrangement differs.
*Fix: one confirmation — which board, in hand or to buy, measured if in
hand — before head CAD and before any camera purchase. CAD/order gate.*

**F5 — The speaker lives in two different places.**
`assembly-design.md` §1.3 puts the speaker in the **trunk front** with the
amp beside it, and §1.5 steps 9–10 build it that way. The fit study §2 and
the locked look (smiling mouth = speaker grille) put the **speaker in the
head** behind the mouth slot, with the amp in the trunk and a 2-core lead up
the neck. These are different head CAD, different trunk CAD and different
wiring.
*Fix: adopt the fit-study placement (it is the one the locked look and the
approved renders show) and correct assembly §1.3/§1.5. CAD gate.*

**F6 — The fitted mic array has no completed electrical design.**
Since the array decision, the trunk back carries the Sipeed R6+1 as a fitted
part — but `assembly-design.md` §2 (the wiring map) doesn't mention it at
all: no power feed, no data path. Fit-study §7 flags the open question and
then files it under "build time": the array's mics are I2S devices, the Pi's
single I2S bus already drives the MAX98357A, and the array was designed for
Sipeed's K210 Maix boards, not as a Pi peripheral. Its 5 V draw (~0.8 A
class per its datasheet) appears in **no** power budget in any document
(see F14). It may work on the Pi's I2S bus; that is currently an assumption,
not a verified fact.
*Fix: bench-test the owned array against the Pi 5 (I2S capture + LED ring)
before the trunk back panel is finalised in CAD; add its power/data to the
wiring map either way. Fallback if it won't talk to the Pi: the parked
ear-mics or a small I2S MEMS mic. CAD gate for the back panel only.*

### SHOULD-FIX

**F7 — Fit-study §2 quotes the wrong servo torque: "30 kg·cm stall".**
The STS3215-**C001** (7.4 V) is rated **19.5 kg·cm stall** (shopping list,
Feetech figures). 30 kg·cm belongs to a different voltage variant. Every
margin the fit study derived from 30 is overstated by ~1.54×. Recomputed
with 19.5 (Pass 2b), the head loads are still comfortable — but the document
must not carry the wrong rating into the policy work.
*Fix: correct §2 to 19.5 kg·cm stall and re-state the margins (7–12% of
stall, static, at 4–6 cm offset).*

**F8 — README's mass target (2.1 kg) is below the recomputed floor.**
README line 9: "Design mass target ~2.1 kg". Pass 2a recomputes
**~2.20 kg central, credible range ~2.15–2.40 kg** — the 2.1 figure survives
only in the README intro and assembly §3.3's framing ("aim"), while the same
assembly report's own planning figure is 2.2–2.4 kg.
*Fix: README intro to read "design mass ~2.2–2.4 kg (3 kg hard ceiling)".*

**F9 — The 11 cm end of the leg range isn't reachable; the honest floor is
~12 cm.**
Pass 2c: three pitch-servo bodies (35 mm each) + the foot sandwich (~14 mm)
already make ~119 mm before any link plate, spacer or cable bend allowance.
Legs of 12–13 cm are credible and stay under the 15 cm limit with margin;
11 cm is not, on the documents' own dimensions.
*Fix: restate the locked leg range as "12–13 cm design point, <15 cm hard
limit" wherever 11–13 appears (README look section, fit-study header).*

**F10 — Fit-study §6 (its own verdict summary) contradicts its resolutions
note.**
§6 still headlines the **trunk-back** charging port as the answer, still
prices the mic array as "fits only if the port moves to the bottom face",
and still gives lidar verdicts for A1/LD06/LD19 classes. The resolutions
note at the top supersedes all three (head port; array fitted externally;
C1 owned, placement parked) — but a reader skimming §6, as readers do, gets
the old answers.
*Fix: add a one-line "superseded by the resolutions note" banner to §6 (and
§4's verdict line), or fold the resolutions into §6.*

**F11 — Assembly report's charge-access and battery-door design is stale.**
§1.2 lists a "USB-C charge port" among the trunk contents, §1.2 item 3 has
the battery sled sliding out of the **back panel**, and §2.8 bolts the
IP2326 at the rear panel with its USB-C through a shell slot. The fit study
dropped the through-slot, moved battery access to the **bottom panel**, and
the resolutions then moved charging to the **head**. The assembly report's
own header update note mentions none of this (it predates the fit study).
*Fix: extend the assembly report's header note to list these supersessions
explicitly, so nobody builds §2.8 as written.*

**F12 — Shopping list carries four stale or self-contradictory lines.**
(a) The charging paragraph still ends with the "open check: 4-pin or 2-pin?"
— **resolved 2026-11-16: it is the 2-point (~2 A) pair.**
(b) It still lists the optional barrel jack, while assembly §4.10 recommends
dropping it and the fit study dropped the shell slot it would have used.
(c) It specifies the IP2326 as "~1.5–2 A" output — impossible through the
2-point pair: 5 V × 2 A = 10 W in cannot produce 1.5–2 A at 8.4 V
(12.6–16.8 W) out (see Pass 2f).
(d) "Eyes/camera" still says the K230 board + cameras are "not yet on the
buy list — to add", while README status lists them under "still to buy" —
one of the two should be the single statement (see also F4).
*Fix: one editing pass over the list with the 2026-11-16 resolutions in
hand.*

**F13 — Purchase status is ambiguous exactly where it matters most.**
The shopping list heads two sections "**reported DONE by him**" (servos &
control; sensing). README status describes the same items as "in cart
(checkout pending)". Assembly §4.9 says "his servo order is reported
placed". These are three different states — reviewed, carted, paid — and
the record never separates them. HW-391 is honestly marked "NOT yet bought
(2026-11-15)"; the Pololu is "his decision… he will just pay" with no
purchase confirmation since. Every status in the repo is dated 2026-11-15
or earlier.
*Fix: one status pass with the user, item by item — owned / paid-and-shipped
/ in cart / not ordered — and date it. Until then, treat "DONE" as
unverified for build planning.*

**F14 — The 5 V rail's paper worst case is now 8.5 A against a 5.5 A buck,
and the documents' watch item predates the mic-array decision.**
Assembly §2.9 already flags Pi (5 A spec) + K230 (0.8) + ear stalls (1.4) +
amp (0.5) = **7.7 A** vs the Pololu's 5.5 A, and files it as "peaks never
coincide". Adding the now-fitted mic array (~0.8 A) makes **8.5 A**. The
real-world draw will be far lower (the Pi's 5 A is a supply spec, not a
load), but a brown-out on this rail reboots the Pi *while it is the walking
controller* (see F19).
*Fix: cheapest robust answer — a second small 5 V buck for peripherals
(ears + amp + K230 + array), the Pololu feeding the Pi alone; alternatively
prove the single rail by bench measurement at §1.5 step 0 before the trunk
is closed. Decide on paper now, not after the first mid-walk reset.*

**F15 — The high-current chain's ratings don't line up, and two links are
unrated in the documents.**
BMS trips at 20 A and the switch is rated 20 A, but the XT30 pairs are
15 A continuous and the specified 18 AWG runs are ~16 A class (assembly
§2.2). Fourteen servos stalling at once would draw 14 × 2.5 A = 35 A — the
design relies on "realistic peaks" with no fuse anywhere: the weakest link
is currently a connector or a battery-holder spring, not a protection
device. Worse, **no document rates the two components that carry everything:
the 18650 holder contacts** (generic holders are often only a few amps per
contact) **and the Waveshare adapter's power input/terminals.**
*Fix: (i) confirm holder contact rating — if low, switch to a seller-built
welded 2S2P pack with leads (keeps the no-solder-at-home rule); (ii) confirm
the adapter's input rating from Waveshare's spec; (iii) add a fuse or PTC
at the pack output sized to the weakest confirmed link (≈15 A), so the
designed failure is a blown fuse.*

**F16 — M3×5×5 heat-set inserts cannot go into the 3 mm panels the fit study
assumes.**
The insert is 5 mm long with a ~5.3 mm knurled OD. A 3 mm cosmetic panel
(the fit study's shell assumption) is thinner than the insert is long and
barely wider than its hole. Assembly §1.4 has inserts going into "body
shells, head" broadly. Inserts are fine in bosses and structural parts
(trunk_bottom, leg spacers, feet) — they are physically impossible in flat
3 mm panels.
*Fix: a CAD rule written into both design docs: inserts only in local
bosses ≥ ~8 mm across and ≥ 6 mm deep; 3 mm panels use captive nuts, nut
slots or direct screws.*

**F17 — The mic array's mounting method doesn't exist yet, and its fixings
land ~3 mm from the panel edges.**
Array 88.8 mm tall on ~89 mm of usable back panel (fit-study §7): any
corner screws sit within ~3 mm of the trunk's top/bottom edges — not enough
material for a reliable fixing, and the array decision (external plate,
"glass hat") postdates that analysis.
*Fix: design a printed cradle/frame that wraps the panel's top and bottom
edges and carries the array on its standoffs — no edge-distance problem,
and it matches the external-plate look he approved.*

**F18 — No low-battery behaviour is specified in any current document.**
The INA226 is fitted "for low-battery behaviour", but no threshold or
behaviour appears in README, assembly or fit-study. The only numbers in the
record are in the stale GOAL.md (return at ~30%, hard floor ~15% — written
for the wheeled plan). Without a specified behaviour, the end of a walk is
the HW-391's hard undervoltage cut-off: instant power loss, servos dead,
robot falls — possibly onto its head and K230.
*Fix: adopt explicit body-layer thresholds now (suggested: warn 30%,
return-to-dock 25%, controlled sit-down at ~6.6 V under load / ~15–20%,
BMS cut-off as last resort only) and write them into the README decisions
table.*

**F19 — README overclaims the crash safety: "if the brain crashes, the legs
don't."**
The body layer is independent of the *agent process*, but it runs on the
same Pi 5 as everything else (settled all-in-one architecture). A Pi hang,
kernel panic or brown-out reboot (F14) takes the walking controller with
it, and no document specifies what the servos/adapter do on USB loss
(hold last position? torque off?).
*Fix: reword the claim to "independent of the agent", and specify the
failsafe: servo behaviour on bus/USB loss configured at §1.5 step 0, plus a
watchdog that commands a crouch/sit on controller loss if the hardware
supports it.*

**F20 — The dock's headline geometry (arm at ~275–280 mm) is not derivable
from the locked proportions yet — because the port's position on the head
is undefined.**
Pass 2h: with legs 120 + trunk 95 + neck 25, the head *centre* sits at
~297 mm (range ~285–305 across the leg/head tolerances). An arm at
275–280 mm meets the head only if the port sits in its **lower** back
panel, ~15–20 mm below centre — plausible, and arguably good design, but
no document says where on the head the port goes. Also note the dock arm
must now reach ~100 mm forward from its post (resolutions), not the 35 mm
of the old trunk-port design — a longer, springier cantilever to align.
*Fix: define the port centre in the head CAD (recommend lower-centre back
panel, on a flat boss — see F31), publish the resulting dock height from
the finished robot's measurements, and don't print the dock until then.*

**F21 — GOAL.md is stale as the current brief.**
Its title and description still describe the **ESP32-P4 pan-tilt head**;
its dated updates end at the 2026-11-14 wheeled GIM4305 plan. The legged
STS3215 pivot (2026-11-15) and everything after it appear nowhere. A fresh
session reading GOAL.md first — as our own conventions instruct — gets a
two-generations-old project. (Its history is valuable and should stay —
as history.)
*Fix: add a top banner/pivot section: "superseded 2026-11-15 — legged
STS3215 panda; current design lives in the GitHub repo", keeping the log
below untouched.*

**F22 — "Adapt the existing head CAD" understates the head job by a lot.**
Assembly §1.2 proposes adapting the v3.5/v3.6 panda head (104 mm wide,
single nose camera hole, eye pits at ±22–24 mm, static ears). The locked
head is ~150 mm across, with two lens eyes at **±35 mm** (fit-study §2),
**no** nose hole, a mouth slot with speaker behind it, MG90S pockets at
both ear roots, a charge port in its back panel, and a K230 + heatsink +
vents inside. That is a new head that borrows the old one's *style*, not
an adaptation of its geometry.
*Fix: restate the CAD scope honestly in the assembly report/README so the
work is planned as a redraw.*

**F23 — The K230's own microSD card is missing from the shopping list.**
Assembly §5.1 notes every K230 board needs a TF card for firmware/models.
The shopping list's two 64 GB cards (README status) are both earmarked for
the Pi (working + pre-imaged spare). No third card for the K230 appears
anywhere.
*Fix: add 1 × microSD (32–64 GB) for the K230 to the list.*

### NOTES

**F24 — The two shopping-list copies currently agree — byte for byte.**
SHA-256 of the repo copy and the goal-files copy are identical
(`1db28d3c…`) at audit time: **no divergence today.** The risk is
structural: the repo copy is the one being edited (per the "GitHub is the
source of truth" instruction), so the goal copy *will* drift silently.
*Fix: replace the goal-files copy with a one-line pointer to the repo
file at the next edit.*

**F25 — Trunk stack re-addition gives zero clearance, where the fit study
said a 2 mm overlap.**
Fit-study §3: mid plate at 26 mm + Pi/cooler envelope ~28 mm = "top at
~56"; servo 30 hangs to 89 − 35 = 54 → "2 mm overlap". Re-adding its own
figures: 26 + 28 = **54**, i.e. the Pi stack top and the servo's bottom
edge are at the *same* height — zero clearance rather than overlap. The
conclusion is unchanged (the Pi-front / servo-rear layout separation is
mandatory, and the rear mezzanine has a genuine 14 mm of air), but the CAD
must work from a *measured* Pi + cooler height, not the ~28 mm estimate.
*Fix: measure the cooler stack at bench stage; keep the layout rule.*

**F26 — Fit-study §3's foot-force line is worded loosely; its conclusion
stands.**
"A ~2.3 kg robot puts ~575 gf on each foot standing" — standing on two
feet, each foot carries ~1.15 kgf; 575 gf is the per-*switch* figure if
the two switches in a foot share evenly. Either way the margin over the
SS-10's 150 gf operating force is comfortable (≥ ~3.8× per switch
standing, more in single support). *Fix: wording only.*

**F27 — No document states a runtime expectation; the honest range is
~1–2 hours of activity, not all-day roaming.**
Pass 2e: usable energy ≈ 35 Wh (of 44.4 nominal, keeping a reserve) gives
~0.7–1.0 h walking, ~1.8–2.3 h standing awake, ~5 h electronics-only idle.
That fits the dock-and-return agent pattern well — but nobody has written
it down, and "24/7 home agent" framing elsewhere could be read as
untethered endurance.
*Fix: one line in the README status: expected untethered activity
~1–2 h between charges.*

**F28 — The INA226's supply/logic voltage is unspecified.**
Assembly §2.4 lists its power as "3.3–5 V". CJMCU-class modules often pull
their I2C lines up to their supply; fed at 5 V, that puts 5 V logic on the
Pi's GPIO 2/3. Fed at 3.3 V there is no issue and readings are unaffected.
*Fix: specify "INA226 powered from 3.3 V" in the wiring map (or verify the
owned module's pull-up arrangement when it's measured for F30-register
item R13).*

**F29 — "Charging works robot-on" needs a net-power caveat.**
Assembly §2.8 says charging is robot-off or robot-on. True, but at the
2-point pair's 10 W input ceiling, an awake robot drawing ~8 W of
electronics leaves ~2 W net — charge effectively stalls, and a robot that
docks awake may sit at the dock slowly *losing* ground once its own load
spikes. *Fix: specify the docked state as asleep/servos-off (which the
park-pose design already implies), and let the INA226 net-current check
(F18 thresholds) confirm actual charging.*

**F30 — The shopping list says "SS-10"; the assembly report builds with
the SS-10T.**
List: plain SS-10 pin plunger (SS-10T noted as the same plunger with
solderless terminals). Assembly §2.7: "use the SS-10T quick-connect
variant's crimp terminals". Given the no-solder rule, the T is the right
one to have in hand. *Fix: name the SS-10T explicitly on the list.*

**F31 — The head's back panel is curved; the magnetic half needs a flat
seat.**
All port discussion treats the head's back as a panel; the locked head is
a rounded shell. A flat pogo half on a curved surface rocks, and magnetic
contact force falls off fast with tilt/gap.
*Fix: the head CAD gets a flat circular boss at the port position (ties
into F20's placement decision) — a one-line CAD requirement, free now,
fiddly later.*

---

## Pass 2 — Numbers, recomputed

### 2a. Total mass (bottom-up, mic array in, lidar out)

Starting from the assembly report §3.1 table — its line items re-add
exactly to its stated **2,134 g** (no eye board). Adjusting only for what
that table omits or where the fit study measured differently:

| Adjustment | Δ |
|---|---|
| K230 system as now designed: mini + heatsink 21 g, cameras + FPCs ~8 g (fit-study §2) | +29 g |
| Sipeed R6+1 mic array, external (fit-study §7: ~38 g gross) — **included** | +38 g |
| Magnetic pogo pair, robot half (fit-study §1 est.) | +10 g |
| Cell holders at fit-study weight (15.5 g ea) vs assembly (13 g ea) | +5 g |
| Charge leads up the neck (2 cores, ~1 m total) | +5 g |
| Adapter / BNO055 / INA226 / IP2326 / speaker-amp at fit-study weights vs assembly estimates | −18 g |
| **Recomputed central total** | **≈ 2,203 g** |

The dominant uncertainty is printed parts (assembly allows 550–700 g):
**credible range ≈ 2.15–2.30 kg**, and the assembly report's own top-down
cross-check says 2.38 kg — so plan on **2.2–2.4 kg**. Margin to the 3 kg
ceiling: **~600–800 g**. Verdict: the planning figure in the documents is
honest; the README's 2.1 kg "target" is not (F8). If the parked C1 lidar
(110 g + ~20 g pod) is ever fitted, add ~130 g — still inside the ceiling.

### 2b. Head mass above the pitch axis vs servo torque

Fit-study §2: contents ≈ 220 g; servos 32 + 33 (2 × 55 g) ride above the
head-pitch servo (31) → **~330 g above the pitch axis**; neck-pitch servo
30 additionally carries servo 31 → **~385 g**. Static torque = mass ×
horizontal centre-of-mass offset from the axis:

| Load | @ 4 cm offset | @ 6 cm offset | % of 19.5 kg·cm stall (at 6 cm) |
|---|---|---|---|
| 330 g (servo 31) | 1.32 kg·cm | 1.98 kg·cm | ~10% |
| 385 g (servo 30) | 1.54 kg·cm | 2.31 kg·cm | ~12% |
| 360 g (31, if shell prints +30 g heavy) | 1.44 kg·cm | 2.16 kg·cm | ~11% |

Verdict: **comfortable, but on the corrected 19.5 kg·cm rating** (F7), and
these are *static* figures — walking dynamics and pitch accelerations
multiply them; the policy training, not this table, is where that gets
proven. The shell remains the swing item: weigh it before final assembly.

### 2c. Leg stack minimum vs the 11–13 cm claim and the <15 cm limit

STS3215 body: 45.2 × 24.7 × **35 mm** (fit-study §1). A leg's vertical
chain contains three pitch-axis servo bodies (hip pitch, knee, ankle).
Naive floor: 3 × 35 = 105 mm + foot sandwich ~14 mm (fit-study §3) =
**~119 mm before any link plate, spacer, horn or cable bend**. Real links
need a few mm each. **Practical minimum ≈ 12–13 cm; the <15 cm limit holds
with 20–30 mm of margin; 11 cm does not close** (F9).

### 2d. Trunk interior stack (fit-study §3 figures, re-added)

Interior ≈ 114 × 104 × **89 mm** (120 × 110 × 95 outer, 3 mm walls).
Floor layer: holders 21.4 → planned 24. Mid plate at 26. Pi + cooler ~28
→ top at **54** (not 56 — F25). Servo 30 hangs from the top: 89 − 35 =
bottom edge at **54** — coincident with the Pi stack top, so the
front/rear zone separation in the fit study is not optional. Rear
mezzanine board tops at ~40 → a genuine 14 mm of clearance under the
servo. Footprints: holders 78 deep × 80 across leave a 26 mm-deep rear
power strip (HW-391 48 × 20, IP2326 32 × 18 — both fit it, the 50 × 30 PD
variant does not, as the fit study warns) and a ~34 mm side channel,
into which the rocker switch intrudes 25–28 mm (4–6 mm spare, right-angle
terminals — tight but real). **Verdict: fits, by zone, with zero spare
height in the Pi column.**

### 2e. Battery energy vs load → runtime

Pack: 2S2P, 7.4 V nominal × 6.0 Ah = **44.4 Wh** nominal. Planning usable
≈ **35 Wh** (don't run to the BMS floor; keep a return reserve):

| State | Assumed draw | Runtime on ~35 Wh |
|---|---|---|
| Electronics only (Pi + K230 idle, servos off) | ~7 W | ~5.0 h |
| Standing awake (holding pose, agent active) | 15–20 W | ~1.8–2.3 h |
| Walking, small steps | 35–45 W | ~0.8–1.0 h |
| Walking, hard / frequent recovery | ~55 W | ~0.65 h |

Draws are estimates from the documents' own figures (Pi 5–9 W working,
K230 2–4 W, servo averages scaled from the Duck class). Verdict: **fine
for a dock-based agent that sorties; not an all-day roamer** (F27).

### 2f. Charge time at the 2-point pair (~2 A)

Input ceiling: 5 V × 2 A = **10 W**. At 85–90% charger efficiency,
8.5–9 W reaches the pack → 44.4 Wh ÷ ~8.75 W ≈ **5.1 h of bulk charge,
plus the constant-voltage taper** (the last ~20% is slower): **realistic
full charge ≈ 5.5–7 h** from empty. The fit study's "≈5–6 h" is the
optimistic edge of that range — acceptable for overnight docking, worth
stating as 6–7 h in planning. Corollary: the shopping list's "IP2326
~1.5–2 A output" cannot happen through this pair (F12c) — it could only
occur on the bench with a ≥3 A USB supply straight into the board.

### 2g. Eye spacing / stereo baseline

Fit-study §2: lens centres at **±35 mm → 70 mm baseline** (human spacing
~63 mm; a sound baseline at this scale). Geometry check: head interior
~144 mm wide; K230 mini 60.3 mm wide centred; camera FPC runs of
~40–60 mm including bends — consistent with the 15 cm FPC requirement
(the 6 cm OV5647 cable cannot reach). **Caveat carried from F4/F-register:
the baseline is only real if a matched camera pair exists for the board —
unproven (fit-study §5.1).** Note the old head CAD's eye pits sit at
±22–24 mm (GOAL.md history) — that geometry is *not* the stereo layout
(F22).

### 2h. Dock geometry vs head-centre height

Head centre height = legs + trunk + neck + ½ head height:
120 + 95 + 25 + 55 ≈ **295 mm** (legs 110–130 and head 110–115 give a
~285–305 mm band). The resolutions note's dock arm at **~275–280 mm**
therefore meets the head **below centre** — consistent only with the port
in the lower back panel (F20). Until the port position is drawn, the dock
height is an estimate with ±20 mm of play; the post's ~100 mm standoff
and the arm's ~100 mm reach are the parts to get right in the dock print,
and both are cheap to adjust *after* measuring the finished robot.

---

## Pass 3 — Systems sanity

### 3a. Power tree, end to end

Cells (2S2P) → holders → HW-391 (B+/BM/B− in; P+/P− out, 20 A) → INA226
shunt in P+ → tee to IP2326 (charge) → main switch (20 A) → **(i)** servo
adapter → 14 × STS3215 on raw pack voltage; **(ii)** Pololu buck → 5 V
rail → Pi 5, K230, ear servos, amp, (and now) mic array. Head charge port
→ 2 leads down the neck → IP2326 input; dock side is a plain 5 V supply.

Ratings vs loads, link by link: pack/BMS 20 A — adequate for realistic
walking peaks, below the 35 A all-stall theoretical worst case (F15).
Switch 20 A ✓. XT30 15 A and 18 AWG ~16 A — the chain's weak links after
the BMS (F15). **Holder contact rating: unstated anywhere — the audit's
biggest power unknown (F15). Adapter input rating: unstated (F15).**
Servo bus at raw pack voltage: correct for 7.4 V C001s across the pack's
8.4→6.0 V range; expect noticeably softer servos near empty — another
reason for F18's voltage-based sit-down. 5 V rail: see F14 — split it or
prove it. Pololu output is 5.0 V, not the Pi's nominal 5.1 V: with a
short, thick feed this is workable, but it leaves no drop budget for
thin/long leads — the assembly report's wiring discipline (20 AWG,
short) is load-bearing, keep it. IP2326↔BMS topology: F2. Charge-while-on
net power: F29.

### 3b. The neck bundle — every wire, counted

| Run | Conductors | Notes |
|---|---|---|
| Servo bus to head servos 31–33 | 3 | V+, GND, signal (daisy chain) |
| K230 power + UART | 4 | 5 V, GND, TX, RX (GPIO 14/15) |
| Ear servos (MG90S × 2) | 4–6 | Shared 5 V/GND pair + 2 signals = 4; full home runs = 6 |
| Speaker pair (amp stays in trunk) | 2 | Fit-study §2 arrangement |
| Charge leads (head port → IP2326) | 2 | Added by the resolutions |
| **Total** | **15 (17 worst case)** | Bundle Ø ~11–14 mm |

(Camera FPCs, ear-servo *bodies* and the K230 all live inside the head —
nothing else crosses. The mic array's ribbon stays in the trunk.) Against
fit-study's "~10, ~12 with charge": **undercounted by the ear servos**
(F3). All of these flex through three axes (pitch/yaw/roll); route the
bundle as close to the roll axis as the channel allows, with the slack
loop the assembly report already specifies for the eye wires, a braided
sleeve, and strain relief at both ends. Add a PTC/fuse in the charge pair
at the trunk end: a chafed charge lead is otherwise an unfused 5 V short
inside a moving joint.

### 3c. Data links

- **Pi ↔ K230 UART** (GPIO 14/15, 115200, JSON lines, 1 Hz heartbeat —
assembly §5.3): sound, bandwidth ~2% utilised. Note for the software
list: on the Pi 5, GPIO 14/15 is UART0 — enabling it interacts with the
Bluetooth/console defaults; a known dtoverlay step, not a risk.
- **Pi ↔ servo adapter USB**: sound; cable end-types unverified until
the adapter is in hand (assembly §2.9 item 2 — register R14).
- **I2C**: BNO055 **0x28**, INA226 **0x40** — no conflict, and nothing
else in the current design joins the Pi's I2C bus (cameras hang off the
K230's own buses; the mic array is I2S, not I2C). INA226 logic-voltage
caveat: F28. BNO055 clock-stretching on Pi I2C is handled by the upstream
runtime, per assembly §2.4.
- **I2S**: MAX98357A (out) is proven-pattern; the mic array (in) sharing
the bus is the unverified part — F6.
- **GPIO map check**: I2C 2/3, PWM 12/13, I2S 18/19/21, UART 14/15,
feet 22/27 — no pin is double-booked in any document. ✓

### 3d. Thermal paths

- **Pi 5**: Active Cooler + the fit study's path (intake low at the
sides, exhaust front/top) is coherent — *provided* the back exhaust is
given up, which the external mic array now forces (fit-study §7 noted
this; assembly §4.3 still describes back vents — stale, fold into F11's
fix). Keep the ≥10 mm plenum above the fan (interacts with F25's zero
clearance — the plenum exists only in the front zone, which is where the
fit study puts it; CAD must not let the servo-30 zone steal it).
- **K230**: heatsink mandatory + crown vents (fit study §2/§5.11) ✓;
fan accessory space reserved, fit only if throttling appears. With the
speaker now in the head too (F5), the head's heat sources are K230
(~2–3 W) + speaker bursts — vents sized for the K230 cover both.
- **Buck**: at ~20 W out and ~90% efficiency it sheds ~2 W into the rear
mezzanine — no document mentions it; the mezzanine's air path is the
trunk's, so it's covered by the same vents, but the buck should not be
foam-taped into a dead corner. One line for the CAD notes.
- **IP2326 while docked**: ~1–2 W of heat inside a closed trunk for
6–7 h — acceptable with the vent path; another reason the docked state
should be servos-off/asleep (F29).

### 3e. Mechanical interfaces

- **6804ZZ seats** (Ø32 OD × 20 bore × 7): the redesigns must inherit the
Duck's seat geometry exactly at hip-roll ×2 and head-roll ×1 — assembly
§4.1 says this; the audit underlines it because *both* surrounding
structures (trunk, head) are being redrawn (F1/F22). Freeze the seat
dimensions on the F1 stack drawing.
- **Inserts**: F16 — bosses or nothing.
- **SS-10 feet**: fit verified in fit-study §3 (14 mm sandwich vs
10.2 mm switch; force margin F26). The plunger must stay ~1.5 mm proud
of the hard sole line so it triggers before the 3 mm neoprene fully
compresses — a dimension for the foot CAD, present in the fit study,
absent from the assembly report; carry it across in F11's fix.
- **Ear servos vs head charge port**: no direct clash *if* the port goes
lower-centre (F20): ear pockets live in the top corners, the K230 frame
is central-upper, the port zone is central-lower back. The three
together tile the head's back/interior without overlap — but only the
port placement decision makes that true rather than lucky.
- **Mic array fixings**: F17.
- **Bottom battery door**: the floor plate now carries holders + door
duty; its screws/inserts take the pack's 220 g under walking vibration —
bosses per F16, and threadlocker where metal meets metal (Loctite still
to buy, register R10).

### 3f. Failure modes

- **Flat battery mid-walk** → BMS hard cut → fall. Mitigated only by
F18's thresholds + controlled sit-down. *This is the single most likely
way the robot gets damaged in its first month.*
- **Pi brown-out / hang mid-walk** → controller loss. Mitigated by F14
(rail) + F19 (failsafe behaviour). The servos' own behaviour on bus loss
must be a configured, tested thing — not discovered.
- **Neck cable chafe** → intermittent servo bus (falls), dead eye, or a
charge-lead short. Mitigated by F3's channel, sleeve, strain relief,
PTC on the charge pair (3b).
- **Dock miss** → robot stands at the post, awake, draining (F29's
numbers: it can net-*discharge* while "plugged in" if awake). The
INA226 gives a free success signal: **no charge current within ~30 s of
parking → re-pose, retry, then alert and sit down.** That behaviour
should be specified alongside F18.
- **One battery string's holder contact fails** → the other 2S string
carries everything, holder heating at the surviving contacts (F15's
rating question again). INA226 current + a sudden voltage sag under
load is the detectable signature; a build-time contact-quality check
is the prevention.
- **Servo stall pile-up in a fall** → 35 A theoretical vs 20 A BMS:
the BMS tripping is the *good* outcome (it's the protection working);
the bad outcome is the trip happening through a hot holder spring
first — F15's fuse reorders that.

---

## Open Items Register — everything undecided, unbought or unverified

| # | Item | State in the record | Whose move |
|---|---|---|---|
| R1 | K230 board identity: mini or full-size; in hand or to buy (F4) | Contradictory | **His** — one look / one measurement |
| R2 | Second camera: does the mini's expansion CSI take a GC2093? (fit-study §5.1) | Unproven; do not order camera 2 | **Seller's** (01Studio) — one email; his to send or delegate |
| R3 | HW-391 BMS (balanced, pre-soldered leads variant) | NOT bought (his word, 2026-11-15) | **His** — purchase |
| R4 | Pololu D36V50F5 buck | Decided twice; purchase unconfirmed | **His** — purchase / confirm |
| R5 | "Servos & sensing DONE" — bought, or reviewed/carted? (F13) | Ambiguous | **His** — one status pass |
| R6 | SparkFun BNO086 return/cancel | "Cancel if free" (2026-11-15); outcome unrecorded | **His** |
| R7 | Spare STS3215 ×1 (+ 2 × SS-10T, spare servo leads) | Recommended, not bought | **His** — add to next order |
| R8 | microSD: 2 × 64 GB A2 for the Pi **and** 1 × for the K230 (F23) | Not bought; K230 card missing from list | **His** — purchase |
| R9 | Pi 5 Active Cooler | In weight budget; on README's "still to buy"; unverified in hand | **His** — confirm/purchase (~£5) |
| R10 | Loctite 243 | "Pending" since 2026-11-15 | **His** — purchase |
| R11 | Sole material (3 mm solid neoprene sheet recommended) | TPU ruled out; sheet not bought | **His** — purchase |
| R12 | Soldering iron (for inserts) + M3 insert tips + multimeter | Ownership unconfirmed; tips needed regardless if his iron lacks them | **His** — confirm/purchase |
| R13 | Cell holders, IP2326 (small 32 × 18 variant only), switch, XT30 pairs, Wagos | On the list; no per-item status; **holder current rating unverified (F15)** | **His** — purchase; **assistant/seller** — rating check |
| R14 | USB cable ends, Pi ↔ servo adapter | Verify on adapter's arrival (assembly §2.9) | **His** — check on arrival |
| R15 | Speaker exact size; INA226 module exact height | "Measure on arrival" (fit-study §5.12) | **His** — measure on arrival |
| R16 | Waveshare adapter power-input rating | Unstated in all documents (F15) | **Assistant's** — spec lookup |
| R17 | Mic array ↔ Pi 5 I2S bench test (F6) | Untested | **His bench / assistant-guided** — before back-panel CAD |
| R18 | Lidar (RPLIDAR C1) placement | **Parked by him** — no provision in v1 CAD | **His** — decision, in his own time |
| R19 | Ear microphones | **Parked by him** | **His** — later |
| R20 | Dock (post + arm) build | Geometry waits on F20's port position + finished-robot measurements | Later — after CAD |
| R21 | Low-battery thresholds + dock-miss behaviour spec (F18, §3f) | Unwritten | **Assistant's** — draft for his approval |

## Top risks, ranked

1. **Power loss mid-walk (brown-out or BMS cut-off) → fall → head/K230
damage.** Cheapest mitigation: F18's software thresholds + sit-down
(free), plus F14's rail decision (a ~£5 second buck at most).
2. **Neck harness under-designed (F3) → chafe, intermittent bus, falls
that look like software bugs.** Mitigation: Ø 14 mm channel + sleeve +
strain relief + PTC — a few pounds, decided at CAD.
3. **Charger wired to the wrong BMS node (F2) → no charging, or charging
that bypasses protection.** Mitigation: one datasheet check and a
redrawn diagram — free, before any wire is cut.
4. **Battery-holder contacts overheat at servo peaks (F15).**
Mitigation: rating check (free); if poor, a seller-welded pack (~the
cost of the holders it replaces).
5. **Second camera can't form the stereo pair (R2).** Mitigation: one
email *before* ordering — the face-tracking design survives a mixed
pair; only calibrated stereo depth is at stake.
6. **Mic array won't talk to the Pi (F6).** Mitigation: bench test now
while the fallback (ear mics / £7 I2S MEMS) costs nothing to adopt.
7. **Head shell prints heavy → contents budget breaks (fit-study §2).**
Mitigation: weigh the shell at print time; the listed infill trims
recover 50–100 g.
8. **K230 is the full-size board (R1) → head layout redraw.**
Mitigation: one measurement today instead of a surprise at assembly.

## Overall verdict

The design is **sound in concept and closer than the finding count
suggests** — the architecture, power system, packaging and look all
survive the audit, the mass lands at ~2.2 kg with real margin, and most
findings are paper inconsistencies between documents written on different
nights, not engineering errors. But it is **not yet ready to go to CAD**:
four paper gates must close first — one dimensioned proportion stack to
replace the assembly report's 45–48 cm baseline (F1), the K230's identity
confirmed (F4), the speaker and charger placements stated once, in one
place (F5, F2/F11), and the neck channel sized for the real 15-wire
bundle (F3) — plus the mic-array bench test before the trunk's back panel
is drawn (F6). None of these needs a purchase or a proportion change;
all of them are cheaper this week than after the first print. Parts
buying can proceed in parallel except camera 2 (R2) and the dock (R20).

*— Overnight audit, 2026-11-16. Report only; no design document was
modified.*
