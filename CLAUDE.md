# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Mod Is

A simulation of David Hudson's real-world BMW E36 M3 endurance race car in Assetto Corsa. The base mod is Ceky Performance's E36 M3, progressively calibrated against real dyno data and VBOX telemetry. **The goal is accuracy to the real car, not maximum sim performance.** If the sim is significantly faster than the real car, something is wrong.

---

## Real Car Specification

| Item | Detail |
|------|--------|
| Engine | BMW S54, VANOS removed, Geoff Steel carbon airbox |
| ECU | DTA S80 Pro, 95% throttle cap for class compliance |
| Weight | 1,175 kg dry / 1,250 kg with driver (sim uses 1,250 kg) |
| Gearbox | Drenth 6-speed sequential, dog engagement |
| Differential | Drexler plate-type LSD, no preload, 45° ramp angles |
| Final drive | 4.1:1 (see gearing note below) |
| Dampers | Moton 3-way adjustable |
| Aero | Geoff Steel fixed splitter, adjustable rear wing |
| Tyres (dry) | Yokohama A052 245/40/18 on 9J rims |
| Tyres (wet) | Dunlop Racing Wets 245/45/18 |
| Fuel cell | 110 litres |
| Brakes | Bias adjustable via rotary dial, ~65% front (unconfirmed) |
| Class | Pro-B (271–310 bhp/tonne at flywheel, at 1175 kg dry: ceiling = 364.2 bhp) |

### Alignment — Shakedown Baseline

| | Front L | Front R | Rear |
|--|---------|---------|------|
| Camber | −4.7° | −3.5° | −2.5° both |
| Toe | 2mm total toe-out | | 2mm total toe-in |
| Caster | 7° | | |

The asymmetric front camber is **deliberate** — biased toward right-hand circuit use. Rear toe moves toward neutral under squat/load.

### Dampers (Moton 3-way, clicks from full soft)
- Front: LS Bump 2, HS Bump 6, Rebound 6
- Rear: LS Bump 2, HS Bump 6, Rebound 6

### ARB
- Front: 3 positions — was full stiff at shakedown (primary understeer suspect), to be moved to middle
- Rear: 2 positions, currently soft

### Ride Height (with 245/40/18)
- Front: 215 mm, Rear: 285 mm — **70 mm rake**

---

## Dyno Data

- **Facility:** MJB Sports Cars Ltd / Dyno Developments
- **Flywheel (uncorrected):** 373.69 hp (already with 95% throttle cap applied)
- **Wheel figures:** 315.3 WHP at 7,451 rpm / 228.5 ft-lbs WTQ at 5,501 rpm
- **Rev limit at dyno time:** 7,500 rpm (since raised to 7,800 rpm in DTA)
- **Power was still rising at cutoff**

The 95% throttle cap was already applied before the dyno run. In the sim, `throttle.lut` is linear (DTA produces a linear throttle response).

---

## Gearing

### CRITICAL: Constant Mesh Ratio

The Drenth sequential has a constant mesh gear (24/25 = 1.042). The ratios below do **not** include the constant mesh, but **the 4.1 final drive already accounts for it in the real driveline**. This was validated against shakedown telemetry:

- Telemetry: 195.32 km/h in 5th gear at 7,480 rpm on 225/40/18
- Using ratios as listed × 4.1 final: matches within rolling radius tolerance
- Using ratios × 1.042 × 4.1: requires physically impossible tyre radius

**In AC: use gear ratios as listed, `FINAL=4.1`, do NOT multiply by 1.042.**

### Gear Ratios

| Gear | Ratio | Speed at 7800 rpm limiter (245/40/18) |
|------|-------|---------------------------------------|
| 1st | 2.500 | ~56 mph |
| 2nd | 1.824 | ~77 mph |
| 3rd | 1.474 | ~95 mph |
| 4th | 1.250 | ~112 mph |
| 5th | 1.091 | ~129 mph |
| 6th | 1.000 | ~140 mph |
| Reverse | −3.750 | |
| Final | 4.100 | |

At Brands Hatch Indy: 5th covers the straight (~121 mph top). 6th only needed at circuits like Spa.

---

## Clutch / Shift Behaviour

Drenth dog engagement:
- **Upshifts:** No clutch. `AUTO_CUTOFF_TIME=50` fires an ignition cut to unload the dogs. This is independent of the autoclutch system.
- **Downshifts:** Clutch required — driver blips throttle and presses clutch to unload dogs.

