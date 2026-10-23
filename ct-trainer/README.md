# CT Anatomy Trainer (sam.med)

Phase 1: one abdomen–pelvis CT with a colour-matched 3D model and linked axial, coronal and sagittal views.

## site/ — drop this folder into the sam.med repo
Static files only; works on GitHub Pages. Serve over http(s), not file://.

- `index.html` — the viewer (three.js r160 from jsDelivr, no build step)
- `ct_00–07.webp` — CT axial slices as WebP mosaics (8 × 6 slices per sheet, HU mapped to 8-bit, see `meta.json → hu_map`)
- `labels_00–07.png` — label volume, same layout, lossless
- `meshes.png` — 3D meshes packed as bytes in a lossless PNG (uint16 positions × 0.05 mm, uint16/uint32 indices)
- `meta.json` — dimensions, spacing, structure list (names, colours, groups), per-slice vertebral level, landmarks
- `content.json` — all teaching text, editable without touching code:
  - `tour`: the Learn steps (title, body HTML, where to jump, which structures to highlight)
  - `cards`: per-structure notes (on CT, relations, clinical, how to find it), keyed by base name (`kidney`, `adrenal_gland`…)
  - `pairs`: explanations for commonly confused structures, used as quiz feedback and Name-it distractors
  - `tiers`: which structures are Core / More in the quiz and structure list

## Viewer features
- Learn: 22 steps in two parts — Reading a CT (orientation, HU, windows, contrast phase, levels) and a 15-stop search pattern, each stop with a find-it task and a look-for checklist
- Explore: labels off by default; hold Space or "Hold to peek" to show them; Core/All list; structure cards
- Levels: landmarks measured in this scan vs textbook levels
- Quiz: Find it / Name it / Level it, Core / More / Everything tiers, confusable distractors, explanations, missed items weighted to come back (progress kept in the browser)
- PACS-style tools: right-drag window/level (or the Window/level tool), ruler, HU circle (mean, SD, area); keys 1–4 window presets, L cycles labels, ↑↓/PgUp/PgDn scroll
- Phones: one plane at a time with Axial/Coronal/Sagittal tabs

## prep/ — how the assets were built
1. CT and labels: TotalSegmentator dataset v2.0.1, case s0204 (36-year-old man, portal venous CT abdomen-pelvis, no pathology recorded), CC BY 4.0. Pulled from the 23.6 GB Zenodo zip with HTTP range requests, so only this case was downloaded.
2. Organ labels: the dataset's own reference segmentations (117 masks merged into one label map with TotalSegmentator's class order).
3. Abdominal wall muscles: `TotalSegmentator -ta abdominal_muscles`, run in chunks to fit in memory; `assemble_muscles.py` stitches them. The model only covers T4–L4.
4. `python prep.py ct.nii.gz total.nii.gz muscles.nii.gz ../site cases/s0204.json`
   - per-case settings live in `cases/*.json` (output grid, label fixes, source credits)
   - meshes: marching cubes + quadric decimation + Taubin smoothing per structure
   - `levels.py` assigns a vertebral level to every axial slice from the vertebral bodies
   - landmarks (aortic bifurcation, IVC formation, iliac crests, renal hila…) and teaching points (Morison's pouch, ileocaecal junction, rectovesical pouch) are measured in this scan

## Known limits
- Organ labels are the dataset's refined labels; the abdominal wall muscles are automatic and stop at L4.
- No coeliac trunk, SMA, renal vessels, hepatic veins, bile ducts, ureters or appendix labels yet.
- One normal male case. Pancreas-CT (TCIA, CC BY 3.0) is a source of more normal scans, abdomen only.

## Licence and attribution
CT and labels: Wasserthal J et al. TotalSegmentator. Radiology: Artificial Intelligence 2023. Dataset: https://doi.org/10.5281/zenodo.10047292 (CC BY 4.0). Keep the credit line in the page footer.
