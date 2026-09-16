# Three bodies

An interactive 2D simulation of the gravitational three-body problem, packaged as a single HTML file with no build step and no dependencies.

Three point masses attract each other under Newtonian gravity. A handful of special starting arrangements produce orbits that repeat forever, but almost any disturbance sends the system into chaos. The simulation lets you pick those arrangements, nudge the bodies, change their masses, and watch what happens.

<img width="945" height="809" alt="image" src="https://github.com/user-attachments/assets/6e17f71a-021e-4ea8-a938-e1708c234ecf" />

## Quick start

Open `three-body.html` in any modern browser (Chrome, Firefox, Safari, Edge). That's it.

The page works offline. It loads two typefaces (Cormorant Garamond and IBM Plex Sans) from Google Fonts when a connection is available and falls back to system fonts otherwise.

To serve it locally instead of opening the file directly:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/three-body.html
```

## Using the simulation

### Starting arrangements

| Preset | What it shows |
| --- | --- |
| Figure eight | The Chenciner–Montgomery solution: three equal masses chasing each other along a single figure-eight path. Periodic and remarkably stable. |
| Triangle | Lagrange's equilateral solution: three equal masses rotating rigidly at the corners of a triangle. It is an exact solution but unstable, so tiny numerical errors eventually grow and the formation breaks apart. |
| Pythagorean | Burrau's problem: masses 3, 4 and 5 released from rest at the corners of a 3-4-5 right triangle (positions scaled by 0.45). Produces a long chaotic dance that typically ends with one body ejected. |
| Star & planets | A heavy central mass (5) with two light bodies (0.05 and 0.15) on roughly circular orbits at radii 1 and 2.1. The planets perturb each other slowly. |
| Random | Random masses (0.6–2.0), positions and velocities. Different every time. |

### Controls

- **Pause / Play** stops and resumes time.
- **Restart** reloads the current preset from its initial conditions.
- **Clear trails** erases the orbit history without affecting the bodies.
- **Masses** sliders (0.05–6) change each body's mass mid-flight. Positions and velocities are kept, so a stable orbit usually falls apart.
- **Speed** (0.1×–4×) scales simulated time per real second.
- **Trail length** (0–3000 points) sets how much history each body leaves behind.
- **Keep centre of mass in view** makes the camera follow the system's barycentre.
- **Show velocity arrows** draws each body's velocity vector.

### Mouse and touch

- **Drag a body** to move it. Its velocity is preserved, and its trail is reset.
- **Scroll** to zoom in and out.

The readout in the bottom-left corner shows simulated time and the relative energy drift since the last reset, drag, or mass change. Drift stays tiny for well-behaved orbits and grows during extremely close encounters, which makes it a useful check on how much to trust what you are seeing.

## How it works

### Physics

Units are chosen so that the gravitational constant G = 1. The acceleration of body *i* is

```
a_i = Σ_j  G · m_j · (r_j − r_i) / (|r_j − r_i|² + ε²)^(3/2)
```

A very small softening term (ε² = 1e-5) prevents division by zero if two bodies pass through the same point, while leaving normal orbits effectively unchanged.

Every preset is shifted into the centre-of-mass frame on load, so total momentum is zero and the system does not drift off screen.

### Integration

The state is advanced with classic fourth-order Runge–Kutta (RK4). The step size adapts every step:

```
h = min(0.004, 0.02 · sqrt(r_min³ / M_total), 0.02 · r_min / v_max)
```

where `r_min` is the closest pair distance and `v_max` the fastest body's speed. Steps shrink automatically during close approaches, which keeps slingshots accurate without slowing down the calm parts of an orbit. Each animation frame runs as many steps as needed to cover the requested simulated time, capped to keep the page responsive.

### Rendering

Drawing uses a 2D canvas scaled for high-DPI screens. Trails are sampled every 0.008 time units and drawn in 24 segments with increasing opacity so older history fades out. The background rings are spaced by powers of two so they stay readable at any zoom level.

The colour palette follows the system light or dark setting through CSS custom properties, which the canvas reads at runtime. If the operating system requests reduced motion, the simulation starts paused.

## Project structure

```
three-body.html   The entire application: markup, styles and script
README.md         This file
```

## Customising

All the tunable pieces live near the top of the `<script>` block in `three-body.html`:

- `presets` holds the initial conditions. Each preset returns three bodies built with `B(mass, x, y, vx, vy)`. Add a new entry and a matching `<button data-p="yourName">` in the presets row to expose it in the UI.
- `EPS2` is the softening constant.
- `adaptiveStep()` contains the step-size rules; lower the `0.02` factors for more accuracy at the cost of speed.
- The colour tokens (`--b1`, `--b2`, `--b3` for the bodies) are defined in the `:root` styles for light and dark themes.

## Browser support

Any browser with Canvas 2D, Pointer Events and `ResizeObserver`, which covers all current desktop and mobile browsers.

## Further reading

- A. Chenciner and R. Montgomery, "A remarkable periodic solution of the three-body problem in the case of equal masses," *Annals of Mathematics*, 2000.
- C. Burrau, "Numerische Berechnung eines Spezialfalles des Dreikörperproblems," *Astronomische Nachrichten*, 1913.
- V. Szebehely and C. F. Peters, "Complete solution of a general problem of three bodies," *Astronomical Journal*, 1967.
