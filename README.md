# From Scene to Stitch

Code and data for the paper *From Scene to Stitch: A Computational Pipeline for Site-Specific Camouflage Generation and Adversarial Evaluation* (Karen Nurlybekov, Computing Science, Thompson Rivers University).

The pipeline takes one photograph of a site and makes a camouflage pattern for it: five colors from the lit, non-sky part of the photograph, and a texture from style transfer of an existing camouflage pattern. Two camouflaged object detectors (SINet-V2 and DGNet) then try to find the new pattern and the pattern it was made from. Two sheets were printed on cloth and tested in the field.

All the code is inside the three notebooks. Each notebook has a "Code" section, and each code cell there has a short note that names the part of the paper it implements.

## Notebooks

| Notebook | Paper section | Makes |
|---|---|---|
| `01_generation_and_colour_limit.ipynb` | Methodology; Results, "Generation and the cost of the five-color limit" | color calibration, the three patterns, `tab:quant`, `fig_luminance`, `fig_patterns` |
| `02_adversarial_test_vs_base.ipynb` | Evaluation; Results, "Detector test against the base pattern" | `tab:base`, `tab:recolor`, `fig_composites` |
| `03_printed_fabric_field_test.ipynb` | Results, "Printed fabric in the field" | `tab:locA`, `tab:locB`, `fig_field` |

Run them in this order, from the project root. Notebook 2 tests the patterns that notebook 1 makes, and its table `tab:recolor` also reads the notebook 1 results. Each notebook writes to its own folder in `results/`.

Running time on an Apple-silicon Mac (Apple GPU): notebook 1 takes about 9 hours (most of it is the color-limit test), notebook 2 about 3 hours, notebook 3 a few minutes.

In the code and the file names, `woodland` is the base pattern that the paper calls M81.

## Data

`images/`

- `site_photos/`: the 40 RAW photographs (DNG) of the site set. `IMG_4171` is the generation frame. `IMG_4174`, `IMG_4193`, `IMG_4194`, `IMG_4198`, `IMG_4204` and `IMG_4209` show the ColorChecker chart, and `IMG_4209` is the one used for calibration. The other 33 are the test scenes.
- `base_patterns/`: the three base patterns (`multicam.jpg`, `woodland.jpg`, `realtree.jpg`) and the RAW photographs of the garments they were made from (`raw/`).
- `field_2026-09-18_two_sites/`: the RAW photographs of the field test at the two locations, and in `_analysis/` the traced outline of each item (`gt_*.png`, the masks that the code reads; `masks/` has the same masks and each outline drawn on its photograph).
- `printed_sheets/`: the two printed designs, `print_v1` (the pipeline's own sheet) and `print_v2` (a contrast-enhanced copy, not pipeline output).

`results/native_resolution_2026-09-24/` holds an earlier run with the base pattern at its original size. Notebooks 1 and 2 compare against it.

The calibrated PNG files are not stored; the notebooks make them from the RAW files.

## Setup

Python 3.12. The versions below are the ones the results were made with.

```
pip install numpy==2.2.6 opencv-python==4.12.0.88 pillow==11.1.0 scikit-learn==1.6.1 scipy==1.15.1 \
    matplotlib==3.10.0 rawpy==0.27.0 colour-science==0.4.7 colour-checker-detection==0.2.3 \
    tensorflow==2.18.0 tensorflow-metal==1.2.0 keras==3.8.0 torch==2.11.0 torchvision==0.26.0 \
    timm==1.0.27 jupyter
```

(`tensorflow-metal` is for the Apple GPU only.)

The two detectors are not in this repository. Clone them into `models/` and download their published weights as their READMEs describe:

```
git clone https://github.com/GewelsJI/SINet-V2 models/SINet-V2
git clone https://github.com/GewelsJI/DGNet models/DGNet
```

The weights must be at:

- `models/SINet-V2/snapshot/SINet_V2/Net_epoch_best.pth`
- `models/DGNet/lib_pytorch/snapshot/DGNet.pth` (the EfficientNet-B4 model)

Newer torchvision versions do not have `torchvision.models.utils`. In `models/DGNet/lib_pytorch/lib/PVTv2.py`, replace the line `from torchvision.models.utils import load_state_dict_from_url` with:

```python
try:
    from torchvision.models.utils import load_state_dict_from_url
except ImportError:
    from torch.hub import load_state_dict_from_url
```

Style transfer uses the ImageNet VGG-19 weights, which Keras downloads on the first run.

## Settings

Each notebook starts with a settings cell. `DEVICE = 'mps'` uses the Apple GPU; set it to `'cpu'` on other machines. `FRESH_RUN = True` deletes the notebook's earlier results before it starts; with `FRESH_RUN = False` an interrupted detector run continues where it stopped.

## Notes

- The k-means palette depends on the number of CPU threads. The results were made with 10 threads.
- Do not import torch before the palette step in the same session. That changes the palette slightly, so the notebooks load torch only when the detectors are built.
- The file that went to the printer for `print_v1` was not kept; `images/printed_sheets/` holds the file of 2026-09-02. The code that made the two sheets is in the Appendix of notebook 1 (`print_v1`) and of notebook 3 (`print_v2`); it makes `print_v2` again from that file with 99.3 % of the pixels equal to the printed file.
