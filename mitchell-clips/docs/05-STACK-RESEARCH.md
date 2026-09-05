# 05: Stack research (Sep 2026)

Researched against GitHub topic pages and repo READMEs on 2026-09-04. Star counts ±5%. Verify a repo's current state before pinning.

## Verdict: no single repo does the whole job locally. Write ~400 lines, borrow from the best parts.

| Capability | Pick | Why | Borrow from |
|---|---|---|---|
| Download | `yt-dlp` | standard | |
| Transcription | **faster-whisper 1.2.x** (`large-v3-turbo`), word timestamps, VAD | fastest CUDA path, MIT | |
| Scene cuts | **PySceneDetect 0.7.1** `AdaptiveDetector` | feeds selector + reframer, BSD | |
| Scoring | **Ollama** + `qwen3:8b` / `qwen3:14b` / `gemma4:12b`, `format=` JSON schema | MIT, Windows installer, structured output | AI-Youtube-Shorts-Generator's virality prompt (MIT) |
| Reframe | **MediaPipe Face Detector** + One-Euro smoothing, scene resets, center-crop fallback | Apache-2.0, CPU-fast, avoids ultralytics AGPL | ClipsAI `resize()` logic, Autocrop-vertical scene-aware crop |
| Captions | **pysubs2 → ASS karaoke → `ffmpeg -vf subtitles=`** | fastest burn-in, no Playwright/MoviePy | ai-video-captions style presets (MIT) |
| Silence | faster-whisper VAD word gaps; `auto-editor.exe` only if tighter cuts needed | one toolchain | |
| Publish | **Composio** Instagram toolkit with `video_file` upload | no bucket needed | |

## End-to-end clippers surveyed

