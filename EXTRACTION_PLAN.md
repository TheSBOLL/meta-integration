# Metaface Core Extraction Plan

## Overview

Extract 7 core deepfake commands from the metaface repository into **2 standalone packages** deployed via **Bob**.

| Package | Commands | Purpose |
|---------|----------|---------|
| **deepface-core** | train, merge | Model training and face merging |
| **face-processing-toolkit** | match-pose, remask, mask-train, face-part-mask, export-dfm | Face processing and masking tools |

**Repository**: Stash  
**Deployment**: Bob artefacts  
**Timeline**: 6 weeks (3 developers)

---

## Two-Phase Approach

### Phase 1: Core Packages (Weeks 1-3)
- Standalone packages that work with **any file paths**
- CLI interfaces via Click
- Python APIs for scripting
- Type-safe configuration via Pydantic
- Comprehensive test coverage

### Phase 2: Ivy Integration (Weeks 4-6)
- Spider adapters for querying datasets/models from Ivy
- PipePublish adapters for publishing outputs to Ivy
- Dependency tracking and model lineage
- **100% backwards compatible** with Phase 1

---

## Package 1: deepface-core

### Commands
- `dfc train` - Train SAEHD face swap models
- `dfc merge` - Merge trained model onto plates

### Directory Structure

```
deepface-core/
├── src/deepface_core/
│   ├── __init__.py
│   ├── cli/
│   │   ├── __init__.py
│   │   ├── main.py              # Entry point: dfc
│   │   ├── train.py             # dfc train
│   │   └── merge.py             # dfc merge
│   ├── config/
│   │   ├── __init__.py
│   │   ├── base.py              # Base config classes
│   │   ├── training.py          # TrainingConfig
│   │   └── merging.py           # MergingConfig
│   ├── core/
│   │   ├── __init__.py
│   │   ├── trainer.py           # Training logic
│   │   ├── merger.py            # Merge logic
│   │   └── grid_generator.py    # Grid generation
│   ├── models/
│   │   ├── __init__.py
│   │   ├── saehd.py             # SAEHD model wrapper
│   │   └── xseg.py              # XSeg model wrapper
│   ├── pipelines/
│   │   ├── __init__.py
│   │   ├── training.py          # TrainingPipeline
│   │   └── merging.py           # MergingPipeline
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── dfl_wrapper.py       # DFL command execution
│   │   ├── dfl_image.py         # DFL image utilities
│   │   ├── checkpoints.py       # Checkpoint management
│   │   ├── latent.py            # Latent manipulation
│   │   ├── warping.py           # Warp calculations
│   │   ├── grid.py              # Grid utilities
│   │   ├── video.py             # Video generation
│   │   └── nuke_export.py       # Nuke tracker export
│   └── ivy/                     # Phase 2
│       ├── __init__.py
│       ├── spider_adapter.py
│       └── publish_adapter.py
├── tests/
│   ├── __init__.py
│   ├── test_training.py
│   ├── test_merging.py
│   └── test_utils.py
├── examples/
│   ├── train_basic.py
│   ├── train_advanced.py
│   ├── merge_basic.py
│   └── merge_with_latent.py
├── docs/
│   ├── training.md
│   ├── merging.md
│   └── configuration.md
├── pyproject.toml
├── README.md
└── bob.yaml                     # Bob deployment config
```

### Dependencies

```toml
[project]
dependencies = [
    "torch>=2.0",
    "torchvision>=0.15",
    "numpy>=1.24",
    "opencv-python>=4.8",
    "pydantic>=2.0",
    "click>=8.0",
    "pyyaml>=6.0",
    "tqdm>=4.65",
    "Pillow>=10.0",
]

[project.optional-dependencies]
ivy = [
    "spider-client",
    "pipepublish",
]
```

**External Dependencies** (via Bob):
- `metaswap` - Training job configuration and execution
- `DFLObjects` - DFL image and face type handling

### Configuration: TrainingConfig

