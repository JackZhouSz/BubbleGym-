# 400x400 thumbnails (callout bubbles only)

This folder is not a full set, by design. It holds the eight 400 px masters that
the paper's Fig. 3 draws as circular callouts over the non-sphericity vs.
frequency scatter -- the bubbles listed as `SELECTED_CALLOUT_MESHES` in
`python/visualization/plot_dataset_distribution.py`:

| | |
|---|---|
| `VOF/bubble.0218.6.png` | `LBM/bubble.0101.33.png` |
| `VOF/bubble.0225.71.png` | `LBM/bubble.0064.115.png` |
| `VOF/bubble.0278.20.png` | `VOF/bubble.0244.12.png` |
| `VOF/bubble.0222.38.png` | `LBM/bubble.0009.2.png` |

The other 9992 bubbles are only ever drawn as small scatter points, so they do
not need this 400x400 resolution. The complete library at 100 px ships in 
[`bubble_mesh_thumbnails_100x100/`](../bubble_mesh_thumbnails_100x100/) (10000 files, 64 MB).

`plot_dataset_distribution.py` searches this folder first and falls back to the
100 px library for anything missing, rescaling whichever it finds, so the figure
still renders if these files are absent. 

## Regenerating

Rendering all 10000 bubbles at 400 px would take roughly **750 MB**, about 12x
the 100 px library, which is why the full set is not in the repository. The
meshes are a separate download (see `dataset/README.md`); point
`BUBBLEGYM_MESH_ROOT` at them, then:

```bash
# all 10000 at 400 px -- large, see the size note above
python python/visualization/render_dataset_thumbnails.py --out-size 0

# smoke test: the first 50 rows of the CSV
python python/visualization/render_dataset_thumbnails.py --out-size 0 --limit 50
```

`--out-size 0` keeps the 400 px render instead of downsampling to 100 px, and
writes back into this folder.

### Rendering only selected bubbles

The script renders one thumbnail per row of the CSV it is given, and has no flag
for picking bubbles by name (`--limit N` only takes the first N rows). To render
an arbitrary subset, filter the dataset CSV down to the `mesh_id`s you want and
render from that copy. Taking the eight callouts above as the example:

```bash
python -c "
import pandas as pd
wanted = [
    'VOF/bubble.0218.6.obj',   'VOF/bubble.0225.71.obj',
    'VOF/bubble.0278.20.obj',  'VOF/bubble.0222.38.obj',
    'LBM/bubble.0101.33.obj',  'LBM/bubble.0064.115.obj',
    'VOF/bubble.0244.12.obj',  'LBM/bubble.0009.2.obj',
]
df = pd.read_csv('dataset/bubble_gym/dataset_bubblegym_10k.csv')
df[df.mesh_id.isin(wanted)].to_csv('subset.csv', index=False)
"
python python/visualization/render_dataset_thumbnails.py \
    --csv subset.csv --out-size 0 --force
```

`--force` re-renders bubbles whose PNG is already present; drop it to fill in
only the missing ones.
