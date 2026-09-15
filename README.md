# Lightweight Photo Editor (C++)

A fast, native, dependency-light photo & paint editor written entirely in C++.
Final binary is ~1.4 MB, far under the 300 MB budget.

## Layout

- **Top bar**: File / Edit / Filters / Plugins / Help menus, plus a live info
  line (current file, canvas size, active tool)
- **Left**: canvas / workspace (checkerboard shows transparency)
- **Right**: collapsible sections — Tools, Layers, Adjustments, Filters, Transform, History
- **Bottom**: status line + imported Filters/Plugins, each with an Amount
  slider (0-200%) and an apply counter

The app opens **maximized** by default, and new documents default to
**1920x1080**.

## Keyboard shortcuts

Menus: `Ctrl+F` File · `Ctrl+E` Edit · `Ctrl+Shift+F` Filters · `Ctrl+P` Plugins · `Ctrl+H` Help
(File and Filters both start with F, so Filters uses `Ctrl+Shift+F`.)

Actions: `Ctrl+N` New · `Ctrl+O` Open · `Ctrl+S` Save · `Ctrl+Shift+S` Save As ·
`Ctrl+Z` Undo · `Ctrl+Y` Redo · `Ctrl+X` Cut · `Ctrl+C` Copy · `Ctrl+V` Paste

**Every slider supports double-click to type an exact value** instead of dragging.

## What changed in this revision

- **Removed** Rectangle Select and Lasso Select tools (Cut/Copy/Paste now act
  on the whole active layer instead)
- **Brush is smoother**: strokes now use a soft/feathered circular edge
  (antialiased) instead of a hard-edged circle, plus denser interpolation
  while dragging so fast strokes don't leave gaps
- **Eraser fixed**: it was always making pixels transparent correctly, but
  with no visual cue that looked like "painting black" — the canvas now
  draws a checkerboard behind the image so transparency is unmistakable
- **Eyedropper fixed**: it was actually picking colors already, but gave no
  feedback — it now shows a live color swatch + RGBA readout and
  automatically switches to the Brush tool after picking
