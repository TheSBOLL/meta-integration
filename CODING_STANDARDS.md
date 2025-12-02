# Python Coding Standards

Modern Python development principles for our packages.

---

## Priority Order

Always build in this order:

```
API  →  CLI  →  Shell Scripts
```

1. **API first** — Core Python functions/classes that can be imported
2. **CLI second** — Thin wrapper calling the API
3. **Shell scripts last** — Only if needed, calling the CLI

If something can't be done via API, the design is wrong.

---

## Repository Structure

```
package-name/
├── src/
│   └── package_name/
│       ├── __init__.py
│       ├── core/                 # Pure logic, no I/O
│       │   ├── __init__.py
│       │   └── processor.py
│       ├── io/                   # All file/disk operations
│       │   ├── __init__.py
│       │   ├── readers.py
│       │   └── writers.py
│       ├── models/               # Data models, ML model wrappers
│       │   ├── __init__.py
│       │   └── predictor.py
│       ├── utils/                # Shared utilities
│       │   ├── __init__.py
│       │   └── helpers.py
│       └── cli/                  # Command line interface
│           ├── __init__.py
│           └── commands.py
├── tests/
│   ├── __init__.py
│   ├── test_core.py
│   └── test_io.py
├── pyproject.toml
├── README.md
└── bob.yaml
```

### Key Principles

- **`src/` layout** — Package lives inside `src/` folder
- **`core/`** — Pure logic, no I/O, no side effects, easily testable
- **`io/`** — All disk operations isolated here
- **No hardcoded paths** — Paths always passed as arguments
- **No interactive prompts** — Config passed in, not asked for

---

## Naming Conventions

### Repositories

```
# Good
face-processing-toolkit
deepface-core
mask-trainer

# Bad
FaceProcessingToolkit    # No PascalCase
face_processing_toolkit  # No underscores
fpt                      # No abbreviations
```

### Packages (inside src/)

```
# Repository: face-processing-toolkit
# Package: face_processing_toolkit (underscores for Python imports)

from face_processing_toolkit.core import process_face
```

### Files

```
# Good
pose_matcher.py
face_aligner.py
landmark_detector.py

# Bad
PoseMatcher.py     # No PascalCase files
posematcher.py     # Use underscores
pm.py              # No abbreviations
```

### Classes

```python
# Good
class PoseMatcher:
class FaceAligner:
class LandmarkDetector:

# Bad
class poseMatcher:      # PascalCase required
class Pose_Matcher:     # No underscores
class PM:               # No abbreviations
```

### Functions and Methods

```python
# Good
def align_face(image: np.ndarray) -> np.ndarray:
def calculate_similarity(a: np.ndarray, b: np.ndarray) -> float:

# Bad
def alignFace():        # No camelCase
def AlignFace():        # No PascalCase
def align():            # Too vague
```

### Constants

```python
# Good
DEFAULT_BATCH_SIZE = 8
MAX_FACE_COUNT = 10
MODEL_INPUT_SIZE = 512

# Bad
defaultBatchSize = 8    # Use UPPER_SNAKE_CASE
```

---

## Core vs I/O Separation

**The Rule:** Methods that generate data should NOT write to disk.

### Bad — Mixed Logic and I/O

```python
def process_faces(input_dir: str, output_dir: str) -> None:
    images = load_images(input_dir)          # I/O
    results = []
    for img in images:
        result = detect_face(img)            # Logic
        result = align_face(result)          # Logic
        results.append(result)
    save_images(results, output_dir)         # I/O
```

### Good — Separated

```python
# core/processor.py — Pure logic
def process_face(image: np.ndarray) -> np.ndarray:
    """Process a single face. No I/O."""
    detected = detect_face(image)
    aligned = align_face(detected)
    return aligned


# io/batch.py — I/O operations
def process_directory(input_dir: Path, output_dir: Path) -> None:
    """Load, process, save. I/O wrapper."""
    for image_path in input_dir.glob("*.png"):
        image = read_image(image_path)
        result = process_face(image)         # Calls core
        write_image(result, output_dir / image_path.name)
```

### Benefits

