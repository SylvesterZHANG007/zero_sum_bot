# Magnetic Piston Actuator — Feasibility Instrument v3

A single-file, client-side design-space tool for a magnetic piston pump driving an elastomeric endcap. It plots the **required force** (Gent hyperelastic endcap coupled to Boyle's law) against the **available force** (exact finite coaxial solenoid, elliptic-integral mutual inductance) along the plunger stroke, and reports whether the force balance closes over the whole stroke.

**[Open the tool →](https://sylvesterzhang007.github.io/zero_sum_bot/)**

<sub>中文：磁力活塞执行器可行性分析仪。单文件纯前端工具，沿活塞行程比较所需力（Gent 超弹性端盖 + 玻意耳定律）与可用力（精确有限共轴螺线管模型），判断力平衡在全行程上是否成立。界面右上角可切换中英文。</sub>

---

## Credit

This is a rebuild and extension of the original **Zero-Sum feasibility instrument by Alexandru Otlacan** ([aotlacan.com/zero-sum-app](https://aotlacan.com/zero-sum-app)), developed for the Zero-Sum Bot project at the University of Michigan (PIs: Prof. Talia Moore, Prof. Cameron Aubin). The physics, the parameter layout and the Gent material calibration originate with that tool; this version was independently reimplemented and cross-validated against it.

<sub>中文：本工具是对 Alexandru Otlacan 原始可行性分析仪的复刻与扩展。物理模型、参数布局与 Gent 材料标定均源自原工具；本版为独立重新实现并与之交叉验证。</sub>

## What was added in v2

| Addition | Why it matters |
|---|---|
| Wire gauge + layer count → turns/m, resistance, power, ΔT, thermal current limit | The original leaves drive current unconstrained; a current that melts the winding would otherwise be reported as feasible |
| Optional clamp of drive current to the thermal limit | Makes every verdict physically attainable |
| Radial clearance as a free parameter | The original locks magnet OD to bore ID, hiding the force-versus-friction trade-off |
| Dead volume | Pressure rise depends on the volume ratio; untracked tubing volume biases results optimistically |
| Friction model k = k₀ + c·r, as live sliders | Seal drag scales with perimeter; a radius-dependent term is what makes an optimal bore radius exist |
| Worst-case margin versus bore radius, with r\* marker | Answers "what bore should we build" directly |
| Two-sided (zero-sum) configuration with pre-inflation | Net load is zero at neutral regardless of working pressure; includes the give-back volume limit |
| Stroke specified in µL | Correct way to compare bore radii for a joint of fixed size |
| Hover readouts, click-through, sorting, CSV export, EN/中文 toggle | Usability |

## What was added in v3

| Addition | Why it matters |
|---|---|
| Pre-wound (catalogue) coil: bobbin ID, height, OD, turns, wire and bare-copper diameter, flange and bobbin wall, measured R | Commercial coils keep their own length and inner radius; the coil is no longer forced to the magnet length. A measured resistance fits the layer spacing, so the mean turn radius is pinned without opening the coil |
| Coil pitch as an input, decoupled from coil and magnet length | Real coils are not always touching; the step and push–pull curves depend strongly on the gap |
| Switching scheme: pull-ahead (original), push–pull, step (one coil at a time) | Push–pull roughly doubles the worst-case pull; step mode starts each move from the next coil's weak tail |
| Bench-supply drive: current = min(V/R, limit) | Matches a CV/CC lab supply, so a voltage setting maps directly to a coil current |
| Dynamics panel: x(t), v(t), a(t), supply current, static thrust vs position, per-voltage summary table | Back-EMF, coil inductance (Wheeler estimate or measured), supply current limit, Coulomb friction, flyback on switch-off; agrees with a SciPy reference to 0.1% |
| Presets: Oct 2026 bench hardware, original defaults | One click to the current build or back to the reference case |

Regression: with the original-defaults preset in gauge + layers mode the verdict is unchanged (81 / 96 positive margin, as in v2).

## Model

- **Magnetics.** The magnet is treated as a uniformly magnetised cylinder, equivalent to surface poles on its end faces: `F = (Br/μ₀)·I·[M(top) − M(bottom)]`, with `M` the exact mutual inductance between the multi-layer coil and a loop of the magnet's radius, evaluated in complete elliptic integrals (AGM). This is not a point-dipole approximation. Commutation energises the nearest *overlap* coils ahead of the magnet; the force dip at each coil centre is real geometry, not numerical noise.
- **Endcap.** Lumped spherical cap, uniform equibiaxial stretch `λ² = 1 + (w₀/a)²`, incompressible thinning `t/λ²`, Gent stress, Laplace law, solved simultaneously with Boyle's law for the trapped gas.
- **Thermal.** Steady-state `ΔT = I²R·duty/(h·A_surf)`, natural convection. Deliberately crude — its purpose is to flag impossible currents, not to predict temperature.

## Validation

- Cross-checked against an independent Python implementation of the same physics: agreement to machine precision (~1e-13) at identical configurations.
- Slice convergence: 30 coil slices are within 0.02% of the converged force.
- Against the original tool: plotted force peaks match to about 1%; tabulated worst-case margins to within 0.01–0.05 N.
- Known divergence: for `w₀/a ≳ 1.5` this model stiffens toward the Gent lock while the original plateaus near the neo-Hookean maximum `1.84 μt/a`. The two agree for `w₀/a ≲ 1`. An equibiaxial bulge test on the real membrane is needed to settle which lumping is correct at large stretch.

## Not modelled

Ferrofluid seal behaviour at a gas interface, magnet tilt and rotation, radial pull from an off-centre coil, eddy currents, wire temperature coefficient, the pneumatic load inside the dynamics run, and anything downstream of the endcap (joint moment, leg kinematics). This is a design-space exploration tool, not a substitute for hardware validation.

## Running it

Open `index.html` in any modern browser — no build step, no server, no dependencies. Google Fonts is loaded for typography and the tool degrades gracefully without network access.

## License

MIT (see `LICENSE`), or adjust to whatever your lab prefers before publishing.
