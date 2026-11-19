# Movement Physics — panda leg check: levers, balance, joint speed

**Date:** 2026-11-19. **Status:** paper calculation from design values —
no CAD, no simulation. It answers one question: do the ~13 cm stacked-servo
legs work on lever and balance physics, before any walking-policy work?
**Verdict: yes, with margin — and the tightest joint is hip roll, not the
knee.** Companion to `assembly-design.md` §1.3/§3 and `fit-study.md`.

> **Correction to the 2026-11-19 chat estimate:** the quick comparison given
> in chat earlier that day used MicroDuck's figures (0.8 kg, XL330 servos).
> MicroDuck is a different robot. The panda's actual design basis, the Open
> Duck Mini v2, totals **2,107 g in its own MuJoCo model** and walks on
> these same STS3215 legs (assembly-design §3: "the panda's legs *are* Duck
> legs — same servos, same geometry"). The precedent is therefore stronger
> than the chat estimate suggested: a same-weight robot already walks on
> this exact leg. The 7.7 kg·cm knee figure quoted in chat is the
> *unbalanced* case below (§2.3), not the balanced crouch (§2.2).

## 1. Inputs

| Input | Value | Source |
|---|---|---|
| Total mass (planning) | 2.20 kg (sensitivity run at 2.40) | Assembly §3 budget ≈2,134 g + K230/Flex additions |
| Mass above the legs | 1.40 kg | §3 budget minus leg items |
| Per leg, below hip pitch | 0.40 kg (thigh group 0.21 + shank/foot 0.19) | 5 × STS3215 @ 55 g (Feetech) + horns/cables/prints |
| Body CoM above hip axis | 95 mm (upright) | §1.3 layout: battery low, Pi above it, head ~0.45 kg |
| Thigh (hip→knee) / shin (knee→ankle) | 55 mm / 55 mm; hip height standing 122 mm | Fit-study leg stack (~120 mm) |
| Foot | 95 mm long × 55 mm wide; ankle 38 mm from heel (toe margin 57 mm) | Duck foot proportions — **assumption, replace with CAD** |
| Hip spacing | 90 mm | **Assumption, replace with CAD** |
| STS3215-C001 stall / rated torque | 19.5 / 5.0 kg·cm @ 7.4 V; 16.5 / 4.0 @ 6.0 V | Feetech spec A/1 (2023-06-23) |
| No-load speed | 0.192 s/60° = 52 RPM @ 7.4 V (42 @ 6 V) | Feetech spec A/1 |
| Torque constant Kt | 7.8 kg·cm/A; rated current 0.65 A; stall 2.5 A | Feetech spec A/1 |

Method: planar statics — torque about a joint = Σ m·g·(horizontal distance
from the joint to each mass beyond it). Walking transients allowed for with
a ×1.4 dynamic factor (stated wherever used). Balance via the inverted-
pendulum (capture point) model. All arithmetic scripted, not hand-computed.

## 2. Lever results — sagittal plane (per stance leg, kg·cm)

| Pose | Hip pitch | Knee | Ankle |
|---|---|---|---|
| Standing straight, 2 legs | ~0 | ~0 | ~0 |
| Crouch 35°, 2 legs | 0.2 | 1.6 | 0.2 |
| Crouch 25°, 1 leg | 0.2 | 2.3 | 0.2 |
| **Crouch 35°, 1 leg** | 0.3 | **3.2** | 0.3 |
| Crouch 45°, 1 leg | 0.4 | 4.0 | 0.4 |

### 2.1 Reading
With the body balanced over the foot, the knee is the only sagittal joint
doing real work, and even single-leg crouching stays **under the 5 kg·cm
rated torque**. Hip pitch and ankle are nearly unloaded in a balanced pose
(the ankle's real job is correction, not holding).

### 2.2 Why the knee number is small
In a balanced crouch the knee sticks only ~17 mm ahead of the hip–ankle
line (55 mm links, 35° flex), so the body's weight acts through a ~17 mm
lever at the knee. Short links are exactly why this works.

### 2.3 The design case is the *unbalanced* one
Stepping, turning and recovering from a nudge put the body CoM ~3.5 cm off
the knee axis: knee torque ≈ **7.7 kg·cm transient** (≈40% of stall at
7.4 V, ≈47% at 6 V). Above rated, so it cannot be held all day — but
walking never holds it; it passes through. At 2.40 kg this scales to
~8.5 kg·cm. This, not the static pose, is what the 3 kg ceiling protects
against, and it is why the planning figure stays 2.2–2.4 kg.

## 3. The tightest joint: hip roll (frontal plane)

Shifting the body sideways onto one foot, pelvis level, loads the stance
hip-roll servo with the full above-leg mass on a ~45 mm lever (half the
hip spacing):

| Pelvis list toward stance side | Hip-roll torque |
|---|---|
| 0° (level) | **8.1 kg·cm** |
| 6° | 5.1 kg·cm |
| 10° | 3.1 kg·cm |

**Finding:** the gait must list (lean the pelvis a few degrees toward the
standing leg) — which Duck-style policies do naturally — or the hip-roll
servos sit at ~8 kg·cm every step and run hot. With a modest list they are
back at rated torque. This is a gait requirement, not a hardware change.

## 4. Balance

- **CoM height ≈ 158 mm** (of 350 mm total) — low for its height, thanks
  to the battery and servos sitting in the legs and trunk floor.
- **Inverted-pendulum rate** ω₀ = √(g/h) ≈ 7.9 rad/s (period ~0.8 s):
  steps must land roughly every half-period, i.e. a cadence of ~1.5–2
  steps/s is the natural gait, matching §5.
- **Capture point** (where the foot must land to stop a fall) at the
  planned speeds: 13 mm ahead at 0.10 m/s, 25 mm at 0.20 m/s — against a
  **57 mm toe margin**. At least 2× margin at twice the planned speed.
- **Static tip-over angles:** 20° forward, 14° backward, 25° sideways on
  two feet, **10° sideways on one foot** — sideways single-support is the
  weak direction, as on every biped; the defence is stepping sideways to
  catch, which the capture-point margin allows.

## 5. Joint speed vs cadence

Servo no-load 52 RPM; a loaded DC servo keeps roughly (1 − torque/stall)
of that, so ~31 RPM available at 40% load.

| Demand | Peak speed | Verdict |
|---|---|---|
| Hip swing ±20°, cadence 1.5 steps/s | 16 RPM | Comfortable |
| Hip swing ±20°, cadence 2.0 steps/s | 21 RPM | OK |
| Hip swing ±20°, cadence 2.5 steps/s | 26 RPM | Near the loaded ceiling |
| Knee fold 40° in 0.25 s (swing) | 27 RPM at ~0.5 kg·cm load | OK (near no-load conditions) |

Step length at ±20° swing ≈ 2 × 120 mm × sin 20° ≈ 82 mm per stride →
**~0.08–0.10 m/s at cadence 2**. Slow, as accepted when the short legs
were locked. Cadence ceiling ~2–2.5 steps/s is set by the 1:345 gearing,
not by torque.

## 6. Current and heat

- Worst sustained joint (knee or hip roll at ~8 kg·cm) draws ≈ 8 / 7.8 ≈
  **1.0 A** against a 0.65 A rated current — transient only; walking
  alternates the load between legs at ~50% duty.
- Whole-bus draw while walking ≈ 3–5 A typical, ~8 A peaks — well inside
  the 20 A BMS and the 2S2P 30 A-class cells.
- The STS3215 reports temperature, load and current over the bus, so the
  software can watch the hip-roll and knee servos and slow the gait if
  they climb — a software guard, planned from the start.

## 7. Verdict and conditions

The 13 cm legs work on paper with real margin:

1. Balanced-pose torques sit **under rated**; worst transients reach
   ~40–47% of stall.
2. Balance margins are ≥2× at the planned speed; CoM is low.
3. Cadence ~2 steps/s is inside the servos' loaded speed; ~0.1 m/s.

Conditions the numbers depend on:

1. **Mass ≤ ~2.4 kg.** Every figure scales with mass; 3 kg remains a
   ceiling, not a target.
2. **The gait lists a few degrees** toward the stance leg (§3), and is
   trained for this geometry — the policy stage still has to happen; this
   check says the hardware gives it a fair chance, not that walking is
   proven.
3. Two inputs are assumptions pending CAD (foot size, hip spacing);
   re-run this calc when the CAD fixes them. The hip-roll result is the
   one most sensitive to hip spacing.
