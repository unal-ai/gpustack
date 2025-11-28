# CI/CD and deployment pipelines

## Overview

This fork now owns its release pipeline. Everything runs on GitHub Actions and publishes to GitHub-owned endpoints:
- `CI` (`.github/workflows/ci.yml`): runs `make ci` on pushes, PRs, and tags across Python 3.10–3.12. Tag builds upload wheels to the GitHub release and optionally publish to PyPI.
- `Pack` (`.github/workflows/pack.yaml`): builds the container image from `pack/Dockerfile` for `linux/amd64` and `linux/arm64`, pushing to `ghcr.io/unal-ai/gpustack` on pushes/tags (PRs build only).
- `Docs` (`.github/workflows/docs.yml`): builds the MkDocs site and deploys it to GitHub Pages at `https://unal-ai.github.io/gpustack`.

## Required secrets and permissions

- `GITHUB_TOKEN` (built-in) is used for GHCR pushes and Pages publishing. No Docker Hub or upstream org credentials are needed.
- Set `PYPI_API_TOKEN` (or keep using `CI_PYPI_API_TOKEN`) to enable PyPI publishing on tag builds. Without it, the PyPI step is skipped.

## Release and promotion flow

1. Create a tag `vX.Y.Z` on `main` (or `v*-dev`) to trigger a release.
2. `CI` rebuilds and uploads the wheel to the GitHub Release; PyPI publish runs only if the token is configured.
3. `Pack` builds and pushes multi-arch images with semantic version tags and `latest` (for non-rc tags) to GHCR.
4. Docs changes merged to `main` automatically refresh the Pages site.

## Local checks before opening a PR

- `make ci` — installs deps, lints, tests, and builds the wheel.
- `make build-docs` — renders the docs locally before pushing doc updates.
- When running the server manually, you can point at a different registry with `--system-default-container-registry <your_registry>` if you mirror the GHCR images internally.
