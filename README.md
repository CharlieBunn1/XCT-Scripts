# XCT-Scripts

Python scripts for post-processing X-ray Computed Tomography (XCT) data, developed for use with Nikon CT systems. Covers the full pipeline from raw reconstructed slices through to a trained ML segmentation model.

---

## Scripts

### Preprocessing

#### `stitch_2_scans.ipynb`
Stitches together two overlapping XCT scans into a single volume. Use this when a sample has been scanned in two positions (e.g. top and bottom) and needs to be combined.

#### `stitch_3_scans.ipynb`
Stitches together three overlapping XCT scans into a single volume. Use this when a sample has been scanned in three positions (e.g. top, middle, and bottom).

#### `tiffs_to_stack.ipynb`
Converts a folder of individual TIFF slices into a single multi-page TIFF stack. Useful for consolidating reconstruction output into a single file for downstream analysis — the segmentation notebooks below expect a single stack as input.

### Segmentation & machine learning

#### `xct_membrane_segmentation.ipynb`
Samples 50 random slices from a full XCT stack (voidage / membrane / polymer cartridge, separable by attenuation) and produces an automatic first-pass 3-class segmentation via multi-Otsu thresholding with denoising and morphological cleanup. Provides a napari-based workflow for hand-correcting those masks into ground truth, plus two ways to speed that up as you go: a random-forest bootstrap trained on whatever you've corrected so far, and a way to pull in predictions from `xct_unet_training.ipynb` once a model's been trained, so later corrections start from an increasingly good baseline rather than the raw threshold output.

#### `inspect_mask_values.ipynb`
Small diagnostic notebook for label mask tif files. Label values (0/1/2) look black/near-black at raw display scaling, which can look like an empty mask even when it isn't — this prints the actual unique pixel values per file and renders a colour-mapped preview so mask content can be checked visually.

#### `xct_unet_training.ipynb`
Trains a U-Net (`segmentation_models`, Keras/TensorFlow backend, ResNet34 encoder) on the hand-corrected ground-truth slices from `xct_membrane_segmentation.ipynb`, using random augmented patches given how few labelled slices there typically are early on. Runs tiled sliding-window inference over the remaining unlabelled samples and saves predictions for use back in the annotation notebook. Designed to be re-run periodically as the ground-truth set grows.

---

## Requirements

- Python 3.9+
- Jupyter Notebook or JupyterLab

**Preprocessing** (`stitch_*`, `tiffs_to_stack`): `numpy`, `tifffile`

**Segmentation & correction** (`xct_membrane_segmentation`, `inspect_mask_values`): `numpy`, `pandas`, `matplotlib`, `tifffile`, `scipy`, `scikit-image`, `scikit-learn`, `tqdm`, plus `napari` and a Qt binding (e.g. `pip install PyQt5`) for the manual-correction step.

**U-Net training** (`xct_unet_training`): `tensorflow<2.16`, `numpy<2.0`, `segmentation-models`, `albumentations`, `scikit-learn`.
- `segmentation_models` hasn't been updated for Keras 3 / TensorFlow >=2.16 — pin both `tensorflow` and `numpy` together, or imports fail with a NumPy/TensorFlow ABI mismatch (`numpy.dtype size changed...`).
- For GPU: install with `pip install "tensorflow[and-cuda]<2.16"` — a plain `pip install tensorflow` on Linux won't discover your GPU even with drivers correctly installed, since the CUDA/cuDNN runtime libraries need to come from the `[and-cuda]` extra.

Install dependencies with:
```
pip install -r requirements.txt
```
*(Add a `requirements.txt` to this repo once your package list is confirmed.)*

---

## Usage

Typical pipeline order:

1. **Stitch**, if the sample was scanned in multiple positions (`stitch_2_scans.ipynb` / `stitch_3_scans.ipynb`).
2. **Consolidate to a stack** (`tiffs_to_stack.ipynb`).
3. **Sample, auto-segment, and hand-correct ground truth** (`xct_membrane_segmentation.ipynb`) — use `inspect_mask_values.ipynb` if a mask ever looks suspiciously empty.
4. **Train a U-Net** on the corrected ground truth and predict the rest (`xct_unet_training.ipynb`).
5. **Feed U-Net predictions back** into `xct_membrane_segmentation.ipynb` to lightly correct them and grow the ground-truth set, then periodically retrain as it grows.

For each notebook: open in Jupyter, edit the `CONFIG`/paths cell at the top, run all cells.

---

## Notes

- Raw scan data, reconstructions, and TIFF files are excluded from this repository via `.gitignore`
- Developed for scans acquired on a Nikon Reflection 225 system
- `xct_membrane_segmentation.ipynb`, `inspect_mask_values.ipynb`, and `xct_unet_training.ipynb` all expect to share the same `output_dir` (`images/`, `masks_corrected/`, `masks_unet/`, `sample_manifest.csv`) so they can read each other's outputs — point all three at the same folder.

---

## Author

Charlie Bunn
