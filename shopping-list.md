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
- 1 × DC-DC buck — Pololu D36V50F5 (item #4091), cheapest found RoboSavvy £35.82 — HIS DECISION 2026-11-15: he doesn't want to waste pack capacity, so no Mini560/7.0 V-cutoff trade. Fixed 5 V, 5.5 A, works from 5.5 V input so the full pack range is usable. Wires attach via solder holes (accepted with this choice). ACCEPTABLE/BETTER ALTERNATIVE (verified 2026-11-15): Pololu D42V55F5 (item #5571) — newer same-form-factor successor, fixed 5 V, 6 A, input 5–60 V, 3% accuracy, ~0.4 mA quiescent; must be the exact 5 V version, NOT the 5.3 V D42V55F5.3. No UK stock found at Pi Hut/RoboSavvy for the exact 5 V version at check time. RE-CONFIRMED 2026-11-15 16:40 after a full afternoon of alternatives (Waveshare UPS HAT (B) two-pack route, B560C £3.94 bench gamble, 3.7→7.4 V servo boost, M5Stack Servo2, feeding the 2S pack into a Waveshare UPS module — all checked and closed): "i guess i will just pay the money for the pololu one".
- 1 × USB-C 2S Li-ion charger board — IP2326 type, 8.4 V / ~1.5–2 A, 3 battery wires (+, −, middle point)
- 1 × power switch (2-pin, 2-position rocker, 12 V / 20 A DC automotive type); 1 × 2.1 mm DC barrel jack (OPTIONAL — only if he wants an 8.4 V wall-adapter charge port; USB-C charging via the IP2326 board needs none); XT30 connector pairs ×2, pre-wired 20 cm leads (XT30U fine, intermates); a few Wago 221 lever connectors (solderless wire joins for the BMS/holder wiring)
- Charging (DECIDED 2026-11-15 evening, confirmed in his words): a charging cable provides the power and connects to one half of the pre-wired magnetic pogo pair bought for the wheeled dock; the other half is on the robot, wired to the IP2326 charger's input (charger + its 3-wire pack connection stay inside the robot unchanged). Halves touch → it charges; separate → it stops. No dock electronics — just a printed holder keeping the cable-side half at the right height. Magnets self-align for hand-parking now; self-docking (robot walks/sits onto it) is a later job once walking works. USB-C straight into the IP2326 remains a bench fallback. Open check: whether his pair is the 4-pin version (~4 A, full-speed charge) or the 2-pin (~2 A, slower).

## Voice
- 1 × MAX98357A I2S amp + bundled waterproof 4 Ω 3 W cavity speaker + PH2.0-to-Dupont lead (¥35.9 set, selected 2026-11-14)

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
- Eyes/camera — DECIDED 2026-11-15: Kendryte K230 with two small cameras (stereo pair at the panda eyes), UART to the Pi. IMX219-83 stereo plan superseded. (K230 board + cameras not yet on the buy list — to add.)
- SG90 pan/tilt kit + M5Stack Unit 8Servos — replaced by the 4 neck/head STS3215s + ear servos
- CAN HAT, GIM4305/GDZ34, 6S pack, wheels, XL4015 dock charger, 24 V dock — wheeled plan, gone
- Cliff sensors — no equivalent on a walking build yet; stair safety is a later policy/sensor problem

## Rough total
≈ £380–450 all-in (duck core BOM is €398.10; this swaps the Pi Zero for the owned Pi 5,
doubles the cells, uprates the 5 V rail, and adds the voice kit).
