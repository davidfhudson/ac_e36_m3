# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Mod Is

A simulation of David Hudson's real-world BMW E36 M3 endurance race car in Assetto Corsa. The base mod is Ceky Performance's E36 M3, progressively calibrated against real dyno data and VBOX telemetry. **The goal is accuracy to the real car, not maximum sim performance.** If the sim is significantly faster than the real car, something is wrong.

**Current model: v1.6 (24 Sep 2026).** Calibrated against Silverstone GP test day 21 Sep 2026 (coastdowns, same-driver laps) and Donington National 19 Jul 2026. Same driver, same tyre, same day: sim 2:25.4 vs real 2:23.2 at Silverstone with the remaining gap located in three braking corners (driver commitment), lateral-G envelope matched to 3 %.

---

## Real Car Specification (as modelled)

| Item | Detail |
|------|--------|
| Engine | BMW S54, VANOS removed, Geoff Steel carbon airbox |
| ECU | DTA S80 Pro, 95 % throttle cap (cable) for class compliance |
| Weight | ~1,211 kg car (ballasted to 1,248 kg at 50 L for 750MC 300 bhp/t) + 75 kg driver → `TOTALMASS=1286` |
| Gearbox | Drenth DG400 6-speed sequential, dog engagement, 6th direct |
| Differential | Drexler plate LSD, no preload, 45° ramps |
| Final drive | 4.10 (4.45 selected for 2027, not fitted) |
| Dampers | Moton 3-way adjustable — **not modelled**, Ceky values retained |
| Aero | Geoff Steel fixed splitter, adjustable rear wing (6.5°) + 5 mm gurney (v17) |
| Tyres (dry) | Vitour Tempesta P1 245/40/18 on 9J (default, index 3); Yokohama A052 (index 1) |
| Tyres (wet) | Dunlop Racing Wets 245/45/18 (index 0) |
| Fuel cell | 110 L; default load 50 L |
| Brakes | OBP dial, ~50 % pressure split ≈ 67 % torque front |
| Class | 750MC Class A 2027 (300 bhp/t): cap target 355 bhp at 1,185 kg |

### Alignment defaults (real setup v17, 21 Sep 2026)

AC defaults are symmetric per axle. `suspensions.ini` carries the RIGHT-side values; set the left side in the setup screen for clockwise circuits.

| | Left (setup screen) | Right (file default) |
|--|---|---|
| Front camber | −5.0° | −4.0° |
| Rear camber | −3.25° | −2.75° |
| Front toe | 2 mm total toe-out (≈ −0.09°/wheel) | |
| Rear toe | 2 mm total toe-in (≈ +0.09°/wheel) | |
| Caster | 7° | |

### Ride height

Real: splitter 80 mm, rear bumper 270 mm, 70 mm rake (215/285 at arches). AC displays ride height at the axles; the front aero element sits 0.53 m ahead of the front axle so the equivalent displayed heights are **~94 mm front / ~164 mm rear**. `ROD_LENGTH` defaults (0.072 / 0.113) give this; `setup.ini` front minimum was lowered to 70 mm to allow it. The front undertray ground-height LUT is steep (×1.3 at 80 mm, ×0.8 at 100 mm) — ride height is a first-order aero-balance lever in this mod.

### ARB / springs

ARB front middle (54188), rear soft (18922) — matches v17. Spring wheel rates (31 / 36 N/mm) are Ceky's; the real car runs ~106 N/mm front wheel rate (120 N/mm coil, MR ~0.94) and an unmeasured rear motion ratio. **Do not use the sim for spring or damper conclusions.**

---

## Engine — power.lut

`power.lut` is a **wheel-torque** curve (AC applies no drivetrain loss). Derived from the MJB dyno trace (25 Jan 2026, 373.7 hp flywheel at 95 % cap, peak ~7,350 rpm, falling by 7,500) × **0.90**. The 0.90 was backed out from real vs sim WOT acceleration at Silverstone with the coastdown-measured drag: real wheel power sits 3–11 % above dyno × 0.85 across 5,750–7,750 rpm, i.e. ~10 % driveline loss, not 15 %.

