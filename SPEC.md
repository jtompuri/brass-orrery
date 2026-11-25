# Real Mechanical Orrery: Build Specification

Build a working brass orrery that runs in a web browser. One hand-cranked drive shaft turns six gear pairs. Each pair turns a nested tube, each tube carries a rigid arm, and each arm carries a planet. Nothing moves unless the drive shaft moves.

The numbers in this file are final. Do not redesign, round, or "improve" them. How you build it (libraries, structure, rendering technique, code organisation) is your decision.

## 1. Deliverable

- One file, `index.html`, that works when opened directly from disk.
- You may load any JavaScript or CSS libraries (for example a 3D engine, a physics or math library) from public CDNs such as cdnjs, jsDelivr, unpkg or esm.sh, in any version, as classic scripts or ES modules.
- No downloaded textures, images, 3D models or fonts: create visual assets in code.
- Handle window resizing, keep rendering cost reasonable (cap the device pixel ratio at 2), and produce no console errors.

## 2. Motion: the one rule that matters

The drive angle is the only animated mechanical value. Play, reverse, the speed slider, and the crank all change it. Every other angle is computed from it each frame. Planets must have no clock, period, or speed variable of their own.

```text
tubeAngle  = -(driveTeeth / tubeTeeth) * driveAngle + meshOffset
meshOffset = PI - PI / tubeTeeth
```

World convention, used by every number in this file: Y is up. Positions are given as (x, y, z) or, for points on the floor plan, as (x, z). Angles are measured in the XZ plane from +X toward +Z. A part "at angle A" has been rotated about the vertical axis so that its reference direction points at angle A.

- A drive gear at drive angle `a` has tooth 0 pointing at world angle `a`. A tube gear has tooth 0 at world angle `tubeAngle`.
- At speed 1x, the drive angle advances `2 * PI / 88` radians per second, so Mercury orbits in 8 s and Saturn in 968 s.
- Speed slider: logarithmic from 0.25x to 32x, default 2x.

## 3. Gear table

Module `m = 0.02`. Pitch radius is `m * teeth / 2`. Every pair totals 144 teeth, so every center distance is `1.44`.

The central axis is at `(0, 0)`. The drive shaft is at `(-1.44, 0)`. Tube gears are on the central axis and drive gears are on the drive shaft.

| Layer k | Planet | Drive teeth | Tube teeth | Gear center Y |
|---:|---|---:|---:|---:|
| 0 | Mercury | 132 | 12 | 0.45 |
| 1 | Venus | 117 | 27 | 0.59 |
| 2 | Earth | 105 | 39 | 0.73 |
| 3 | Mars | 84 | 60 | 0.87 |
| 4 | Jupiter | 26 | 118 | 1.01 |
| 5 | Saturn | 12 | 132 | 1.15 |

Gear thickness is `0.07`.

### Tooth profile

Use exactly this profile. It was checked for collisions across a full tooth pitch on all six pairs. Tooth `j` is centered at angle `2 * PI * j / N` in the gear's own frame, so tooth 0 is on the gear's reference direction.

```text
pitchStep = 2 * PI / N
rPitch = m * N / 2
rTip   = rPitch + 1.00 * m
rRoot  = rPitch - 1.25 * m
halfPitch = 0.38 * pitchStep / 2
halfTip   = 0.55 * halfPitch
halfRoot  = halfPitch + (halfPitch - halfTip) * 1.25
```

Each tooth outline runs through these polar points, then along the root circle to the next tooth:

```text
(rRoot, -halfRoot) (rPitch, -halfPitch) (rTip, -halfTip) (rTip, +halfTip) (rPitch, +halfPitch) (rRoot, +halfRoot)
```

Gears are flat solids of that outline, with no bevel. Large gears should have 4 to 6 cut-out spokes, leaving a solid rim and hub.

## 4. Fixed structure

| Part | Y range or Y | Radius or position |
|---|---|---|
| Base (dark wood with brass trim) | -0.35 to 0.00 | radius 4.35 |
| Bevel gears and crank input shaft | 0.30 | bevel radius 0.06 |
| Bottom plate (open frame) | 0.375 to 0.405 | radius 3.00, center bore 0.24, drive bore 0.09 |
| Top plate (open frame) | 1.25 to 1.29 | radius 3.00, center bore 0.24, drive bore 0.09 |
| Six frame pillars | 0.00 to 1.25 | radius 0.04, at distance 2.92, angles 30, 90, 150, 210, 270, 330 degrees |
| Fixed central post | 0.00 to 1.77 | radius 0.03 |
| Drive shaft | 0.30 to 1.27 | radius 0.07, at (-1.44, 0) |
| Glass dome (upper hemisphere) | center Y 0 | radius 4.20 |

The sun sits on the fixed central post and does not orbit.

