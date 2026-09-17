# qwen3.5-abliterated-local

Run a 9B coding model entirely on your own machine. No API key, no account, no
per-token bill, no session limits, no request leaving your network.

This repo is **configuration, not weights** — a Modelfile, setup steps, and
measured benchmark numbers for running `Qwen3.5-9B-abliterated` locally under
[Ollama](https://ollama.com) on consumer hardware.

## Quick start

```bash
ollama pull hf.co/mradermacher/Huihui-Qwen3.5-9B-abliterated-GGUF:IQ4_XS
ollama create qwen-local -f Modelfile
ollama run qwen-local
```

Serves an OpenAI-compatible endpoint on `http://127.0.0.1:11434/v1`, so most
tools that accept a custom base URL will talk to it unchanged.

## What it runs on

Benchmarked on an **RTX 2060 SUPER (8 GB)** — a six-year-old mid-range card.

| | |
|---|---|
| Quant | IQ4_XS |
| Size on disk | 5.7 GB |
| VRAM at 16K context | ~6.9 GB |
| Coding score | 8/12 |
| Tool calling | 100% |
| Edit validity | 100% |

The 8 GB figure is the point: this is not a datacenter model. If you have a
gaming GPU from 2019, you can run it.

### Fitting it in 8 GB

VRAM is the whole game. With a typical desktop running, ~3 GB is already gone
before the model loads, and Ollama will quietly put the overflow on your CPU —
which drops throughput by more than an order of magnitude.

```bash
nvidia-smi --query-compute-apps=pid,used_memory,process_name --format=csv
```

Check `ollama ps` after a request. If it says anything other than `100% GPU`,
close whatever is holding VRAM (compositor effects and animated wallpapers are
common culprits) or drop to a smaller quant.

Recommended server settings:

```
OLLAMA_CONTEXT_LENGTH=16384
OLLAMA_KV_CACHE_TYPE=q8_0
OLLAMA_FLASH_ATTENTION=true
```

`q8_0` roughly halves KV cache VRAM versus the f16 default.

### Thinking mode

Qwen3.5 defaults to thinking **on**, and on a small budget it will spend the
entire allowance reasoning and return an empty answer. Disable per request:

```bash
curl http://127.0.0.1:11434/api/generate \
  -d '{"model":"qwen-local","prompt":"...","think":false}'
```

`think` is a top-level field, not an `options` key. `PARAMETER think false` in a
Modelfile is rejected by current Ollama versions.

## Honest notes

**This is an abliterated model.** Abliteration suppresses the model's refusal
behaviour. That is a real modification with real consequences — it will answer
things the stock model declines, and you own what you do with it.

**It costs accuracy.** Measured on the same harness, the stock Qwen3.5-9B scores
**11/12** on coding versus **8/12** here. If you want a local coding model and
nothing else, use the stock build — it is strictly better at the job:

```bash
ollama pull hf.co/unsloth/Qwen3.5-9B-GGUF:IQ4_XS
```

**Benchmarks are mine, not official.** 12-task coding suite on one machine, one
GPU. Treat them as a sighting shot, not a leaderboard.

## Credits

None of the model weights are my work. The chain:

| Stage | By |
|---|---|
| Base model (Qwen3.5-9B) | [Alibaba / Qwen](https://huggingface.co/Qwen) |
| Abliteration | [huihui_ai](https://huggingface.co/huihui-ai) |
| GGUF quantisation | [mradermacher](https://huggingface.co/mradermacher) |
| Modelfile, benchmarks, setup notes | this repo |

Weights are distributed by the parties above and carry **their** licence terms —
check the upstream model cards before redistributing or using commercially.
Nothing here relicenses anything.

## Licence

The configuration and documentation in this repo are MIT. The model weights are
not covered by it and are not distributed here.
