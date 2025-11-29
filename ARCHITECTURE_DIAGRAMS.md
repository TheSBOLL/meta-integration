# Architecture Diagrams

## System Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              NEW PACKAGES                                   │
│                                                                             │
│  ┌──────────────────────────────┐  ┌──────────────────────────────────┐     │
│  │       deepface-core          │  │   face-processing-toolkit        │     │
│  │                              │  │                                  │     │
│  │  CLI Layer (Click)           │  │  CLI Layer (Click)               │     │
│  │  ┌──────┐  ┌──────┐          │  │  ┌─────────┐  ┌──────┐  ┌──────┐ │     │
│  │  │train │  │merge │          │  │  │match-   │  │remask│  │mask- │ │     │
│  │  └──────┘  └──────┘          │  │  │pose    │  │      │  │train ││ │     │
│  │                              │  │  └─────────┘  └──────┘  └──────┘ │     │
│  │  Config Layer (Pydantic)     │  │                                  │     │
│  │  ┌──────────────────────┐    │  │  Config Layer (Pydantic)         │     │
│  │  │TrainingConfig        │    │  │  ┌────────────────────────────┐  │     │
│  │  │MergingConfig         │    │  │  │PoseMatchingConfig          │  │     │
│  │  └──────────────────────┘    │  │  │RemaskingConfig             │  │     │
│  │                              │  │  │MaskTrainingConfig          │  │     │
│  │  Core Logic                  │  │  └────────────────────────────┘  │     │
│  │  ┌──────────────────────┐    │  │                                  │     │
│  │  │Trainer, Merger       │    │  │  Core Logic                      │     │
│  │  │GridGenerator         │    │  │  ┌────────────────────────────┐  │     │
│  │  └──────────────────────┘    │  │  │PoseMatcher, Remasker       │  │     │
│  │                              │  │  │MaskTrainer, FacePartMasker │  │     │
│  │  Pipeline Layer              │  │  └────────────────────────────┘  │     │
│  │  ┌──────────────────────┐    │  │                                  │     │
│  │  │TrainingPipeline      │    │  │  Models                          │     │
│  │  │MergingPipeline       │    │  │  ┌────────────────────────────┐  │     │
│  │  └──────────────────────┘    │  │  │FaceDetector, LandmarkDet   │  │     │
│  │                              │  │  │MaskModel (DeepLabV3+)      │  │     │
│  │  Utils                       │  │  └────────────────────────────┘  │     │
│  │  ┌──────────────────────┐    │  │                                  │     │
│  │  │dfl_wrapper, latent   │    │  │  Utils                           │     │
│  │  │warping, grid, video  │    │  │  ┌────────────────────────────┐  │     │
│  │  └──────────────────────┘    │  │  │alignment, masks, polygons  │  │     │
│  │                              │  │  └────────────────────────────┘  │     │
│  │  [Phase 2] Ivy Integration   │  │                                  │     │
│  │  ┌──────────────────────┐    │  │  [Phase 2] Ivy Integration       │     │
│  │  │SpiderAdapter         │    │  │  ┌────────────────────────────┐  │     │
│  │  │PublishAdapter        │    │  │  │SpiderAdapter               │  │     │
│  │  └──────────────────────┘    │  │  │PublishAdapter              │  │     │
│  │                              │  │  └────────────────────────────┘  │     │
│  └──────────────────────────────┘  └──────────────────────────────────┘     │
│                                                                             │
│  Deployment: Bob artefacts                                                  │
│  Repository: Stash                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Training Pipeline