```python
from pydantic import BaseModel, Field, field_validator
from pathlib import Path
from enum import Enum
from typing import Optional

class ModelArchitecture(str, Enum):
    LIAE_UD = "LIAE_UD"
    LIAE_UDT = "LIAE_UDT"
    LIAE_UDT_2X = "LIAE_UDT_2X"
    SAEHD = "SAEHD"

class Optimizer(str, Enum):
    ADAM = "adam"
    ADAMW = "adamw"
    LAMB = "lamb"
    RANGER = "ranger"

class BorderMode(str, Enum):
    CONSTANT = "constant"
    REPLICATE = "replicate"
    REFLECT = "reflect"

class FreezeMode(str, Enum):
    NONE = "none"
    ENCODER = "encoder"
    INTER_AB = "inter_ab"
    DECODER = "decoder"

class TrainingConfig(BaseModel):
    """Configuration for SAEHD model training."""
    
    # Required
    model_name: str = Field(..., description="Name of the model")
    src_dataset: Path = Field(..., description="Path to source aligned faces")
    dst_dataset: Path = Field(..., description="Path to destination aligned faces")
    output_dir: Path = Field(..., description="Output directory for model")
    
    # Architecture
    model_architecture: ModelArchitecture = Field(
        default=ModelArchitecture.LIAE_UDT,
        description="Model architecture variant"
    )
    resolution: int = Field(default=128, ge=64, le=512, description="Training resolution")
    encoder_dims: int = Field(default=64, ge=16, le=256)
    inter_dims: int = Field(default=128, ge=32, le=1024)
    decoder_dims: int = Field(default=64, ge=16, le=256)
    mask_dims: int = Field(default=22, ge=8, le=64)
    
    # Training
    batch_size: int = Field(default=8, ge=1, le=64)
    num_epochs: int = Field(default=0, ge=0, description="0 = indefinite")
    learning_rate: float = Field(default=6e-6, gt=0)
    learning_rate_dropout: bool = Field(default=True)
    
    # Optimizer
    optimizer: Optimizer = Field(default=Optimizer.ADAM)
    optimizer_gradient_clip: bool = Field(default=True)
    optimizer_gradient_clip_value: float = Field(default=1.0)
    
    # Loss weights
    loss_weight_ssim: float = Field(default=0.0, ge=0)
    loss_weight_style: float = Field(default=0.0, ge=0)
    loss_weight_dssim: float = Field(default=10.0, ge=0)
    loss_weight_ms_ssim: float = Field(default=0.0, ge=0)
    loss_weight_mse: float = Field(default=0.0, ge=0)
    loss_weight_mae: float = Field(default=0.0, ge=0)
    loss_weight_idleak: float = Field(default=0.0, ge=0)
    loss_weight_lpips: float = Field(default=0.0, ge=0)
    loss_weight_src: float = Field(default=1.0, ge=0)
    loss_weight_dst: float = Field(default=1.0, ge=0)
    
    # Augmentation
    border_mode: BorderMode = Field(default=BorderMode.REPLICATE)
    uniform_yaw: bool = Field(default=False)
    random_blur: bool = Field(default=False)
    random_blur_chance: float = Field(default=0.5, ge=0, le=1)
    random_flip: bool = Field(default=True)
    random_downsample: bool = Field(default=False)
    random_downsample_chance: float = Field(default=0.5, ge=0, le=1)
    random_noise: bool = Field(default=False)
    random_noise_chance: float = Field(default=0.5, ge=0, le=1)
    random_warp: bool = Field(default=True)
    random_warp_samples: int = Field(default=3, ge=1, le=10)
    random_warp_scale: float = Field(default=0.05, ge=0, le=0.5)
    random_transform: bool = Field(default=True)
    random_transform_rotation: float = Field(default=5.0, ge=0, le=45)
    random_transform_scale: float = Field(default=0.05, ge=0, le=0.5)
    random_transform_translate: float = Field(default=0.05, ge=0, le=0.5)
    
    # Advanced
    masked_training: bool = Field(default=True)
    background_grayscaling: bool = Field(default=False)
    freeze_mode: FreezeMode = Field(default=FreezeMode.NONE)
    use_fp16: bool = Field(default=False)
    
    # Checkpoints
    output_interval_minutes: int = Field(default=15, ge=1)
    checkpoint_interval_hours: int = Field(default=4, ge=1)
    checkpoint_count: int = Field(default=4, ge=1)
    preview_images_count: int = Field(default=4, ge=1, le=16)
    
    # Resume training
    model_path: Optional[Path] = Field(default=None, description="Path to existing model to resume")
    
    @field_validator('src_dataset', 'dst_dataset')
    @classmethod
    def validate_dataset_exists(cls, v: Path) -> Path:
        if not v.exists():
            raise ValueError(f"Dataset path does not exist: {v}")
        return v
    
    @field_validator('resolution')
    @classmethod
    def validate_resolution(cls, v: int) -> int:
        valid = [64, 96, 128, 160, 192, 224, 256, 288, 320, 352, 384, 416, 448, 480, 512]
        if v not in valid:
            raise ValueError(f"Resolution must be one of {valid}")
        return v
```

