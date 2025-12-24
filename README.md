# 🎄 3D Christmas Tree in Julia (CairoMakie) — Complete Guide

A **Julia Jupyter notebook** that procedurally generates a 3D Christmas tree using a cone-shaped point cloud, adds ornaments + a trunk, overlays a **3D billboard-style message**, and exports both **PNG** (still image) and **GIF** (rotating camera animation).

---

## Files

- `xmas_tree_billboard.ipynb` — main notebook
- `README.md` — this guide
- `outputs/` — saved PNG/GIF examples

---

## Requirements

- **Julia** (recommended 1.9+)
- **Packages:**
  - `CairoMakie`
  - `Colors`
  - `GeometryBasics`

### Install packages (Julia REPL)

```julia
import Pkg
Pkg.add(["CairoMakie", "Colors", "GeometryBasics"])
```

---

## How to Use (Notebook)

1. **Open the notebook** file (`xmas_tree_billboard.ipynb`) in JupyterLab, Jupyter Notebook, or VS Code.

2. **Run the cells** from top to bottom (Run All).

3. The notebook will:
   - Generate the 3D tree figure
   - Save a PNG and a GIF into your `outdir`
   - Preview them inside the notebook using embedded HTML

4. **Look at the final printed lines:**
   ```
   Saved PNG: ...
   Saved GIF: ...
   ```

### Common Run Modes

- **Jupyter / VS Code notebook:** Just run cells.
- **Terminal (optional):** If you exported the notebook code into a `.jl` file, run:
  ```bash
  julia xmas_tree_billboard.jl
  ```

---

## Customize (User Parameters)

All user-editable settings are in the `P = (; ... )` block near the top.  
**You usually don't need to touch anything else.**

### Output Controls

- `outdir`: Folder where the PNG and GIF are saved
- `png_name`: Output PNG filename
- `gif_name`: Output GIF filename

### Randomness

- `seed = 1234` → Reproducible (same tree each run)
- `seed = nothing` → New random tree each run

### Animation (GIF Rotation)

- `fps`: Frames per second (higher = smoother/faster playback)
- `step_deg`: Degrees per frame
  - Smaller (e.g., 1–2) = smoother but more frames
  - Larger (e.g., 5–10) = faster but fewer frames
- `revolutions`: Number of full 360° rotations
- `start_az_deg`: Starting azimuth angle (degrees)
- `elev_deg`: Camera elevation (degrees)

### 3D Billboard Message (Inside the 3D Scene)

- `show_message`: Turn message on/off
- `message`: Message text
- `message_pos = Point3f(x, y, z)`: Where the billboard sits in data coordinates
  - Increase `z` to move it upward
  - Change `y` to move it "in front/behind" depending on your view
- `wrap_px`: Wrapping width (pixels); bigger = fewer line breaks
- `msg_fontsize`: Text size
- `msg_bg_alpha`: Background box transparency (0..1)
- `msg_padding`: Padding inside the background box (pixels)

### Tree Appearance

- `ornaments_total`: Number of ornaments total (split among 6 colors/shapes)
- `tree_marker_size`: Size of tree "leaf/star" points
- `ornament_size`: Ornament marker size
- `top_star_size`: Top star size

### Stable Framing (Prevents GIF Jitter)

- `xlim`, `ylim`, `zlim`: Fixed axis limits
- If your message or tree is clipped, widen these ranges

---

## Maths Behind the Tree

The tree is built as a **cone-like 3D point cloud**.

### 1) Random Angle Around the Vertical Axis

We sample an angle uniformly:

```
a ~ Uniform(0, 2π)
```

This spreads points evenly around the z-axis.

### 2) Random Height

We sample a height uniformly:

```
z ~ Uniform(0, 1)
```

### 3) Cone Radius Shrinks with Height

A cone narrows linearly as you go up:

```
r_max(z) = R(1 - z)
```

In the script, `R = 0.4` sets the overall "tree width".

### 4) Cylindrical → Cartesian

Convert from `(r, a)` to `(x, y)`:

```
x = r·cos(a),  y = r·sin(a)
```

### 5) Filling the Cone Volume (Not Just the Surface)

If we used `(x, y) = (r·cos(a), r·sin(a))` directly, points would lie on the cone surface.

To fill the interior, we sample:

```
t ~ Uniform(0, 1)
```

and scale the radius:

```
ρ = t · r_max(z)
```

So plotted points are:

```
(x_t, y_t, z) = (x·t,  y·t,  z)
```

**Note:** If you want uniform density in disk area, a common trick is to use:

```
ρ = r√u  with  u ~ U(0,1)
```

In code: `t = sqrt.(rand(N))`.

Your current `t = rand(N)` produces slightly more points near the center, which often looks more "tree-like".

---

## Ornaments: Why the `.*t` Scaling Matters

The tree points are plotted at:

```
(x·t,  y·t,  z)
```

Ornaments are chosen from the same underlying point set (random indices).

To place ornaments **exactly on the tree points**, they must use the same scaling:

```
(x_o, y_o, z_o) = (x[L]·t[L],  y[L]·t[L],  z[L])
```

If you forget `.*t`, ornaments are placed at `(x[L], y[L], z[L])`, which corresponds to the unscaled cone surface radius and can look "floating" or misplaced.

---

## GIF Rotation 

The object is **static**. The animation rotates the **camera azimuth**.

If the azimuth at frame `k` is:

```
θ_k = θ_0 + k·Δθ
```

then the camera completes one full rotation after `Δθ` accumulates to `360°`.

### In the Script:

- `start_az_deg` is `θ_0`
- `step_deg` is `Δθ`
- `revolutions` controls total angle range: `360° × revolutions`

**Elevation is kept constant** to avoid "wobble".

---

## Tips / Troubleshooting

### Message Overlaps the Tree

**Adjust:**

- `message_pos` (try increasing `z` or changing `y`)
- `wrap_px` (to change line breaks)
- `msg_fontsize` (reduce if too large)

**Example:**

```julia
message_pos = Point3f(0f0, -0.25f0, 1.25f0)
```

### GIF Looks Jittery

Keep **fixed limits**:

```julia
xlim, ylim, zlim
```

If you changed tree scale (e.g., radius), update these limits accordingly.

### Want a Denser Tree

Increase `N` (in the Data section).

This increases render time, especially for GIF frames.

---

## Happy Holidays!

Enjoy creating your procedurally generated 3D Christmas tree! Feel free to customize the parameters and share your creations. 

## License

This project is licensed under the **MIT License** — you are free to use, modify, and distribute it as long as the license notice is included. See the `LICENSE` file for details.

## Contributing

Contributions, issues, and feature requests are welcome!

## Show Your Support

If you found this project helpful, please consider giving it a star!

---

**Made with using Julia and CairoMakie**
