# Aurora — Interactive Procedural 3D EV

A single self-contained `index.html` that renders an interactive 3D electric sedan.
Every surface, material, texture and sound is generated in code at runtime — there
are no model files, no image textures, no environment maps and no audio files
anywhere in this repository.

Open `index.html` in any modern browser, or serve the folder
(`python3 -m http.server`) and visit it. There is no build step.

## Interactions

| Control | What it does |
| --- | --- |
| Drag / pinch / scroll | Orbit, zoom and pan. Auto-rotate stops on first input. |
| Click the car | Doors, frunk and light bars are all clickable in 3D. |
| Front L/R, Rear L/R, All | Open and close each door on its own hinge. |
| Frunk / Trunk | Open the front drive-unit bay and the decklid. |
| Lights | Front and rear LED bars, plus real headlight beams. |
| Motor | Ramps the rotor up, pulses the coil glow, adds vibration and a synth hum. |
| Drive | Spins all four wheels, adds suspension bob and a live speed readout. |
| Night | Studio lighting drops to a night rig with stronger bloom. |
| Paint | Five live paint colours. |

Door handles pop out when the pointer is over their door, and each panel runs a
`closed → opening → open → closing` state machine that ignores repeat clicks
mid-animation.

## How the body is built

The whole shell is one parametric loft. A cross-section is defined by eight
control points on a Catmull-Rom curve (underbody, rocker, widest point,
beltline, roof rail, crown), and each of those heights and widths is driven by a
smooth curve along the length of the car. Sweeping that section over ~135
stations produces an `(i, j)` grid, and every panel is simply a sub-rectangle of
that grid:

- **Doors** are grid cells between the rocker and beltline bands, extracted with
  their own solid geometry and re-parented to a hinge group, so they fit their
  own aperture exactly.
- **Glass** is the same surface above the beltline — the windshield, side
  windows and panoramic roof are one continuous plane, split only where a door
  needs to carry its glass with it.
- **Hood and decklid** are the top arc of the section over their length range.
- **Wheel arches and shut lines** are cells removed from the grid; a swept
  elliptical flare covers each arch opening.

`buildSolid()` turns any masked set of cells into a panel with a real outer skin,
an inward-offset inner skin and rims along every open edge, so panels have
thickness and open doorways show a proper interior.

Textures (paint flake, tyre tread, floor gradient, contact shadow, the centre
display) are drawn to `<canvas>` at runtime. Reflections come from three.js's
`RoomEnvironment` through a `PMREMGenerator`, which is code, not an asset. The
motor hum is a Web Audio oscillator pair through a lowpass filter.

## Technical notes

- three.js r169 core + official addons (OrbitControls, RoomEnvironment,
  RectAreaLightUniformsLib, EffectComposer / RenderPass / UnrealBloomPass /
  OutputPass) via an import map. Nothing else.
- ACES filmic tone mapping, sRGB output, `devicePixelRatio` capped at 2,
  PCF soft shadows plus a blurred contact shadow.
- ~89k triangles in the whole scene; repeated parts (spokes, lug nuts, coil
  windings, rotor magnets, diffuser fins, battery ribs) are instanced.
- Debounced resize, clamped orbit distance and polar angle so the camera can
  neither enter the car nor drop below the floor.
- `window.aurora` exposes the scene, camera, state and toggles for scripting
  from the console.

The design is an original minimalist EV. It carries no real manufacturer's
logos, badges or model names.
