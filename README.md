# Julia Set Renderer

A small, dependency-light Python script that renders a Julia set fractal
and saves it as a PNG. No command-line arguments, no `argparse` — just
edit a few constants at the top of the file and run it. This also makes
it safe to paste directly into a Jupyter/Colab cell, since it never
reads `sys.argv`.

## What it does

For a fixed complex constant `c`, the script iterates `z = z² + c` for
every pixel's starting value `z0` across a 2D grid, and colors each
pixel by how quickly (if ever) that point escapes to infinity. Points
that escape quickly are colored one way; points that stay bounded
forever (the actual Julia set) are left as solid background. A
*smooth* (continuous) escape-time formula is used instead of a plain
integer iteration count, which avoids the "banded" look you get from
naive escape-time coloring.

## Requirements

- Python 3.8+
- `numpy`
- `matplotlib`

Install with:

```bash
pip install numpy matplotlib
```

## Usage

Run it directly:

```bash
python julia_set.py
```

This renders the fractal, saves it to `julia.png` in the current
directory, and (if a display is available) also opens a preview
window via `plt.show()`.

In Colab/Jupyter, either:
- paste the whole file into a cell and run it, or
- upload `julia_set.py` and run `%run julia_set.py`, or
- `import julia_set` (the render happens on import, since the script
  has no `if __name__ == "__main__":` guard).

## Parameters

All configuration lives in one block near the top of the file — edit
these values directly, no flags needed:

| Variable | Default | Meaning |
|---|---|---|
| `C` | `complex(-0.7, 0.27015)` | The fixed Julia constant. This single value determines the whole fractal's shape. |
| `MAX_ITER` | `300` | Max iterations per pixel before a point is considered "never escapes." Higher = more detail, slower render. |
| `WIDTH`, `HEIGHT` | `900`, `700` | Output image resolution in pixels. |
| `XMIN`, `XMAX`, `YMIN`, `YMAX` | `-1.6, 1.6, -1.2, 1.2` | The region of the complex plane to render. Shrinking this range zooms in. |
| `OUT_FILE` | `"julia.png"` | Output filename. |

### Trying different fractals

Change `C` to explore different Julia sets. A few interesting ones:

```python
C = complex(-0.7, 0.27015)   # default — dendrite-like, delicate filaments
C = complex(-0.8, 0.156)     # spiral arms
C = complex(-0.4, 0.6)       # denser, more organic clusters
C = complex(0.285, 0.01)     # rabbit-like lobes
```

Any `c` inside the Mandelbrot set tends to produce a connected Julia
set; any `c` outside it produces a "dust" of disconnected points.

## How it works (code walkthrough)

- **`make_grid(...)`** — builds a 2D grid of complex numbers `z0`, one
  per output pixel, spanning the `[XMIN, XMAX] x [YMIN, YMAX]` window.
- **`escape_time(c, z0, ...)`** — vectorized (NumPy, no Python loops
  over pixels) escape-time computation. Iterates `z = z² + c` for every
  point simultaneously, tracking which points have crossed the bailout
  radius and recording a *smooth* escape value for each. This is the
  core fractal math and is shared logic you'd reuse for a Mandelbrot
  set too (there, `z0 = 0` and `c` varies instead of `z0`).
- **`render_escape_image(...)`** — normalizes the smooth escape values
  into a `[0, 1]` range for use as image data, leaving never-escaped
  (interior) points at `0`.
- **`julia_image(...)`** — ties the above together: builds the grid,
  runs escape-time, returns the normalized image array.
- **Render/save block** — calls `julia_image`, plots it with
  `matplotlib` using a custom dark-to-white "fractal" colormap
  (`FRACTAL_CMAP`), saves to `OUT_FILE`, and displays it.

## Performance notes

- Runtime scales roughly with `WIDTH * HEIGHT * MAX_ITER` in the worst
  case (points that never escape run the full iteration count). Most
  points escape early, so real runtimes are much lower in practice.
- To render faster while experimenting, temporarily lower `MAX_ITER`
  (e.g. `100`) and/or the resolution, then raise them again for a
  final high-quality render.

## License

Use, modify, and share freely.
