# latent-crowd-mpc

Minimal research codebase skeleton for latent world models, MPC, and crowded robot navigation.

This repository is intentionally small at this stage. It does not include algorithms,
training code, datasets, checkpoints, or PyTorch dependencies yet.

## Setup

```bash
pip install -e ".[dev]"
```

## Tests

```bash
python -m pytest -q
```

## Layout

```text
configs/                 Experiment and project configuration files
src/latent_crowd_mpc/    Python package source
tests/                   Minimal test suite
```
