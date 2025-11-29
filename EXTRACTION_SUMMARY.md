# Metaface Core Extraction - Executive Summary

## Overview

Extract 7 core deepfake commands from metaface into **2 standalone packages** deployed via **Bob**.

| Package | Commands | Purpose |
|---------|----------|---------|
| **deepface-core** | train, merge | Model training and face merging |
| **face-processing-toolkit** | match-pose, remask, mask-train, face-part-mask, export-dfm | Face processing and masking |

**Repository**: Stash  
**Deployment**: Bob artefacts  
**Timeline**: 6 weeks (3 developers)

---

## Two-Phase Approach

### Phase 1: Core Packages (Weeks 1-3)
- Standalone packages working with **any file paths**
- CLI interfaces via Click
- Python APIs for scripting
- Type-safe configuration via Pydantic
- Comprehensive test coverage
- Deployed via Bob

### Phase 2: Ivy Integration (Weeks 4-6)
- Spider adapters for querying datasets/models
- PipePublish adapters for publishing outputs
- Dependency tracking and model lineage
- **100% backwards compatible** with Phase 1

---

## Package Summary

### deepface-core

**Commands**:
- `dfc train` - Train SAEHD face swap models
- `dfc merge` - Merge trained model onto plates

**Structure**:
```
deepface-core/
├── cli/         # Click CLIs
├── config/      # Pydantic configs
├── core/        # Trainer, Merger, GridGenerator
├── pipelines/   # High-level APIs
├── utils/       # DFL wrapper, latent, warping, etc.
└── ivy/         # Phase 2: Spider/PipePublish
```

**Dependencies**: torch, metaswap, DFLObjects, pydantic, click

### face-processing-toolkit

**Commands**:
- `fpt match-pose` - Match source faces to destination poses
- `fpt remask` - Apply XSeg masks to aligned faces
- `fpt mask-train` - Train XSeg mask models
- `fpt face-part-mask` - Generate face part masks
- `fpt export-dfm` - Export model to DFM format

**Structure**:
```
face-processing-toolkit/
├── cli/         # Click CLIs (5 commands)
├── config/      # Pydantic configs
├── core/        # PoseMatcher, Remasker, etc.
├── models/      # FaceDetector, LandmarkDetector, MaskModel
├── data/        # Dataset, augmentations
├── pipelines/   # High-level APIs
├── utils/       # Alignment, masks, polygons
└── ivy/         # Phase 2: Spider/PipePublish
```

**Dependencies**: torch, segmentation-models-pytorch, DFLObjects, pydantic, click

---

## Usage Examples

### CLI

```bash
# Train
dfc train --model my_model --src /path/to/src --dst /path/to/dst --output /path/to/output

# Merge
dfc merge --model /path/to/model --plates /path/to/plates --aligned /path/to/aligned --output /path/to/output

# Face tools
fpt match-pose --src /path/to/src --dst /path/to/dst --output /path/to/output
fpt remask --input /path/to/aligned --checkpoint /path/to/mask.pt
```

### Python API

```python
from deepface_core import TrainingConfig, TrainingPipeline
from pathlib import Path

config = TrainingConfig(
    model_name="my_model",
    src_dataset=Path("/data/src"),
    dst_dataset=Path("/data/dst"),
    output_dir=Path("/output"),
    resolution=128,
    batch_size=8,
)

pipeline = TrainingPipeline(config)
pipeline.train()
```

### Phase 2: With Ivy

```bash
dfc train --src-stem "SHOW/shots/shot_010" --dst-stem "SHOW/assets/person_b" --publish --kind "saehd"
```

---

## Implementation Timeline

| Week | Developer 1 | Developer 2 | Developer 3 |
|------|-------------|-------------|-------------|
| **1** | TrainingConfig, Trainer, `dfc train` | MergingConfig, Merger, `dfc merge` | Package setup, alignment, match-pose |
| **2** | Checkpoints, batch, resume | Latent, warping, grid, all modes | Remask, mask-train, FP-mask, export-dfm |
| **3** | Docs, examples, tests | Docs, examples, tests | Docs, examples, tests |
| **4** | Spider: datasets/models | Spider: plates/aligned | Spider: faces/masks |
| **5** | Publish models, deps | Publish merged, versions | Publish masks/XSeg/DFMs |
| **6** | Integration tests | Integration tests | Release |

---

## Key Improvements

| Aspect | Before | After |
|--------|--------|-------|
| **Interface** | Interactive prompts | CLI + Python API |
| **Configuration** | Prompts only | Pydantic + YAML |
| **Type Safety** | None | 100% |
| **Paths** | Hardcoded | Any paths |
| **Testing** | Difficult | Easy |
| **Publishing** | Manual | PipePublish to Ivy |
| **Lineage** | None | Automatic via Ivy |

---

## Metrics

| Metric | Value |
|--------|-------|
| Commands extracted | 7 |
| Interactive prompts eliminated | ~2,200 lines |
| Core logic preserved | ~4,000 lines |
| New package files | ~62 |
| Target test coverage | 80%+ |

---

## Success Criteria

### Phase 1
- [ ] Both packages deployed via Bob
- [ ] All 7 commands functional
- [ ] Python APIs available
- [ ] Zero interactive prompts
- [ ] Type-safe Pydantic configuration
- [ ] 80%+ test coverage
- [ ] Complete documentation

### Phase 2
- [ ] Spider queries working
- [ ] PipePublish working
- [ ] Dependency tracking
- [ ] Backwards compatible

---

## Questions for Stakeholders

1. **Stash Repository**: Confirm location and naming
2. **Bob Platform**: Confirm `platform-pipe2024.1` target
3. **metaswap Package**: Confirm availability via Bob
4. **DFLObjects Package**: Confirm availability via Bob
5. **Checkpoints Storage**: Location for pretrained weights
6. **Ivy TwigType Codes**: Codes for models, aligned faces, outputs