- **Testable** — Core functions can be unit tested without disk
- **Reusable** — Core can be called from API, CLI, or other code
- **Flexible** — I/O layer can be swapped (disk, S3, database)

---

## Type Hints

**Required on all functions.** Use `mypy` for validation.

### Basic Types

```python
from pathlib import Path
import numpy as np

def load_image(path: Path) -> np.ndarray:
    ...

def resize_image(image: np.ndarray, size: tuple[int, int]) -> np.ndarray:
    ...

def calculate_score(values: list[float]) -> float:
    ...
```

### Optional and Union

```python
from typing import Optional

def process(image: np.ndarray, mask: Optional[np.ndarray] = None) -> np.ndarray:
    ...

# Python 3.10+ — use | instead of Union
def get_path(value: str | Path) -> Path:
    ...
```

### Complex Types

```python
from typing import TypeAlias
import numpy as np

Image: TypeAlias = np.ndarray
Landmarks: TypeAlias = np.ndarray  # Shape: (68, 2)

def detect_landmarks(image: Image) -> Landmarks:
    ...
```

### Pydantic for Configs

```python
from pydantic import BaseModel, Field
from pathlib import Path

class TrainingConfig(BaseModel):
    model_dir: Path
    batch_size: int = Field(default=8, ge=1, le=64)
    learning_rate: float = Field(default=1e-4, gt=0)
    iterations: int = Field(default=100000, ge=1)

    class Config:
        frozen = True  # Immutable after creation
```

---

## Function Design

### Single Responsibility

```python
# Bad — Does too much
def process_and_save_and_upload(image, path, bucket):
    ...

# Good — One thing each
def process(image: np.ndarray) -> np.ndarray:
    ...

def save(image: np.ndarray, path: Path) -> None:
    ...

def upload(path: Path, bucket: str) -> str:
    ...
```

### Return Values

```python
# Bad — Returns None, caller doesn't know if it worked
def process(image):
    try:
        # do stuff
        pass
    except:
        pass

# Good — Returns result or raises
def process(image: np.ndarray) -> np.ndarray:
    if image is None:
        raise ValueError("Image cannot be None")
    result = do_processing(image)
    return result
```

### Docstrings

```python
def align_face(
    image: np.ndarray,
    landmarks: np.ndarray,
    output_size: int = 512,
) -> np.ndarray:
    """Align a face using landmarks.

    Args:
        image: Input image as HWC numpy array.
        landmarks: Facial landmarks, shape (68, 2).
        output_size: Output image size in pixels.

    Returns:
        Aligned face image.

    Raises:
        ValueError: If landmarks shape is incorrect.
    """
    ...
```

---

## CLI Design (Click)

CLI should be a thin wrapper over the API.

```python
# cli/commands.py
import click
from pathlib import Path
from ..core.processor import process_face
from ..io.readers import read_image
from ..io.writers import write_image


@click.command()
@click.option("--input", "-i", type=click.Path(exists=True), required=True)
@click.option("--output", "-o", type=click.Path(), required=True)
@click.option("--size", "-s", type=int, default=512)
def align(input: str, output: str, size: int) -> None:
    """Align a face image."""
    image = read_image(Path(input))
    result = process_face(image, output_size=size)
    write_image(result, Path(output))
    click.echo(f"Saved to {output}")
```

### CLI Principles

- **No logic in CLI** — Just parse args and call API
- **Use Click** — Standard, well-documented
- **Consistent naming** — `--input`, `--output`, `--config`
- **Help text on all options**

---

## Error Handling

```python
# Define custom exceptions
class ProcessingError(Exception):
    """Base exception for processing errors."""
    pass

class FaceNotFoundError(ProcessingError):
    """No face detected in image."""
    pass

class InvalidConfigError(ProcessingError):
    """Configuration validation failed."""
    pass


# Use them
def detect_face(image: np.ndarray) -> np.ndarray:
    faces = detector.detect(image)
    if len(faces) == 0:
        raise FaceNotFoundError("No face detected in image")
    return faces[0]
```

---

## Testing

