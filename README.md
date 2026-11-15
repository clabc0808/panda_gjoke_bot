# Panda Robot — legged home agent on STS3215 servos

A small **walking panda robot** that lives in the house: it walks, sees, hears,
talks, and runs a home-agent brain. Built on the **Open Duck Mini v2**
architecture (Antoine Pirrone's open-source biped) with a panda body, a
Raspberry Pi 5 brain, and a Kendryte K230 vision co-processor as its eyes.

Design mass target ~2.1 kg (3 kg hard ceiling — the servo torque budget).
This repo holds the proposal, the current build status, the shopping list,
and the detailed assembly/wiring design.

## The idea

- **Body:** Open Duck Mini v2 legged platform — 14 bus servos, RL walking
  policy retrained for the panda's mass distribution (duck policies do not
  transfer directly).
- **Brain:** Raspberry Pi 5 (8 GB, already owned) running the Hermes home
  agent (OpenClaw fallback). LLM via OpenRouter; speech-to-text on the
  household Jetson (Whisper). The walking/safety layer runs independently of
  the agent — if the brain crashes, the legs don't.
- **Eyes:** Kendryte **K230 with two small cameras** — a stereo pair at the
  panda's eye positions. Face detection/tracking runs on the K230 itself; it
  sends results to the Pi over UART, so vision costs the Pi nothing.
- **Voice out:** MAX98357A I2S amp + 4 Ω 3 W waterproof speaker.
- **Power:** 4 × high-drain 18650 in 2S2P (7.4 V, ~44 Wh), HW-391 balanced
  BMS, Pololu 5 V buck for the Pi. Charging is **magnetic, not plug-in**: a
  pre-wired 4-pin magnetic pogo pair (already bought) is the inlet — the dock
  half is fed by a plain 5 V supply, the robot half feeds the IP2326 charger
  inside the robot. Hand-park to charge for now; self-docking comes later.

## Key design decisions (all settled 2026-11-15)

| Area | Decision |
|---|---|
| Servos | 14 × Feetech STS3215-C001 **7.4 V** (5 per leg, 4 neck/head) + 2 × MG90S ear servos |
| Servo control | Waveshare Bus Servo Adapter (A), USB/UART from the Pi |
| IMU | Adafruit BNO055 (STEMMA QT 4646) in the trunk — the Duck's native IMU |
| Feet | 4 × SS-10 pin-plunger contact switches, 2 per foot |
| Battery monitor | INA226 ±20 A module (already owned) |
| 5 V rail | Pololu D36V50F5 buck (full pack range usable, no early cutoff) |
| Eyes | **K230 + 2 cameras** (stereo), UART results link to Pi GPIO 14/15 |
| Charging | Magnetic pogo pair inlet (bought for the old wheeled dock, reused); hand-park now, self-dock later |
| Not fitted (v1) | Lidar (weight), cliff sensors |

Superseded plans, for the record: an earlier wheeled self-balancing design
(2 × GIM4305 + GDZ34, 6 S pack, CAN HAT) was dropped on 2026-11-15 in favour
of the legged build; a Waveshare IMX219-83 stereo board for the eyes was
dropped because it does not physically fit the 104 mm head — the K230 pair
replaces it.

## Status — what we have now (2026-11-15)

- **Bought:** 4 × high-drain 18650 cells (30 A class); 3 × 6804ZZ bearings
  (hip-roll ×2, head-roll ×1) + tools.
- **Reported done / in cart (checkout pending):** the 14 STS3215 servos,
  MG90S ears, Waveshare adapter, BNO055 + cable, SS-10 foot switches,
  M3 heat-set inserts (M3×5×5 ×50), M3 screw assortment, cable wrap/Dupont/USB.
- **Still to buy:** HW-391 BMS (pre-soldered-leads variant), Pololu buck
  (Pi Hut), 2S cell holders (series, 3-wire), IP2326 charger board, rocker
  switch, XT30 pre-wired pairs, Wago 221s, Loctite 243, **K230 board + 2
  cameras**, 2 × 64 GB A2 microSD, Pi 5 Active Cooler, sole material
  (3 mm neoprene sheet recommended).
- **Already owned:** Pi 5 8 GB, Bambu Lab A1 mini (prints the shells),
  INA226, Sipeed mic array, PLA (black/white/pink matte).
- **Design:** assembly + wiring design complete (see
  [assembly-design.md](assembly-design.md)); weight estimate 2.2–2.4 kg.
  Panda trunk is a redesign around the Pi 5 + pack; legs/hips/neck carry
  over from the Duck; head adapts the existing panda v3.5/v3.6 CAD.
- **Not started:** panda CAD changes, MJCF model, walking-policy training,
  K230 firmware (CanMV), Pi↔K230 protocol.

Full item-by-item list with specs and statuses:
[shopping-list.md](shopping-list.md)

## Still open

- Microphone choice (report §6): I2S MEMS in the head vs the owned Sipeed
  array in the trunk vs v1 ships without a mic.
- Spares: 1 spare STS3215 recommended before first walks.
- Stair/cliff safety: no sensor solution on a walking build yet — a later
  policy/sensor problem, flagged not solved.

## Files

- `README.md` — this proposal + status (kept current)
- `shopping-list.md` — full parts list with specs, traps and buy status
- `assembly-design.md` — detailed assembly sequence, wiring/power map,
  weight budget, gap analysis, K230 evaluation (2026-11-16 report)
