# Technology Stack

## Language & Runtime

- **Python 3.10+** (mypy configured for 3.10, CI runs 3.11)
- No virtual environment committed; dependencies via `pip install -r requirements.txt`

## Core Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| cadquery | >=2.4.0 | OpenCASCADE B-Rep kernel for parametric 3D geometry |
| numpy | >=1.24.0 | Numerical arrays, airfoil coordinate manipulation |
| scipy | >=1.10.0 | CubicSpline interpolation, Savitzky-Golay filtering |
| ezdxf | >=1.1.0 | DXF reading/writing for nesting and template export |
| pytest | >=7.0.0 | Test framework |
| pytest-cov | >=4.0.0 | Coverage reporting |
| mkdocs | >=1.5.0 | Documentation generation |
| mkdocs-material | >=9.0.0 | Documentation theme |

## Dev Dependencies (`requirements-dev.txt`)

| Package | Version | Purpose |
|---------|---------|---------|
| pre-commit | >=3.7.0 | Git hooks for linting |
| ruff | >=0.5.4 | Linting and formatting |
| mypy | >=1.10.0 | Static type checking |

## Optional / External Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| OpenVSP (NASA) | Aerodynamic validation via vortex lattice solver | Commented out in requirements.txt; must be installed separately from openvsp.org. Code gracefully degrades when absent (surrogate models used instead). |

## Configuration

- **`mypy.ini`**: Python 3.10, `ignore_missing_imports = True`, `disallow_untyped_defs = False`, follows imports silently for `core.*`
- **`.pre-commit-config.yaml`**: ruff (lint + format), mypy (scripts/ only), custom geometry smoke test hook
- **No `pyproject.toml` or `setup.py`** -- not packaged as a library; runs via `python main.py`

## Build / Run

```bash
pip install -r requirements.txt
python main.py --generate-all          # Full artifact generation
python main.py --validate              # Config validation only
python main.py --validate-physics      # Physics regression check
python main.py --canard                # Canard only
python main.py --wing                  # Main wing only
python main.py --compliance            # FAA compliance report
python main.py --nest-sheets           # DXF nesting optimizer
python main.py --summary               # Config summary
pre-commit run --all-files             # Lint + format + smoke test
```

## CI / CD

- **GitHub Actions** (`.github/workflows/ci.yml`)
- Two jobs: `lint` (pre-commit) and `geometry-validation` (smoke test + artifact upload)
- Runs on `ubuntu-latest` with Python 3.11
- Uploads STEP/STL/DXF/GCODE artifacts on success

## Output Formats

| Format | Tool | Use Case |
|--------|------|----------|
| STEP | CadQuery `exporters.export()` | CAM software import |
| STL | CadQuery `exporters.export()` | 3D printed jigs/fixtures |
| DXF | CadQuery + ezdxf | Laser-cut templates, nested sheets |
| G-code (.tap) | Custom `GCodeWriter` | 4-axis hot-wire CNC foam cutting |
| VSP3 JSON | Custom adapter | OpenVSP geometry metadata |
| JSON | stdlib `json` | Metadata, compliance tracking, physics baselines |
| Markdown | Custom generators | Compliance reports, layup schedules |
| CSV | Custom nesting export | Nest manifests for CAM import |