AC implementation:
- `UPSHIFT_PROFILE=NONE`, `DOWNSHIFT_PROFILE=NONE` — no autoclutch on either direction
- `USE_ON_CHANGES=0`
- Player must have autoclutch **OFF** in AC settings for this to work correctly

---

## Power Curve — Target Values (power.lut)

Rebuilt from dyno data using 12% drivetrain loss (crank = wheel ÷ 0.88):

```
0|50
1000|100
1500|120
2000|135
2500|139
3000|205
3500|251
4000|297
4500|339
5000|382
5500|408
6000|418
6500|413
7000|408
7500|404
7800|383
8000|150
8500|0
```

Peak crank torque: 418 Nm at 6,000 rpm. Peak wheel power: ~374 WHP at 7,500 rpm. Sub-2,500 rpm is extrapolated — the S54 without VANOS is genuinely weak down low. Sharp cutoff above 7,800 models the DTA hard limiter.

---

## Tyre Compounds

| Index | File sections | Compound | Use |
|-------|--------------|----------|-----|
| 0 | FRONT / REAR | Dunlop Racing Wet | Wet track |
| 1 (default) | FRONT_1 / REAR_1 | Yokohama A052 | **Primary — real car's dry tyre** |
| 2 | FRONT_2 / REAR_2 | Slicks | Maximum dry grip |

`COMPOUND_DEFAULT=1` — defaults to A052.

### A052 Tyre Dimensions (245/40/18 on 9J)
- Free radius: 0.3266 m
- Width in AC: 0.249 m (245 mm nominal, marginal stretch on 9J)
- Ideal pressure: 32 PSI (~2.21 bar) — at the high end of the hot target range (2.0–2.3 bar), which is correct for this application
- `DX_REF=DY_REF=1.280` — real-world peak lateral μ for A052 is ~1.35–1.45; 1.280 is deliberately conservative to account for AC's load model

### A052 Thermal Behaviour
- Useful from ~50–60°C, peaks ~70–90°C tread surface temperature
- Above ~100°C sustained: risk of graining/blistering, especially in high-load corners
- The `THERMAL_FRONT_1`/`THERMAL_REAR_1` sections in tyres.ini model this — if the sim shows persistent underperformance on early laps, the thermal model may need recalibrating
- The compound has a progressive limit (gives warning before full slip) — this should feel true in the sim

### Shakedown telemetry note
All shakedown telemetry was recorded on **225/40/18** (free radius 0.3186 m), not the current 245s. Slightly more grip available on 245s once dialled in.

---

## Key Configuration Files

| File | What it controls |
|------|-----------------|
| `data/car.ini` | Mass (1,250 kg), inertia, fuel (110 L), steering ratio (17.6), FFB |
| `data/engine.ini` | RPM limiter (7,800), inertia, power curve reference (`power.lut`) |
| `data/power.lut` | Crank torque curve — see target values above |
| `data/drivetrain.ini` | Gear ratios, final drive (4.1), Drexler diff values, shift profiles |
| `data/suspensions.ini` | Geometry, spring rates, dampers, camber, ARB stiffness |
| `data/tyres.ini` | A052 / Wet / Slick compound data, thermal model, wear curves |
| `data/aero.ini` | 5 aero elements: body, front undertray, undertray, diffuser, rear wing |
| `data/brakes.ini` | Max torque (3,600 Nm), ~67% front bias, thermal model |
| `data/electronics.ini` | ABS=0, TC=0, EDL=0 — no driver aids on real car |
| `data/throttle.lut` | Linear map (DTA produces linear throttle response) |
| `data/setup.ini` | Adjustable parameters exposed in-game |
| `extension/ext_config.ini` | PBR materials, animations, extension features |
| `ui/ui_car.json` | Specs and power/torque curves shown in game UI |

---

## Target State for Key Parameters

These are the validated v1.2 values. If a file differs, investigate before changing:

