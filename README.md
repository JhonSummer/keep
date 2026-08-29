# Keep

Saved from [voxium’s post](https://x.com/v0xium/status/2093527405759693240). The article under that screenshot is Michal Pitr’s C++ series, not an LLM server writeup.

## The blog

[Build Your Own Inference Engine: From Scratch to "7"](https://michalpitr.substack.com/p/build-your-own-inference-engine-from) — Michal Pitr, 4 Aug 2024. Building a C++ inference engine from scratch, part 1.

Train a small MNIST net, export it as ONNX, then write an engine that loads the protobuf graph, topologically sorts it, and runs Flatten, Gemm, ReLU, and Add. The last step in the post is a handwritten 7. Argmax of the output is class 7.

Code lives in [MichalPitr/inference_engine](https://github.com/MichalPitr/inference_engine).

## Series

1. [From Scratch to "7"](https://michalpitr.substack.com/p/build-your-own-inference-engine-from) — load ONNX, build the DAG, first correct prediction.
2. [Optimizing Performance](https://michalpitr.substack.com/p/inference-engine-optimizing-performance) — drop Tensor copies, about 15× faster. GEMM is the real cost after that.
3. [Accelerating with CUDA](https://michalpitr.substack.com/p/inference-engine-accelerating-with) — execution providers, keep weights on GPU, reuse a memory pool. Batch 1 is about 5× the CPU path. Batch 128 is about 60×.

## Why this and not the quoted thread

The quote is a five-layer map for an LLM engine (weights, ops, autoregressive loop, KV cache, then CPU/GPU speed). Pitr’s series is the first working engine: a real model, a real graph, a correct digit. Keep that first.
