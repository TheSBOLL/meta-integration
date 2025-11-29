# File Extraction Map

Detailed mapping from metaface source files to new package locations.

---

## Legend

| Action | Description |
|--------|-------------|
| EXTRACT | Core logic to extract and modernize |
| DELETE | Interactive prompts/telemetry - do not extract |
| NEW | New file to create |
| MERGE | Combine multiple source files |

---

## deepface-core

### CLI Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| cli/main.py | NEW | NEW | Click group entry point |
| cli/train.py | train_handler.py (33 lines) | EXTRACT | Convert to Click command |
| cli/merge.py | merge_in_batch_handler.py (69 lines) | EXTRACT | Convert to Click command |

### Config Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| config/base.py | NEW | NEW | Base config classes, enums |
| config/training.py | train_prompt.py (455 lines) | EXTRACT | Extract parameters only, delete prompts |
| config/merging.py | merge_in_batch_prompt.py (788 lines) | EXTRACT | Extract parameters only, delete prompts |

### Core Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| core/trainer.py | train_runner.py (62 lines), dfl.py (159 lines) | MERGE | DFL command execution |
| core/merger.py | merge_runner.py (469 lines), rawPredMerge.py (223 lines) | MERGE | All merge logic |
| core/grid_generator.py | grid_runner.py (~800 lines), grid_utils.py (180 lines) | MERGE | Grid generation |

### Models Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| models/saehd.py | NEW | NEW | SAEHD model wrapper |
| models/xseg.py | NEW | NEW | XSeg model wrapper |

### Pipelines Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| pipelines/training.py | NEW | NEW | High-level training API |
| pipelines/merging.py | NEW | NEW | High-level merging API |

### Utils Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| utils/dfl_wrapper.py | dfl.py (159 lines), run_cmd.py | MERGE | DFL execution wrapper |
| utils/dfl_image.py | dfl_image_utils.py (120 lines) | EXTRACT | DFL image utilities |
| utils/checkpoints.py | NEW | NEW | Checkpoint management |
| utils/latent.py | latent.py (150 lines), latent_utils.py | MERGE | Latent manipulation |
| utils/warping.py | warp_utils.py (200 lines), warp.py | MERGE | Warp calculations |
| utils/grid.py | grid_utils.py (180 lines) | EXTRACT | Grid utilities |
| utils/video.py | ffmpeg.py (200 lines), generate_mp4.py | MERGE | Video generation |
| utils/nuke_export.py | nuke_parser.py (150 lines) | EXTRACT | Nuke tracker export |

### Ivy Layer (Phase 2)

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| ivy/spider_adapter.py | NEW | NEW | Spider query adapter |
| ivy/publish_adapter.py | NEW | NEW | PipePublish adapter |

---

## face-processing-toolkit

### CLI Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| cli/main.py | NEW | NEW | Click group entry point |
| cli/match_pose.py | match_pose_handler.py (85 lines) | EXTRACT | Convert to Click |
| cli/remask.py | remask_handler.py (32 lines) | EXTRACT | Convert to Click |
| cli/mask_train.py | mask_train_handler.py (45 lines) | EXTRACT | Convert to Click |
| cli/face_part_mask.py | face_part_mask_handler.py (60 lines) | EXTRACT | Convert to Click |
| cli/export_dfm.py | export_DFM_handler.py (40 lines) | EXTRACT | Convert to Click |

### Config Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| config/base.py | NEW | NEW | Base config classes |
| config/pose_matching.py | match_pose_prompt.py | EXTRACT | Extract parameters |
| config/remasking.py | remask_prompt.py | EXTRACT | Extract parameters |
| config/mask_training.py | mask_train_prompt.py | EXTRACT | Extract parameters |
| config/face_part_masking.py | face_part_mask_prompt.py | EXTRACT | Extract parameters |
| config/export.py | export_DFM_prompt.py | EXTRACT | Extract parameters |

### Core Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| core/pose_matcher.py | match_pose.py (135 lines), match_pose_handler.py (85 lines) | MERGE | Pose matching logic |
| core/remasker.py | remask.py (78 lines) | EXTRACT | XSeg remask logic |
| core/mask_trainer.py | mask_train.py (200 lines) | EXTRACT | Mask training logic |
| core/face_part_masker.py | fp_mask.py (335 lines) | EXTRACT | Face part masking |
| core/dfm_exporter.py | export_DFM_runner.py (33 lines) | EXTRACT | DFM export |

### Models Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| models/face_detector.py | face_detect.py | EXTRACT | Face detection |
| models/landmark_detector.py | LandmarksProcessor.py (1,128 lines) | EXTRACT | Landmark detection |
| models/mask_model.py | face_mask.py (41 lines) | EXTRACT | DeepLabV3+ masking |
| models/xseg_model.py | NEW | NEW | XSeg model wrapper |

### Data Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| data/face_dataset.py | dataset.py (104 lines) | EXTRACT | PyTorch dataset |
| data/augmentations.py | augmentations.py (267 lines) | EXTRACT | Training augmentations |

### Pipelines Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| pipelines/pose_matching.py | NEW | NEW | High-level API |
| pipelines/remasking.py | NEW | NEW | High-level API |
| pipelines/mask_training.py | NEW | NEW | High-level API |
| pipelines/face_part_masking.py | NEW | NEW | High-level API |
| pipelines/export.py | NEW | NEW | High-level API |

