# XCT-Scripts

Python scripts for post-processing X-ray Computed Tomography (XCT) data, developed for use with Nikon CT systems.

---

## Scripts

### `stitch_2_scans.ipynb`
Stitches together two overlapping XCT scans into a single volume. Use this when a sample has been scanned in two positions (e.g. top and bottom) and needs to be combined.

### `stitch_3_scans.ipynb`
Stitches together three overlapping XCT scans into a single volume. Use this when a sample has been scanned in three positions (e.g. top, middle, and bottom).

### `tiffs_to_stack.ipynb`
Converts a folder of individual TIFF slices into a single TIFF stack. Useful for consolidating reconstruction output into a single file for downstream analysis.

---

## Requirements

- Python 3.x
- Jupyter Notebook or JupyterLab

Install dependencies with:
```
pip install -r requirements.txt
```
*(Add a `requirements.txt` to this repo once your package list is confirmed.)*

---

## Usage

1. Open the relevant notebook in Jupyter
2. Update the file paths at the top of the notebook to point to your data
3. Run all cells

---

## Notes

- Raw scan data, reconstructions, and TIFF files are excluded from this repository via `.gitignore`
- Developed for scans acquired on a Nikon Reflection 225 system

---

## Author

Charlie Bunn
