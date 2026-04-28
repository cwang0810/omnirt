# Support Status

This document tracks the models already integrated into `omnirt`, the ones that have completed real hardware smoke tests, and the high-priority targets that are still pending.

Last updated: `2026-04-28`

## Current public task surfaces

- `text2image`
- `image2image`
- `text2video`
- `image2video`
- `audio2video`

## Integrated models

The full list is generated from the live registry: [Supported Models](supported_models.md). This page only tracks real-hardware smoke status and partial-support notes.

## Real hardware smoke completed

The following models have completed real hardware smoke tests using local model directories:

- `sdxl-base-1.0`
  CUDA: `internal CUDA validation host`
  Ascend: `internal Ascend validation host`
- `svd-xt`
  CUDA: `internal CUDA validation host`
  Ascend: `internal Ascend validation host`
- `soulx-flashtalk-14b`
  Ascend: `internal Ascend validation host`
  Notes: the `persistent_worker` path has completed an 8-card `Ascend 910B2` hardware run.
- `soulx-liveact-14b`
  Ascend: `internal Ascend validation host`
  Notes: the external SoulX-LiveAct `generate.py` path has been aligned to the 4-card `Ascend 910B` official case; OmniRT exposes it through a script-backed wrapper. By default it prepares text context on one NPU before the 4-card inference job. Use `--text-cache-visible-devices <single-card> --visible-devices <four-cards> --sample-steps 1` for quick smoke.

## Integrated but still waiting for real hardware smoke

These models already have registry entries, request-surface integration, and local unit coverage, but they do not yet have repository-tracked local model directories plus verified dual-backend smoke results:

- `sdxl-refiner-1.0`
- `flux-fill`
- `flux-kontext`
- `qwen-image-edit`
- `qwen-image-edit-plus`
- `chronoedit`
- `flux-depth`
- `flux-canny`
- `qwen-image-layered`
- `animate-diff-sdxl`
- `kolors`
- `pixart-sigma`
- `bria-3.2`
- `lumina-t2x`
- `mochi`
- `skyreels-v2`

Relevant smoke tests already exist. For the now-public `image2image` surface, the recommended starting models are `sdxl-base-1.0`, `sdxl-refiner-1.0`, `sd15`, and `sd21`:

- `tests/integration/test_sdxl_refiner_cuda.py`
- `tests/integration/test_sdxl_refiner_ascend.py`
- `tests/integration/test_flux_fill_cuda.py`
- `tests/integration/test_flux_fill_ascend.py`
- `tests/integration/test_image_edit_cuda.py`
- `tests/integration/test_image_edit_ascend.py`

## Partial support

- `helios`
  Currently exposed as two registry keys: `helios-t2v` and `helios-i2v`.
- `hunyuan-video-1.5`
  Currently exposed as two registry keys: `hunyuan-video-1.5-t2v` and `hunyuan-video-1.5-i2v`.

## High-priority targets not completed yet

- No new high-priority model gap is tracked here; the current priority is to keep adding real-hardware smoke evidence and deployable local model sources for integrated models.

## Related docs

- [Model Support Roadmap](roadmap.md)
- [China Deployment](../deployment/china_mirrors.md)