```python
# tests/test_core.py
import numpy as np
import pytest
from face_processing_toolkit.core.processor import process_face


def test_process_face_returns_correct_shape():
    image = np.zeros((256, 256, 3), dtype=np.uint8)
    result = process_face(image, output_size=512)
    assert result.shape == (512, 512, 3)


def test_process_face_raises_on_none():
    with pytest.raises(ValueError):
        process_face(None)
```

### Test Principles

- **Test core functions** — They have no I/O, easy to test
- **Use pytest** — Standard, simple
- **One assertion per test** — Clear failure messages
- **Test edge cases** — None, empty, wrong types

---

## Configuration Files

### pyproject.toml

```toml
[project]
name = "face-processing-toolkit"
version = "1.0.0"
description = "Face processing tools"
requires-python = ">=3.10"
dependencies = [
    "numpy>=1.24.0",
    "opencv-python>=4.8.0",
    "click>=8.0.0",
    "pydantic>=2.0.0",
]

[project.scripts]
fpt = "face_processing_toolkit.cli:main"

[tool.mypy]
python_version = "3.10"
strict = true

[tool.pytest.ini_options]
testpaths = ["tests"]

[tool.ruff]
line-length = 100
select = ["E", "F", "I", "N", "W"]
```

> **Note:** For DNEG deployment, you also need a `setup.py` — see DNEG Specific section below.

---

## DNEG Specific

For internal DNEG deployment, packages go through **Bob** (our build/deployment system).

### Key Points

- Bob uses **`setup.py`** (not `pyproject.toml`) for package detection
- You need both: `pyproject.toml` for modern tooling, `setup.py` for Bob compatibility
- Packages are defined in **builders** (Python functions that describe how to build)
- Deployed to **targets** (e.g., `platform-pipe2024.1`)

### Minimal setup.py for Bob

```python
from setuptools import setup, find_packages

setup(
    name="face-processing-toolkit",
    version="1.0.0",
    packages=find_packages(where="src"),
    package_dir={"": "src"},
    install_requires=[
        "numpy",
        "opencv-python",
        "click",
        "pydantic",
    ],
    entry_points={
        "console_scripts": [
            "fpt=face_processing_toolkit.cli:main",
        ],
    },
)
```

### Builder (brief example)

Builders live in `bobdeployment` repo and tell Bob how to build your package:

```python
@registrar.register()
def face_processing_toolkit(params, ctx):
    do_dneg_retrieve(ctx, 'face_processing_toolkit', params)
    
    build_deps = [
        P('python'),
        P('python_numpy'),
        P('python_click'),
        P('python_pydantic'),
    ]
    
    runtime_deps = [
        P('python'),
        P('python_numpy'),
        P('python_click'),
        P('python_pydantic'),
    ]
    
    setup_world(ctx, *build_deps)
    do_python_build(ctx, params)
    set_runtime_info(ctx, 'face_processing_toolkit', runtime_deps)
```

### Deployment

```bash
# Build for a target
build-deployment pipeline pipeline-launchers platform-pipe2024-1 --wait-for-deployment -v

# Enter a world to test (using build output path)
bob-world -d /builds/targets/gen-2025-11-29T18:39:22.094543 -t platform-pipe2024-1
```

For full Bob documentation, see internal DNEG resources.

---

## Summary Checklist

Before committing, verify:

- [ ] All functions have type hints
- [ ] Core logic has no I/O
- [ ] I/O is isolated in `io/` module
- [ ] CLI only calls API, no logic
- [ ] No hardcoded paths
- [ ] No interactive prompts
- [ ] Custom exceptions for errors
- [ ] Docstrings on public functions
- [ ] Tests for core functions
- [ ] `pyproject.toml` is complete

---

## Quick Reference

| What | Convention | Example |
|:-----|:-----------|:--------|
| Repo names | `kebab-case` | `face-processing-toolkit` |
| Package names | `snake_case` | `face_processing_toolkit` |
| File names | `snake_case.py` | `pose_matcher.py` |
| Class names | `PascalCase` | `PoseMatcher` |
| Function names | `snake_case` | `align_face()` |
| Constants | `UPPER_SNAKE` | `DEFAULT_SIZE` |
| Private | `_prefix` | `_internal_method()` |

---

*Add to this document as patterns emerge.*
