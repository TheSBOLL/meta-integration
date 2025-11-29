# 🏗️ Architecture Diagrams

<div align="center">

![Packages](https://img.shields.io/badge/Packages-2-green?style=for-the-badge)
![Commands](https://img.shields.io/badge/Commands-7-blue?style=for-the-badge)
![Deployment](https://img.shields.io/badge/Deployment-Bob-orange?style=for-the-badge)

**Visual architecture documentation for the extraction project**

[Extraction Plan](./EXTRACTION_PLAN.md) • [Quick Reference](./QUICK_REFERENCE.md) • [File Map](./FILE_EXTRACTION_MAP.md) • [Summary](./EXTRACTION_SUMMARY.md)

</div>

---

## 📋 Table of Contents

- [System Overview](#-system-overview)
- [Package Architecture](#-package-architecture)
- [Data Flow Diagrams](#-data-flow-diagrams)
- [Team Parallelization](#-team-parallelization)
- [Phase 2: Ivy Integration](#-phase-2-ivy-integration)

---

## 🎯 System Overview

### High-Level Architecture

```mermaid
flowchart TB
    subgraph Users["👥 Users"]
        CLI[CLI Interface]
        API[Python API]
        YAML[YAML Config]
    end
    
    subgraph Packages["📦 Packages"]
        subgraph DFC["deepface-core"]
            Train[dfc train]
            Merge[dfc merge]
        end
        
        subgraph FPT["face-processing-toolkit"]
            MP[fpt match-pose]
            RM[fpt remask]
            MT[fpt mask-train]
            FP[fpt face-part-mask]
            EX[fpt export-dfm]
        end
    end
    
    subgraph External["🔌 External Dependencies"]
        DFL[DeepFaceLab]
        MS[metaswap]
        DFLO[DFLObjects]
        SMP[segmentation-models-pytorch]
    end
    
    subgraph Storage["💾 Storage"]
        Models[(Models)]
        Aligned[(Aligned Faces)]
        Plates[(Plates)]
        Output[(Output)]
    end
    
    CLI --> Packages
    API --> Packages
    YAML --> Packages
    
    DFC --> DFL
    DFC --> MS
    DFC --> DFLO
    FPT --> DFLO
    FPT --> SMP
    
    Packages --> Storage
    
    style DFC fill:#e8f5e9
    style FPT fill:#e3f2fd
    style External fill:#fff3e0
    style Storage fill:#fce4ec
```

### Package Comparison

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'pie1': '#4CAF50', 'pie2': '#2196F3', 'pie3': '#FF9800', 'pie4': '#9C27B0', 'pie5': '#F44336', 'pie6': '#00BCD4'}}}%%
pie showData
    title Package Composition
    "deepface-core CLI" : 300
    "deepface-core Config" : 500
    "deepface-core Core" : 800
    "face-toolkit CLI" : 400
    "face-toolkit Core" : 900
    "face-toolkit Models" : 600
```

---

## 📦 Package Architecture

### deepface-core Structure

```mermaid
flowchart TB
    subgraph CLI["📂 cli/"]
        MainCLI[main.py<br/>Entry point]
        TrainCLI[train.py<br/>Training CLI]
        MergeCLI[merge.py<br/>Merge CLI]
    end
    
    subgraph Config["📂 config/"]
        BaseConfig[base.py<br/>Base classes]
        TrainConfig[training.py<br/>TrainingConfig]
        MergeConfig[merging.py<br/>MergingConfig]
    end
    
    subgraph Core["📂 core/"]
        Trainer[trainer.py<br/>Trainer class]
        Merger[merger.py<br/>Merger class]
        Grid[grid_generator.py<br/>Grid generation]
    end
    
    subgraph Models["📂 models/"]
        SAEHD[saehd.py<br/>SAEHD wrapper]
        XSeg[xseg.py<br/>XSeg wrapper]
    end
    
    subgraph Pipelines["📂 pipelines/"]
        TrainPipe[training.py<br/>TrainingPipeline]
        MergePipe[merging.py<br/>MergingPipeline]
    end
    
    subgraph Utils["📂 utils/"]
        DFLWrap[dfl_wrapper.py]
        Latent[latent.py]
        Warp[warping.py]
        Video[video.py]
    end
    
    MainCLI --> TrainCLI
    MainCLI --> MergeCLI
    
    TrainCLI --> TrainConfig
    MergeCLI --> MergeConfig
    
    TrainConfig --> TrainPipe
    MergeConfig --> MergePipe
    
    TrainPipe --> Trainer
    MergePipe --> Merger
    MergePipe --> Grid
    
    Trainer --> SAEHD
    Merger --> XSeg
    
    Trainer --> DFLWrap
    Merger --> Latent
    Merger --> Warp
    Grid --> Video
    
    style CLI fill:#e3f2fd
    style Config fill:#fff3e0
    style Core fill:#e8f5e9
    style Models fill:#fce4ec
    style Pipelines fill:#f3e5f5
    style Utils fill:#e0f2f1
```

---

### face-processing-toolkit Structure

```mermaid
flowchart TB
    subgraph CLI["📂 cli/"]
        MainCLI[main.py]
        MPCLI[match_pose.py]
        RMCLI[remask.py]
        MTCLI[mask_train.py]
        FPCLI[face_part_mask.py]
        EXCLI[export_dfm.py]
    end
    
    subgraph Config["📂 config/"]
        BaseConfig[base.py]
        PMConfig[pose_matching.py]
        RMConfig[remasking.py]
        MTConfig[mask_training.py]
        FPConfig[face_part_masking.py]
        EXConfig[export.py]
    end
    
    subgraph Core["📂 core/"]
        PM[pose_matcher.py]
        RM[remasker.py]
        MT[mask_trainer.py]
        FP[face_part_masker.py]
        EX[dfm_exporter.py]
    end
    
    subgraph ModelsLayer["📂 models/"]
        FD[face_detector.py]
        LD[landmark_detector.py]
        MM[mask_model.py]
        XS[xseg_model.py]
    end
    
    subgraph Data["📂 data/"]
        DS[face_dataset.py]
        Aug[augmentations.py]
    end
    
    subgraph Utils["📂 utils/"]
        Align[alignment.py]
        Masks[masks.py]
        Poly[polygons.py]
        Math[math.py]
    end
    
    MainCLI --> MPCLI & RMCLI & MTCLI & FPCLI & EXCLI
    
    MPCLI --> PMConfig --> PM
    RMCLI --> RMConfig --> RM
    MTCLI --> MTConfig --> MT
    FPCLI --> FPConfig --> FP
    EXCLI --> EXConfig --> EX
    
    PM --> LD
    RM --> XS
    MT --> DS & Aug
    FP --> MM
    
    PM --> Align
    RM --> Masks
    FP --> Masks
    
    style CLI fill:#e3f2fd
    style Config fill:#fff3e0
    style Core fill:#e8f5e9
    style ModelsLayer fill:#fce4ec
    style Data fill:#f3e5f5
    style Utils fill:#e0f2f1
```

---

## 🔄 Data Flow Diagrams

### Training Pipeline

```mermaid
sequenceDiagram
    participant U as User
    participant CLI as dfc train
    participant C as TrainingConfig
    participant P as TrainingPipeline
    participant T as Trainer
    participant DFL as DeepFaceLab
    participant FS as File System
    
    U->>CLI: dfc train --model X --src /path
    CLI->>C: Parse & validate config
    C->>C: Validate paths exist
    C->>C: Validate resolution
    C->>P: Create pipeline
    
    P->>P: Initialize trainer
    P->>T: Start training
    
    loop Training Loop
        T->>FS: Load batch (src + dst)
        T->>DFL: Execute training step
        DFL-->>T: Updated model
        T->>FS: Save checkpoint
    end
    
    T-->>P: Training complete
    P->>FS: Save final model
    P-->>CLI: Success
    CLI-->>U: Model saved to /path
```

### Merging Pipeline

```mermaid
sequenceDiagram
    participant U as User
    participant CLI as dfc merge
    participant C as MergingConfig
    participant P as MergingPipeline
    participant M as Merger
    participant G as GridGenerator
    participant FS as File System
    
    U->>CLI: dfc merge --model /path
    CLI->>C: Parse & validate config
    C->>P: Create pipeline
    
    P->>FS: Load model
    P->>M: Initialize merger
    
    loop For each plate
        M->>FS: Load plate + aligned
        M->>M: Get transform matrix
        M->>M: Apply model prediction
        M->>M: Warp to plate
        M->>FS: Save merged output
    end
    
    alt Generate Grid
        P->>G: Create grid
        G->>FS: Save grid images
    end
    
    alt Generate MP4
        P->>FS: Create video
    end
    
    P-->>CLI: Success
    CLI-->>U: Output saved
```

### Face Processing Pipelines

```mermaid
flowchart LR
    subgraph Input
        A[Aligned Faces]
        P[Plates]
        M[Model]
    end
    
    subgraph MatchPose["fpt match-pose"]
        MP1[Load landmarks]
        MP2[Compare poses]
        MP3[Find matches]
    end
    
    subgraph Remask["fpt remask"]
        RM1[Load checkpoint]
        RM2[Generate masks]
        RM3[Apply to faces]
    end
    
    subgraph MaskTrain["fpt mask-train"]
        MT1[Load dataset]
        MT2[Augment data]
        MT3[Train model]
    end
    
    subgraph FPMask["fpt face-part-mask"]
        FP1[Detect landmarks]
        FP2[Generate part masks]
        FP3[Save masks]
    end
    
    subgraph Export["fpt export-dfm"]
        EX1[Load model]
        EX2[Convert format]
        EX3[Save DFM]
    end
    
    subgraph Output
        O1[Matched Pairs]
        O2[Masked Faces]
        O3[Trained Model]
        O4[Part Masks]
        O5[DFM File]
    end
    
    A --> MP1 --> MP2 --> MP3 --> O1
    A --> RM1 --> RM2 --> RM3 --> O2
    A --> MT1 --> MT2 --> MT3 --> O3
    A --> FP1 --> FP2 --> FP3 --> O4
    M --> EX1 --> EX2 --> EX3 --> O5
    
    style Input fill:#e3f2fd
    style Output fill:#e8f5e9
```

---

## 👥 Team Parallelization

### 6-Week Timeline

```mermaid
gantt
    title Extraction Timeline
    dateFormat  YYYY-MM-DD
    
    section Phase 1
    Week 1 - Foundation           :p1w1, 2024-01-01, 7d
    Week 2 - Core Features        :p1w2, after p1w1, 7d
    Week 3 - Polish               :p1w3, after p1w2, 7d
    
    section Phase 2
    Week 4 - Spider               :p2w1, after p1w3, 7d
    Week 5 - Publish              :p2w2, after p2w1, 7d
    Week 6 - Release              :p2w3, after p2w2, 7d
```

### Developer Assignments

```mermaid
flowchart TB
    subgraph Phase1["Phase 1: Core Packages"]
        subgraph W1["Week 1"]
            D1W1[Dev 1: Package setup<br/>TrainingConfig<br/>Trainer + CLI]
            D2W1[Dev 2: MergingConfig<br/>Merger<br/>rawPredMerge]
            D3W1[Dev 3: face-toolkit setup<br/>alignment utils<br/>match-pose]
        end
        
        subgraph W2["Week 2"]
            D1W2[Dev 1: Checkpoints<br/>Batch training<br/>Resume feature]
            D2W2[Dev 2: Latent utils<br/>Warping<br/>Grid generator]
            D3W2[Dev 3: Remask<br/>Mask train<br/>FP mask<br/>Export DFM]
        end
        
        subgraph W3["Week 3"]
            D1W3[Dev 1: Documentation<br/>Examples<br/>Tests]
            D2W3[Dev 2: Documentation<br/>Benchmarks<br/>Tests]
            D3W3[Dev 3: Documentation<br/>All CLIs<br/>Tests]
        end
    end
    
    subgraph Phase2["Phase 2: Ivy Integration"]
        subgraph W4["Week 4"]
            D1W4[Dev 1: Spider<br/>Query datasets<br/>Query models]
            D2W4[Dev 2: Spider<br/>Query plates<br/>Query aligned]
            D3W4[Dev 3: Spider<br/>Query faces<br/>Query masks]
        end
        
        subgraph W5["Week 5"]
            D1W5[Dev 1: Publish models<br/>Track deps<br/>Model lineage]
            D2W5[Dev 2: Publish merged<br/>Versions<br/>Dependencies]
            D3W5[Dev 3: Publish masks<br/>Publish XSeg<br/>Publish DFMs]
        end
        
        subgraph W6["Week 6"]
            D1W6[Dev 1: Integration tests]
            D2W6[Dev 2: Documentation]
            D3W6[Dev 3: Release]
        end
    end
    
    W1 --> W2 --> W3 --> W4 --> W5 --> W6
    
    style Phase1 fill:#e8f5e9
    style Phase2 fill:#e3f2fd
```

### Deliverables

```mermaid
flowchart LR
    subgraph P1["Phase 1 Deliverables"]
        P1D1[✅ deepface-core package]
        P1D2[✅ face-processing-toolkit]
        P1D3[✅ 7 CLI commands]
        P1D4[✅ Python APIs]
        P1D5[✅ 80% test coverage]
        P1D6[✅ Documentation]
        P1D7[✅ Bob deployment]
    end
    
    subgraph P2["Phase 2 Deliverables"]
        P2D1[✅ Spider adapters]
        P2D2[✅ Publish adapters]
        P2D3[✅ Dependency tracking]
        P2D4[✅ Model lineage]
        P2D5[✅ Backwards compatible]
        P2D6[✅ Integration tests]
    end
    
    P1 --> P2
    
    style P1 fill:#e8f5e9
    style P2 fill:#e3f2fd
```

---

## 🔗 Phase 2: Ivy Integration

### Integration Architecture

```mermaid
flowchart TB
    subgraph Phase1["Phase 1: Local Paths"]
        LP[Local Paths]
        CLI1["dfc train --src /path"]
        API1["TrainingConfig(src=Path(...))"]
    end
    
    subgraph Phase2["Phase 2: + Ivy"]
        subgraph IvyDB["Ivy Database"]
            Spider[(Spider<br/>Query)]
            Publish[(PipePublish<br/>Publish)]
            Deps[(Dependencies<br/>Lineage)]
        end
        
        subgraph Adapters["Adapters"]
            SA[SpiderAdapter]
            PA[PublishAdapter]
        end
        
        CLI2["dfc train --src-stem SHOW/shots/010"]
        API2["ivy.query_aligned(...)"]
    end
    
    LP --> CLI1 --> API1
    
    Spider --> SA --> CLI2
    Spider --> SA --> API2
    PA --> Publish
    PA --> Deps
    
    CLI1 -.->|"Still works!"| Phase2
    API1 -.->|"Still works!"| Phase2
    
    style Phase1 fill:#e1f5fe
    style Phase2 fill:#e8f5e9
    style IvyDB fill:#fff3e0
```

### Backwards Compatibility

```mermaid
flowchart LR
    subgraph Before["Phase 1 Code"]
        B1["config = TrainingConfig(<br/>    src_dataset=Path('/local/src'),<br/>    dst_dataset=Path('/local/dst'),<br/>)"]
    end
    
    subgraph After["Phase 2 Code"]
        A1["# Option 1: Local paths (still works!)<br/>config = TrainingConfig(<br/>    src_dataset=Path('/local/src'),<br/>)"]
        
        A2["# Option 2: Ivy paths<br/>ivy = IvyAdapter(job='SHOW')<br/>config = TrainingConfig(<br/>    src_dataset=ivy.query(...),<br/>)"]
        
        A3["# Option 3: Mixed!<br/>config = TrainingConfig(<br/>    src_dataset=ivy.query(...),<br/>    dst_dataset=Path('/local/dst'),<br/>)"]
    end
    
    Before --> A1
    Before --> A2
    Before --> A3
    
    style Before fill:#e3f2fd
    style After fill:#e8f5e9
```

---

<div align="center">

**[📖 Extraction Plan](./EXTRACTION_PLAN.md)** • **[📖 Quick Reference](./QUICK_REFERENCE.md)** • **[📁 File Map](./FILE_EXTRACTION_MAP.md)** • **[📋 Summary](./EXTRACTION_SUMMARY.md)**

</div>