**Base.** Dark wood with a brass trim ring at the edge: wood out to radius `4.25`, brass trim from `4.25` to `4.35`. No two surfaces may lie on top of each other at the same height (they flicker).

**Plates.** The gears must stay visible, so each plate is an open brass frame, never a solid disc:

| Piece | Shape |
|---|---|
| Rim | annulus, radius 2.80 to 3.00 |
| Center bearing | annulus, radius 0.24 to 0.42 |
| Drive bearing | annulus, radius 0.09 to 0.17, centered at (-1.44, 0) |
| Six spokes | flat bars, width 0.10, from radius 0.40 to 2.82, at the six pillar angles |
| Drive bar | flat bar, width 0.10, along the X axis at Z = 0, in two segments: x from -0.40 to -1.29 and x from -1.59 to -2.82 |

## 5. Nested tubes, arms, and planets

The orbital plane is `Y = 2.15`. The sun has center `(0, 2.15, 0)` and radius `0.38`.

Each tube runs from its gear's center Y up to its arm's Y. Inner tubes reach farther in both directions: Mercury's tube is innermost, lowest at the gear end, and highest at the arm end. Saturn's tube is outermost and shortest.

| Planet | Tube inner r | Tube outer r | Arm Y | Orbit radius | Planet radius | Arm offset (deg) |
|---|---:|---:|---:|---:|---:|---:|
| Mercury | 0.04 | 0.06 | 1.62 | 0.75 | 0.05 | 20 |
| Venus | 0.07 | 0.09 | 1.57 | 1.10 | 0.085 | 135 |
| Earth | 0.10 | 0.12 | 1.52 | 1.50 | 0.09 | 250 |
| Mars | 0.13 | 0.15 | 1.47 | 1.90 | 0.07 | 320 |
| Jupiter | 0.16 | 0.18 | 1.42 | 2.50 | 0.19 | 75 |
| Saturn | 0.19 | 0.21 | 1.37 | 3.15 | 0.16 | 200 |

Each planet's tube gear, tube, collar, arm hub, arm, riser and planet are rigidly attached and turn together about the central axis by `tubeAngle`:

- **Arm:** a flat brass bar, thickness `0.025`, width `0.05`, from the hub to the orbit radius, pointing at world angle `tubeAngle + armOffset`.
- **Hub:** a short cylinder, radius `tubeOuter + 0.02`, height `0.025`, at the arm's Y.
- **Riser:** a vertical rod, radius `0.012`, from the arm tip up to `Y = 2.15`.
- **Planet:** centered on top of the riser at the orbit radius.

The arm offset is a fixed part of the arm, so it must not be added to the gear's tooth phase.

Saturn has a flat textured ring with inner radius `0.21`, outer radius `0.30`, tilted 20 degrees. Orbits are circular and lie in the plane `Y = 2.15`. Draw each one as a very thin, half-transparent brass ribbon.

## 6. Hand crank

A horizontal input shaft runs along -X at `Y = 0.30`, from the drive shaft to `x = -3.70`. A pair of 1:1 bevel gears at the bottom of the drive shaft connects them. At the outer end is a crank arm of length `0.24` with a turned wooden knob. The crank angle equals the drive angle.

Dragging the knob turns the crank and pauses auto-play while dragging. It must change only the drive angle.

## 7. Interface

A compact, elegant panel with:

- Play/pause (`id="orrery-play"`), reverse (`id="orrery-reverse"`), and the speed slider (`id="orrery-speed"`, an `<input type="range">`)
- A section slider (`id="orrery-section"`) that cuts away the dome, plates, and gears with a vertical plane, revealing the nested tubes
- A readout (`id="orrery-readout"`) of the drive-shaft turns and the equivalent elapsed Earth years

Camera: drag to orbit, wheel to zoom, and the camera stays above the base. The default view shows the whole base and dome.

**Click to trace:** clicking a planet highlights its arm, tube, tube gear, and drive gear, and dims the other mechanisms. A label (`id="orrery-label"`) shows the planet's name, gear pair, ratio, and real orbital period. Clicking empty space clears it.

Real periods in days: Mercury 87.969, Venus 224.701, Earth 365.256, Mars 686.980, Jupiter 4332.59, Saturn 10759.22.

## 8. Look

Polished brass under soft studio light, on a dark seamless background, with believable reflections. The sun glows and its surface moves (cosmetic, may use real time). Each planet looks distinct. The glass dome keeps the mechanism clearly visible. Any glow or bloom is subtle and does not wash out the gears. Do not add an asteroid belt or a piston.

## 9. Verification

No browser is available while you work. How you check your work is up to you. The result will be judged by opening `index.html` in a desktop browser: by how it looks, and by whether the mechanism, crank, controls, click to trace, and section view work as specified.