- `LIMITER=7950` — the real soft cut overshoots to 7,900–8,070 rpm (Silverstone S3/S9). 7,800 is the shift target, not the cut.
- Curve holds to 7,950 then drops; the real engine pulls cleanly to 7,750.
- `COAST_REF` 130 Nm at 7,300 unchanged.

### Rebuilding after a new dyno run
1. Read flywheel torque (ft·lb) at each rpm point from the trace.
2. Nm = ft·lb × 1.35582.
3. Wheel = flywheel × 0.90 (validated loss factor; re-check with an acceleration comparison after any driveline change).
4. Extend below 2,500 with a smooth taper; above the limiter drop to 0 by ~8,500.
5. Verify with a same-driver VBO: matched-gear WOT acceleration at 150–210 km/h should sit within ±3 % once drag is right.

---

## Gearing — SETTLED (Sep 2026)

Effective ratios **include** the 24/25 constant mesh (1.0417); AC's `GEAR_x` wants the effective value. Measured by steady-state rpm/speed regression and the rpm drop across ~50 upshifts, three sessions, two circuits, three drivers, agreeing to 0.4 %.

| Gear | Effective (drivetrain.ini) | Pair | km/h per 1,000 rpm |
|------|------|------|------|
| 1st | 2.394 | 2.286 | 12.35 (±1 %, wheelspin) |
| 2nd | 1.829 | 1.765 | 16.16 |
| 3rd | 1.493 | 1.444 | 19.80 |
| 4th | 1.236 | 1.182 | 23.91 |
| 5th | 1.091 | 1.042 | 27.09 |
| 6th | 1.000 | direct | 29.56 |
| Final | 4.10 | | |

Master formula: km/h per 1,000 rpm = 29.559 / effective ratio (measured circumference ÷ final drive = 0.49265 m). With P1 `RADIUS=0.3281` the sim reproduces real rpm/km/h in every gear to 0.5 %.

**The earlier claim that "the 4.1 final already accounts for the constant mesh" was wrong** — it was fitted with an assumed tyre radius. The old set (2.500 / 1.824 / 1.474 / 1.250 / 1.091) is the car's setup sheet from a different box and is not what is fitted.

---

## Aero — measured drag, inferred lift

Silverstone coastdowns 21 Sep 2026 (Phil Keen, clutch in, slope- and density-corrected, 142–184 km/h) vs the sim's own coastdowns: real total drag = **1.134 × sim**, uniform across speed → aero elements ×1.16. Real wing-off = −10 to −12 % of total drag; gurney = +3.6 %.

| Element | Change | Basis |
|---|---|---|
| BODY CD_GAIN | 1.103 → 1.155 | measured |
| WING_4 CD/CL LUTs | ×10.9 | wing was 1 % of drag in the mod, 11 % in the car |
| WING_4 CD_GAIN | 0.735 (gurney) / 0.540 (no gurney) | measured |
| WING_4 CL_GAIN | 1.06 (gurney) / 0.85 (no gurney) | **ASSUMED** — L/D kept at Ceky's; downforce unmeasured |
| ANGLE | 6 ≡ real 6.5° | datum arbitrary; use deltas |

Sim coastdown reference (ρ = 1.225): decel = 0.191 + 0.000414·v² m/s² before the ×1.134 — expect 0.216 + 0.000480·v² after. Wing angle costs ~0.9 % of total drag per degree.

**Open:** rear wing downforce. A real measurement (rear ride height or damper pots at two speeds) would close it. Until then, high-speed balance in the sim is provisional.

---

## Tyres

| Index | Compound | Radius | DX/DY_REF | Ideal psi | Notes |
|---|---|---|---|---|---|
| 0 | Dunlop Wet | 0.3389 | 1.34 | 32 | Ceky-derived |
| 1 | Yokohama A052 | 0.3266 | 1.280 | 29 | validated Mar 2026 (Brands) |
| **3 (default)** | Vitour P1 245/40 | 0.3281 | 1.300 | 28 F / 30 R | grip validated by lat-G envelope (sim 1.27 vs real 1.23 p95) |
| 4 | Vitour P1 265/35 on 9J | 0.3228 | 1.326 | 28 / 30 | **test compound — grip, RATE, FZ0 are hypotheses** |

