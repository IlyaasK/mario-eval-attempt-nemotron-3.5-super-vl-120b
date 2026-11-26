# Mario Eval Attempt: Nemotron 3.5 Super VL 120B (NVIDIA NIM, omp)

One 24-hour autonomous run of [Mario Eval](https://github.com/IlyaasK/mario-eval) ([ilya.as/mario-eval](https://ilya.as/mario-eval/)): an autonomous agent gets the C decompilation of **Super Mario 64** (`n64decomp/sm64`), pinned N64 compilers (`ido` + `mips64-elf-gcc`), an instrumented cycle/cache/RDP-accurate `ares` emulator (`n64run`), and the `mev` profiler/self-check CLI, and optimizes real 4 MB N64 console execution while preserving gameplay, visuals, and audio fidelity.

No Nintendo code or ROM assets are redistributed here — `patches/` contains only Nemotron 3.5 Super VL 120B's unified diffs against `n64decomp/sm64`, alongside the full v3.1 grading summaries and compressed agent session transcripts.

| | |
|---|---|
| Model | `nvidia/nemotron-3.5-super-vl-120b` (NVIDIA NIM API) |
| Agent harness | `omp` (oh-my-pi) |
| Environment | Mario Eval SM64 offline task container (24.0h active budget) |
| Tool calls & commits | ~4,150 tool calls, 58 git commits; best valid checkpoint `934580c` (`checkpoint-9`, 18.2h) |
| Evaluation split | 25 routes (6 public courses + 8 private scripted routes + 11 private seeded button-mash routes) + `PLAY=1` console boot smoke test |

## Scores (v3.1 Symmetric RDP Bus Model, Section-Weighted 40/50/10)

| Evaluation | Routes | Status | Console Speedup | Time Saved | vs GCC `-O2` | Reward | `PLAY=1` Smoke |
|---|---:|---|---:|---:|---:|---:|---|
| 1996 Retail US Baseline (`r0`) | 25 | `valid` | `1.000×` | `0.0%` | `0.745×` | `0.000` | pass |
| Compiler Only (`r1-gcc-o2`) | 25 | `valid` | `1.343×` | `25.5%` | `1.000×` | `+0.106` | pass |
| **Nemotron 3.5 Super VL best valid (`934580c`, CP9)** | **25** | **`valid`** | **`1.390×`** | **`28.0%`** | **`1.035×`** | **`+0.119`** | **pass** |
| DeepSeek-V4.1-Flash-1017 (`b4c0cd1`, CP2) | 25 | `valid` | `1.566×` | `36.1%` | `1.166×` | `+0.162` | pass |
| Kimi K3.1 Pro Trial (`4b54858`, CP4) | 25 | `valid` | `1.640×` | `39.0%` | `1.221×` | `+0.178` | pass |
| Kaze Master Reference (`r10-kaze-master`) | 25 | `valid` | `1.663×` | `39.9%` | `1.238×` | `+0.183` | pass |
| **Nemotron 3.5 Super VL final (`314214c`, HEAD)** | **25** | **`invalid`** | `—` | `—` | `—` | `-1.000` | `play_no_video_progress` |

## Trajectory Summary

- **Hours 0–18.2 (`0e76292` → `934580c`):** Switched to `GRUCODE=f3dex2` and `COMPILER=gcc` `-O3 -fno-fast-math`, added Y-bounds surface collision pre-filtering, inlined hot animation/distance helpers, and unrolled `guMtxF2L`, climbing cleanly to **`1.390×`** across all 25 routes with `PLAY=1` boot smoke passing.
- **Hours 18.2–24.0 (`cc2cf35` → `314214c`):** Replaced `osWritebackDCacheAll()` with `osWritebackDCache(spTask, sizeof(*spTask))` (flushing the 64-byte task header struct rather than the pointed-to `gGfxPool` display list), causing 616 RSP DMA coherence errors and boot hangs across 28 trailing unverified commits.
