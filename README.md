# SIMFLATTC

Browser driving sim (three.js, no build step). Serve the folder statically (GitHub Pages works) and open `index.html`; add to the iPhone home screen if you like.
Locally: `python3 -m http.server`, then http://localhost:8000.

Controls: W/S throttle/brake, A/D steer, Space handbrake, E/Q shift, M manual/auto, L lights, N day/night, **P cycles M140i -> M6 GT3 -> Alfa 4C**, T traction control, Enter respawn, C next camera (Shift+C previous), R live reflections on/off, **G circuit / grid**, V reset chase view.

Chase camera (C, first view): drag = orbit, mouse wheel / two-finger pinch = zoom, V or double-click / double-tap = reset. Touch devices have a TRACK button for G.

## Circuit (G toggles circuit / 3 km x 3 km grid)
One original fictional circuit, 3.76 km, generated at start-up from a closed Catmull-Rom centreline (`TRACK_DEF` in `index.html`: centreline points, half-width profile, elevation profile, kerb / run-off / barrier rules). Long start straight (about 700 m), fast sweepers, a 22 m radius hairpin, a technical esses section, 29 m of elevation change (max grade about 3.6 %). Racing direction is set by the grid; the car starts on pole facing the line, and a lap timer (HUD above the speed) starts when you cross the line and only counts laps that pass all quarter checkpoints.

| Surface | Grip (x asphalt) | Extra |
|---|---|---|
| Asphalt, asphalt run-off | 1.0 / 0.95 | |
| Kerb (red / white, 1.4 m, raised 5 cm) | 0.88 | tyre-load rumble |
| Grass | 0.50 | +0.04 rolling resistance, light rumble |
| Gravel trap (outside of corners) | 0.40 | +0.10 rolling resistance |
| Barrier (steel, 1.05 m, 12 m or more from the edge, further at gravel traps) | solid | impact removes the normal velocity (small bounce), scrapes lose speed |

Ground height, slope (gravity along and across the car) and the car's pitch / roll come from the circuit geometry at each wheel, so hills are real, not a flat plane. Surface and barrier lookups use a spatial grid plus a warm-start index (about 2 ms of CPU per simulated second). Textures are procedural (canvas), so nothing extra is downloaded; the whole circuit is about 9 draw calls (plus posts) and 83k triangles. Visual extras: vertex-colour rubbered-in racing line and wheel paths, darkened grass at barrier feet (cheap AO), existing sun shadows and night / headlight shader. Not included: dynamic tyre marks, weather, other circuits.

The grid world is now 3 km x 3 km (was 1 km); G switches back to it at any time.

## Lighting
Fixed low sun (~30 deg) with soft 2048 shadows that follow the car, a gradient sky, a contact-shadow patch, and live reflections (R toggles; off by default on touch devices). N switches to night.

## Cars
The BMW M140i is the default car. The 116d and M4 GT3 were removed in this version.

### 2018 BMW M6 GT3
No working lights yet (L does nothing in this mode). 4.4 l twin-turbo V8, ~585 PS, 1330 kg, RWD, 6-speed sequential, slicks plus downforce, ABS and TC. Checked in the sim: 0-100 km/h 3.6 s, 0-200 km/h 9.2 s. Values are in `MODES.m6` in `index.html`.

### 2014 Alfa Romeo 4C Launch Edition (new)
Loaded the first time you select it (P).

| | Real car (factory / tested) | Sim |
|---|---|---|
| Engine | 1750 TBi I4, 177 kW (240 PS) at 6000 rpm, 350 Nm 2200-4250 rpm, limit 6500 | same curve (plateau a little above 350 Nm, as measured cars make more) |
| Gearbox | Alfa TCT 6-speed dual dry clutch: 4.154 / 2.269 / 1.435 / 0.978 / 0.755 / 0.622, final drive 4.118 | same ratios, 0.12 s shifts, launch control behaviour |
| Chassis | RWD mid-engine, wheelbase 2.380 m, track 1.639 / 1.604 m, 41 / 59 % weight, ~1,025-1,118 kg | 1050 kg, same geometry and distribution |
| Tyres | 205/40 R18 front, 235/35 R19 rear | grip about 1.1 g lateral |
| 0-100 km/h | 4.5 s claimed, about 4.2 s tested | 4.6 s |
| 0-200 km/h | about 17.6 s tested | 16.2 s |
| Quarter mile | about 12.5 s tested | 12.8 s |
| Top speed | 258 km/h | 262 km/h |
| 100-0 km/h | 36 m claimed, about 31 m tested | 34 m |
| Lateral grip | 1.1 g claimed, about 1.0 g tested | about 1.05-1.1 g |
| Turning circle | 12.1 m | about 12.6 m |

Every sim figure is within 10 % of the real car. Mass, CdA (0.68 m2), downforce and tyre grip are estimates. All values are in the `MODES.a4c` table in `index.html`. Body paint is Rosso Alfa (the model ships white).

**4C lights** (L = headlights + tail lights, brake pedal = brake lights, `window.A4L.force={head,run,brake}` forces states for testing, set it to `null` afterwards). All light comes from the model's own lamp mesh (`...LightA_Material1`, one mesh, one material). That material is cloned, the original is untouched, and its own base-colour atlas (texture 21, UV0) and emissive texture (texture 22, TEXCOORD_1) are used; a small shader reads texture 22 through UV1 and gates each region:

| Texture 22 region (UV1) | Geometry | Behaviour |
|---|---|---|
| white disc, u>0.55, 0.45<v<0.93 | front bi-halogen projector lens | headlights, warm white |
| small white dot, u>0.8, v<0.22 | 5 LED bullets per side | headlights, cool white |
| outer ring of 20 squares (radius > 0.2 around 0.33,0.33) | rear ring, one quad per LED | running (dim) and brake (bright) |
| inner ring + centre cup (radius < 0.2) | rear ring interior | brake only |
| six white bars, u 0.67-0.80 | number-plate lamps | running |
| UV0 strip at the top of the base atlas | high-mounted brake lamp | brake (faint with running lights) |
| amber arcs | side markers / indicators | not driven, stay dark |

Running, brake and head states are independent. The dark gaps between the LEDs are preserved. The only extra light source is a dim red point light behind the car and the existing ground beam (halogen coloured). Which rear elements belong to the brake function was inferred from the model (the inner ring is dim in the supplied emissive map) and the reference photo (both rings plus the high-mount lamp lit when braking), not from Alfa documentation.

## Credits and licences
- BMW M140i model: Ddiaz Design, CC BY 4.0.
- BMW M6 GT3 model: "2018 BMW M6 GT3" by Ddiaz Design, **CC BY-NC-SA 4.0** (non-commercial, share-alike). Do not use it in anything commercial.
- Alfa Romeo 4C model: "2014 Alfa Romeo 4C Launch Edition (MY15)" by Ddiaz Design, **CC BY-NC-SA 4.0** (non-commercial, share-alike). Do not use it in anything commercial.