### Configuration: MergingConfig

```python
from pydantic import BaseModel, Field, field_validator
from pathlib import Path
from enum import Enum
from typing import Optional

class MergeMode(str, Enum):
    RAW_RGB = "raw_rgb"
    RAW_PRED = "raw_pred"
    RAW_RGB_PRED = "raw_rgb+pred"
    SEAMLESS = "seamless"

class Interpolator(str, Enum):
    NEAREST = "nearest"
    LINEAR = "linear"
    CUBIC = "cubic"
    AREA = "area"
    LANCZOS4 = "lanczos4"

class MergingConfig(BaseModel):
    """Configuration for face merging."""
    
    # Required
    model_path: Path = Field(..., description="Path to trained model")
    plates_dir: Path = Field(..., description="Path to plate images")
    aligned_dir: Path = Field(..., description="Path to aligned faces")
    output_dir: Path = Field(..., description="Output directory")
    
    # Mode
    merge_mode: MergeMode = Field(default=MergeMode.RAW_RGB)
    merge_on_alignments: bool = Field(default=False)
    is_degradation_merge: bool = Field(default=False)
    
    # Latent
    latent_shift_file: Optional[Path] = Field(default=None)
    latent_shift_scale: float = Field(default=1.0, ge=0, le=2)
    
    # Warping
    enable_warp: bool = Field(default=False)
    n_matches_to_warp: int = Field(default=3, ge=1, le=10)
    dataset_src_to_warp: Optional[Path] = Field(default=None)
    
    # Grid generation
    generate_grid: bool = Field(default=False)
    generate_yaw_grid: bool = Field(default=False)
    generate_mask_grid: bool = Field(default=False)
    
    # Output
    generate_mp4: bool = Field(default=False)
    mp4_quality: str = Field(default="normal", pattern="^(low|normal|high)$")
    
    # Batch processing
    batch_size: int = Field(default=8, ge=1, le=64)
    
    # Advanced
    interpolator: Interpolator = Field(default=Interpolator.LANCZOS4)
    rescale_output: bool = Field(default=False)
    
    @field_validator('model_path', 'plates_dir', 'aligned_dir')
    @classmethod
    def validate_path_exists(cls, v: Path) -> Path:
        if not v.exists():
            raise ValueError(f"Path does not exist: {v}")
        return v
```

---

## Package 2: face-processing-toolkit

### Commands
- `fpt match-pose` - Match source faces to destination poses
- `fpt remask` - Apply XSeg masks to aligned faces
- `fpt mask-train` - Train XSeg mask models
- `fpt face-part-mask` - Generate face part masks (eyes, nose, mouth, etc.)
- `fpt export-dfm` - Export model to DFM format

### Directory Structure