```
User Input
│
├─> dfc train --model X --src /path --dst /path
│   OR
├─> TrainingConfig(model_name="X", src_dataset=..., dst_dataset=...)
│
▼
┌─────────────────────────────────────────┐
│         TrainingConfig (Pydantic)       │
│                                         │
│ • model_name: str                       │
│ • src_dataset: Path                     │
│ • dst_dataset: Path                     │
│ • output_dir: Path                      │
│ • resolution: int (64-512)              │
│ • batch_size: int                       │
│ • model_architecture: enum              │
│ • loss weights, augmentation, etc.      │
│                                         │
│ Validation:                             │
│ ✓ Paths exist                           │
│ ✓ Resolution valid                      │
│ ✓ Types correct                         │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                    TrainingPipeline                             │
│                                                                 │
│  1. Validate Configuration                                      │
│     • Check paths exist                                         │
│     • Verify dataset contents                                   │
│     • Check GPU availability                                    │
│                                                                 │
│  2. Initialize Trainer                                          │
│     • Load or create model                                      │
│     • Setup optimizer                                           │
│     • Configure checkpoints                                     │
│                                                                 │
│  3. Execute Training                                            │
│     • DFL subprocess execution                                  │
│     • Progress callbacks                                        │
│     • Checkpoint saving                                         │
│                                                                 │
│  4. Post-Processing                                             │
│     • Save final checkpoint                                     │
│     • Generate previews                                         │
│     • Log metrics                                               │
└─────────────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│              OUTPUT                     │
│                                         │
│ output_dir/                             │
│ ├── model/                              │
│ │   ├── model.dat                       │
│ │   ├── model.json                      │
│ │   └── checkpoints/                    │
│ ├── previews/                           │
│ └── training_config.yaml                │
└─────────────────────────────────────────┘
```

---

## Merging Pipeline

```
User Input
│
├─> dfc merge --model /path --plates /path --aligned /path
│
▼
┌─────────────────────────────────────────┐
│         MergingConfig (Pydantic)        │
│                                         │
│ • model_path: Path                      │
│ • plates_dir: Path                      │
│ • aligned_dir: Path                     │
│ • output_dir: Path                      │
│ • merge_mode: enum                      │
│ • latent_shift_file: Optional[Path]     │
│ • enable_warp: bool                     │
│ • generate_grid: bool                   │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MergingPipeline                              │
│                                                                 │
│  1. Load Model                                                  │
│     • Read model config                                         │
│     • Verify model type                                         │
│                                                                 │
│  2. Prepare Merge                                               │
│     • Fix aligned filenames                                     │
│     • Load latent shift (optional)                              │
│     • Setup output directories                                  │
│                                                                 │
│  3. Execute Merge                                               │
│     • For each plate/aligned pair:                              │
│       - Load face data                                          │
│       - Get transform matrix                                    │
│       - Apply model prediction                                  │
│       - Warp result to plate                                    │
│       - Save output                                             │
│                                                                 │
│  4. Post-Processing                                             │
│     • Generate MP4 video                                        │
│     • Create merge summary                                      │
│     • Run warp (if enabled)                                     │
│     • Generate grid (if enabled)                                │
└─────────────────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│              OUTPUT                     │
│                                         │
│ output_dir/                             │
│ ├── rawrgb/                             │
│ │   └── frame_*.jpg                     │
│ ├── rawpred/                            │
│ │   └── frame_*.jpg                     │
│ ├── output.mp4                          │
│ ├── output_summary.json                 │
│ └── output_grid.mp4                     │
└─────────────────────────────────────────┘
```

---

## Face Processing Pipelines

```
┌───────────────────┐  ┌───────────────────┐  ┌───────────────────┐
│   Match Pose      │  │     Remask        │  │   Mask Train      │
└─────────┬─────────┘  └─────────┬─────────┘  └─────────┬─────────┘
          │                      │                      │
          ▼                      ▼                      ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│PoseMatchConfig  │    │RemaskingConfig  │    │MaskTrainConfig  │
│• src_dir        │    │• input_dir      │    │• data_paths     │
│• dst_dir        │    │• checkpoint     │    │• output_dir     │
│• threshold      │    │• features       │    │• epochs         │
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         ▼                      ▼                      ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ PoseMatcher     │    │   Remasker      │    │  MaskTrainer    │
│                 │    │                 │    │                 │
│ • detect()      │    │ • get_mask()    │    │ • train()       │
│ • compare()     │    │ • apply()       │    │ • validate()    │
│ • match()       │    │ • save()        │    │ • save()        │
└────────┬────────┘    └────────┬────────┘    └────────┬────────┘
         │                      │                      │
         ▼                      ▼                      ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ OUTPUT:         │    │ OUTPUT:         │    │ OUTPUT:         │
│ matched/        │    │ (in-place)      │    │ model/          │
│ └── pairs.json  │    │ XSeg mask in    │    │ └── mask.pt     │
│ └── matched_*   │    │ aligned files   │    │ └── config.yaml │
└─────────────────┘    └─────────────────┘    └─────────────────┘


┌───────────────────┐  ┌───────────────────┐
│  Face Part Mask   │  │   Export DFM      │
└─────────┬─────────┘  └─────────┬─────────┘
          │                      │
          ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│FPMaskConfig     │    │ExportDFMConfig  │
│• input_folder   │    │• model_path     │
│• features       │    │• output_path    │
│• mask_type      │    │• latent_support │
└────────┬────────┘    └────────┬────────┘
         │                      │
         ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│FacePartMasker   │    │  DFMExporter    │
│                 │    │                 │
│ Features:       │    │ • export()      │
│ • face          │    │                 │
│ • eyes          │    │                 │
│ • nose          │    │                 │
│ • mouth         │    │                 │
│ • hair          │    │                 │
└────────┬────────┘    └────────┬────────┘
         │                      │
         ▼                      ▼
┌─────────────────┐    ┌─────────────────┐
│ OUTPUT:         │    │ OUTPUT:         │
│ masks/          │    │ model.dfm       │
│ ├── face/       │    │                 │
│ ├── eyes/       │    │                 │
│ └── ...         │    │                 │
└─────────────────┘    └─────────────────┘
```

