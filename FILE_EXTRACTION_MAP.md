# 📁 File Extraction Map

<div align="center">

![Files](https://img.shields.io/badge/Source%20Files-~50-blue?style=for-the-badge)
![Extract](https://img.shields.io/badge/Extract-~4000%20lines-green?style=for-the-badge)
![Delete](https://img.shields.io/badge/Delete-~2200%20lines-red?style=for-the-badge)

**Detailed mapping from metaface source files to new package locations**

[Extraction Plan](./EXTRACTION_PLAN.md) • [Architecture](./ARCHITECTURE_DIAGRAMS.md) • [Quick Reference](./QUICK_REFERENCE.md) • [Summary](./EXTRACTION_SUMMARY.md)

</div>

---

## 📋 Table of Contents

- [Legend](#-legend)
- [deepface-core](#-deepface-core)
- [face-processing-toolkit](#-face-processing-toolkit)
- [Shared Utilities](#-shared-utilities)
- [Files to Delete](#-files-to-delete)
- [Priority Order](#-priority-order)
- [Summary](#-summary)

---

## 🏷️ Legend

| Badge | Action | Description |
|:-----:|:-------|:------------|
| 🟢 | **EXTRACT** | Core logic to extract and modernize |
| 🔴 | **DELETE** | Interactive prompts/telemetry - do not extract |
| 🔵 | **NEW** | New file to create |
| 🟣 | **MERGE** | Combine multiple source files |

---

## 📦 deepface-core

### CLI Layer

```mermaid
flowchart LR
    subgraph Source
        A[train_handler.py<br/>33 lines]
        B[merge_in_batch_handler.py<br/>69 lines]
    end
    
    subgraph Target["cli/"]
        C[main.py]
        D[train.py]
        E[merge.py]
    end
    
    A -->|🟢 EXTRACT| D
    B -->|🟢 EXTRACT| E
    N[NEW] -->|🔵 NEW| C
    
    style Source fill:#ffeeee
    style Target fill:#eeffee
```

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `cli/main.py` | - | 🔵 NEW | ~50 | Click group entry point |
| `cli/train.py` | `train_handler.py` | 🟢 EXTRACT | 33 | Convert to Click command |
| `cli/merge.py` | `merge_in_batch_handler.py` | 🟢 EXTRACT | 69 | Convert to Click command |

---

### Config Layer

```mermaid
flowchart LR
    subgraph Source
        A[train_prompt.py<br/>455 lines<br/>🔴 DELETE prompts]
        B[merge_in_batch_prompt.py<br/>788 lines<br/>🔴 DELETE prompts]
    end
    
    subgraph Target["config/"]
        C[base.py]
        D[training.py]
        E[merging.py]
    end
    
    A -.->|Extract params only| D
    B -.->|Extract params only| E
    N[NEW] -->|🔵 NEW| C
    
    style Source fill:#ffeeee
    style Target fill:#eeffee
```

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `config/base.py` | - | 🔵 NEW | ~100 | Base config classes, enums |
| `config/training.py` | `train_prompt.py` | 🟢 EXTRACT | 455 | Extract parameters only, delete prompts |
| `config/merging.py` | `merge_in_batch_prompt.py` | 🟢 EXTRACT | 788 | Extract parameters only, delete prompts |

---

### Core Layer

```mermaid
flowchart LR
    subgraph Source
        A[train_runner.py<br/>62 lines]
        B[dfl.py<br/>159 lines]
        C[merge_runner.py<br/>469 lines]
        D[rawPredMerge.py<br/>223 lines]
        E[grid_runner.py<br/>~800 lines]
        F[grid_utils.py<br/>180 lines]
    end
    
    subgraph Target["core/"]
        G[trainer.py]
        H[merger.py]
        I[grid_generator.py]
    end
    
    A -->|🟣 MERGE| G
    B -->|🟣 MERGE| G
    C -->|🟣 MERGE| H
    D -->|🟣 MERGE| H
    E -->|🟣 MERGE| I
    F -->|🟣 MERGE| I
    
    style Source fill:#ffeeee
    style Target fill:#eeffee
```

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `core/trainer.py` | `train_runner.py`, `dfl.py` | 🟣 MERGE | 221 | DFL command execution |
| `core/merger.py` | `merge_runner.py`, `rawPredMerge.py` | 🟣 MERGE | 692 | All merge logic |
| `core/grid_generator.py` | `grid_runner.py`, `grid_utils.py` | 🟣 MERGE | ~980 | Grid generation |

---

### Models Layer

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `models/saehd.py` | - | 🔵 NEW | ~100 | SAEHD model wrapper |
| `models/xseg.py` | - | 🔵 NEW | ~100 | XSeg model wrapper |

---

### Pipelines Layer

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `pipelines/training.py` | - | 🔵 NEW | ~150 | High-level training API |
| `pipelines/merging.py` | - | 🔵 NEW | ~150 | High-level merging API |

---

### Utils Layer

```mermaid
flowchart LR
    subgraph Source
        A[dfl.py]
        B[run_cmd.py]
        C[dfl_image_utils.py]
        D[latent.py]
        E[latent_utils.py]
        F[warp_utils.py]
        G[warp.py]
        H[ffmpeg.py]
        I[generate_mp4.py]
        J[nuke_parser.py]
    end
    
    subgraph Target["utils/"]
        K[dfl_wrapper.py]
        L[dfl_image.py]
        M[checkpoints.py]
        N[latent.py]
        O[warping.py]
        P[grid.py]
        Q[video.py]
        R[nuke_export.py]
    end
    
    A --> K
    B --> K
    C --> L
    D --> N
    E --> N
    F --> O
    G --> O
    H --> Q
    I --> Q
    J --> R
    
    style Source fill:#ffeeee
    style Target fill:#eeffee
```

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `utils/dfl_wrapper.py` | `dfl.py`, `run_cmd.py` | 🟣 MERGE | ~200 | DFL execution wrapper |
| `utils/dfl_image.py` | `dfl_image_utils.py` | 🟢 EXTRACT | 120 | DFL image utilities |
| `utils/checkpoints.py` | - | 🔵 NEW | ~100 | Checkpoint management |
| `utils/latent.py` | `latent.py`, `latent_utils.py` | 🟣 MERGE | ~200 | Latent manipulation |
| `utils/warping.py` | `warp_utils.py`, `warp.py` | 🟣 MERGE | ~250 | Warp calculations |
| `utils/grid.py` | `grid_utils.py` | 🟢 EXTRACT | 180 | Grid utilities |
| `utils/video.py` | `ffmpeg.py`, `generate_mp4.py` | 🟣 MERGE | ~250 | Video generation |
| `utils/nuke_export.py` | `nuke_parser.py` | 🟢 EXTRACT | 150 | Nuke tracker export |

---

### Ivy Layer (Phase 2)

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `ivy/spider_adapter.py` | - | 🔵 NEW | ~150 | Spider query adapter |
| `ivy/publish_adapter.py` | - | 🔵 NEW | ~150 | PipePublish adapter |

---

## 🎭 face-processing-toolkit

### CLI Layer

```mermaid
flowchart LR
    subgraph Source
        A[match_pose_handler.py<br/>85 lines]
        B[remask_handler.py<br/>32 lines]
        C[mask_train_handler.py<br/>45 lines]
        D[face_part_mask_handler.py<br/>60 lines]
        E[export_DFM_handler.py<br/>40 lines]
    end
    
    subgraph Target["cli/"]
        F[main.py]
        G[match_pose.py]
        H[remask.py]
        I[mask_train.py]
        J[face_part_mask.py]
        K[export_dfm.py]
    end
    
    A -->|🟢| G
    B -->|🟢| H
    C -->|🟢| I
    D -->|🟢| J
    E -->|🟢| K
    N[NEW] -->|🔵| F
    
    style Source fill:#ffeeee
    style Target fill:#eeeeff
```

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `cli/main.py` | - | 🔵 NEW | ~50 | Click group entry point |
| `cli/match_pose.py` | `match_pose_handler.py` | 🟢 EXTRACT | 85 | Convert to Click |
| `cli/remask.py` | `remask_handler.py` | 🟢 EXTRACT | 32 | Convert to Click |
| `cli/mask_train.py` | `mask_train_handler.py` | 🟢 EXTRACT | 45 | Convert to Click |
| `cli/face_part_mask.py` | `face_part_mask_handler.py` | 🟢 EXTRACT | 60 | Convert to Click |
| `cli/export_dfm.py` | `export_DFM_handler.py` | 🟢 EXTRACT | 40 | Convert to Click |

---

### Config Layer

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `config/base.py` | - | 🔵 NEW | ~50 | Base config classes |
| `config/pose_matching.py` | `match_pose_prompt.py` | 🟢 EXTRACT | ~80 | Extract parameters |
| `config/remasking.py` | `remask_prompt.py` | 🟢 EXTRACT | ~60 | Extract parameters |
| `config/mask_training.py` | `mask_train_prompt.py` | 🟢 EXTRACT | ~100 | Extract parameters |
| `config/face_part_masking.py` | `face_part_mask_prompt.py` | 🟢 EXTRACT | ~80 | Extract parameters |
| `config/export.py` | `export_DFM_prompt.py` | 🟢 EXTRACT | ~50 | Extract parameters |

---

### Core Layer

```mermaid
flowchart LR
    subgraph Source
        A[match_pose.py<br/>135 lines]
        B[match_pose_handler.py<br/>85 lines]
        C[remask.py<br/>78 lines]
        D[mask_train.py<br/>200 lines]
        E[fp_mask.py<br/>335 lines]
        F[export_DFM_runner.py<br/>33 lines]
    end
    
    subgraph Target["core/"]
        G[pose_matcher.py]
        H[remasker.py]
        I[mask_trainer.py]
        J[face_part_masker.py]
        K[dfm_exporter.py]
    end
    
    A --> G
    B --> G
    C --> H
    D --> I
    E --> J
    F --> K
    
    style Source fill:#ffeeee
    style Target fill:#eeeeff
```

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `core/pose_matcher.py` | `match_pose.py`, `match_pose_handler.py` | 🟣 MERGE | 220 | Pose matching logic |
| `core/remasker.py` | `remask.py` | 🟢 EXTRACT | 78 | XSeg remask logic |
| `core/mask_trainer.py` | `mask_train.py` | 🟢 EXTRACT | 200 | Mask training logic |
| `core/face_part_masker.py` | `fp_mask.py` | 🟢 EXTRACT | 335 | Face part masking |
| `core/dfm_exporter.py` | `export_DFM_runner.py` | 🟢 EXTRACT | 33 | DFM export |

---

### Models Layer

```mermaid
flowchart LR
    subgraph Source
        A[face_detect.py]
        B[LandmarksProcessor.py<br/>1,128 lines]
        C[face_mask.py<br/>41 lines]
    end
    
    subgraph Target["models/"]
        D[face_detector.py]
        E[landmark_detector.py]
        F[mask_model.py]
        G[xseg_model.py]
    end
    
    A --> D
    B --> E
    C --> F
    N[NEW] --> G
    
    style Source fill:#ffeeee
    style Target fill:#eeeeff
```

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `models/face_detector.py` | `face_detect.py` | 🟢 EXTRACT | ~150 | Face detection |
| `models/landmark_detector.py` | `LandmarksProcessor.py` | 🟢 EXTRACT | 1,128 | Landmark detection |
| `models/mask_model.py` | `face_mask.py` | 🟢 EXTRACT | 41 | DeepLabV3+ masking |
| `models/xseg_model.py` | - | 🔵 NEW | ~100 | XSeg model wrapper |

---

### Data Layer

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `data/face_dataset.py` | `dataset.py` | 🟢 EXTRACT | 104 | PyTorch dataset |
| `data/augmentations.py` | `augmentations.py` | 🟢 EXTRACT | 267 | Training augmentations |

---

### Pipelines Layer

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `pipelines/pose_matching.py` | - | 🔵 NEW | ~80 | High-level API |
| `pipelines/remasking.py` | - | 🔵 NEW | ~80 | High-level API |
| `pipelines/mask_training.py` | - | 🔵 NEW | ~80 | High-level API |
| `pipelines/face_part_masking.py` | - | 🔵 NEW | ~80 | High-level API |
| `pipelines/export.py` | - | 🔵 NEW | ~80 | High-level API |

---

### Utils Layer

```mermaid
flowchart LR
    subgraph Source
        A[aligned_face.py<br/>790 lines]
        B[umeyama.py<br/>50 lines]
        C[mathlib.py<br/>100 lines]
        D[IEPolys.py<br/>150 lines]
        E[SegIEPolys.py<br/>180 lines]
        F[mask_utils.py]
        G[image_utils.py]
    end
    
    subgraph Target["utils/"]
        H[alignment.py]
        I[math.py]
        J[polygons.py]
        K[masks.py]
        L[image_io.py]
    end
    
    A --> H
    B --> I
    C --> I
    D --> J
    E --> J
    F --> K
    G --> L
    
    style Source fill:#ffeeee
    style Target fill:#eeeeff
```

| Target | Source | Action | Lines | Notes |
|:-------|:-------|:------:|------:|:------|
| `utils/alignment.py` | `aligned_face.py` | 🟢 EXTRACT | 790 | Face alignment math |
| `utils/masks.py` | `mask_utils.py` | 🟢 EXTRACT | ~100 | Mask utilities |
| `utils/polygons.py` | `IEPolys.py`, `SegIEPolys.py` | 🟣 MERGE | 330 | Polygon handling |
| `utils/math.py` | `umeyama.py`, `mathlib.py` | 🟣 MERGE | 150 | Math utilities |
| `utils/image_io.py` | `image_utils.py` | 🟢 EXTRACT | ~150 | Image I/O |

---

## 🔄 Shared Utilities

> Files used by both packages - extract to each package or create shared library

| Source | Lines | Used By | Target |
|:-------|------:|:--------|:-------|
| `image_utils.py` | ~150 | Both | `utils/image_io.py` in each |
| `folder_utils.py` | ~100 | Both | `utils/paths.py` in each |
| `batched.py` | ~50 | Both | `utils/batched.py` in each |

---

## 🗑️ Files to Delete

> [!CAUTION]
> These files contain interactive prompts, telemetry, or workflow-specific logic and should NOT be extracted

### Interactive Prompts

<details>
<summary>🔴 <b>All *_prompt.py files (~2,200 lines total)</b></summary>

| File | Lines | Reason |
|:-----|------:|:-------|
| `train_prompt.py` | 455 | InquirerPy prompts |
| `merge_in_batch_prompt.py` | 788 | InquirerPy prompts |
| `match_pose_prompt.py` | ~100 | InquirerPy prompts |
| `remask_prompt.py` | ~80 | InquirerPy prompts |
| `mask_train_prompt.py` | ~120 | InquirerPy prompts |
| `face_part_mask_prompt.py` | ~90 | InquirerPy prompts |
| `export_DFM_prompt.py` | ~60 | InquirerPy prompts |
| `extract_prompt.py` | ~200 | InquirerPy prompts |
| All other `*_prompt.py` | ~300 | InquirerPy prompts |

</details>

### Telemetry & Workflow

<details>
<summary>🔴 <b>Telemetry and workflow-specific files</b></summary>

| File | Lines | Reason |
|:-----|------:|:-------|
| `tracking.py` | ~200 | Telemetry |
| `slack_metaface_app.py` | ~150 | Slack integration |
| `progress_handler.py` | ~100 | Workflow-specific |
| `metaflow_config_handler.py` | ~80 | Workflow-specific |
| `memory_usage_monitor.py` | ~60 | Telemetry |

</details>

### Menu Systems

<details>
<summary>🔴 <b>Menu routing files</b></summary>

| File | Lines | Reason |
|:-----|------:|:-------|
| `face_tools_handler.py` | 130 | Menu routing |
| `start_handler.py` | ~100 | Menu routing |
| `main.py` (handlers) | ~50 | Menu routing |

</details>

---

## 📊 Priority Order

### 🔥 High Priority (Week 1)

```mermaid
flowchart TB
    subgraph Week1["Week 1: Foundation"]
        direction TB
        A[1. Training Core]
        B[2. Merging Core]
        C[3. Face Tools Foundation]
    end
    
    A --> A1[train_runner.py → trainer.py]
    A --> A2[dfl.py → dfl_wrapper.py]
    A --> A3[TrainingConfig from prompts]
    
    B --> B1[merge_runner.py → merger.py]
    B --> B2[rawPredMerge.py → merger.py]
    B --> B3[MergingConfig from prompts]
    
    C --> C1[aligned_face.py → alignment.py]
    C --> C2[umeyama.py + mathlib.py → math.py]
    C --> C3[match_pose.py → pose_matcher.py]
```

### ⚡ Medium Priority (Week 2)

```mermaid
flowchart TB
    subgraph Week2["Week 2: Features"]
        direction TB
        A[4. Training Advanced]
        B[5. Merging Advanced]
        C[6. Face Tools Commands]
    end
    
    A --> A1[Checkpoint management]
    A --> A2[Progress callbacks]
    A --> A3[Batch processing]
    
    B --> B1[grid_runner.py → grid_generator.py]
    B --> B2[latent.py → latent.py]
    B --> B3[warp_utils.py → warping.py]
    
    C --> C1[remask.py → remasker.py]
    C --> C2[mask_train.py → mask_trainer.py]
    C --> C3[fp_mask.py → face_part_masker.py]
    C --> C4[export_DFM_runner.py → dfm_exporter.py]
```

### 📝 Lower Priority (Week 3)

```mermaid
flowchart TB
    subgraph Week3["Week 3: Polish"]
        direction TB
        A[7. Models]
        B[8. Documentation]
    end
    
    A --> A1[LandmarksProcessor.py → landmark_detector.py]
    A --> A2[face_mask.py → mask_model.py]
    A --> A3[Model wrappers]
    
    B --> B1[Integration tests]
    B --> B2[API documentation]
    B --> B3[Examples]
```

---

## 📈 Summary

### deepface-core

| Layer | Files | Estimated Lines |
|:------|------:|----------------:|
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
|:------|------:|----------------:|
| CLI | 6 | 400 |
| Config | 6 | 400 |
| Core | 5 | 900 |
| Models | 4 | 600 |
| Data | 2 | 400 |
| Pipelines | 5 | 400 |
| Utils | 5 | 800 |
| Tests | 5 | 600 |
| **Total** | **38** | **4,500** |

### Overall

| Metric | Value |
|:-------|------:|
| Total new files | ~62 |
| Total new lines | ~8,400 |
| Prompts eliminated | ~2,200 lines |
| Core logic preserved | ~4,000 lines |

---

<div align="center">

**[📖 Extraction Plan](./EXTRACTION_PLAN.md)** • **[🏗️ Architecture](./ARCHITECTURE_DIAGRAMS.md)** • **[📖 Quick Reference](./QUICK_REFERENCE.md)** • **[📋 Summary](./EXTRACTION_SUMMARY.md)**

</div>