```
face-processing-toolkit/
├── src/face_processing_toolkit/
│   ├── __init__.py
│   ├── cli/
│   │   ├── __init__.py
│   │   ├── main.py              # Entry point: fpt
│   │   ├── match_pose.py        # fpt match-pose
│   │   ├── remask.py            # fpt remask
│   │   ├── mask_train.py        # fpt mask-train
│   │   ├── face_part_mask.py    # fpt face-part-mask
│   │   └── export_dfm.py        # fpt export-dfm
│   ├── config/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── pose_matching.py     # PoseMatchingConfig
│   │   ├── remasking.py         # RemaskingConfig
│   │   ├── mask_training.py     # MaskTrainingConfig
│   │   ├── face_part_masking.py # FacePartMaskingConfig
│   │   └── export.py            # ExportDFMConfig
│   ├── core/
│   │   ├── __init__.py
│   │   ├── pose_matcher.py
│   │   ├── remasker.py
│   │   ├── mask_trainer.py
│   │   ├── face_part_masker.py
│   │   └── dfm_exporter.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── face_detector.py     # Face detection
│   │   ├── landmark_detector.py # Landmark detection
│   │   ├── mask_model.py        # DeepLabV3+ masking
│   │   └── xseg_model.py        # XSeg model
│   ├── data/
│   │   ├── __init__.py
│   │   ├── face_dataset.py      # PyTorch dataset
│   │   └── augmentations.py     # Training augmentations
│   ├── pipelines/
│   │   ├── __init__.py
│   │   ├── pose_matching.py
│   │   ├── remasking.py
│   │   ├── mask_training.py
│   │   ├── face_part_masking.py
│   │   └── export.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── alignment.py         # Face alignment math
│   │   ├── masks.py             # Mask utilities
│   │   ├── polygons.py          # Polygon handling
│   │   ├── math.py              # Math utilities
│   │   └── image_io.py          # Image loading/saving
│   └── ivy/                     # Phase 2
│       ├── __init__.py
│       ├── spider_adapter.py
│       └── publish_adapter.py
├── checkpoints/                 # Pretrained weights
│   └── .gitkeep
├── tests/
│   ├── __init__.py
│   ├── test_pose_matching.py
│   ├── test_remasking.py
│   ├── test_mask_training.py
│   └── test_face_part_masking.py
├── examples/
│   ├── match_pose_example.py
│   ├── remask_example.py
│   └── mask_train_example.py
├── docs/
│   ├── pose_matching.md
│   ├── remasking.md
│   ├── mask_training.md
│   └── face_part_masking.md
├── pyproject.toml
├── README.md
└── bob.yaml                     # Bob deployment config
```

### Dependencies

```toml
[project]
dependencies = [
    "torch>=2.0",
    "torchvision>=0.15",
    "numpy>=1.24",
    "opencv-python>=4.8",
    "pydantic>=2.0",
    "click>=8.0",
    "pyyaml>=6.0",
    "tqdm>=4.65",
    "Pillow>=10.0",
    "segmentation-models-pytorch>=0.3",
    "albumentations>=1.3",
    "natsort>=8.0",
]

[project.optional-dependencies]
ivy = [
    "spider-client",
    "pipepublish",
]
```

**External Dependencies** (via Bob):
- `DFLObjects` - DFL image and face type handling

### Configuration Classes

```python
# PoseMatchingConfig
class PoseMatchingConfig(BaseModel):
    src_dir: Path
    dst_dir: Path
    output_dir: Path
    similarity_threshold: float = Field(default=0.8, ge=0, le=1)
    max_matches: int = Field(default=10, ge=1)
    use_3d_landmarks: bool = Field(default=True)

# RemaskingConfig
class RemaskingConfig(BaseModel):
    input_dir: Path
    checkpoint_path: Path
    features: list[str] = Field(default=["face"])
    batch_size: int = Field(default=16, ge=1)

# MaskTrainingConfig
class MaskTrainingConfig(BaseModel):
    training_data_paths: list[Path]
    output_dir: Path
    epochs: int = Field(default=100, ge=1)
    batch_size: int = Field(default=8, ge=1)
    learning_rate: float = Field(default=1e-4, gt=0)
    resolution: tuple[int, int] = Field(default=(512, 512))

# FacePartMaskingConfig
class FacePartMaskingConfig(BaseModel):
    input_folder: Path
    features: list[str]
    mask_type: str = Field(default="both", pattern="^(aligned|plate|both)$")
    checkpoint_path: Path
    plates_folder: Optional[Path] = None

# ExportDFMConfig
class ExportDFMConfig(BaseModel):
    model_path: Path
    output_path: Path
    include_latent_support: bool = Field(default=True)
```

---

## Source File Mapping

### deepface-core Sources

