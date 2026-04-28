# 数字人（audio2video / talking head）

给一张人脸 portrait + 一段音频，生成口型、头部动作或长时数字人动画对齐的 MP4。OmniRT 通过 `soulx-flashtalk-14b` 和 `soulx-liveact-14b` 支持这一任务面。

## 最小示例

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

## 关键参数

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `image` | `str` | **必填** | 人脸 portrait 路径 |
| `audio` | `str` | **必填** | 音频路径（推荐 `.wav`；支持 ffmpeg 能解的格式） |
| `prompt` | `str?` | `None` | 可选提示（表情 / 情绪） |
| `preset` | `fast`/`balanced`/`quality`/`low-vram` | `balanced` | 见 [预设](../features/presets.md) |
| `fps` | `int?` | 模型默认 | 输出帧率 |
| `repo_path` | `str?` | auto | 外部仓库 checkout 路径（SoulX-FlashTalk 为 script-backed 模型，首次运行会自动克隆） |

## 支持模型

| 模型 | 输入 | 输出 | 显存 |
|---|---|---|---|
| `soulx-flashtalk-14b` | portrait + audio | MP4 | ≥ 20 GB |
| `soulx-liveact-14b` | portrait + audio | MP4 | 4 卡 Ascend 910B 推荐 |

!!! info "SoulX-FlashTalk 是 script-backed 模型"
    当前实现使用外部 SoulX-FlashTalk checkout。内网/离线环境请通过 `repo_path`、`ckpt_dir`、`wav2vec_dir` 指向本地路径。

!!! info "SoulX-LiveAct 的 Ascend 910B 推荐配置"
    `soulx-liveact-14b` 使用外部 SoulX-LiveAct checkout 的 `generate.py`。Ascend 上默认设置 `PLATFORM=ascend_npu`，默认会先用单张 NPU 运行 `prepare_text_cache.py`，再启动 4 卡推理；显式多卡时可用 `--text-cache-visible-devices 2 --visible-devices 2,3,4,5` 固定为 1 卡 T5 + 4 卡推理。快速 smoke 可加 `--sample-steps 1`。若使用 LightVAE，请同时设置 `--vae-path models/vae/lightvaew2_1.pth --use-lightvae --use-cache-vae`，并预热 `--condition-cache-dir`。

## 错误与排查

!!! warning

    - **音频采样率不匹配** — FlashTalk 要求 16 kHz 单声道；非此格式会自动 resample，但过长音频会放大误差
    - **portrait 对齐差** — 正面、上半身、眼睛可见效果最好
    - **外部仓库克隆失败** — 在国内网络下走 `GHPROXY` 或离线提供 `repo_path`
    - **Ascend 上速度显著低于 CUDA** — FlashTalk 的部分自定义算子尚未在 Ascend 上优化，会触发 eager 回退（见 `RunReport.backend_timeline`）
    - **LiveAct 启动时报 CUDA 设备不可用** — 确认 `PLATFORM=ascend_npu` 已设置；OmniRT wrapper 默认会设置，但手工运行外部仓库时容易漏掉
    - **LiveAct T5 OOM** — 优先使用默认的单卡 NPU text context cache；显式设置 `--text-cache-visible-devices`，并避免让 T5 和 4 卡主推理同时加载
