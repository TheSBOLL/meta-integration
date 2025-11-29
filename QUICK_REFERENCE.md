# Quick Reference

## Packages

| Package | Commands | Purpose |
|---------|----------|---------|
| **deepface-core** | `dfc train`, `dfc merge` | Model training and face merging |
| **face-processing-toolkit** | `fpt match-pose`, `fpt remask`, `fpt mask-train`, `fpt face-part-mask`, `fpt export-dfm` | Face processing and masking |

---

## CLI Commands

### Train Model

```bash
# Basic
dfc train \
  --model my_model \
  --src /path/to/aligned_src \
  --dst /path/to/aligned_dst \
  --output /path/to/output

# Full options
dfc train \
  --model character_swap \
  --src /data/src_faces \
  --dst /data/dst_faces \
  --output /models/output \
  --resolution 256 \
  --batch-size 4 \
  --architecture LIAE_UDT \
  --learning-rate 6e-6 \
  --uniform-yaw \
  --random-warp \
  --checkpoint-interval 4

# From config file
dfc train --config /path/to/training_config.yaml

# Resume training
dfc train --model-path /path/to/existing_model --output /path/to/output
```

### Merge Faces

```bash
# Basic
dfc merge \
  --model /path/to/model \
  --plates /path/to/plates \
  --aligned /path/to/aligned \
  --output /path/to/output

# Full options
dfc merge \
  --model /models/my_model \
  --plates /shots/shot_010/plates \
  --aligned /shots/shot_010/aligned \
  --output /output/merged \
  --mode raw_rgb \
  --merge-on-alignments \
  --latent-shift /path/to/latent.npy \
  --enable-warp \
  --generate-grid \
  --generate-mp4
```

**Merge Modes**: `raw_rgb`, `raw_pred`, `raw_rgb+pred`, `seamless`

### Match Pose

```bash
fpt match-pose \
  --src /path/to/src_aligned \
  --dst /path/to/dst_aligned \
  --output /path/to/matched \
  --threshold 0.8 \
  --max-matches 10
```

### Remask

```bash
fpt remask \
  --input /path/to/aligned \
  --checkpoint /path/to/mask.pt \
  --features face,nose,mouth \
  --batch-size 16
```

**Features**: face, eyes, iris, eyebrow, nose, lip, mouth, ear, hair

### Mask Training

```bash
fpt mask-train \
  --data /path/to/training_data \
  --output /path/to/model_output \
  --epochs 100 \
  --batch-size 8 \
  --learning-rate 1e-4
```

### Face Part Mask

```bash
fpt face-part-mask \
  --input /path/to/aligned \
  --features face,eyes,mouth \
  --mask-type both \
  --plates /path/to/plates \
  --checkpoint /path/to/checkpoint.pt
```

**Mask Types**: `aligned`, `plate`, `both`

### Export DFM

```bash
fpt export-dfm \
  --model /path/to/model \
  --output /path/to/output.dfm \
  --latent-support
```

---

## Python API

### Training

```python
from deepface_core import TrainingConfig, TrainingPipeline
from pathlib import Path

# Basic
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

```python
# Advanced
from deepface_core.config import (
    TrainingConfig,
    ModelArchitecture,
    BorderMode,
    Optimizer,
)

config = TrainingConfig(
    model_name="character_swap",
    src_dataset=Path("/data/character_a"),
    dst_dataset=Path("/data/character_b"),
    output_dir=Path("/models/output"),
    
    model_architecture=ModelArchitecture.LIAE_UDT,
    resolution=256,
    encoder_dims=64,
    inter_dims=256,
    decoder_dims=64,
    
    batch_size=4,
    learning_rate=6e-6,
    optimizer=Optimizer.ADAM,
    
    loss_weight_dssim=10.0,
    loss_weight_lpips=0.5,
    
    uniform_yaw=True,
    random_warp=True,
    
    checkpoint_interval_hours=4,
)

pipeline = TrainingPipeline(config)
pipeline.train()
```

### Merging

```python
from deepface_core import MergingConfig, MergingPipeline
from deepface_core.config import MergeMode
from pathlib import Path

config = MergingConfig(
    model_path=Path("/models/my_model"),
    plates_dir=Path("/shots/shot_010/plates"),
    aligned_dir=Path("/shots/shot_010/aligned"),
    output_dir=Path("/output/merged"),
    
    merge_mode=MergeMode.RAW_RGB,
    merge_on_alignments=True,
    
    latent_shift_file=Path("/latent/shift.npy"),
    latent_shift_scale=1.0,
    
    enable_warp=True,
    generate_grid=True,
    generate_mp4=True,
)