### Utils Layer

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| utils/alignment.py | aligned_face.py (790 lines) | EXTRACT | Face alignment math |
| utils/masks.py | mask_utils.py | EXTRACT | Mask utilities |
| utils/polygons.py | IEPolys.py (150 lines), SegIEPolys.py (180 lines) | MERGE | Polygon handling |
| utils/math.py | umeyama.py (50 lines), mathlib.py (100 lines) | MERGE | Math utilities |
| utils/image_io.py | image_utils.py | EXTRACT | Image I/O |

### Ivy Layer (Phase 2)

| Target | Source | Action | Notes |
|--------|--------|--------|-------|
| ivy/spider_adapter.py | NEW | NEW | Spider query adapter |
| ivy/publish_adapter.py | NEW | NEW | PipePublish adapter |

---

## Shared Utilities

Files used by both packages - extract to shared location or duplicate with minimal changes.

| Source | Lines | Used By | Target |
|--------|-------|---------|--------|
| image_utils.py | ~150 | Both | utils/image_io.py in each |
| folder_utils.py | ~100 | Both | utils/paths.py in each |
| batched.py | ~50 | Both | utils/batched.py in each |

---

## Files to Delete

Do not extract these files - they contain interactive prompts, telemetry, or workflow-specific logic.

### Interactive Prompts (DELETE)

| File | Lines | Reason |
|------|-------|--------|
| train_prompt.py | 455 | InquirerPy prompts |
| merge_in_batch_prompt.py | 788 | InquirerPy prompts |
| match_pose_prompt.py | ~100 | InquirerPy prompts |
| remask_prompt.py | ~80 | InquirerPy prompts |
| mask_train_prompt.py | ~120 | InquirerPy prompts |
| face_part_mask_prompt.py | ~90 | InquirerPy prompts |
| export_DFM_prompt.py | ~60 | InquirerPy prompts |
| extract_prompt.py | ~200 | InquirerPy prompts |
| All other *_prompt.py | ~1,500 | InquirerPy prompts |

### Telemetry & Workflow (DELETE)

| File | Lines | Reason |
|------|-------|--------|
| tracking.py | ~200 | Telemetry |
| slack_metaface_app.py | ~150 | Slack integration |
| progress_handler.py | ~100 | Workflow-specific |
| metaflow_config_handler.py | ~80 | Workflow-specific |
| memory_usage_monitor.py | ~60 | Telemetry |

### Menu Systems (DELETE)

| File | Lines | Reason |
|------|-------|--------|
| face_tools_handler.py | 130 | Menu routing |
| start_handler.py | ~100 | Menu routing |
| main.py (handlers) | ~50 | Menu routing |

---

## Extraction Priority

### High Priority (Week 1)

1. **Training Core**
   - train_runner.py → core/trainer.py
   - dfl.py → utils/dfl_wrapper.py
   - TrainingConfig from train_prompt.py parameters

2. **Merging Core**
   - merge_runner.py → core/merger.py
   - rawPredMerge.py → core/merger.py
   - MergingConfig from merge_in_batch_prompt.py parameters

3. **Face Tools Foundation**
   - aligned_face.py → utils/alignment.py
   - umeyama.py + mathlib.py → utils/math.py
   - match_pose.py → core/pose_matcher.py

### Medium Priority (Week 2)

4. **Training Advanced**
   - Checkpoint management
   - Progress callbacks
   - Batch processing

5. **Merging Advanced**
   - grid_runner.py → core/grid_generator.py
   - latent.py → utils/latent.py
   - warp_utils.py → utils/warping.py

6. **Remaining Face Tools**
   - remask.py → core/remasker.py
   - mask_train.py → core/mask_trainer.py
   - fp_mask.py → core/face_part_masker.py
   - export_DFM_runner.py → core/dfm_exporter.py

### Lower Priority (Week 3)

7. **Models**
   - LandmarksProcessor.py → models/landmark_detector.py
   - face_mask.py → models/mask_model.py
   - Model wrappers

8. **Documentation & Testing**
   - Integration tests
   - API documentation
   - Examples

---

## Line Count Summary

### deepface-core

| Layer | Files | Estimated Lines |
|-------|-------|-----------------|
| CLI | 3 | 300 |
| Config | 3 | 500 |
| Core | 3 | 800 |
| Models | 2 | 200 |
| Pipelines | 2 | 300 |
| Utils | 8 | 1,200 |
| Tests | 3 | 600 |
| **Total** | **24** | **3,900** |

### face-processing-toolkit

| Layer | Files | Estimated Lines |
|-------|-------|-----------------|
| CLI | 6 | 400 |
| Config | 6 | 400 |
| Core | 5 | 900 |
| Models | 4 | 600 |
| Data | 2 | 400 |
| Pipelines | 5 | 400 |
| Utils | 5 | 800 |
| Tests | 5 | 600 |
| **Total** | **38** | **4,500** |

### Combined

| Metric | Value |
|--------|-------|
| Total new files | ~62 |
| Total new lines | ~8,400 |
| Prompts eliminated | ~2,200 lines |
| Core logic preserved | ~4,000 lines |
