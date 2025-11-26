# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is openpilot?

openpilot is an operating system for robotics that upgrades the driver assistance system in 300+ supported cars. It provides Adaptive Cruise Control (ACC) and Automated Lane Centering (ALC).

## Build Commands

```bash
# Initial setup (installs dependencies, submodules, LFS files)
tools/op.sh setup

# Activate Python virtual environment
source .venv/bin/activate

# Build openpilot (from activated venv)
scons -j$(nproc)

# Build with options
scons --asan    # Address sanitizer
scons --ubsan   # Undefined behavior sanitizer
```

## Testing

```bash
# Run all tests
pytest

# Run a single test file
pytest path/to/test_file.py

# Run a specific test
pytest path/to/test_file.py::test_function_name

# Run tests excluding slow ones
pytest -m 'not slow'

# Run with verbose output
pytest -v

# Process replay tests (run separately, not collected by pytest)
python selfdrive/test/process_replay/test_processes.py
```

## Linting

```bash
# Run all linters (ruff, mypy, codespell, etc.)
scripts/lint/lint.sh

# Fast lint (skip mypy and codespell)
scripts/lint/lint.sh --fast

# Run specific linter
scripts/lint/lint.sh ruff
scripts/lint/lint.sh mypy
```

## Architecture Overview

### Core Directories

- **selfdrive/**: Main driving logic
  - `car/`: Car interfaces via opendbc (CAN communication, car-specific implementations)
  - `controls/`: Vehicle control (controlsd - steering, gas, brake)
  - `selfdrived/`: Main state machine coordinating all driving components
  - `modeld/`: Neural network models for driving and driver monitoring
  - `locationd/`: Localization, calibration, and vehicle parameter estimation
  - `monitoring/`: Driver monitoring system
  - `ui/`: User interface (raylib-based)
  - `pandad/`: Panda hardware interface daemon

- **system/**: System services
  - `manager/`: Process manager that starts/stops all services
  - `loggerd/`: Logging and encoding
  - `camerad/`: Camera interface
  - `athena/`: Cloud connectivity
  - `hardware/`: Hardware abstraction layer

- **common/**: Shared utilities (params, logging, filters)

- **cereal/**: Message definitions (Cap'n Proto schemas in `log.capnp`)

- **tools/**: Development tools (replay, cabana, simulator, plotjuggler)

### Submodules (symlinked)

- `opendbc` → `opendbc_repo/opendbc`: CAN database and car interfaces
- `panda`: Safety code and hardware interface
- `msgq` → `msgq_repo/msgq`: Inter-process messaging
- `rednose` → `rednose_repo/rednose`: Kalman filter library
- `tinygrad` → `tinygrad_repo/tinygrad`: ML framework for models

### Process Architecture

The system runs as multiple processes managed by `system/manager/`. Key processes defined in `system/manager/process_config.py`:
- **selfdrived**: Main state machine
- **controlsd**: Vehicle controls
- **modeld**: Driving model inference
- **card**: Car interface
- **camerad**: Camera capture
- **loggerd**: Data logging

Processes communicate via ZeroMQ pub/sub using Cap'n Proto messages defined in `cereal/log.capnp`.

## Code Style

- Python: 2-space indentation, ruff for linting, line length 160
- Use `openpilot.` prefix for imports (e.g., `from openpilot.common.params import Params`)
- Use `time.monotonic` instead of `time.time`
- Use pytest, not unittest
- UI: Use `system.ui.lib` functions, not raw raylib calls

## Safety Requirements

openpilot follows ISO26262 guidelines. Do not:
- Disable or nerf driver monitoring
- Disable or nerf excessive actuation checks in `selfdrive/selfdrived/helpers.py`
- Modify safety code in `opendbc/safety/` without maintaining full test coverage

## PR Guidelines

- PRs should be against master branch
- Keep PRs under 500 lines
- Every line must directly contribute to stated purpose
- Include verification method and benchmarks if optimizing
