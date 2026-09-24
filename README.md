# Clearwater

Real-time, photoreal shallow water in a single HTML file. WebGL2, no libraries, no build step, no external assets.

**[Live demo](https://aureliengmz.github.io/clearwater/)** · drag to look around · tap the water

![Clearwater](media/landscape.png)

## Run

Open `index.html` in a browser, from disk or any static host. Nothing to install.

Needs WebGL2 with float render targets (`EXT_color_buffer_float`). Resolution adapts to keep the frame rate up.

| URL option | Effect |
| --- | --- |
| `?debug` | Frame rate, resolution, quality level |
| `?noglare` | Disable lens-diffraction glare |
| `?t=5` | Freeze time at 5 s (screenshots) |
| `?yaw=0.5&pitch=-0.4` | Initial camera direction, radians |
| `?view=caus` | Show the raw caustics texture |

## Jetski simulator

`jetski.html` turns the renderer into a personal-watercraft simulator. It is also a single file with no dependencies.

- **Waves**: two CPU FFT cascades (256 m and 27 m, JONSWAP wind sea plus swell) drive both the physics and the rendering, so what you see is what you hit. The GPU cascade above adds fine detail. Waves shoal and fade over shallows, and steep crests make whitecaps.
- **Physics**: a 430 kg rigid body. Each hull panel gets hydrostatic pressure plus hydrodynamic pressure, suction and skin friction, so planing, porpoising, slamming and jumps come out of the model. A thrust-vectoring jet pump ventilates when the intake leaves the water. Keel and sponson foils give the hull its grip. The rider balances and leans into turns.
- **Wake**: the ski makes real water waves. A dispersive wave solver (FFT, exact deep-water dispersion `ω = √(gk)`) runs in a 96 m window that follows the ski. It is driven by the hull's measured hydrodynamic lift spread over its wetted footprint. Moving produces a Kelvin wake, and landing a jump sends out rings. Wake heights are read back asynchronously each frame, so the physics can ride and jump your own wake. The ski's own pressure hollow is masked out of that readback.
- **Surf**: a long-period swell shoals over the beach profile using linear finite-depth theory (wavenumber, phase and shoaling gain are tabulated and shared with the shader). Near the break point it steepens, peaks, saturates at 0.78 × depth and breaks into white water. A bigger set rolls in every seventh wave. Ride out through it to jump.
- **World**: a sandy mainland beach with a foreshore, berm, strand line of seaweed and dunes with marram grass, backed by wooded hills. The sand is wet and glossy near the swash line and there's a sandy seabed offshore. Further out are rocky islands with pebble coves. The terrain uses the same formula in JS and GLSL, so collisions match what you see. You can drive up onto the beach and park; with the ski out of the water, `W`/`S` walk it forward or back into the water.
- **Effects**: bow spray, a rooster tail, landing splashes, wake foam and synthesized engine audio.

| Control | |
| --- | --- |
| `W`/`S` or arrows | Throttle / brake and reverse |
| `A`/`D` | Steer (under throttle, like a real jet) |
| `Q`/`E` | Trim; pitch in the air |
| `Shift` | Stand and lean forward for sharper turns |
| `C` | Chase / first-person camera |
| `1`–`4` | Sea state |
| `R`, `M`, `H` | Reset, mute, help |

Gamepad and touch are supported. URL options: `?debug` shows stats, `?fp` starts in first person, `?auto` turns on the autopilot, `?wake=256` uses a lighter wake grid (the default on phones), `?view=wake` / `?view=surf` show debug maps of the wake and surf height fields.

## Code map

Everything lives in `index.html`, in sections marked `/* ---- Name ---- */`:

| Section | What to tweak |
| --- | --- |
| Ocean spectrum (FFT) | `L` patch size, `DEPTH`, `TARGET_SLOPE` wave steepness |
| Interactive ripples | `RN`, `RSIZE` simulation grid |
| Caustics | `G` ray grid, `C` caustics resolution, `IORS` per-channel refraction |
| Main water shader | Fresnel, absorption, seabed shading (GLSL) |
| Post / Lens diffraction glare | Bloom, glare, tone curve, grain |
| Camera & input | `SUN_EL`, `SUN_AZ` sun position, `VFOV` |
| Loop | Frame loop, adaptive quality |

The seabed texture is base64 in `<script id="pebbles-texture">` at the end of the file. To regenerate it: `python tools/make_pebbles.py` (numpy, scipy, pillow), then paste the base64 JPEG into that block.

## References

- Jerry Tessendorf, *Simulating Ocean Water*: FFT waves
- Jerry Tessendorf, *eWave: Using an Exponential Solver on the iWave Problem*: the jetski wake solver
- Jacob Kerner, *Water interaction model for boats in video games*: per-panel hull forces
- Evan Wallace, *WebGL Water*: refracted-grid caustics
- Inigo Quilez, *Texture repetition*: seabed tiling
- Marc Olano & Dan Baker, *LEAN Mapping*: distant highlights

## Credits

<a href="https://x.com/Aurelien_Gz"><img src="media/aurelien.jpg" width="20" height="20" alt=""></a> Made by [Aurélien](https://x.com/Aurelien_Gz) at
<a href="https://lumaris.works"><picture><source media="(prefers-color-scheme: dark)" srcset="media/lumaris-dark.svg"><img src="media/lumaris-light.svg" width="14" height="14" alt=""></picture></a> [Lumaris](https://lumaris.works).

MIT License, see [LICENSE](LICENSE).
