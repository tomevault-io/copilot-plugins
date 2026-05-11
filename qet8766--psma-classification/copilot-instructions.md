## psma-classification

> Object classifier over RGB images. Ground-truth pipeline:

# color_axis

Object classifier over RGB images. Ground-truth pipeline:

1. **Rule #1 (per-pixel):** `(R - B) / max(B, 1) ≥ 0.046332746744155884`
   — precomputed in `rgb_lut.npy` (256³ bool LUT).
2. **Morphological cleanup:** 3-step erosion, then 5-step dilation
   (scipy defaults: 4-connected cross, `iterations=3` then `iterations=5`).
3. **Objects:** connected components of the cleaned mask.
4. **Rule #2 (per-object):** `(meanR − meanG) / max(meanG, 1) ≥ 0.034`,
   where means are over the CC's rule-#1-passing pixels. Implemented in
   `rule2.py`.

## Data layout

```
SOURCE IMAGES (variable size, RGB uint8)
────────────────────────────────────────
capture/positive/              capture/negative/
  0000001.png (3475×1979×3)      0000007.png (896×1144×3)
  0000002.png (2639×1897×3)      0000008.png (683×566×3)
  …  (11 files)                  …  (8 files)

             │  flatten each (H,W,3) → (H·W, 3)
             │  concat all images in folder
             ▼

LOOKUP TABLE  (precomputed rule)
────────────────────────────────
rgb_lut.npy   shape (256, 256, 256)   dtype bool   17 MB

   index axes:  [ R ][ G ][ B ]   ← currently only R,B matter
                                    G axis free for future rules

   rule: (R - B) / max(B, 1) ≥ 0.04633…
   positives: 8,017,920 / 16,777,216  cells

             │  for each image:
             │     px   = img.reshape(-1, 3)
             │     keep = px[ lut[px[:,0], px[:,1], px[:,2]] ]
             │  concat keeps within each folder
             ▼

PASSED-PIXEL TABLES   structured dtype = [('R','u1'),('G','u1'),('B','u1')]
───────────────────
positive.npy  (  324,140 ,)   ◄── only pixels that satisfy the rule
negative.npy  (1,050,491 ,)       row = one passing pixel
                                  duplicates kept (preserve frequency)
         R    G    B
       ┌────┬────┬────┐
   0   │ 212│ 204│ 202 │     access:
   1   │ 224│ 216│ 214 │       a = np.load("positive.npy")
   …                          a["R"], a["G"], a["B"]
       └────┴────┴────┘
```

## Scripts

**Rule #1 (pixel layer):**
- `build_lut.py` — builds `rgb_lut.npy` from the rule.
- `build_datasets.py` — builds `positive.npy` / `negative.npy` from `capture/`.

**Rule #2 (object layer):**
- `rule2.py` — production inference. No trained model; `THRESHOLD` is a constant. Entry points: `rule2.build_objects(m1)`, `rule2.object_scores(img, m1, labels_img, n_cc)`, `rule2.classify(img, m1, labels_img, n_cc)`.
- `eval_rule2.py` — image loading, per-image prep (`build_objects`, constants `EROSION_STEPS=3`, `DILATION_STEPS=5`).
- `train_final.py` — recomputes the threshold across all 19 captures and prints it (for sanity-checking when data changes).
- `object_rules.py`, `compare_object.py` — kept as exploratory harnesses for evaluating future candidate rules (not on the production path).

---
> Source: [qet8766/psma_classification](https://github.com/qet8766/psma_classification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