---

## Package Structure

```
deepface-core/                      face-processing-toolkit/
├── src/deepface_core/              ├── src/face_processing_toolkit/
│   ├── cli/                        │   ├── cli/
│   │   ├── main.py                 │   │   ├── main.py
│   │   ├── train.py                │   │   ├── match_pose.py
│   │   └── merge.py                │   │   ├── remask.py
│   │                               │   │   ├── mask_train.py
│   ├── config/                     │   │   ├── face_part_mask.py
│   │   ├── base.py                 │   │   └── export_dfm.py
│   │   ├── training.py             │   │
│   │   └── merging.py              │   ├── config/
│   │                               │   │   ├── base.py
│   ├── core/                       │   │   ├── pose_matching.py
│   │   ├── trainer.py              │   │   ├── remasking.py
│   │   ├── merger.py               │   │   ├── mask_training.py
│   │   └── grid_generator.py       │   │   ├── face_part_masking.py
│   │                               │   │   └── export.py
│   ├── models/                     │   │
│   │   ├── saehd.py                │   ├── core/
│   │   └── xseg.py                 │   │   ├── pose_matcher.py
│   │                               │   │   ├── remasker.py
│   ├── pipelines/                  │   │   ├── mask_trainer.py
│   │   ├── training.py             │   │   ├── face_part_masker.py
│   │   └── merging.py              │   │   └── dfm_exporter.py
│   │                               │   │
│   ├── utils/                      │   ├── models/
│   │   ├── dfl_wrapper.py          │   │   ├── face_detector.py
│   │   ├── dfl_image.py            │   │   ├── landmark_detector.py
│   │   ├── checkpoints.py          │   │   ├── mask_model.py
│   │   ├── latent.py               │   │   └── xseg_model.py
│   │   ├── warping.py              │   │
│   │   ├── grid.py                 │   ├── data/
│   │   └── video.py                │   │   ├── face_dataset.py
│   │                               │   │   └── augmentations.py
│   └── ivy/          [Phase 2]     │   │
│       ├── spider_adapter.py       │   ├── pipelines/
│       └── publish_adapter.py      │   │   ├── pose_matching.py
│                                   │   │   ├── remasking.py
├── tests/                          │   │   ├── mask_training.py
├── examples/                       │   │   ├── face_part_masking.py
├── docs/                           │   │   └── export.py
├── pyproject.toml                  │   │
├── bob.yaml                        │   ├── utils/
└── README.md                       │   │   ├── alignment.py
                                    │   │   ├── masks.py
                                    │   │   ├── polygons.py
                                    │   │   ├── math.py
                                    │   │   └── image_io.py
                                    │   │
                                    │   └── ivy/      [Phase 2]
                                    │       ├── spider_adapter.py
                                    │       └── publish_adapter.py
                                    │
                                    ├── checkpoints/
                                    ├── tests/
                                    ├── examples/
                                    ├── docs/
                                    ├── pyproject.toml
                                    ├── bob.yaml
                                    └── README.md
```

