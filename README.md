# road-enhance_oswald-topo-print

Blender project (macOS, Blender 5.2): a terrain model of the Koralpe/Soboth area, built from
16 BEV ALS-DTM STL tiles (1 m grid, EPSG:3035, CC BY 4.0). The tiles are merged into one
seamless mesh, `ALS_DTM_CRS3035RES50000mN2600000E4700000_model_21_seamless`, with 3.04 M vertices.

Scale: 1 model unit ≈ 51.5 m, with no vertical exaggeration. The scale was fitted from 9 summits
and the lake level (1080 m).

Two scripts find the B69 (Südsteirische Grenzstraße) in the DTM and gently emphasise it.
They use only `bpy`, `mathutils` and `numpy` (no SciPy).

## Layout

```
scripts/b69/
├── b69_1_find_road.py      # detection; writes data only, no geometry change
└── b69_2_enhance_road.py   # z-only enhancement, reversible
```

Not tracked in the repo:
- `*.blend` files (~280 MB): use Git LFS or keep them out of the repo.
- `*.blend1` backup files.
- Map screenshots: these are mentioned here and not committed.

## B69 road enhancement

### Script 1: `b69_1_find_road.py`

Pipeline: map pixels → fragment chain (MST, gap penalty) → corridor raster (0.05 u)
→ bench score (small-scale slope ÷ large-scale slope) → Dijkstra least-cost path.

It writes these outputs on the terrain object:
- curve `B69_centerline` (hidden in render)
- vertex group `B69_road`
- point attributes `b69_dist`, `b69_zc`, `b69_w`

Runtime is about 18 s. The geometry is not changed.

### Script 2: `b69_2_enhance_road.py`

```
z += w · STRENGTH · ((zc − DEPTH) − z)
```

This applies to road vertices only. It skips vertices more than `MAX_DZ` from the road grade,
which protects the lake. The original heights are stored in `b69_z_orig`, so the script can be
re-run. `STRENGTH = 0` restores the original.

| Parameter  | Default | Note      |
|------------|---------|-----------|
| `STRENGTH` | 0.5     |           |
| `DEPTH`    | 0.02    | ≈ 1 m     |
| `MAX_DZ`   | 0.15    |           |

With the defaults, the median shift is ≈ 0.36 m.

## Usage

1. In script 1, set:
   - `OBJ_NAME`: the terrain object's name as shown in the Outliner. This is not a file path.
   - `MAP_IMAGE`: the absolute path to the uncropped OSM screenshot with the full red 4×4 grid,
     e.g. `/Users/crse/Desktop/grid.png`. The committed value stays the placeholder
     `"/path/to/map_screenshot.png"`.
2. Run script 1. Check the result with `B69_centerline` or the `B69_road` weight paint.
3. Run script 2. Tune its parameters and re-run as needed.

## Compatibility

Tested with Blender 5.0.1. Not yet confirmed on Blender 5.2.

## Data attribution

Terrain: BEV ALS DTM (data.bev.gv.at), CC BY 4.0.
Map background: © OpenStreetMap contributors.