pipeline = MergingPipeline(config)
pipeline.merge()
```

### Pose Matching

```python
from face_processing_toolkit import PoseMatchingConfig, PoseMatchingPipeline
from pathlib import Path

config = PoseMatchingConfig(
    src_dir=Path("/src_faces"),
    dst_dir=Path("/dst_faces"),
    output_dir=Path("/matched"),
    similarity_threshold=0.8,
    max_matches=10,
)

pipeline = PoseMatchingPipeline(config)
results = pipeline.match()
```

### Remasking

```python
from face_processing_toolkit import RemaskingConfig, RemaskingPipeline
from pathlib import Path

config = RemaskingConfig(
    input_dir=Path("/aligned"),
    checkpoint_path=Path("/checkpoints/mask.pt"),
    features=["face", "nose", "mouth"],
    batch_size=16,
)

pipeline = RemaskingPipeline(config)
pipeline.remask()
```

### Mask Training

```python
from face_processing_toolkit import MaskTrainingConfig, MaskTrainingPipeline
from pathlib import Path

config = MaskTrainingConfig(
    training_data_paths=[
        Path("/data/aligned_1/masks"),
        Path("/data/aligned_2/masks"),
    ],
    output_dir=Path("/models/xseg"),
    epochs=100,
    batch_size=8,
    learning_rate=1e-4,
)

pipeline = MaskTrainingPipeline(config)
pipeline.train()
```

---

## Configuration Files

### Training Config (YAML)

```yaml
model_name: character_swap
src_dataset: /data/src
dst_dataset: /data/dst
output_dir: /models/output

model_architecture: LIAE_UDT
resolution: 256
encoder_dims: 64
inter_dims: 256
decoder_dims: 64
mask_dims: 22

batch_size: 4
learning_rate: 0.000006
learning_rate_dropout: true

optimizer: adam
optimizer_gradient_clip: true

loss_weight_dssim: 10.0
loss_weight_lpips: 0.5

border_mode: replicate
uniform_yaw: true
random_warp: true

checkpoint_interval_hours: 4
checkpoint_count: 4
```

### Merging Config (YAML)

```yaml
model_path: /models/my_model
plates_dir: /shots/shot_010/plates
aligned_dir: /shots/shot_010/aligned
output_dir: /output/merged

merge_mode: raw_rgb
merge_on_alignments: true

latent_shift_file: /latent/shift.npy
latent_shift_scale: 1.0

enable_warp: true
n_matches_to_warp: 3

generate_grid: true
generate_mp4: true
```

---

## Phase 2: Ivy Integration

### CLI with Ivy

```bash
# Train with Ivy sources
dfc train \
  --src-stem "SHOW/shots/shot_010" \
  --dst-stem "SHOW/assets/person_b" \
  --publish \
  --kind "saehd"

# Merge with Ivy
dfc merge \
  --model-stem "SHOW/shots/shot_010" \
  --plates-stem "SHOW/shots/shot_010" \
  --publish \
  --kind "cgr"
```

### Python with Ivy

```python
from deepface_core import TrainingConfig, TrainingPipeline
from deepface_core.ivy import IvyAdapter, IvyPublisher

ivy = IvyAdapter(job="SHOWNAME")
src_paths = ivy.query_aligned_faces("shots/shot_010", "aligned_src")
dst_paths = ivy.query_aligned_faces("assets/person_b", "aligned_dst")

config = TrainingConfig(
    model_name="my_model",
    src_dataset=src_paths,
    dst_dataset=dst_paths,
    output_dir=Path("/local/output"),
)

pipeline = TrainingPipeline(config)
pipeline.train()

publisher = IvyPublisher(job="SHOWNAME")
publisher.publish_model(
    model_path=config.output_dir,
    scope_stem="shots/shot_010",
    kind="saehd",
    dependencies=[src_paths, dst_paths],
)
```

---

## Troubleshooting

### Dataset Not Found

```bash
# Verify paths exist
ls -la /path/to/dataset

# Check permissions
chmod -R 755 /path/to/dataset
```

### CUDA Out of Memory

```bash
# Reduce batch size
dfc train --batch-size 4

# Reduce resolution
dfc train --resolution 128
```

### GPU Batch Size Guide

| GPU | VRAM | Batch Size (256 res) |
|-----|------|---------------------|
| RTX 3090 | 24GB | 8-16 |
| RTX 4090 | 24GB | 8-16 |
| A100 | 40GB | 16-32 |

### Merge Artifacts

- Verify model trained sufficiently (check loss values)
- Check alignment quality
- Review merge mode settings
- Try different interpolators