| Source File | Lines | Action | Target |
|-------------|-------|--------|--------|
| train_handler.py | 33 | EXTRACT | core/trainer.py |
| train_prompt.py | 455 | DELETE | - |
| train_runner.py | 62 | EXTRACT | utils/dfl_wrapper.py |
| merge_in_batch_handler.py | 69 | EXTRACT | core/merger.py |
| merge_in_batch_prompt.py | 788 | DELETE | - |
| merge_runner.py | 469 | EXTRACT | core/merger.py |
| rawPredMerge.py | 223 | EXTRACT | core/merger.py |
| grid_runner.py | ~800 | EXTRACT | core/grid_generator.py |
| latent.py | 150 | EXTRACT | utils/latent.py |
| warp_utils.py | 200 | EXTRACT | utils/warping.py |
| grid_utils.py | 180 | EXTRACT | utils/grid.py |
| dfl.py | 159 | EXTRACT | utils/dfl_wrapper.py |
| dfl_image_utils.py | 120 | EXTRACT | utils/dfl_image.py |
| ffmpeg.py | 200 | EXTRACT | utils/video.py |
| nuke_parser.py | 150 | EXTRACT | utils/nuke_export.py |

### face-processing-toolkit Sources

| Source File | Lines | Action | Target |
|-------------|-------|--------|--------|
| match_pose_handler.py | 85 | EXTRACT | core/pose_matcher.py |
| match_pose.py | 135 | EXTRACT | core/pose_matcher.py |
| remask.py | 78 | EXTRACT | core/remasker.py |
| remask_handler.py | 32 | EXTRACT | core/remasker.py |
| mask_train.py | 200 | EXTRACT | core/mask_trainer.py |
| mask_train_handler.py | 45 | EXTRACT | core/mask_trainer.py |
| fp_mask.py | 335 | EXTRACT | core/face_part_masker.py |
| face_part_mask_handler.py | 60 | EXTRACT | core/face_part_masker.py |
| export_DFM_runner.py | 33 | EXTRACT | core/dfm_exporter.py |
| export_DFM_handler.py | 40 | EXTRACT | core/dfm_exporter.py |
| aligned_face.py | 790 | EXTRACT | utils/alignment.py |
| LandmarksProcessor.py | 1,128 | EXTRACT | models/landmark_detector.py |
| face_mask.py | 41 | EXTRACT | models/mask_model.py |
| umeyama.py | 50 | EXTRACT | utils/math.py |
| mathlib.py | 100 | EXTRACT | utils/math.py |
| IEPolys.py | 150 | EXTRACT | utils/polygons.py |
| SegIEPolys.py | 180 | EXTRACT | utils/polygons.py |
| augmentations.py | 267 | EXTRACT | data/augmentations.py |
| dataset.py | 104 | EXTRACT | data/face_dataset.py |

### Files to Delete (Not Extract)

| File | Reason |
|------|--------|
| *_prompt.py (all) | Interactive prompts |
| tracking.py | Telemetry |
| slack_metaface_app.py | Slack integration |
| progress_handler.py | Workflow-specific |
| metaflow_config_handler.py | Workflow-specific |

---

## Implementation Timeline

### Phase 1: Core Packages (Weeks 1-3)

#### Week 1: Foundation

**Developer 1 - Training**
- [ ] Create deepface-core package structure
- [ ] Implement TrainingConfig with validation
- [ ] Implement Trainer class (DFL wrapper)
- [ ] Create `dfc train` CLI
- [ ] Unit tests for training config

**Developer 2 - Merging**
- [ ] Implement MergingConfig with validation
- [ ] Implement Merger class
- [ ] Extract rawPredMerge logic
- [ ] Create `dfc merge` CLI
- [ ] Unit tests for merging config

**Developer 3 - Face Tools Setup**
- [ ] Create face-processing-toolkit package structure
- [ ] Extract alignment utilities (aligned_face.py → alignment.py)
- [ ] Extract math utilities (umeyama.py, mathlib.py → math.py)
- [ ] Implement PoseMatchingConfig
- [ ] Implement PoseMatcher class
- [ ] Create `fpt match-pose` CLI

#### Week 2: Core Features

**Developer 1 - Training Advanced**
- [ ] Checkpoint management
- [ ] Batch processing
- [ ] Resume training support
- [ ] Progress callbacks
- [ ] TrainingPipeline high-level API

