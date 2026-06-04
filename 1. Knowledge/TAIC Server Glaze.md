## Checking the Specs:
1. GPU:
```shell
nvidia-smi
```
2. CPU & RAM:
```shell
lscpu               # CPU cores, architecture
free -h             # available RAM
```
3. Disk Space:
```shell
df -h
```

---

## Specs:
*GPUs*: 2× NVIDIA RTX 4000 Ada Generation
- ~20 GB VRAM each, ~40 GB combined
- Both nearly idle right now
- CUDA 12.6 — fully current
*CPU*: Xeon Gold 5416S — 16 cores / 32 threads, fast
*RAM: 30 GB* — decent but will be a bottleneck for very large models
*Disk*: Main drive is 96% full (only 44 GB free)
- The `/share` drive has 9.5 TB free though — store models there

---
### Recommended model:
llama3:70b-instruct-q4_0

![[Pasted image 20260602142546.png]]