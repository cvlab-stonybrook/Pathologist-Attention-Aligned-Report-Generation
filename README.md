# Pathologist-Attention-Aligned-Report-Generation

Official dataset and code for **"Pathologist Attention–Aligned Report Generation for Prostate Histopathology"** (MICCAI 2026).

> **Data requests:** for scanpath and other data beyond what is released here, please email **ruoxue@cs.stonybrook.edu**.
>
> The code and dataset are coming soon.

## Keyword Attention Heatmap Dataset

Pathologists reviewed digitized H&E diagnostic slides from TCGA-PRAD. The dataset provides, for every slide and diagnostic finding (keyword), a heatmap of the regions the pathologists attended to when they reported that finding as **present** or **absent**.

| | |
|---|---|
| Slides | 116 TCGA-PRAD whole-slide images |
| Pathologists | 6 (4 specialists, 1 general pathologist, 1 resident) |
| Heatmaps | 952 (836 keyword-specific + 116 aggregate) |
| Resolution | 64× downsampled from the level-0 (40×) slide |
| Size | ~65 MB |

### Download

The download link will be added here once the dataset is released.

The original whole-slide images (`.svs`) are not included. They can be downloaded from the [NCI Genomic Data Commons](https://portal.gdc.cancer.gov/) (project **TCGA-PRAD**), using the slide ID as the folder name (for example `TCGA-2A-A8VL-01Z-00-DX1`).

### Keywords

| Keyword (file prefix) | # slides with *present* map | # slides with *absent* map |
|---|---:|---:|
| `pattern_3` (Gleason pattern 3) | 94 | 7 |
| `pattern_4` (Gleason pattern 4) | 111 | 9 |
| `pattern_5` (Gleason pattern 5) | 46 | 15 |
| `tertiary_pattern` | 23 | 10 |
| `cribriform_pattern` | 65 | 17 |
| `intraductal_carcinoma` | 39 | 33 |
| `perineural_invasion` | 66 | 31 |
| `lymphovascular_invasion` | 2 | 31 |
| `extraprostatic_extension` | 14 | 87 |
| `positive_surgical_margins` | 20 | 87 |
| `seminal_vesicle_invasion` | 2 | 27 |
| `all_keywords` (aggregate) | 116 | – |

A map is provided only when that keyword and condition were reported for the slide.

### Folder structure

```
keyword_heatmaps_64x/
├── README.md
├── index.json                          # metadata for all slides and maps
├── TCGA-2A-A8VL-01Z-00-DX1/
│   ├── wsi_thumbnail.jpg               # 64× slide thumbnail (same pixel grid as the heatmaps)
│   ├── all_keywords__present.png       # aggregate over all keywords reported as present
│   ├── pattern_3__present.png
│   ├── pattern_4__present.png
│   ├── extraprostatic_extension__absent.png
│   └── ...
└── ...
```

The file name is `<keyword>__<condition>.png`, where `condition` is `present` or `absent`.

### Heatmaps

- **Per keyword:** each map shows the attention associated with one keyword and condition (`present` / `absent`).
- **Aggregate:** `all_keywords__present` combines the attention for all keywords reported as present.
- **Readers:** attention from all pathologists who read a slide is combined. Three slides (`TCGA-G9-6329`, `TCGA-G9-6385`, `TCGA-G9-7522`) were read by all six pathologists.
- **Smoothing:** maps are smoothed with a Gaussian of σ = 2048 level-0 pixels (32 pixels on the 64× grid).

### File format

- **Heatmaps.** 8-bit greyscale PNG of size `ceil(W/64) × ceil(H/64)`, where `W × H` is the level-0 size of the slide. Each map is normalised to its own maximum. The unnormalised intensity (arbitrary units) is `pixel / 255 * value_max`.
- **Pixel grid.** Pixel `(row i, col j)` covers level-0 region `[64j, 64j+64) × [64i, 64i+64)`. To map a level-0 point `(x, y)` onto the grid, use `(x / 64 - 0.5, y / 64 - 0.5)` in image coordinates.
- **Thumbnail.** `wsi_thumbnail.jpg` is resampled from the slide pyramid using each level's exact scale. It lies on the same grid as the heatmaps, so the two can be overlaid directly.
- **`index.json`.** Top-level fields:

```jsonc
{
  "downsample": 64,
  "sigma_level0_px": 2048.0,
  "sigma_64x_px": 32.0,
  "readers": {"<user_id>": "specialist" | "general" | "resident", ...},
  "slides": {
    "TCGA-2A-A8VL-01Z-00-DX1": {
      "img_size_level0": [109559, 81897],        // W, H of the original .svs
      "size_64x": [1712, 1280],                  // W, H of the PNGs and thumbnail
      "thumbnail_source": {...},                 // svs pyramid level used for the thumbnail
      "maps": {
        "pattern_4__present": {
          "file": "pattern_4__present.png",
          "value_max": 4414.68,                  // unnormalised intensity = pixel / 255 * value_max
          "readers": ["OpIEy0dvj8"]              // user_ids that contributed
        }
      }
    }
  }
}
```

Pathologists are identified only by an anonymous `user_id` and their expertise level.

### Quick start

```python
import json
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt

root = "keyword_heatmaps_64x"
index = json.load(open(f"{root}/index.json"))

slide, key = "TCGA-2A-A8VL-01Z-00-DX1", "pattern_4__present"
meta = index["slides"][slide]["maps"][key]

thumb = np.array(Image.open(f"{root}/{slide}/wsi_thumbnail.jpg"))
heat = np.array(Image.open(f"{root}/{slide}/{key}.png")).astype(np.float32)
value = heat / 255 * meta["value_max"]            # unnormalised intensity

plt.imshow(thumb)
plt.imshow(np.ma.masked_less(heat / 255, 0.05), cmap="jet", alpha=0.5)  # hide near-zero values
plt.title(f"{slide}  {key}  readers={[index['readers'][r] for r in meta['readers']]}")
plt.axis("off")
plt.show()
```

## Citation

If you use this dataset, please cite:

```bibtex
@article{xue2026pathologist,
  title={Pathologist Attention-Aligned Report Generation for Prostate Histopathology},
  author={Xue, Ruoyu and Singh, Suryakant and Chakraborty, Souradeep and Marza, Pierre and Yaskiv, Oksana and Friedman, Constantin and Sheuka, Natallia and Friedman, Paul and Ramlal, Bharat and Knudsen, Beatrice and others},
  journal={arXiv preprint arXiv:2607.19624},
  year={2026}
}
```

## Acknowledgements

The slides are from The Cancer Genome Atlas (TCGA-PRAD), accessed through the NCI Genomic Data Commons. Please follow the [GDC data use policies](https://gdc.cancer.gov/access-data/data-access-processes-and-tools) when using the original slides.