- `BRAKE_DX_MOD=0.00` on A052 and P1 (was 0.08): sim peak braking was 1.4 g vs real 1.19–1.26.
- P1 thermal (`FRICTION_K=0.055, ROLLING_K=0.22, COOL_FACTOR=2.0`) validated 24 Sep: sim 60–80 °C at Silverstone vs real pyrometer 51–80 °C. Ceky's values ran 40–49 °C and cost ~5 % grip through the performance curve. Set session track temperature to the real day's (e.g. 32 °C).
- Cold defaults 20 F / 22 R → ~28 / 30 hot.

---

## Brakes

`MAX_TORQUE=1500` per wheel (was 3600). At 3600, 28 % pedal produced a real-car full stop and the driver arrived under-speed at every braking corner. 1500 gives ~1.4 g at full pedal so the pedal is lock-limited like the car. `FRONT_SHARE=0.67` is torque split; the car's 50 % pressure dial ≈ 60–67 % torque. Setup bias range 50–70.

---

## Key Configuration Files

| File | What it controls / v1.6 state |
|------|-----------------|
| `data/car.ini` | `TOTALMASS=1286`, `INERTIA=1.39,1.10,4.02`, fuel default 50 L |
| `data/engine.ini` | `LIMITER=7950` |
| `data/power.lut` | wheel torque = dyno × 0.90 |
| `data/drivetrain.ini` | settled effective ratios, `FINAL=4.1`, diff 0.30 / 0.15 / 0 (estimates) |
| `data/suspensions.ini` | v17 right-side geometry, rod lengths for 94/164 mm |
| `data/tyres.ini` | four compounds, P1 default, P1 thermal calibrated |
| `data/aero.ini` + `wing_rear_AOA_*.lut` | measured drag, assumed lift |
| `data/brakes.ini` | `MAX_TORQUE=1500` |
| `data/setup.ini` | height 1 mm steps, front min 70 mm, rear camber to −3.5, bias 50–70, rear wing 0–16 |

---

## Validation Record

| Date | Circuit | Check | Result |
|---|---|---|---|
| Mar 2026 | Brands Indy | v1.2 vs Nigel 52.9 (225 A052) | sim 52.4 |
| 19 Jul 2026 | Donington Nat | rpm/km/h per gear, WOT accel (David) | ratios to 0.5 %, accel +2–10 % → fixed by drag |
| 21 Sep 2026 | Silverstone GP | coastdown, vmax, accel vs power | drag ×1.134; power ×1.06; vmax 219 sim vs 221.8 real |
| 24 Sep 2026 | Silverstone GP | same driver, P1, v1.5 | lat p95 1.27 vs 1.23; temps 60–80 vs 51–80 °C; 2:25.4 vs 2:23.2 |

---

## Known Assumptions (in order of likely impact)

1. Rear wing downforce (CL) — inferred from drag at Ceky's L/D.
2. Springs and dampers — Ceky values; front wheel rate ~⅓ of real.
3. Differential lock — POWER 0.30 / COAST 0.15 estimates; sim shows ~17 % rear slip at hairpin exits.
4. Brake bias mapping — set by feel.
5. 265/35 compound — geometry real, everything else hypothesis.

---

## Development Principles

1. One change at a time — in the sim and on the real car.
2. Every change needs a hypothesis and a measurable validation target, written before the run.
3. Validate against telemetry traces (rpm/speed per gear, WOT acceleration by speed band, lateral-G envelope, tyre temperatures, corner apex speeds), not lap time alone.
4. Same driver, same tyre, same day is the only clean sim-vs-real comparison; between-driver comparisons measure the driver.
5. Physics files are the mod; setups are the car's configuration. Don't bake a circuit-specific setup into the physics.

---

## LUT File Format

`.lut` files map input→output (`input|output` per line). Rear-wing LUTs were rescaled ×10.9 in v1.4; the body/undertray LUTs are Ceky's.

## Workflow

Physics changes need a full AC restart. If a `data.acd` exists in the car folder AC reads it and ignores the loose `data\` folder — rename it during development. Content Manager's "Replace sound" can swap the borrowed Hemi sound bank (which stops rising at ~6,500 rpm) for a higher-revving donor. Large binaries (`.kn5`, `.bank`, `.dds`) are tracked via Git LFS.
