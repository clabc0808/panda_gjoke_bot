# Legged Panda — STS3215 Shopping List (2026-11-15)

Supersedes the 2026-11-14 wheeled GIM4305 list in full. Architecture: Open Duck Mini v2
(legged biped, RL policy), panda body, Raspberry Pi 5 brain, design target 2–3 kg.
Prices are Open Duck BOM / AliExpress-class figures, not live-checked checkout totals.

## Servos & control — reported DONE by him, 2026-11-15
- 14 × Feetech STS3215-C001, **7.4 V** version (19.5 kg·cm, 1:345) — 5 per leg (IDs 10–14 / 20–24), 4 neck/head (IDs 30–33). NOT the lookalike 12 V version. ~€14 ea ≈ €196
- 2 × MG90S 9 g metal-gear micro servos — panda ears (the duck's antenna servos)
- 1 × Waveshare Bus Servo Adapter (A) — Pi ↔ TTL servo bus + servo power distribution

## Sensing — reported DONE by him, 2026-11-15
- 1 × Adafruit BNO055 STEMMA QT / Qwiic breakout (product 4646) + STEMMA QT-to-jumper cable — trunk-mounted, solderless via the QT connectors. This is the Open Duck's native IMU, so the upstream runtime reads it as-is. (His purchased SparkFun BNO086 becomes unnecessary — cancel/return it if free to do so.)
- 4 × SS-10 microswitches — PLAIN pin plunger, no suffix (GL/GL2/GL13 lever variants do NOT fit the press-fit foot pocket; SS-10T is the same plunger with solderless quick-connect terminals) — foot contact, 2 per foot
- INA226 module — ALREADY OWNED (his unit is the −20 A to +20 A variant, confirmed 2026-11-15 — ample for total battery current, and bidirectional so it can also see charge current). Battery voltage/current monitoring for low-battery behaviour. No INA219 needed.

## Power (uprated from the duck for the Pi 5)
- 4 × high-drain 18650 cells — ALREADY BOUGHT (his cells are 30 A continuous-discharge class, confirmed 2026-11-15). Wired 2S2P ≈ 7.4 V / ~6000 mAh ≈ 44 Wh. Match cell voltages before assembly.
- 2 × 2-cell SERIES 18650 holders, 3 wires out (+, −, and the middle point — needed by both the BMS balance wire and the IP2326 charger); NOT the parallel/3.7 V type (or one 2S2P holder with 3 wires out)
- 1 × 2S Li-ion BMS / protection board — HW-391 — **NOT yet bought (his word, 2026-11-15)** — (the balanced 20 A version; the near-identical HW-392 is the UNbalanced one, don't buy that). 7.4 V / 8.4 V, balance function, common charge/discharge port. This is the board the Tnkr Open Duck kit uses. NO-SOLDER NOTE: the bare board has solder pads — order the variant with leads pre-soldered, or ask the seller to solder the 5 leads (B+, B−, BM, P+, P−) before dispatch; joins at his end are then Wago/crimp only.
- 2 × DC-DC buck — CR3520 fixed-5 V module (Taobao ¥27.72 ea ≈ £3) — DECIDED 2026-11-17, SUPERSEDES the Pololu D36V50F5 (£35.82, re-confirmed 2026-11-15): 6–35 V in, 1 V dropout so 5 V holds to the ~6 V pack floor (same as the Pololu, and the bit that killed the Mini560/XL4015), 15 A continuous / 20 A peak, 42×24 mm, 14 g, TVS + short-circuit + over-temp protection, fixed 5 V with no pads to bridge. He will buy 2 — one for the robot, one spare for the bench (~£6 for both vs ~£36–40 for one Pololu). BENCH TEST FIRST when they land: 6.0 V in, 3–4 A load, must hold ≥4.9 V out — the seller's rating is untested, and a droop under load would show as Pi undervoltage throttling. If it fails, fall back to the Pololu D36V50F5 (item #4091) or the D42V55F5 exact-5 V version (item #5571, NOT the 5.3 V variant). Rejected in favour of the fixed CR3520: the adjustable-pad variant (pads marked 3.3/12, 4.2/15, 5/20, 6.2/24 — voltage set by a solder blob across a pair; must be metered before a Pi ever connects). Wires solder onto the big corner pads.
- 1 × USB-C 2S Li-ion charger board — IP2326 type, 8.4 V / ~1.5–2 A, 3 battery wires (+, −, middle point)
- 1 × power switch (2-pin, 2-position rocker, 12 V / 20 A DC automotive type); 1 × 2.1 mm DC barrel jack (OPTIONAL — only if he wants an 8.4 V wall-adapter charge port; USB-C charging via the IP2326 board needs none); XT30 connector pairs ×2, pre-wired 20 cm leads (XT30U fine, intermates); a few Wago 221 lever connectors (solderless wire joins for the BMS/holder wiring)
- Charging (DECIDED 2026-11-15 evening, confirmed in his words): a charging cable provides the power and connects to one half of the pre-wired magnetic pogo pair bought for the wheeled dock; the other half is on the robot, wired to the IP2326 charger's input (charger + its 3-wire pack connection stay inside the robot unchanged). Halves touch → it charges; separate → it stops. No dock electronics — just a printed holder keeping the cable-side half at the right height. Magnets self-align for hand-parking now; self-docking (robot walks/sits onto it) is a later job once walking works. USB-C straight into the IP2326 remains a bench fallback. Open check: whether his pair is the 4-pin version (~4 A, full-speed charge) or the 2-pin (~2 A, slower).

## Voice
- 1 × MAX98357A I2S amp + bundled waterproof 4 Ω 3 W cavity speaker + PH2.0-to-Dupont lead (¥35.9 set, selected 2026-11-14)
- 1 × **ReSpeaker Flex XVF3800 Circular-4** mic array (Seeed) — DECIDED 2026-11-17, **BOUGHT 2026-11-17** (Taobao, ¥365.5): ring on the head, processor board in the trunk, USB to the Pi. Replaces the Sipeed R6+1 plan (Sipeed stays owned, as a spare). The **no-XIAO** version (ships with USB firmware).

## Structure & hardware — status 2026-11-15 (his word): #1, #2, #3, #6 IN CART; #4 pending; #5 ruled out
- 1 × pack (50) M3 brass heat-set inserts — M3×5×5 size chosen. IN CART
- 1 × M3 socket-head screw assortment (6 / 8 / 10 / 12 / 16 / 20 mm, ~50 each, stainless, cup-head) + M3 nuts. IN CART
- 3 × 6804ZZ ball bearings (20 × 32 × 7 mm) — hip-roll ×2 + head-roll ×1 support. CORRECTED by him 2026-11-15 (the cart originally held 2 × 608ZZ, 8 mm bore — wrong, per the overnight assembly study §4.1) and bought with the tools below.
- 1 × Loctite 243 threadlocker (metal-to-metal screws only) — PENDING (he will need it)
- Foot soles: TPU 95A spool RULED OUT by him 2026-11-15 — any soft, grippy material firmly attached does the job (rubber pads/sheet etc.). (PLA shells print from stock on hand: black / white / pink PLA Matte)
- Cable sheath / spiral wrap, Dupont jumper wires, short USB cable (Pi ↔ servo adapter). IN CART

## Already owned — do NOT buy
Raspberry Pi 5 8GB · Bambu Lab A1 mini · Sipeed mic array · PLA filament · lidar units (not fitted to the v1 walker — weight)

## Deliberately OFF this list
- Eyes/camera — DECIDED 2026-11-15: Kendryte K230 with two small cameras (stereo pair at the panda eyes), UART to the Pi. IMX219-83 stereo plan superseded. UPDATE 2026-11-16: his board is the **full-size CanMV K230 (85 × 56 mm, 31 g), already owned** — built-in GC2093 + 2 expansion ports (CSI0/CSI1, OV5647 documented). Eye cameras BOUGHT 2026-11-19: FOUR fixed-focus OV5647 modules — a matched 72° pair and a matched 100° pair (fixed cables, ~10 cm as bought; the in-head run needs ~9–12 cm, so a bench dry-fit confirms reach). Eye pair = the 100° pair (his pick 2026-11-19); the 72° pair is the alternate/spare set. His earlier autofocus OV5647 pair is NOT usable here — the K230 cannot drive the focus motor, and they are Pi 15-pin.
- SG90 pan/tilt kit + M5Stack Unit 8Servos — replaced by the 4 neck/head STS3215s + ear servos
- CAN HAT, GIM4305/GDZ34, 6S pack, wheels, XL4015 dock charger, 24 V dock — wheeled plan, gone
- Cliff sensors — no equivalent on a walking build yet; stair safety is a later policy/sensor problem

## Rough total
≈ £380–450 all-in (duck core BOM is €398.10; this swaps the Pi Zero for the owned Pi 5,
doubles the cells, uprates the 5 V rail, and adds the voice kit).