---

## Team Parallelization

```
                   Week 1              Week 2              Week 3
                   ──────              ──────              ──────

Developer 1    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
(Training)     │ Package setup │   │ Checkpoints   │   │ Documentation │
               │ TrainingConfig│──►│ Batch training│──►│ Examples      │
               │ Trainer class │   │ Resume feature│   │ Tests         │
               │ CLI: train    │   │ Progress      │   │               │
               └───────────────┘   └───────────────┘   └───────────────┘

Developer 2    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
(Merging)      │ MergingConfig │   │ Latent utils  │   │ Documentation │
               │ Merger class  │──►│ Warping utils │──►│ Examples      │
               │ CLI: merge    │   │ Grid generator│   │ Tests         │
               │ rawPredMerge  │   │ All modes     │   │               │
               └───────────────┘   └───────────────┘   └───────────────┘

Developer 3    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
(Face Tools)   │ Package setup │   │ Remask        │   │ Documentation │
               │ Alignment     │──►│ Mask train    │──►│ Examples      │
               │ Match pose    │   │ FP mask       │   │ Tests         │
               │               │   │ Export DFM    │   │               │
               └───────────────┘   └───────────────┘   └───────────────┘

                              │
                              ▼
               ┌─────────────────────────────────────┐
               │  Phase 1 Complete: Week 3           │
               │  • Both packages deployed via Bob   │
               │  • 7 commands working               │
               │  • Works with any file paths        │
               └─────────────────────────────────────┘
                              │
                              ▼

                   Week 4              Week 5              Week 6
                   ──────              ──────              ──────

Developer 1    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
(Spider)       │ Spider adapter│   │ Publish models│   │ Integration   │
               │ Query datasets│──►│ Track deps    │──►│ tests & docs  │
               │ Query models  │   │ Model lineage │   │ Release       │
               └───────────────┘   └───────────────┘   └───────────────┘

Developer 2    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
(Publish)      │ Spider adapter│   │ Publish merged│   │ Integration   │
               │ Query plates  │──►│ Track deps    │──►│ tests & docs  │
               │ Query aligned │   │ Versions      │   │ Release       │
               └───────────────┘   └───────────────┘   └───────────────┘

Developer 3    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
(Ivy Tools)    │ Spider adapter│   │ Publish masks │   │ Integration   │
               │ Query faces   │──►│ Publish XSeg  │──►│ tests & docs  │
               │ Query masks   │   │ Publish DFMs  │   │ Release       │
               └───────────────┘   └───────────────┘   └───────────────┘

                              │
                              ▼
               ┌─────────────────────────────────────┐
               │  Phase 2 Complete: Week 6           │
               │  • Ivy integration                  │
               │  • Spider queries                   │
               │  • PipePublish                      │
               │  • Backwards compatible             │
               └─────────────────────────────────────┘
```

---

## Phase 2: Ivy Integration

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        IVY INTEGRATION                                      │
│                                                                             │
│  ┌─────────────────────┐      ┌─────────────────────┐                       │
│  │   SpiderAdapter     │      │   PublishAdapter    │                       │
│  │                     │      │                     │                       │
│  │ • query_datasets()  │      │ • publish_model()   │                       │
│  │ • query_models()    │      │ • publish_output()  │                       │
│  │ • query_aligned()   │      │ • track_deps()      │                       │
│  │ • query_masks()     │      │ • create_version()  │                       │
│  └─────────┬───────────┘      └──────────┬──────────┘                       │
│            │                             │                                  │
│            ▼                             ▼                                  │
│  ┌─────────────────────────────────────────────────────────────┐            │
│  │                     IVY DATABASE                            │            │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐             │            │
│  │  │   Spider   │  │PipePublish │  │Dependencies│             │            │
│  │  │  (Query)   │  │ (Publish)  │  │ (Lineage)  │             │            │
│  │  └────────────┘  └────────────┘  └────────────┘             │            │
│  └─────────────────────────────────────────────────────────────┘            │
│                                                                             │
│  Backwards Compatible:                                                      │
│  • Phase 1 code continues to work with local paths                          │
│  • Ivy is optional - only used when --stem flags provided                   │
└─────────────────────────────────────────────────────────────────────────────┘
```
