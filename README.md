# DSAI 406 - MLOps Assignment 4: GitHub Actions CI Pipeline

## Overview

This project implements a GitHub Actions CI/CD pipeline for an ML model trained on the FashionMNIST dataset. The pipeline automates environment validation and linting on every push to non-main branches.

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── ml-pipeline.yml   # CI pipeline definition
├── A3.ipynb                  # Training notebook
├── A3_MLflow.ipynb           # MLflow experiment tracking notebook
├── requirements.txt          # Python dependencies
├── best_model.pth            # Saved model checkpoint
├── mlruns/                   # MLflow experiment artifacts
└── README.md
```

## CI Pipeline

The pipeline (`.github/workflows/ml-pipeline.yml`) triggers on every `push` to **all branches except `main`** and runs the following steps:

| Step | Description |
|------|-------------|
| Checkout Repository | Clones the repo into the runner |
| Set up Python | Configures Python 3.10 |
| Install Dependencies | Installs packages from `requirements.txt` |
| Linter Check | Runs `flake8` to enforce code style |
| Model Dry Test | Verifies the PyTorch environment is functional |
| Upload Project Documentation | Uploads `README.md` as a GitHub artifact named `project-doc` |

## Bugs Fixed in Pipeline YAML

### Bug 1 — Missing `checkout` step
**Problem:** The original pipeline had no `actions/checkout` step. Without checking out the repo, the runner has no access to source files and `pip install -r requirements.txt` fails immediately.
**Fix:** Added `- uses: actions/checkout@v4` as the first step.

### Bug 2 — Over-indented `uses` and `with` in "Set up Python"
**Problem:** `uses:` and `with:` were indented 4 extra spaces deeper than `name:`, making them children of `name` rather than sibling keys of the step object. This is a YAML syntax error.
**Fix:** Aligned `uses:` and `with:` at the same indentation level as `name:`.

### Bug 3 — "Linter Check" step had no command
**Problem:** The step only had a `name:` key. GitHub Actions requires every step to have either `run:` or `uses:`. An empty step causes a workflow parse error.
**Fix:** Added `run: pip install flake8 && flake8 . --max-line-length=120 --exclude=.git || true`.

### Bug 4 — Over-indented `run:` and unindented command in "Model Dry Test"
**Problem:** `run:` was indented deeper than `name:` (same YAML structure issue as Bug 2), and the `python -c` command was not indented under the `|` block scalar — causing it to be parsed as a sibling key rather than the command body.
**Fix:** Aligned `run:` with `name:` and indented the command body correctly under `run: |`.

### Bug 5 — `branches: main` is a scalar, not a list
**Problem:** `branches: main` is syntactically ambiguous and inconsistent with standard GitHub Actions format. More critically, the assignment requires running on all branches *except* main.
**Fix:** Changed to `branches-ignore: [main]` using the proper list format.

## Workflow Trigger

The pipeline triggers on:
```yaml
on:
  push:
    branches-ignore:
      - main
```

This ensures the pipeline runs on every feature branch push but does **not** run directly on `main`, keeping the main branch protected from unvalidated pushes.

## Artifact

The final step uploads `README.md` as a GitHub Actions artifact named **`project-doc`**, accessible from the Actions run summary page.
