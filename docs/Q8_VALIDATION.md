# Q8_0 validation receipt

This receipt records checks run on 2026-07-13 against commit `ea59224`, the head
of the branch submitted as
[TrevorS/voxtral-mini-realtime-rs#15](https://github.com/TrevorS/voxtral-mini-realtime-rs/pull/15).
The tracked files in this mirror at `ccac074` are identical to that branch
outside `README.md`; the mirror commit adds its status introduction.

## Environment

- MacBook Pro with Apple M1 Pro
- macOS 27.0
- Rust toolchain selected by the repository

No model weights were downloaded for these checks.

## Commands and observed results

```bash
cargo fmt --all -- --check
```

Exit 0.

```bash
cargo clippy --features "wgpu,cli,hub" -- -D warnings
```

Exit 0. Cargo separately reported a future-incompatibility warning for the
transitive dependency `block v0.1.6`.

```bash
cargo test --features "wgpu,cli,hub" --no-run
```

Exit 0. The library, binaries and integration-test targets compiled.

The non-GPU selection used by the upstream CI workflow also exited 0:

```bash
cargo test --features "wgpu,cli,hub" -- \
  --skip "gguf::" \
  --skip "models::adapter::tests" \
  --skip "models::decoder::tests" \
  --skip "models::encoder::tests" \
  --skip "models::layers" \
  --skip "models::time_embedding::tests" \
  --skip "models::voxtral::tests" \
  --skip "tts::" \
  --skip "test_q4_"
```

Observed result: 63 passed, 0 failed. GPU, GGUF and TTS groups were filtered by
the upstream skip list.

Focused Q8 execution on Metal:

```bash
WGPU_BACKEND=metal cargo test --features "wgpu,cli,hub" test_q8_
cargo test --features "wgpu,cli,hub" test_gguf_reader_q8_dtype
```

Observed results: seven Q8 tests passed and the GGUF Q8 reader test passed. The
focused set covers:

- CPU block quantization/dequantization and edge cases;
- byte-count rejection;
- GPU dequantization;
- GPU matmul on small, attention and feed-forward shapes;
- Q8-backed linear-layer shape;
- GGUF Q8 dtype parsing and payload reading.

The largest reported difference in the focused matmul output was approximately
`1.53e-5`, below the test tolerance.

## What this does not prove

These checks use synthetic tensors. They do not execute a complete Voxtral TTS
model, evaluate generated audio, measure peak GPU memory or latency, or exercise
Vulkan. Claims about listening quality, an 8 GB fit, real-time behavior or
cross-backend parity require separate reproducible receipts.
