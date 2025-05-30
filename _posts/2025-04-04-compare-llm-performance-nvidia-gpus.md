---
layout: post
title: "Benchmark LLM Performance on Turing and Ampere Nvidia GPUs"
description: "I'm benchmarking and comparing the performance of multiple Nvidia GPUs using Ollama."
tags: server linux nvidia gpu ai
---

I wanted to see how various Nvidia GPUs of the Turing and Ampere generations compare in terms of LLM performance. I developed the script [stefanthoss/ollama-server-benchmark](https://github.com/stefanthoss/ollama-server-benchmark) for benchmarking Ollama prompts.

## GPU Selection

I'm comparing the following Nvidia GPUs:

| Model | Generation | CUDA Cores | Memory [GB] | Memory Bandwidth [GB/s] | TDP [W] | PCIe Interface |
|---|---|---|---|---|---|---|
| Tesla T4 | Turing | 2560 | 16 | 320 | 75 | 3.0x16 |
| Quadro RTX 8000 | Turing | 4608 | 48 | 672 | 250 | 3.0x16 |
| A2 | Ampere | 1280 | 16 | 200 | 60 | 4.0x8 |
| RTX A4000 | Ampere | 6144 | 16 | 448 | 140 | 4.0x16 |
| RTX A5000 | Ampere | 8192 | 24 | 768 | 230 | 4.0x16 |

I have skipped older Nvidia generations like Pascal and Volta because [Nvidia is phasing out driver support](https://www.tomshardware.com/pc-components/gpu-drivers/nvidia-starts-phasing-out-maxwell-pascal-and-volta-gpus-geforce-driver-support-status-unclear) and I have skipped newer generations like Ada Lovelace and Blackwell because they're still pretty high in price. I'm only looking at workstation and data center cards because these tend to be smaller and use less energy compared to their desktop counterparts. I'm not using any AMD or Intel cards since those aren't supported by the arm64 architecture of my server.

## Test Setup

For the 100% GPU load tests I'm using [wilicc/gpu-burn](https://github.com/wilicc/gpu-burn) with the command

```shell
docker run --rm -it --gpus all gpu_burn "./gpu_burn" "300"
```

I'm using my [Ampere homelab server]({% post_url 2025-02-06-ampere-server %}) with 6 sticks of RAM and 1 NVMe SSD as of April 2025. The driver installation is documented [here]({% post_url 2025-04-16-install-nvidia-drivers-arm64.md }) (currently running driver version 535.216.01 with CUDA version 12.2). For the LLM benchmarks I'm using [stefanthoss/ollama-server-benchmark](https://github.com/stefanthoss/ollama-server-benchmark) together with a Docker-based Ollama installation. I evaluated the following models because they fit in 16GB VRAM:

- deepseek-r1:14b
- gemma3:12b
- llama3.1:8b
- mistral-nemo:12b
