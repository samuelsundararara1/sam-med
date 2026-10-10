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
- Learn: 14-step guided tour; highlights only the structures each step is about
- Explore: labels off by default; hold Space or "Hold to peek" to show them; Core/All list; structure cards
- Levels: landmarks measured in this scan vs textbook levels
- Quiz: Find it / Name it / Level it, Core / More / Everything tiers, confusable distractors, explanations, missed items weighted to come back (progress kept in the browser)
- PACS-style tools: right-drag window/level (or the Window/level tool), ruler, HU circle (mean, SD, area); keys 1–4 window presets, L cycles labels, ↑↓/PgUp/PgDn scroll
- Phones: one plane at a time with Axial/Coronal/Sagittal tabs

## prep/ — how the assets were built
1. CT: Medical Segmentation Decathlon Task03 `liver_100` (portal venous, 0.7 mm), via 3D Slicer sample data.
2. Labels: `TotalSegmentator -ml` (task `total`), plus `-ta abdominal_muscles` run in four chunks to fit in 4 GB RAM (`assemble_muscles.py` stitches them).
3. `python prep.py ct.nii.gz total.nii.gz muscles.nii.gz ../site`
   - crops to liver dome → below the hips, removes the table, resamples to 1.0 × 1.0 × 1.25 mm
   - marching cubes + quadric decimation + Taubin smoothing per structure
   - `levels.py` assigns a vertebral level to every axial slice from the vertebral bodies (anterior to the cord)
   - landmarks (aortic bifurcation, IVC formation, iliac crests, renal hila…) are measured in this scan

Manual fix applied for this case: TotalSegmentator split T11 into two labels and numbered T9–T11 one too low. Corrected in `prep.py` (`FIX`), checked against the 10th and 11th rib heads.

## Known limits
- Labels are automatic and not hand-corrected.
- The abdominal-wall muscle model only covers T4–L4, so rectus abdominis and the obliques stop at L4.
- No coeliac trunk, SMA, renal vessels or hepatic veins yet (not in the free TotalSegmentator tasks).
- The patient has liver tumours (it is a liver-tumour dataset).

## Licences
CT: CC BY-SA 4.0 (Medical Segmentation Decathlon) — derived assets (slices, meshes) should carry the same licence and attribution.
Labels: TotalSegmentator (Wasserthal et al., Radiology: AI 2023), Apache-2.0 code / open weights for the tasks used.