**Developer 2 - Merging Advanced**
- [ ] Latent shift utilities
- [ ] Warping utilities
- [ ] Grid generator (all modes)
- [ ] All merge modes (raw_rgb, raw_pred, seamless)
- [ ] MergingPipeline high-level API

**Developer 3 - Face Tools Commands**
- [ ] Implement RemaskingConfig + Remasker
- [ ] Implement MaskTrainingConfig + MaskTrainer
- [ ] Implement FacePartMaskingConfig + FacePartMasker
- [ ] Implement ExportDFMConfig + DFMExporter
- [ ] Create all remaining CLIs

#### Week 3: Polish

**All Developers**
- [ ] Integration tests
- [ ] Documentation (README, API docs, examples)
- [ ] Performance benchmarking
- [ ] Type checking (mypy strict mode)
- [ ] Bob deployment configuration
- [ ] Code review and refinement

**Deliverable**: Both packages deployed via Bob, working with any file paths

### Phase 2: Ivy Integration (Weeks 4-6)

#### Week 4: Spider Adapters

**Developer 1**
- [ ] Spider adapter for querying datasets
- [ ] Spider adapter for querying models
- [ ] Integration with TrainingPipeline

**Developer 2**
- [ ] Spider adapter for querying plates
- [ ] Spider adapter for querying aligned faces
- [ ] Integration with MergingPipeline

**Developer 3**
- [ ] Spider adapter for querying masks
- [ ] Spider adapter for face tool inputs
- [ ] Integration with face tool pipelines

#### Week 5: Publish Adapters

**Developer 1**
- [ ] PipePublish adapter for models
- [ ] Dependency tracking for training
- [ ] Model lineage support

**Developer 2**
- [ ] PipePublish adapter for merged outputs
- [ ] Version management
- [ ] Dependency tracking for merging

**Developer 3**
- [ ] PipePublish adapter for masks
- [ ] PipePublish adapter for XSeg models
- [ ] PipePublish adapter for DFM exports

#### Week 6: Integration & Release

**All Developers**
- [ ] End-to-end integration tests
- [ ] Documentation updates for Ivy features
- [ ] Performance testing with Ivy
- [ ] Production deployment
- [ ] Release notes

**Deliverable**: Full Ivy integration, backwards compatible with Phase 1

---

## Bob Deployment

### bob.yaml (deepface-core)

```yaml
name: deepface-core
version: "1.0.0"

requires:
  - platform-pipe2024.1
  - python-3.10
  - torch-2.0
  - metaswap
  - DFLObjects

build:
  type: python
  entry_points:
    dfc: deepface_core.cli.main:cli

variants:
  - platform: [linux]
    arch: [x86_64]
```

### bob.yaml (face-processing-toolkit)

```yaml
name: face-processing-toolkit
version: "1.0.0"

requires:
  - platform-pipe2024.1
  - python-3.10
  - torch-2.0
  - DFLObjects
  - segmentation-models-pytorch

build:
  type: python
  entry_points:
    fpt: face_processing_toolkit.cli.main:cli

variants:
  - platform: [linux]
    arch: [x86_64]
```

---

## Success Criteria

### Phase 1
- [ ] Both packages deployed via Bob
- [ ] All 7 commands functional via CLI
- [ ] Python APIs available for all commands
- [ ] Zero interactive prompts
- [ ] Type-safe Pydantic configuration
- [ ] 80%+ test coverage
- [ ] Complete documentation

### Phase 2
- [ ] Spider queries working for all data types
- [ ] PipePublish working for all outputs
- [ ] Dependency tracking implemented
- [ ] Model lineage tracked
- [ ] Backwards compatible with Phase 1
- [ ] Integration tests passing

---

## Questions for Stakeholders

1. **Stash Repository**: Confirm repository location and naming convention
2. **Bob Platform**: Confirm `platform-pipe2024.1` as target
3. **metaswap Package**: Confirm availability via Bob
4. **DFLObjects Package**: Confirm availability via Bob
5. **Checkpoints Storage**: Location for pretrained model weights
6. **Ivy TwigType Codes**: Codes for models, aligned faces, outputs
