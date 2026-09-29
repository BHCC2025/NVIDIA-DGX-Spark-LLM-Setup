# NVIDIA DGX Spark LLM Setup — Start Here

Run open LLMs such as Qwen and Gemma locally on 1, 2 or 3 NVIDIA DGX Sparks (GB10) with vLLM: tested recipes, one
setup script, and multi-Spark tensor parallelism over direct QSFP cables (one box, two joined by a cable, or three in
a triangle).

Every speed below was measured on our own Sparks with the recipe's own benchmark, and every setup passed its smoke
test.

## Start here

1. Pick a recipe below: the model you want, on as many Sparks as you have.
2. On the Spark you'll serve from:
   ```bash
   git clone https://github.com/BHCC2025/<recipe>.git
   cd <recipe>
   ./setup.sh
   ```
   `./setup.sh` checks and installs what's missing, finds how your Sparks are cabled and gives the ports IP addresses,
   tests the network with a real NCCL all-reduce, and downloads the model. It asks before every change.
3. `./run.sh tp1` on one Spark (`tp2` on two, `tp3` on three), then `scripts/smoke-test.sh`. The API is
   OpenAI-compatible, on port 8000 of that Spark.

You don't need to download anything else: each recipe includes the setup kit.

**Before you start:** DGX OS 7 with its current updates on every Spark, and no Hugging Face login needed for the
recipes below. For more than one Spark, connect the cables first (see below). No IP addresses to configure:
`./setup.sh` finds which port goes where and assigns them.

### Cabling

Each DGX Spark has two high-speed QSFP network ports. No switch is needed: the Sparks are cabled to each other
directly with 200GbE QSFP cables.

**Two Sparks, for TP2:** one cable, from either port on one Spark to either port on the other.

**Three Sparks, for TP3 (triangle):** three cables, so that each Spark's two ports go to its two different neighbours:

| Cable | From | To |
|---|---|---|
| 1 | Spark A, port 1 | Spark B, port 1 |
| 2 | Spark B, port 2 | Spark C, port 1 |
| 3 | Spark C, port 2 | Spark A, port 2 |

Any other arrangement works too, as long as no Spark has both cables going to the same neighbour; `./setup.sh`
detects the layout and tells you if it is wrong. Run `./setup.sh` on Spark A, which becomes the head node.

## Recipes

| Recipe | Sparks | 1 Spark (code / prose) | Fastest (code / prose) |
|---|---|---|---|
| [Qwen3.8-Flash-Next](https://github.com/BHCC2025/Qwen3.8-Flash-Next-DGX-Spark-TP1-TP3) | 1–3 | 40.1 / 26.4 tok/s | 62.8 / 40.7 tok/s on 3; up to 1M context |
| [Gemma-4-31B-IT](https://github.com/BHCC2025/Gemma-4-31B-IT-DGX-Spark-TP1-TP2) | 1–2 | 24.2 / 16.6 tok/s | 40.6 / 28.0 tok/s on 2 |

Single-stream decode. Each recipe's `bench/results/` has the raw logs.

## Something not working?

Run `./setup.sh --check` in the recipe (it changes nothing) and open an issue on that recipe with the
`.setup/report.txt` it writes.

## Writing a recipe

[dgx-spark-recipe-kit](https://github.com/BHCC2025/dgx-spark-recipe-kit) has the shared setup, the benchmark and
`new-recipe.sh`, which starts a new recipe in the same layout.
