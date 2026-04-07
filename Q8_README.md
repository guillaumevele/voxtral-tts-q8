# Voxtral TTS Q8 — Near-Lossless 8-bit Text-to-Speech

Fork of [voxtral-mini-realtime-rs](https://github.com/TrevorS/voxtral-mini-realtime-rs) with **Q8_0 quantization support** for Voxtral 4B TTS.

## What's New

This fork adds **Q8_0 (8-bit) quantization** to the Voxtral TTS inference engine, providing **near-lossless audio quality** at roughly half the BF16 model size.

| Quantization | Model Size | Audio Quality | RTF (RTX 4070) |
|---|---|---|---|
| BF16 (original) | 8.8 GB | Reference | N/A (needs 16GB+) |
| **Q8_0 (this fork)** | **4.5 GB** | **~99% of BF16** | ~4.8x |
| Q4_0 (original) | 2.7 GB | Good, some artifacts | ~2.7x |

## Changes

- **2 new WGSL compute shaders** (`shader_q8.wgsl`, `shader_naive_q8.wgsl`) for fused Q8 dequant+matmul on GPU
- **Rust GGUF reader** extended with Q8_0 dtype support (code 8, 34 bytes per block of 32 elements)
- **Tensor/linear/op modules** updated to handle Q8 blocks
- **Python quantization script** with `--quant-type q8_0` option
- **Model loader** supporting Q8 weight tensors for both ASR and TTS

### Q8_0 Block Format
```
Block of 32 elements = 34 bytes:
  - 2 bytes: f16 scale
  - 32 bytes: signed int8 quantized values
Dequantization: value = scale * int8_value
```

## Quick Start

### Generate Q8 TTS GGUF
```bash
# Download BF16 weights
uv run --with huggingface_hub hf download mistralai/Voxtral-4B-TTS-2603 --local-dir models/voxtral-tts

# Quantize to Q8
uv run scripts/quantize_tts_gguf.py models/voxtral-tts/ -o voxtral-tts-q8.gguf --quant-type q8_0
```

### Build & Run
```bash
cargo build --release --features "wgpu,cli,hub"

# TTS inference with Q8
cargo run --release --features "wgpu,cli,hub" --bin voxtral -- \
  speak --text "Hello, I am Voxtral in 8-bit quality." \
  --gguf voxtral-tts-q8.gguf \
  --voice casual_female \
  --euler-steps 8
```

### Available Voices (20 presets, 9 languages)
| Language | Voices |
|---|---|
| English | casual_female, casual_male, neutral_female, neutral_male, cheerful_female |
| French | fr_female, fr_male |
| Spanish | es_female, es_male |
| German | de_female, de_male |
| Italian | it_female, it_male |
| Portuguese | pt_female, pt_male |
| Dutch | nl_female, nl_male |
| Arabic | ar_male |
| Hindi | hi_female, hi_male |

## Requirements

- NVIDIA GPU with Vulkan/WebGPU support (tested on RTX 4070 Laptop, 8GB VRAM)
- Rust 1.94+
- ~4.5 GB VRAM for Q8 model

## Credits

- Original project: [TrevorS/voxtral-mini-realtime-rs](https://github.com/TrevorS/voxtral-mini-realtime-rs)
- Model: [Mistral AI / Voxtral-4B-TTS-2603](https://huggingface.co/mistralai/Voxtral-4B-TTS-2603)
- Q8 shaders & integration: Guillaume Vele + Claude Code

## License

MIT (same as original)
