---
layout: post
title: "Benchmark LLM Performance on Multiple Nvidia GPUs"
description: "I'm benchmarking and comparing the performance of multiple Nvidia GPUs using Ollama."
tags: server linux nvidia gpu
---

Using my [Ampere homelab server]({% post_url 2025-02-06-ampere-server %}) with 6 sticks of RAM and 1 NVMe SSD as of April 2025. Driver installation see _posts/2025-02-07-install-nvidia-drivers-arm64.md. I'm currently running driver version 535.216.01 with CUDA version 12.2.

Not using any AMD cards since those aren't supported by the ARM64 architecture of my server.

All benchmarks are performed in Docker containers. For the 100% GPU load tests I'm using https://github.com/wilicc/gpu-burn with the command

```shell
docker run --rm -it --gpus all gpu_burn "./gpu_burn" "120"
```

For the LLM benchmarks I'm using the metrics output by Ollama itself. I evaluated the following three models because they fit in 16GB VRAM:

* llama3.1:8b
* codestral:22b
* deepseek-r1:14b

```
docker exec -it ollama ollama run "llama3.1:8b" --verbose "How does Kant use the distinction between things and persons in expressing the supreme principle of morality?"

docker exec -it ollama ollama stop llama3.1:8b

docker exec -it ollama ollama run "codestral:22b" --verbose "Write a Python function that calculates AQI based on PM2.5 and PM10 values."

docker exec -it ollama ollama stop codestral:22b

docker exec -it ollama ollama run "deepseek-r1:14b" --verbose "Why is the blue sky blue?"
docker exec -it ollama ollama stop deepseek-r1:14b
```

## No GPU
Power consumption idle: 71W

## Nvidia RTX A4000
Power consumption idle: 89W -> 18W
Power consumption with `gpu-burn`: 216W -> 145W

Llama results:
```
total duration:       10.550899069s
load duration:        2.295360475s
prompt eval count:    28 token(s)
prompt eval duration: 215.26018ms
prompt eval rate:     130.08 tokens/s
eval count:           445 token(s)
eval duration:        8.038562751s
eval rate:            55.36 tokens/s

total duration:       12.702390746s
load duration:        2.319831899s
prompt eval count:    28 token(s)
prompt eval duration: 226.416081ms
prompt eval rate:     123.67 tokens/s
eval count:           561 token(s)
eval duration:        10.154546426s
eval rate:            55.25 tokens/s

total duration:       10.464312387s
load duration:        2.305546304s
prompt eval count:    28 token(s)
prompt eval duration: 198.468944ms
prompt eval rate:     141.08 tokens/s
eval count:           453 token(s)
eval duration:        7.958521916s
eval rate:            56.92 tokens/s
```

Codestral results:
```
total duration:       30.837413424s
load duration:        2.936504836s
prompt eval count:    27 token(s)
prompt eval duration: 261.043206ms
prompt eval rate:     103.43 tokens/s
eval count:           690 token(s)
eval duration:        27.638590295s
eval rate:            24.97 tokens/s

total duration:       41.082998432s
load duration:        2.884138581s
prompt eval count:    27 token(s)
prompt eval duration: 263.985199ms
prompt eval rate:     102.28 tokens/s
eval count:           942 token(s)
eval duration:        37.933491323s
eval rate:            24.83 tokens/s

total duration:       34.820410353s
load duration:        2.925243607s
prompt eval count:    27 token(s)
prompt eval duration: 275.07597ms
prompt eval rate:     98.15 tokens/s
eval count:           784 token(s)
eval duration:        31.618714887s
eval rate:            24.80 tokens/s
```

Deepseek results:
```
total duration:       24.642735145s
load duration:        2.732116959s
prompt eval count:    10 token(s)
prompt eval duration: 246.532954ms
prompt eval rate:     40.56 tokens/s
eval count:           694 token(s)
eval duration:        21.662608864s
eval rate:            32.04 tokens/s

total duration:       39.815176546s
load duration:        2.773568021s
prompt eval count:    10 token(s)
prompt eval duration: 237.431802ms
prompt eval rate:     42.12 tokens/s
eval count:           1159 token(s)
eval duration:        36.802781395s
eval rate:            31.49 tokens/s

total duration:       38.516851577s
load duration:        2.740291299s
prompt eval count:    10 token(s)
prompt eval duration: 259.0575ms
prompt eval rate:     38.60 tokens/s
eval count:           1138 token(s)
eval duration:        35.516034729s
eval rate:            32.04 tokens/s
```

## RTX A5000
Power consumption idle: 89W -> 18W
Power consumption with `gpu-burn`: 309W -> 238W

Llama results:
```
total duration:       9.800584636s
load duration:        2.295455169s
prompt eval count:    28 token(s)
prompt eval duration: 219.270022ms
prompt eval rate:     127.70 tokens/s
eval count:           564 token(s)
eval duration:        7.283505762s
eval rate:            77.44 tokens/s

total duration:       8.898229932s
load duration:        2.372185876s
prompt eval count:    28 token(s)
prompt eval duration: 190.643919ms
prompt eval rate:     146.87 tokens/s
eval count:           492 token(s)
eval duration:        6.332728134s
eval rate:            77.69 tokens/s

total duration:       11.483094259s
load duration:        2.245776441s
prompt eval count:    28 token(s)
prompt eval duration: 227.055463ms
prompt eval rate:     123.32 tokens/s
eval count:           690 token(s)
eval duration:        9.008750264s
eval rate:            76.59 tokens/s
```