| Repo | Stars | License | Verdict |
|---|---|---|---|
| [Anil-matcha/AI-Youtube-Shorts-Generator](https://github.com/Anil-matcha/AI-Youtube-Shorts-Generator) | ~4.8k | MIT | Closest. yt-dlp → faster-whisper → LLM virality rank → OpenCV face crop. Ranking is OpenAI/Gemini only (swap base URL to Ollama). No karaoke captions. Borrow the prompt and crop code. |
| [mutonby/openshorts](https://github.com/mutonby/openshorts) | ~2.5k | MIT | Polished, but hard cloud deps (Gemini, fal.ai). Docker-first (GPU passthrough via WSL2 on Windows). Reference for MediaPipe+YOLO reframe modes. |
| [JayWebtech/autoshorts](https://github.com/JayWebtech/autoshorts) | ~650 | unstated | Local-first Tauri app with Ollama support. Center-crop only, basic drawtext captions. |
| [ClipsAI/clipsai](https://github.com/ClipsAI/clipsai) | ~540 | MIT | Library. Unmaintained, pinned to old WhisperX/torch. Its `resize()` active-speaker logic is the reference. |
| [Shaarav4795/ClippedAI](https://github.com/Shaarav4795/ClippedAI) | ~170 | CC BY-NC | All-local reference with ASS captions. NonCommercial license: read, don't vendor. |
| [atherion005-byte/agent-opus](https://github.com/atherion005-byte/agent-opus) | ~10 | MIT | faster-whisper GPU + Ollama + YOLOv8 tracking + word highlight, Windows path pre-wired. Tiny community. Borrow freely. |
| [xixihhhh/hotclip](https://github.com/xixihhhh/hotclip) | ~60 | AGPL | Electron, libass karaoke, Ollama. AGPL: read only. |
| ShortGPT (~7.7k), FunClip (~6.2k) | | | Not applicable (faceless generator / Chinese-first ASR, no reframe). |

## Transcription

- [faster-whisper](https://github.com/SYSTRAN/faster-whisper) ~25k, MIT, v1.2.1. `BatchedInferencePipeline`, Silero VAD.
- [whisperX](https://github.com/m-bain/whisperX) ~23.5k, BSD-2. Tighter word alignment (wav2vec2). Upgrade if karaoke timing looks loose. Use ≥3.8.7rc1 on Windows (CUDA detection fix). Can't align non-dictionary tokens ("$13.60").
- insanely-fast-whisper: flash-attn pain on Windows. Skip.

**CUDA/cuDNN rules:** faster-whisper ≥1.0 → CTranslate2 ≥4.0 → CUDA 12 + cuDNN 9. Easiest: `pip install nvidia-cublas-cu12 nvidia-cudnn-cu12` and `os.add_dll_directory` on their `bin` dirs, or install the torch cu12x wheel and `import torch` first. Symptom of mismatch: `Could not locate cudnn_ops64_9.dll`. Stuck on cuDNN 8: pin `ctranslate2==4.4.0`. Zero-Python fallback: [Purfview/whisper-standalone-win](https://github.com/Purfview/whisper-standalone-win).

## Reframe

- [mediapipe](https://github.com/google-ai-edge/mediapipe) ~37k, Apache-2.0. BlazeFace hundreds of fps at 720p on CPU. **AutoFlip is legacy (support ended 2023), don't use.** Wheels lag Python releases: 3.11 safe.
- ultralytics YOLOv8/11 + face weights: best on small/profile faces but **AGPL-3.0**. Only if MediaPipe fails on his footage.
- [Autocrop-vertical](https://github.com/kamilstanuch/Autocrop-vertical) ~336: PySceneDetect → per-scene crop-vs-letterbox. Scene detection ≈ 50% of runtime, so cache it. Normalize VFR sources first.
- Single-host commentary = largest face + One-Euro smoothing + scene-cut reset is sufficient. Multi-person would need diarization → skip.

## Captions

- [ai-video-captions](https://github.com/nicolaigaina/ai-video-captions) MIT: faster-whisper → pysubs2 ASS with per-word `\k` → `ffmpeg -vf subtitles`. 6 style presets. This is the approach.
- [pycaps](https://github.com/francozanardi/pycaps) MIT: CSS-styled via Playwright. Most flexible, heaviest. Later if Mitchell wants motion styles.
- [captacity](https://github.com/unconv/captacity) MIT: MoviePy, slow. Skip.
- Windows: escape drive colon in `subtitles=` path, pass `fontsdir=`, use gyan.dev "full" ffmpeg (has libass + nvenc).

## Local LLM

- [Ollama](https://github.com/ollama/ollama) ~178k MIT. `format=<json schema>` in the Python client. OpenAI-compatible `/v1` on :11434.
- Models (Q4_K_M): 8GB → `qwen3:8b` (~5GB) or `llama3.1:8b`; 12GB → `gemma4:12b` or `qwen3:14b`; 16GB → `phi4:14b` or `gpt-oss:20b`. Sub-7B degrades badly. Disable Qwen3 thinking (`think=False`) for speed. `num_ctx ≥ 8192`.
- Chunk transcript into 60-90s windows with 10-15s overlap; ask for a JSON array per window.

## Scenes / silence

- [PySceneDetect](https://github.com/Breakthrough/PySceneDetect) ~5.1k BSD-3, v0.7.1 (fixed VFR timestamps). `pip install scenedetect[opencv]`.
- [auto-editor](https://github.com/WyattBlue/auto-editor) ~5.1k, now Nim, **not on pip**: download `auto-editor.exe`, call via subprocess. Only if VAD gaps aren't tight enough.

## Install (PowerShell, Python 3.11 venv at C:\clips)

```
winget install Gyan.FFmpeg
# new shell
python -m venv .venv; .\.venv\Scripts\Activate.ps1
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
pip install faster-whisper==1.2.1 nvidia-cublas-cu12 nvidia-cudnn-cu12
pip install "scenedetect[opencv]" mediapipe opencv-python pysubs2 yt-dlp ollama pydantic pyyaml fastapi uvicorn python-dotenv composio
# Ollama: https://ollama.com/download/OllamaSetup.exe ; then: ollama pull qwen3:8b
```

## Windows gotchas (read before debugging)

- `ffmpeg` not found after winget → open a new terminal. Verify `ffmpeg -filters | findstr subtitles` and `ffmpeg -encoders | findstr nvenc`.
- cu128 wheels need driver ≥ 570 (`nvidia-smi` shows max CUDA). Never mix PyPI CPU torch with the cu wheel.
- Long paths / spaces break tools. Project at `C:\clips`.
- Ollama on hybrid graphics: `ollama ps` must say 100% GPU; if CPU, update the NVIDIA driver.
- auto-editor.exe triggers SmartScreen on first run.

Sources: repo pages linked above; [opus-clip-alternative topic](https://github.com/topics/opus-clip-alternative?o=desc&s=stars); [Ollama structured outputs](https://docs.ollama.com/capabilities/structured-outputs); [CTranslate2 releases](https://github.com/OpenNMT/CTranslate2/releases); [Meta IG media reference](https://developers.facebook.com/docs/instagram-platform/instagram-graph-api/reference/ig-user/media).