| File | Parameter | Target value | Notes |
|------|-----------|-------------|-------|
| engine.ini | LIMITER | 7800 | Match DTA |
| engine.ini | COAST_REF RPM | 7300 | Single section only |
| engine.ini | COAST_REF TORQUE | 130 | S54 no-VANOS, higher engine braking |
| engine.ini | RPM_THRESHOLD | 8100 | 300 rpm above limiter |
| car.ini | TOTALMASS | 1250 | 1175 kg dry + 75 kg driver |
| car.ini | INERTIA | 1.35,1.07,3.91 | Scaled with mass |
| car.ini | MAX_FUEL | 110 | 110 L cell |
| drivetrain.ini | PRELOAD | 0 | No preload confirmed |
| drivetrain.ini | POWER | 0.30 | Drexler estimate — needs telemetry validation |
| drivetrain.ini | COAST | 0.15 | Drexler estimate — needs telemetry validation |
| tyres.ini | A052 RADIUS | 0.3266 | 245/40/18 |
| tyres.ini | A052 WIDTH | 0.249 | 245 mm on 9J |
| tyres.ini | A052 DX_REF/DY_REF | 1.280 | Corrected from Ceky's 1.430 |
| tyres.ini | COMPOUND_DEFAULT | 1 | A052 as default |
| electronics.ini | ABS PRESENT/ACTIVE | 0/0 | No ABS on real car |

---

## Suspension at a Glance

| Parameter | Front | Rear |
|-----------|-------|------|
| Type | MacPherson strut | Double wishbone |
| Static camber | −3.0° (setup grid covers to −5.0° for real −4.7° LF) | −2.0° |
| Spring rate | 31,927 N/m | 36,305 N/m |
| ARB stiffness | 54,188 Nm | 18,922 Nm |
| Bump (slow) | 5,023 Ns/m | 4,400 Ns/m |
| Rebound (slow) | 8,203 Ns/m | 8,000 Ns/m |

---

## Aero Elements

| Wing | Role | Notes |
|------|------|-------|
| WING_0 BODY | Body drag | CL_GAIN=0, CD_GAIN=1.103 |
| WING_1 FRONT | Front undertray | Height-sensitive (ground effect) |
| WING_2 UNDERTRAY | Centre undertray | Height-sensitive |
| WING_3 DIFFUSER | Rear diffuser | ANGLE=6°, height-sensitive |
| WING_4 REAR | Rear wing | CL_GAIN=0.85, ANGLE=6° |

All CD_GAIN values scaled ×1.199 from Ceky base to match terminal velocity target (~191 km/h). Calibrated on cold/damp reference lap — needs refinement against dry warm-up top speed.

---

## Validation Targets (Brands Hatch Indy)

| Metric | Target |
|--------|--------|
| Lap time | ~52.4 s sim vs 52.9 s real reference |
| Top speed | ~195 km/h in 5th gear on Brabham straight |
| Gear at Druids | 2nd |
| Gear on straight | 5th (no 6th needed at Brands) |
| RPM range | 3,800–7,480 rpm |
| Peak lateral G | ~1.42 g (on 225s; slightly higher expected on 245s) |
| Peak braking G | −1.02 g (real car was conservative — potential is higher) |

**Reference lap:** Nigel Greensall, Lap 2, 52.90 s (cold, slightly damp, 225/40/18 tyres).

---

## Known Outstanding Issues

| Issue | Detail |
|-------|--------|
| Braking lockup | Sim locks earlier than real car. `BRAKE_DX_MOD` reduction from 0.08 → ~0.04 identified but deferred |
| Diff values | POWER/COAST are estimates. Validate against real exit wheel speed telemetry. Check git state — files may have reverted to Ceky values (0.60/0.35) |
| Aero calibration | CD_GAIN set from cold/damp lap. Refine with dry warm top speed data |
| Tyre grip | DX_REF/DY_REF=1.280 set in v1.2. Check current repo state |

---

## Development Principles

1. One change at a time — in the sim and on the real car
2. Every change needs a specific hypothesis and measurable validation target
3. Gear usage mismatch between sim and real car is a reliable diagnostic for power calibration errors
4. Validate against telemetry traces (speed, RPM, gear, G-force profiles), not just lap time
5. The sim should match the real car's behaviour — if the sim is significantly quicker, something is wrong

---

## LUT File Format

`.lut` files map input→output values for all non-linear relationships (torque, aero, brake temps, tyre performance):
```
|input=output
|1000=180
|2000=240
```

## Workflow

Changes are tested by loading the car in Assetto Corsa. Content Manager's Developer Mode can hot-reload some values; most physics changes require a full game restart. Large binary files (`.kn5`, `.bank`, `.dds`) are tracked via **Git LFS**.

### DATA_CHECKSUM_EXCLUDE

AC verifies checksums of packed `data.acd` files in multiplayer to prevent cheating. Files listed in `DATA_CHECKSUM_EXCLUDE` are excluded from this check, allowing them to be modified without being flagged as a "modified car" by servers. During development this is useful — files under active iteration can be added to the exclude list to avoid repacking the `.acd` after every change.