Codestral results:
```
total duration:       14.139452138s
load duration:        2.86317158s
prompt eval count:    27 token(s)
prompt eval duration: 243.22701ms
prompt eval rate:     111.01 tokens/s
eval count:           420 token(s)
eval duration:        11.031706627s
eval rate:            38.07 tokens/s

total duration:       23.020000995s
load duration:        2.890126106s
prompt eval count:    27 token(s)
prompt eval duration: 203.720573ms
prompt eval rate:     132.53 tokens/s
eval count:           750 token(s)
eval duration:        19.924837155s
eval rate:            37.64 tokens/s

total duration:       21.138402439s
load duration:        2.848679443s
prompt eval count:    27 token(s)
prompt eval duration: 247.544021ms
prompt eval rate:     109.07 tokens/s
eval count:           682 token(s)
eval duration:        18.040770374s
eval rate:            37.80 tokens/s
```

Deepseek results:
```
total duration:       19.496116938s
load duration:        2.727597637s
prompt eval count:    10 token(s)
prompt eval duration: 228.808678ms
prompt eval rate:     43.70 tokens/s
eval count:           745 token(s)
eval duration:        16.53694358s
eval rate:            45.05 tokens/s

total duration:       24.314787258s
load duration:        2.735340204s
prompt eval count:    10 token(s)
prompt eval duration: 222.915635ms
prompt eval rate:     44.86 tokens/s
eval count:           965 token(s)
eval duration:        21.354493453s
eval rate:            45.19 tokens/s

total duration:       17.660754481s
load duration:        3.035558455s
prompt eval count:    10 token(s)
prompt eval duration: 238.794266ms
prompt eval rate:     41.88 tokens/s
eval count:           649 token(s)
eval duration:        14.383528338s
eval rate:            45.12 tokens/s
```

## Nvidia T4
Using case fans and a shroud for cooling, no additional fan added.

Power consumption idle: 79W -> 8W
Power consumption with `gpu-burn`: 141W -> 70W

Llama results:
```
total duration:       20.381795116s
load duration:        3.179664751s
prompt eval count:    28 token(s)
prompt eval duration: 191.908575ms
prompt eval rate:     145.90 tokens/s
eval count:           584 token(s)
eval duration:        17.00875429s
eval rate:            34.34 tokens/s

total duration:       21.516093539s
load duration:        3.137821894s
prompt eval count:    28 token(s)
prompt eval duration: 196.76127ms
prompt eval rate:     142.30 tokens/s
eval count:           639 token(s)
eval duration:        18.180120885s
eval rate:            35.15 tokens/s

total duration:       23.368700914s
load duration:        4.040593456s
prompt eval count:    28 token(s)
prompt eval duration: 192.173758ms
prompt eval rate:     145.70 tokens/s
eval count:           622 token(s)
eval duration:        19.134474533s
eval rate:            32.51 tokens/s
```

Codestral results:
```
total duration:       1m0.469023427s
load duration:        3.851923752s
prompt eval count:    27 token(s)
prompt eval duration: 271.07711ms
prompt eval rate:     99.60 tokens/s
eval count:           807 token(s)
eval duration:        56.344632307s
eval rate:            14.32 tokens/s

total duration:       53.429056311s
load duration:        4.006094875s
prompt eval count:    27 token(s)
prompt eval duration: 286.928357ms
prompt eval rate:     94.10 tokens/s
eval count:           690 token(s)
eval duration:        49.134650429s
eval rate:            14.04 tokens/s

total duration:       1m15.718878807s
load duration:        3.96648466s
prompt eval count:    27 token(s)
prompt eval duration: 299.609389ms
prompt eval rate:     90.12 tokens/s
eval count:           1051 token(s)
eval duration:        1m11.451337228s
eval rate:            14.71 tokens/s
```

Deepseek results::
```
total duration:       54.103588898s
load duration:        3.693419224s
prompt eval count:    10 token(s)
prompt eval duration: 231.07106ms
prompt eval rate:     43.28 tokens/s
eval count:           845 token(s)
eval duration:        50.177700958s
eval rate:            16.84 tokens/s

total duration:       37.839830061s
load duration:        3.847850677s
prompt eval count:    10 token(s)
prompt eval duration: 235.584945ms
prompt eval rate:     42.45 tokens/s
eval count:           638 token(s)
eval duration:        33.754573346s
eval rate:            18.90 tokens/s

total duration:       46.741535764s
load duration:        3.850300759s
prompt eval count:    10 token(s)
prompt eval duration: 224.462449ms
prompt eval rate:     44.55 tokens/s
eval count:           733 token(s)
eval duration:        42.665387586s
eval rate:            17.18 tokens/s
```
