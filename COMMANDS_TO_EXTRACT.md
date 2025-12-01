# Commands to Extract

A working list of commands and their source files for extraction.

---

## Training

| Source File | Lines | Notes |
|:------------|:------|:------|
| `train_runner.py` | 62 | DFL subprocess wrapper |
| `train_handler.py` | ~150 | Config handling |
| `training_config.py` | ~200 | Config definitions |

**Status:** To evaluate against DFLEssentials

---

## Inference (encode/decode)

| Source File | Lines | Notes |
|:------------|:------|:------|
| `merge_runner.py` | 469 | Contains inference logic |
| `rawPredMerge.py` | 223 | Raw prediction handling |

**Status:** To evaluate against DFLEssentials (`encode`, `decode`, `inference` commands)

---

## Merge (composite onto plate)

| Source File | Lines | Notes |
|:------------|:------|:------|
| `rawPredMerge.py` | 223 | `merge_rawpred()` - warps face onto plate |
| `merge_runner.py` | 469 | Post-processing, MP4 generation |

**Status:** May deprecate in favour of separate inference + composite steps

---

## Match Pose

| Source File | Lines | Notes |
|:------------|:------|:------|
| `match_pose.py` | 135 | Core pose matching logic |
| `match_pose_handler.py` | ~100 | Handler |

**Status:** To extract

---

## Remask

| Source File | Lines | Notes |
|:------------|:------|:------|
| `remask.py` | 78 | XSeg mask application |
| `remask_handler.py` | ~80 | Handler |

**Status:** Diogo working on masks - align scope

---

## Mask Training

| Source File | Lines | Notes |
|:------------|:------|:------|
| `mask_train.py` | 200 | XSeg training loop |
| `mask_train_runner.py` | ~150 | Runner |
| `augmentations.py` | 180 | Training augmentations |

**Status:** Diogo working on masks - align scope

---

## Face Part Mask

| Source File | Lines | Notes |
|:------------|:------|:------|
| `fp_mask.py` | 335 | DeepLabV3+ inference |
| `face_part_mask_handler.py` | ~100 | Handler |

**Status:** To extract

---

## Export DFM

| Source File | Lines | Notes |
|:------------|:------|:------|
| `export_DFM_runner.py` | ~150 | Model export |
| `export_DFM_handler.py` | ~80 | Handler |

**Status:** To extract

---

## Shared Utilities

| Source File | Lines | Notes |
|:------------|:------|:------|
| `aligned_face.py` | 790 | Face alignment utilities |
| `LandmarksProcessor.py` | 1,128 | Landmark detection |
| `latent.py` | 150 | Latent space utilities |
| `warp_utils.py` | 200 | Warping functions |
| `image_utils.py` | ~300 | Image I/O |
| `mask_utils.py` | ~200 | Mask utilities |

---

## Files to DELETE (not extract)

| File Pattern | Reason |
|:-------------|:-------|
| `*_prompt.py` | Interactive prompts (~2,200 lines) |
| `tracking.py` | Telemetry |
| `progress_handler.py` | Workflow-specific |

---

## Notes

- **DFLEssentials** already has: `train`, `inference`, `encode`, `decode`
- **Terminology:** "inference" = encode/decode, "merge" = composite onto plate
- **Masks:** Diogo doing first pass - align before duplicating work

---

*Last updated: [date]*