- **Layers panel fixed**: it was deep-copying the entire document (every
  layer's full pixel buffer) every single frame just to draw the list, which
  caused the sluggishness that looked like "not working." It now reads the
  document by reference and only copies/commits when you actually interact
  with a control
- **Imported filters/plugins moved to the bottom bar**, each with an Amount
  slider (0-200%) that blends the effect against the original, and a
  running "(Nx)" apply counter
- **Sidebar sections are now collapsible** (`CollapsingHeader`) instead of
  one long scroll
- **Built-in filters show an apply counter** next to their button (e.g.
  "Apply Vignette (2x)") so stacked effects are visible
- **Blur sigma / sharpen amount / vignette amount reset to their default**
  after clicking Apply, instead of drifting
- **Flip/Rotate use compact icon-style buttons** (FlipH, FlipV, 90 CW, 180,
  90 CCW) instead of full words, with tooltips. Note: Dear ImGui's bundled
  font doesn't include arrow/unicode glyphs without adding a font file as a
  dependency, so these are short abbreviations styled as a toolbar rather
  than true icon glyphs — flagging this trade-off rather than silently
  shipping tofu boxes.

## Features

**Import / Export** — Open JPG/PNG, Save/Save As PNG or JPG, New document (with 1080p/4K/Square presets)

**Layers** — add, delete, reorder, toggle visibility, adjust opacity; tools/filters/adjustments act on the active layer

**Paint tools** — Brush (size + color, smoothed edges), Eraser (size + strength, smoothed edges), Paint Bucket fill (tolerance), Eyedropper (with swatch feedback), Crop

**Adjustments (live preview on active layer)** — Brightness, Contrast, Saturation, Hue, Exposure, Gamma

**Filters (active layer)** — Grayscale, Sepia, Invert, Gaussian Blur, Sharpen, Vignette — each shows how many times it's been applied

**Transforms (whole canvas)** — Flip H/V, Rotate 90/180/270, free rotation, Resize

**Filters & Plugins menus** — each has a single **Import...** item:
- `Filters > Import...` loads a `.cpfilter` file (one filter)
- `Plugins > Import...` loads a `.cpplugin` file (a bundle of several filters)

Both appear in the **bottom bar** with an Amount slider and Apply button.
Two example files are in `examples/` (`vintage.cpfilter`, `starter_pack.cpplugin`).

### Custom filter/plugin file format

Plain text, hand-editable in Notepad:

```
# my_filter.cpfilter
NAME=Vintage Warmth
brightness=8
contrast=-5
saturation=-15
sepia=1
vignette=0.45
```

```
# starter_pack.cpplugin — bundles multiple filters as [Sections]
[Cool Blue]
hue=200
saturation=-10

[High Contrast B&W]
grayscale=1
contrast=35
```
Recognized keys: `brightness`, `contrast`, `saturation`, `hue`, `exposure`, `gamma`,
`grayscale`, `sepia`, `invert`, `blur`, `sharpen`, `vignette` (Help > Custom Filter/Plugin Format in-app has ranges).

**Undo / Redo** — full history across layers, paint strokes, filters, transforms

## Tech stack

| Purpose | Library | Notes |
|---|---|---|
| GUI | Dear ImGui + GLFW + OpenGL3 | No heavy toolkit like Qt |
| Image decode/encode | stb_image / stb_image_write | Single header, no libjpeg/libpng |
| Resize | stb_image_resize2 | Single header |
| File dialogs | tinyfiledialogs | Native OS dialogs, single file |

## Project layout

```
PhotoEditor/
├── CMakeLists.txt
├── examples/
│   ├── vintage.cpfilter
│   └── starter_pack.cpplugin
├── src/
│   ├── main.cpp          # GUI, menu bar, collapsible panels, bottom bar, canvas
│   ├── Image.h            # RGBA8 image buffer struct
│   ├── ImageIO.h/.cpp     # Load/save JPG/PNG
│   ├── Editor.h/.cpp      # Adjustments, filters, geometric transforms
│   ├── Layer.h             # Layer struct + compositing
│   ├── Selection.h         # Selection mask (paint-tool clipping infra)
│   ├── PaintTools.h/.cpp  # Brush, eraser (feathered), flood fill, eyedropper
│   ├── Preset.h/.cpp      # .cpfilter / .cpplugin parser + applier
│   └── History.h          # Undo/redo stack
└── external/
    ├── imgui/
    ├── stb/
    └── tinyfiledialogs/
```

## Building

### Linux (Debian/Ubuntu)
```bash
sudo apt install cmake build-essential libglfw3-dev libgl1-mesa-dev \
    libx11-dev libxrandr-dev libxinerama-dev libxcursor-dev libxi-dev
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
./PhotoEditor
```

### macOS
```bash
brew install cmake glfw
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(sysctl -n hw.ncpu)
./PhotoEditor
```

### Windows (Visual Studio + vcpkg)
```powershell
cd C:\vcpkg
.\vcpkg install glfw3:x64-windows opengl:x64-windows
cd C:\PhotoEditor
mkdir build; cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=C:\vcpkg\scripts\buildsystems\vcpkg.cmake -DVCPKG_TARGET_TRIPLET=x64-windows
cmake --build . --config Release
.\Release\PhotoEditor.exe
```
Builds as a proper GUI app (no console window).

## Publishing / Distribution — what end users need

**git/CMake/vcpkg/Visual Studio are for you (the developer) only.** Someone
you send the finished `.exe` to needs none of them.

**Default build:** nothing extra, except possibly the free Microsoft Visual
C++ Redistributable (commonly already present): https://aka.ms/vs/17/release/vc_redist.x64.exe

**Fully self-contained build** (zero dependencies, recommended before publishing):
```powershell
cd C:\vcpkg
.\vcpkg install glfw3:x64-windows-static opengl:x64-windows-static
cd C:\PhotoEditor\build
cmake .. -DCMAKE_TOOLCHAIN_FILE=C:\vcpkg\scripts\buildsystems\vcpkg.cmake ^
         -DVCPKG_TARGET_TRIPLET=x64-windows-static ^
         -DSTATIC_RUNTIME=ON
cmake --build . --config Release
```
The resulting `.exe` needs nothing on the target machine beyond the OpenGL
driver every GPU already ships.

## How it was verified

Built and compiled in a sandboxed Linux environment to confirm it compiles
cleanly and links successfully (~1.4 MB binary). That sandbox has no display
server, so the underlying image-processing engine was separately verified
with a headless test harness in earlier revisions of this project; this
revision's UI/interaction logic (menus, panels, canvas tools) follows the
same tested calling conventions into `Editor.cpp`/`PaintTools.cpp`.

## Known simplifications

- History stores full document snapshots (all layers) per undo step.
- Undo/redo resets tool sliders for simplicity.
- All layers share the canvas' width/height (no per-layer offset).
- Transforms (flip/rotate/resize/crop) apply to all layers together, keeping
  canvas size consistent; adjustments/filters apply to the active layer only.
- Flip/Rotate buttons use short text abbreviations, not true icon glyphs
  (no font dependency added) — see "What changed" above.
