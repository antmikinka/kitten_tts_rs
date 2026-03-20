# kitten-tts-rs 🐱🦀

Rust port of [KittenTTS](https://github.com/KittenML/KittenTTS) — ultra-lightweight ONNX-based text-to-speech.

Models range from 15M to 80M parameters (25–80 MB on disk). Runs on CPU by default, with optional GPU acceleration via CUDA (NVIDIA) or CoreML (Apple Silicon).

## Features

- **Single binary** — no Python, no pip, no venv
- **ONNX Runtime** via the [`ort`](https://crates.io/crates/ort) crate
- **8 built-in voices** — Bella, Jasper, Luna, Bruno, Rosie, Hugo, Kiki, Leo
- **GPU acceleration** — CUDA, TensorRT, or CoreML via Cargo features
- **Text preprocessing** — number/currency expansion built-in
- **24 kHz WAV output**

## Quick Start

```bash
# Build (CPU only)
cargo build --release

# Build with NVIDIA GPU support
cargo build --release --features cuda

# Build with Apple Silicon (CoreML) support
cargo build --release --features coreml
```

### Download a model

Download from Hugging Face (e.g. the 80M mini model):

```bash
# Using huggingface-cli
pip install huggingface_hub
huggingface-cli download KittenML/kitten-tts-mini-0.8 --local-dir ./models/kitten-tts-mini

# Or manually download config.json, the .onnx file, and voices.npz
```

### Generate speech

```bash
# Basic usage
./target/release/kitten-tts ./models/kitten-tts-mini "Hello, world!" Bruno

# With options
./target/release/kitten-tts ./models/kitten-tts-mini "Hello, world!" \
  --voice Luna --speed 1.2 --output hello.wav

# List voices
./target/release/kitten-tts ./models/kitten-tts-mini "" --list-voices
```

## Prerequisites

- **espeak-ng** for phonemization:
  ```bash
  # macOS
  brew install espeak-ng

  # Ubuntu/Debian
  sudo apt install espeak-ng

  # Windows
  # Download from https://github.com/espeak-ng/espeak-ng/releases
  ```

## GPU Acceleration

### NVIDIA (CUDA / TensorRT)

```bash
cargo build --release --features cuda
# or
cargo build --release --features tensorrt
```

Requires CUDA toolkit and cuDNN installed on the system.

### Apple Silicon (CoreML)

```bash
cargo build --release --features coreml
```

Uses Apple's Neural Engine / Metal GPU via CoreML. The ONNX model is automatically converted to CoreML format at runtime.

> **Note:** CoreML EP requires building ONNX Runtime from source with CoreML support. The `ort` crate handles this when the feature is enabled, but the first build may take longer.

### Execution Provider Priority

When multiple features are enabled, the priority order is:
1. TensorRT (if available)
2. CUDA (if available)
3. CoreML (if available)
4. CPU (always available as fallback)

## Architecture

```
src/
├── main.rs        # CLI entry point (clap)
├── lib.rs         # Library root
├── model.rs       # ONNX session, inference, chunking
├── phonemize.rs   # espeak-ng phonemization + token ID mapping
├── preprocess.rs  # Text normalization (numbers, currency, etc.)
└── voices.rs      # NPZ voice embedding loader
```

## Compared to Python KittenTTS

| | Python | Rust |
|---|---|---|
| Dependencies | onnxruntime, misaki, phonemizer, numpy, soundfile, spacy | ort, hound, espeak-ng (system) |
| Binary size | ~500MB (with venv) | ~10MB |
| Startup time | ~2s (Python import) | ~100ms |
| Deployment | pip install + venv | Single binary |
| GPU support | onnxruntime-gpu pip | Cargo feature flag |

## License

Apache-2.0 (same as KittenTTS)
