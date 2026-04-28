# Talking Head (audio2video)

Given a face portrait plus an audio clip, produce an MP4 where lips, head motion, or long-form avatar animation are aligned to the speech. OmniRT supports this task via `soulx-flashtalk-14b` and `soulx-liveact-14b`.

## Minimal example

=== "Python"

    ```python
    from omnirt import generate
    from omnirt.requests import audio2video

    result = generate(audio2video(
        model="soulx-flashtalk-14b",
        image="inputs/portrait.png",
        audio="inputs/speech.wav",
        preset="balanced",
    ))
    ```

=== "CLI"

    ```bash
    omnirt generate \
      --task audio2video \
      --model soulx-flashtalk-14b \
      --image inputs/portrait.png \
      --audio inputs/speech.wav \
      --preset balanced
    ```

=== "HTTP"

    ```bash
    curl -sS http://localhost:8000/v1/generate \
      -H 'Content-Type: application/json' \
      -d '{
        "task": "audio2video",
        "model": "soulx-flashtalk-14b",
        "inputs": {
          "image": "inputs/portrait.png",
          "audio": "inputs/speech.wav"
        },
        "config": {"preset": "balanced"}
      }'
    ```

## Key parameters

| Parameter | Type | Default | Notes |
|---|---|---|---|
| `image` | `str` | **required** | path to the face portrait |
| `audio` | `str` | **required** | audio path (prefer `.wav`; any ffmpeg-decodable format works) |
| `prompt` | `str?` | `None` | optional hint (expression / emotion) |
| `preset` | `fast`/`balanced`/`quality`/`low-vram` | `balanced` | see [Presets](../features/presets.md) |
| `fps` | `int?` | model default | output frame rate |
| `repo_path` | `str?` | auto | external repo checkout path (SoulX-FlashTalk is script-backed; first run auto-clones) |

## Supported models

| Model | Inputs | Output | VRAM |
|---|---|---|---|
| `soulx-flashtalk-14b` | portrait + audio | MP4 | ≥ 20 GB |
| `soulx-liveact-14b` | portrait + audio | MP4 | 4-card Ascend 910B recommended |

!!! info "SoulX-FlashTalk is a script-backed model"
    The current implementation uses an external SoulX-FlashTalk checkout. In offline or restricted networks, point `repo_path`, `ckpt_dir`, and `wav2vec_dir` at local paths.

!!! info "Recommended SoulX-LiveAct settings on Ascend 910B"
    `soulx-liveact-14b` launches `generate.py` from an external SoulX-LiveAct checkout. The wrapper sets `PLATFORM=ascend_npu` by default, prepares text context with `prepare_text_cache.py` on a single NPU, then launches the 4-card inference job. With explicit placement, use `--text-cache-visible-devices 2 --visible-devices 2,3,4,5` for the 1-card T5 + 4-card inference split. Add `--sample-steps 1` for quick smoke tests. For LightVAE, pair `--vae-path models/vae/lightvaew2_1.pth --use-lightvae --use-cache-vae` and warm `--condition-cache-dir`.

## Troubleshooting

!!! warning

    - **Audio sample-rate mismatch** — FlashTalk expects 16 kHz mono. Other formats are resampled automatically, but long audio amplifies error.
    - **Poor portrait alignment** — frontal, upper-body, eyes-visible portraits give the best output.
    - **External repo clone fails** — behind the GFW, route through `GHPROXY` or supply an offline `repo_path`.
    - **Ascend much slower than CUDA** — some FlashTalk custom ops aren't optimized on Ascend and fall back to eager (tracked in `RunReport.backend_timeline`).
    - **LiveAct reports CUDA device unavailable** — confirm `PLATFORM=ascend_npu`; the OmniRT wrapper sets it, but manual external runs often miss it.
    - **LiveAct T5 OOM** — prefer the default single-NPU text-context cache path; set `--text-cache-visible-devices` explicitly and avoid loading T5 inside the 4-card inference process.
