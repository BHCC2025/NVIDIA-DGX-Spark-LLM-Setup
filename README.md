# NVIDIA DGX Spark LLM Setup — Start Here

Run open LLMs such as Qwen locally on one or two NVIDIA DGX Sparks (GB10). We now serve with
[TensorFold](https://github.com/ashhart/TensorFold) by @ashhart. On one Spark, with its recommended checkpoint, it decodes
Qwen3.8 Flash Next about twice as fast as our vLLM recipe did, serves several users at once, and installs with one
`pip` command inside NVIDIA's PyTorch container.

This page tells you what to run and where the recipe lives. Some recipes are ours; for others we point to the people
who already publish them. Every speed below was measured on our own Sparks.

## Start here

1. **Pick a model and the number of Sparks** from the table below.
2. **Follow that recipe's README.** For TensorFold on one Spark it comes down to three commands:
   ```bash
   docker run -it --gpus all --ipc=host --network host nvcr.io/nvidia/pytorch:26.07-py3
   python -m pip install git+https://github.com/ashhart/TensorFold.git
   tensorfold serve <checkpoint> --host 0.0.0.0
   ```
   The API is OpenAI-compatible.
3. **Two Sparks** need one cable and a working network between them first; see [Cabling](#cabling).

**Before you start:** DGX OS 7 with its current updates on every Spark. No Hugging Face login is needed for the
models below.

## Recipes

| Model | Sparks | Engine | Recipe | 1 user (code / chat) | 8 users (code / chat) |
|---|---|---|---|---|---|
| Qwen3.8-27B (MLX 4-bit) | 1 | TensorFold | [TensorFold docs](https://github.com/ashhart/TensorFold/blob/main/docs/recipes/qwen3.8-27b.md) · [MiaAI-Lab recipe](https://github.com/MiaAI-Lab/Qwen3.8-27B-DGX-Spark-TensorFold) | 69.1 / 51.3 tok/s | 319 / 230 tok/s |
| Qwen3.8-27B (MLX 4-bit) | 2 | TensorFold | [ours](https://github.com/BHCC2025/Qwen3.8-27B-DGX-Spark-TP2-TensorFold) | 100.1 / 73.3 tok/s | 387 / 279 tok/s |
| Qwen3.8 Flash Next | 1 | TensorFold | [TensorFold docs](https://github.com/ashhart/TensorFold/blob/main/docs/recipes/qwen3.8-flash-next.md) · [MiaAI-Lab recipe](https://github.com/MiaAI-Lab/Qwen3.8-Flash-Next-Single-DGX-Spark-TensorFold) | 85.8 / 72.0 tok/s | 425 / 388 tok/s |
| Qwen3.6-35B-A3B (MLX 4-bit) | 1 | TensorFold | [ours](https://github.com/BHCC2025/Qwen3.6-35B-A3B-DGX-Spark-TP1-TensorFold) | 214.4 / 164.2 tok/s | 558 / 472 tok/s |
| Gemma-4-31B-IT | 1–2 | vLLM | [ours](https://github.com/BHCC2025/Gemma-4-31B-IT-DGX-Spark-TP1-TP2) | 24.2 / 16.6 tok/s (1 Spark) | — |

How we measured: the TensorFold rows use TensorFold 0.6.2 and its own `tools/bench_concurrent.py` (greedy, drafts on
as shipped). The Gemma row uses that recipe's `bench/bench.sh`, whose `bench/results/` has the raw logs. All measured on our
Sparks, 2026-09-24 to 2026-10-02.

The Qwen3.6-35B-A3B recipe also fits a 64 GB Spark (simulated on a 128 GB one): `./run.sh tp1` picks 4 users at 65,536 tokens
there, with the same single-user speed.

**Both engines, same benchmark** (one Spark, Qwen3.8 Flash Next, our recipe kit's `bench/bench.sh`, thinking off):

| Engine and checkpoint | Decode, code / prose | Cold prompt, 8K tokens |
|---|---|---|
| vLLM, `nvidia/Qwen3.8-Flash-Next-NVFP4` | 40.1 / 26.4 tok/s | 1,779 tok/s |
| TensorFold 0.6.2, the same NVFP4 checkpoint | 44.7 / 33.8 tok/s | 1,621 tok/s |
| TensorFold 0.6.2, `Vontra/Qwen3.8-Flash-Next-MLX-4bit-MTP` (recommended) | 79.8 / 60.7 tok/s | 2,261 tok/s |

On NVIDIA's NVFP4 checkpoint TensorFold decodes faster but reads prompts a little slower (it reports that checkpoint
as untested); on its recommended MLX 4-bit checkpoint it is faster at both. Each engine sampled with its own default
temperature.

TensorFold also runs Nemotron 3.5 Lightning, and GLM-5.3-Flash on two Sparks; see its
[model list](https://github.com/ashhart/TensorFold#models).

## Cabling

Each DGX Spark has two 200GbE QSFP ports. No switch is needed.

**Two Sparks:** one QSFP cable, from either port on one Spark to either port on the other.

**Four Sparks:** we have not benched a four-Spark ring ourselves yet.
[MiaAI-Lab's switchless ring recipe](https://github.com/MiaAI-Lab/glm-5.3-flash-4x-dgx-spark-switchless) shows one.

Our [setup kit](https://github.com/BHCC2025/dgx-spark-recipe-kit) finds which port goes where, gives the ports
addresses and tests the link with a real NCCL all-reduce (`./setup.sh`; it asks before every change).

## Older vLLM recipes

[Qwen3.8-Flash-Next-DGX-Spark-TP1-TP3](https://github.com/BHCC2025/Qwen3.8-Flash-Next-DGX-Spark-TP1-TP3) (vLLM,
1–3 Sparks) is archived. It still works as published, but we no longer maintain it.

## Something not working?

Open an issue on the recipe you followed. For TensorFold itself, use
[TensorFold's issues](https://github.com/ashhart/TensorFold/issues).
