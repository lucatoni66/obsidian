---
title: "Running ~30B MoE models on an old GTX 1080"
source: "https://mdda.net/blog/tech/dl/llama-cpp-moe-on-an-old-gtx-1080"
author:
  - "[[Martin Andrews]]"
published: 2026-05-11
created: 2026-06-18
description: "( Fedora 42, llama.cpp )"
tags:
  - "clippings llama"
---
This post covers how I got the Qwen 3.6 35B-A3B and Gemma 4 26B-A4B Mixture-of-Experts models running on a fairly elderly GeForce GTX 1080 (8 GiB VRAM, Pascal / Compute Capability 6.1) under Fedora 42.

The starting point was an awesome [YouTube video by Codacus](https://www.youtube.com/watch?v=8F_5pdcD3HY) that touted essentially this setup, but with the usual YouTube-shaped emphasis on “OMG this works” rather than the grindy details of actually getting it working. In contrast, this post *shows* the grindy details: which forks to use, which CUDA/glibc/gcc combinations break, which flags actually matter, and how the numbers came out.

The trick, in both cases, is that with MoE models only a few experts activate per token, so you can park the bulk of the (cold) expert weights in system RAM and stream them via PCIe, while keeping the always-hot layers and KV cache on the GPU. `llama.cpp` exposes this directly via `--n-cpu-moe N` (keep the MoE weights of the first N layers on the CPU) and `--n-gpu-layers 999` (everything else on the GPU).

Getting there on Fedora 42 involves going through a few hoops - mostly because the GTX 1080 is past NVIDIA’s mainline driver EOL, and current CUDA + current `glibc` don’t agree out of the box.

## The hardware

The full system spec, since the CPU and RAM matter as much as the GPU when you’re streaming MoE weights over PCIe:

- **CPU**: Intel i7-6700 (Skylake, 4 cores / 8 threads, 2015 vintage)
- **RAM**: 32 GiB system RAM
- **GPU**: NVIDIA GeForce GTX 1080, 8 GiB VRAM (Pascal, 2016 vintage)
- **OS**: Fedora 42

Nothing on this list is what you’d *new* buy today. But I did actually buy it second-hand in 2025 for < 200 USD (and that was a reasonable price at the time).

The GPU’s compute capability matters for the build; and the PCIe link speed matters for inference throughput.

```bash
lspci | grep NVIDIA  # Find your Nvidia card
01:00.0 VGA compatible controller: NVIDIA Corporation GP104 [GeForce GTX 1080] (rev a1)

lspci -vv -s 01:00.0 | grep -E "LnkCap|LnkSta"
#   LnkCap: Port #0, Speed 8GT/s, Width x16, ASPM L0s L1, Exit Latency L0s <512ns, L1 <4us
#   LnkSta: Speed 2.5GT/s (downgraded), Width x16
```

NB: (the ‘downgraded’ LnkSta is the link sitting idle - it negotiates a speed-up under load). From the web: 8.0 GT/s => PCIe 3.0 (sadly not PCIe 4.0)

Re-running the same `lspci` query *while* `llama-server` is generating tokens (this is worth doing once you’ve got the setup working — the diagnostic is what convinced me the PCIe link, not the GPU, was the bottleneck):

```bash
# In another terminal, while the model is actively generating:
lspci -vv -s 01:00.0 | grep LnkSta
#   LnkSta: Speed 8GT/s, Width x16
# i.e. the link is now running at PCIe 3.0 8 GT/s max rates
```

At the same time, when running the model, I found that `nvidia-smi` reported the GPU sitting at roughly 40-50% utilisation. PCIe maxed out + GPU half-idle = the system is **bandwidth-limited**, not compute-limited. That’s the single most important fact for thinking about what’s likely to make things faster on this kind of setup: anything that reduces the volume of weight data crossing the PCIe bus per token (more aggressive quantisation, fewer CPU-side MoE layers, speculative decoding when it works) helps; anything that just makes the GPU faster (e.g. a newer-but-similarly-bandwidth-constrained card) wouldn’t.

“The GeForce GTX 1080 is based on the Pascal architecture, which has a Compute Capability of 6.1 (SM 61).” That matters later when telling CMake which architecture to build for.

## First Steps: Staple the NVIDIA driver to the 580xx branch

Pascal is on its way to legacy status. On Fedora 42 (and later) you want to pin `akmod-nvidia` to the 580xx branch rather than letting it move to whatever the current mainline is — which can drop Pascal at any point.

```bash
# Swap to the 580xx legacy branch
dnf swap akmod-nvidia akmod-nvidia-580xx --allowerasing --releasever=44
```

(The `--releasever=44` is so dnf pulls the 580xx packaging that lives in the newer repo metadata, even though the running system is Fedora 42.)

## First Steps: CUDA toolkit and a working nvcc

```bash
dnf reinstall cuda-nvcc-12-9.x86_64
find / | grep nvcc
# /usr/local/cuda-12.9/bin/nvcc

export CUDACXX=/usr/local/cuda-12.9/bin/nvcc
```

The CMake build will need `nvcc` on the path, or `CUDACXX` set.

## Preparation for llama.cpp: Force gcc-14 for the CUDA build

CUDA 12.9 doesn’t accept the newest gcc that Fedora 42 ships by default, so we need an older `gcc` / `g++` available. Installing the side-by-side `gcc14` packages is easy:

```bash
dnf install gcc14 gcc14-c++
```

You’d *expect* to be able to point CMake at the right compiler via the usual `-DCMAKE_C_COMPILER=/usr/bin/gcc-14 -DCMAKE_CXX_COMPILER=/usr/bin/g++-14` (or `CC` / `CXX` environment variables). That doesn’t work here: somewhere inside the nvidia / CUDA-toolkit CMake modules, plain `gcc` is hard-coded, and overrides further up the chain are quietly ignored.

The least-distasteful workaround I could find is to put a `gcc` -> `gcc-14` symlink early on PATH, so the hard-coded reference resolves to the right binary. It’s ugly, and I’d love to know the proper fix, but it works:

```bash
mkdir -p ~/.local/bin
pushd ~/.local/bin/
  ln -s /usr/bin/gcc-14 gcc
  ln -s /usr/bin/g++-14 g++
popd

# Make sure ~/.local/bin is at the front of PATH
echo $PATH
```

Remember to undo this later if you don’t want every other build on the machine also using `gcc-14`.

## Patch CUDA’s math\_functions.h for glibc 2.41

CUDA 12.9’s headers were written against an older glibc, and on Fedora 42 (glibc 2.41) some inline definitions collide. The “least bad solution” is to patch CUDA’s header in place. Gentoo has a clean version of the patch:

- [nvidia-cuda-toolkit-glibc-2.41-r1.patch](https://github.com/stefantalpalaru/gentoo-overlay/blob/75b7793fdc88314165ca28c75b557defcb38f6d8/dev-util/nvidia-cuda-toolkit/files/nvidia-cuda-toolkit-glibc-2.41-r1.patch)

```bash
# Edit by hand, applying the patch above:
joe /usr/local/cuda-12.9/targets/x86_64-linux/include/crt/math_functions.h
```

Essentially, what needs to be done is to replace:

```cpp
... rsqrt(double x);
# with
... rsqrt(double x) noexcept (true);
```

and

```cpp
__func__(double rsqrt(double a));
# with
__func__(double rsqrt(double a)) throw();
```

for the functions:

```cpp
double rsqrt(double a);
double sinpi(double a);
double cospi(double a);
float rsqrtf(float a);
float sinpif(float a);
float cospif(float a);
```

## Choosing the right llama.cpp fork

Vanilla `llama.cpp` works fine for most usecases, but for an 8 GiB card you want a version that supports extra-aggressive KV-cache quantisations (`turbo2`, `turbo3`, `turbo4`). Those make the difference between “model fits at 16k context” and “model fits at 128k context” on this card.

The original video used the [TurboQuant](https://github.com/TheTom/llama-cpp-turboquant) fork, which worked fine for Qwen. However, it does not support the Gemma 4 MTP (speculative decoding) stuff.

To add Gemma 4 MTP functionality, I found the [AtomicChat](https://huggingface.co/AtomicChat/gemma-4-26B-A4B-it-assistant-GGUF) GGUFs, which pointed at a more modern fork: [AtomicBot-ai/atomic-llama-cpp-turboquant](https://github.com/AtomicBot-ai/atomic-llama-cpp-turboquant) (which is itself forked from `TheTom/llama-cpp-turboquant`) and is the right combination of features for the MTP head + RotorQuant cache.

```bash
git clone https://github.com/AtomicBot-ai/atomic-llama-cpp-turboquant.git
cd atomic-llama-cpp-turboquant/
```

## Now Build llama.cpp

Within the `llama.cpp` fork directory, do the cmake build:

```bash
export CUDACXX=/usr/local/cuda-12.9/bin/nvcc

cmake --fresh -B build -DGGML_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES=native
# 'native' picks up the compute capability of the installed 1080 GTX GPU.
# In practice cmake reports it as "75" (rather than the textbook 61 I was expecting
# for Pascal/SM 6.1) - but the resulting build runs fine, so I haven't pushed on
# where that comes from.
# Equivalently you could pin it explicitly: -DCMAKE_CUDA_ARCHITECTURES="61"

cmake --build build --config Release
# NB: --parallel seemed to cause problems, so I left it off
```

Sanity check that the GPU is visible:

```bash
cd ./build/bin  # This is where the binaries you just compiled will appear

./llama-cli --list-devices
# ggml_cuda_init: found 1 CUDA devices (Total VRAM: 8107 MiB):
#   Device 0: NVIDIA GeForce GTX 1080, compute capability 6.1, VMM: yes, VRAM: 8107 MiB
# Available devices:
#   CUDA0: NVIDIA GeForce GTX 1080 (8107 MiB, 7992 MiB free)
```

## Keep a local copy of llama-server --help

Before doing anything else, it’s worth dumping the full help text to a file you can `grep` against — there are a *lot* of flags, and you’ll be reaching for the docs constantly once you start tuning:

```bash
./llama-server --help > llama.cpp-man.txt
wc -l llama.cpp-man.txt
# 570 llama.cpp-man.txt
```

The dump is fork-dependent: flags like `turbo2` / `turbo3` / `turbo4`, `--mtp-head`, `--spec-type mtp`, `--cache-type-k-draft`, etc., only show up when you’ve built one of the TurboQuant/RotorQuant forks. Against vanilla `ggml-org/llama.cpp` you’ll get a shorter file.

The companion file [`llama.cpp-man.txt`](https://mdda.net/static/images/blog/2026-05//llama.cpp-man.txt) attached to this post is the one I generated from the atomic fork against my GTX 1080 — handy as a quick reference even if you haven’t built anything yet.

## Getting a model to test out

I used the Qwen 3.6 35B-A3B MoE in 4-bit (Q4\_K\_M) quantisation, from bartowski’s GGUFs:

- [Qwen/Qwen3.6-35B-A3B](https://huggingface.co/Qwen/Qwen3.6-35B-A3B) (base model card)
- [bartowski/Qwen\_Qwen3.6-35B-A3B-GGUF](https://huggingface.co/bartowski/Qwen_Qwen3.6-35B-A3B-GGUF) (GGUF quants)

The convenient `-hf` flag worked, but stashed the file deep in the HF cache where I couldn’t easily find it:

```bash
# This works, but hides the .gguf inside ~/.cache/huggingface/hub/models--bartowski--...
# ./llama-cli -hf bartowski/Qwen_Qwen3.6-35B-A3B-GGUF
```

So I just `wget` ed it directly to the current directory:

```bash
wget https://huggingface.co/bartowski/Qwen_Qwen3.6-35B-A3B-GGUF/resolve/main/Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf
# downloads into ./Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf
```

I then moved the gguf files into `~/Models/` for sanity’s sake.

## How to go about testing…

What follows is a sequence of `llama-server` invocations with their timing output. The search wasn’t very scientific - this was more “see if it runs, then see if I can make it better” than a proper benchmark sweep. The rough order of operations, for each model in turn, was:

1. **Does `llama-server` work at all?**: pick a deliberately conservative `--n-cpu-moe` value, see if the server starts, fire a small `curl` at it, get a reply. Expect OOMs on the first few attempts and dial back.
2. **What’s a sensible CPU-MoE starting point?**: start high (more layers on CPU) so there’s headroom on the GPU for the KV cache.
3. **Can `--ctx-size` go up?**: bump from the default to 64k, then 128k, watching the memory-breakdown print at startup.
4. **Optimise over `--n-cpu-moe` for speed**: drop the value one layer at a time (pushing more weights onto the GPU) until OOM, then back off by one. That gives the working floor.
5. **Do the other flags actually do anything?**: toggle `--mmap` / `--mlock`, `--flash-attn`, the K/V cache quantisations. Some matter, some don’t.

For Gemma 4 26B-A4B, the same five-step process was used, with the addition of:

6. **Re-run the whole thing with the MTP “assistant” head** for speculative decoding, to see if it’s a win.
7. **Re-optimise `--n-cpu-moe`** with the MTP head also resident (it isn’t very responsive — the floor turns out to be the same 20 layers with or without MTP).

A couple of caveats on what I *didn’t* check:

- **I didn’t grade the actual quality of the output.** Quick eyeballing only - the point of the exercise was getting wall-clock numbers rather than choosing the best model output.
- I ran two prompt styles: a text-y “explain how a server query works” prompt (~29 input tokens) and a code-y “stream the Fibonacci numbers under 1000 with only one print statement in the loop” prompt (~52 input tokens). Since the per-token numbers were close enough, I’m reporting them together.
- I *did* confirm that `llama-server` is correctly applying *different* chat templates for Qwen vs Gemma (so they’re not being inadvertently homogenised by some default template falling through): the model metadata embedded in the GGUF drives this, and the server logs the chosen template on startup.

## First run: Qwen 35B-A3B, baseline

```bash
./llama-server \
    --model ~/Models/Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf \
\
\
    --no-mmap
```

The relevant flags (full text is in `llama.cpp-man.txt`):

```js
-m,    --model FNAME                    model path to load
-ngl,  --gpu-layers, --n-gpu-layers N   max. number of layers to store in VRAM,
                                        either an exact number, 'auto', or 'all'
                                        (default: auto)
-ncmoe, --n-cpu-moe N                   keep the Mixture of Experts (MoE) weights
                                        of the first N layers in the CPU
--mmap, --no-mmap                       whether to memory-map model. (if mmap
                                        disabled, slower load but may reduce
                                        pageouts if not using mlock) (default:
                                        enabled)
--mlock                                 force system to keep model in RAM rather
                                        than swapping or compressing
```

A simple smoke-test query against the server’s OpenAI-compatible endpoint (use another terminal window running on the same machine):

```bash
curl -X POST http://localhost:8080/v1/chat/completions \
     -H "Content-Type: application/json" \
     -d '{
       "messages": [
         {"role": "system", "content": "You are a helpful assistant."},
         {"role": "user", "content": "Explain how a server query works."}
       ]
     }'
```

After quite a while (since we didn’t include `"stream": true` in the curl command `data` hash), the server returns timings as part of the overall JSON response:

```json
"timings": {
  "cache_n": 0,
  "prompt_n": 29, "prompt_ms": 515.945,
  "prompt_per_token_ms": 17.79, "prompt_per_second": 56.20,
  "predicted_n": 2122, "predicted_ms": 85069.442,
  "predicted_per_token_ms": 40.08, "predicted_per_second": 24.94
}
```

but I found it more convenient to grab the timing information from the llama-server window logging:

```js
prompt eval time =     515.95 ms /    29 tokens (   17.79 ms/tok,    56.21 tok/s)
       eval time =   85069.44 ms /  2122 tokens (   40.09 ms/tok,    24.94 tok/s)
      total time =   85585.39 ms /  2151 tokens
```

So: ~25 tokens/sec generation, on a card that has no business running a 35B model!

Here’s a more realistic test query for an assistant-style workload (this is the one I keep re-running for comparable timings):

```bash
curl -X POST http://localhost:8080/v1/chat/completions \
     -H "Content-Type: application/json" \
     -d '{
       "messages": [
         {"role": "system", "content": "You are a helpful assistant."},
         {"role": "user", "content": "Please write a program to stream the Fibonacci numbers under 1000 - with the restriction that there should be only one print statement in the loop"}
       ]
     }'
```

## Qwen 35B-A3B with TurboQuant KV cache

Add `--cache-type-k turbo4 --cache-type-v turbo3` and bump the context:

```bash
./llama-server \
    --model ~/Models/Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf \
\
\
    --no-mmap --mlock \
    --cache-type-k turbo4 --cache-type-v turbo3 --ctx-size 64000
```

```js
-ctk,  --cache-type-k TYPE              KV cache data type for K
                                        allowed values: f32, f16, bf16, q8_0, q4_0,
                                        q4_1, iq4_nl, q5_0, q5_1, turbo2, turbo3,
                                        turbo4   (default: f16)
-ctv,  --cache-type-v TYPE              (same allowed values)   (default: f16)
-c,    --ctx-size N                     size of the prompt context
                                        (default: 0, 0 = loaded from model)
```

Result:

```js
prompt eval time =     497.54 ms /    29 tokens (   17.16 ms/tok,    58.29 tok/s)
       eval time =   87924.70 ms /  2052 tokens (   42.85 ms/tok,    23.34 tok/s)
      total time =   88422.24 ms /  2081 tokens
```

Same model running with `--mmap` (i.e. without `--mlock`) for comparison:

```js
prompt eval time =     500.95 ms /    29 tokens (   17.27 ms/tok,    57.89 tok/s)
       eval time =  102627.30 ms /  2367 tokens (   43.36 ms/tok,    23.06 tok/s)
      total time =  103128.25 ms /  2396 tokens
```

Pushing the context out to 128k, while bumping more layers onto the CPU (`--n-cpu-moe 35` vs `30`) to free up memory for the context (expected to be slower, since more weights now travel over PCIe per token):

```bash
./llama-server \
    --model ~/Models/Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf \
\
\
    --no-mmap --mlock \
    --cache-type-k turbo4 --cache-type-v turbo3 --ctx-size 128000
```

```js
prompt eval time =     539.69 ms /    29 tokens (   18.61 ms/tok,    53.73 tok/s)
       eval time =  110986.98 ms /  2396 tokens (   46.32 ms/tok,    21.59 tok/s)
      total time =  111526.67 ms /  2425 tokens
```

Now, hone the number of CPU MoE layers:

**WORKING Qwen3.6-35B-A3B**

```bash
./llama-server \
    --model ~/Models/Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf \
\
\
    --no-mmap --mlock \
    --cache-type-k turbo4 --cache-type-v turbo3 --ctx-size 128000
```

```js
prompt eval time =     530.27 ms /    29 tokens (   18.29 ms/tok,    54.69 tok/s)
       eval time =   91362.24 ms /  2203 tokens (   41.47 ms/tok,    24.11 tok/s)
      total time =   91892.52 ms /  2232 tokens
```

Since `--n-cpu-moe 29` gives an Out-Of-Memory (OOM) error, 30 CPU-MoE layers is the sweet spot for Qwen 35B-A3B at 128k context on this card (this was actually a lucky initial guess).

## Fixing --mlock permission warnings (not essential)

`--mlock` was throwing warnings at startup (it still loaded, but couldn’t lock pages in RAM, which silently undoes most of the point of using it). Clearly, the default `ulimits` on a regular user account were too low for a 24 GiB model.

Using a root account:

```bash
ulimit -a   # See all the limits in operation

# 24 GiB = 25165824 KiB

# Bump the limit for the current shell (as root):
ulimit -l 25165824

# Make it persistent (as root, edit limits.conf):
joe /etc/security/limits.conf
# Add at the bottom:
#   your_user_name        -       memlock         25165824
```

After re-login (or doing a new `ssh` into the box), `--mlock` stops complaining.

In practice, though, this turned out to be a non-event: timings *with* a working `--mlock` weren’t measurably different from timings without it. A couple of other write-ups I came across suggest `--mlock` is mostly relevant in containerised setups (where the host can otherwise reclaim pages out from under the container). On bare-metal Linux with plenty of free RAM, the default mmap behaviour already keeps the hot pages resident. I left the `limits.conf` change in place anyway, but it’s not where the speedups come from.

## Gemma 4 26B-A4B

The other model I wanted to run is Gemma 4 26B-A4B, which is a similar shape (roughly 25B total / 3.8B active parameters, 30 layers, 256K trained context):

- [HF: gemma-4-26B-A4B-it](https://huggingface.co/google/gemma-4-26B-A4B-it#mixture-of-experts-moe-model)
- [HF: all quantised versions](https://huggingface.co/models?other=base_model:quantized:google/gemma-4-26B-A4B)

| Property | Value |
| --- | --- |
| Total parameters | 25.2B |
| Active parameters | 3.8B |
| Layers | 30 |
| Context length | 256K tokens |

Gemma 4 needs the *RotorQuant* extension (a Gemma-specific KV-cache scheme), which lives on a `llama.cpp` branch, and is incorporated in the Atomic fork we’ve been working with already.

The GGUFs can be found here: [AtomicChat GGUFs](https://huggingface.co/AtomicChat/gemma-4-26B-A4B-it-assistant-GGUF)

There’s also an MTP (Multi-Token Prediction) “assistant” head that pairs with the main model for speculative decoding:

- [google/gemma-4-26B-A4B-it-assistant](https://huggingface.co/google/gemma-4-26B-A4B-it-assistant)

Get the Gemma 4 models. Either via `huggingface-cli` …

```bash
hf download AtomicChat/gemma-4-26B-A4B-it-assistant-GGUF \
    --include "*Q4_K_M.gguf" --local-dir ./Models

# Any GGUF build of the matching target model works; e.g. unsloth's:
hf download unsloth/gemma-4-26B-A4B-it-GGUF \
    --include "*Q4_K_M*.gguf" --local-dir ./Models
```

… or just `wget` (and move them into `~/Models/`):

```bash
wget https://huggingface.co/AtomicChat/gemma-4-26B-A4B-it-assistant-GGUF/resolve/main/gemma-4-26B-A4B-it-assistant.Q4_K_M.gguf
wget https://huggingface.co/ggml-org/gemma-4-26B-A4B-it-GGUF/resolve/main/gemma-4-26B-A4B-it-Q4_K_M.gguf
```

### Gemma 4 26B-A4B: baseline runs (no MTP)

```bash
cd ./build/bin
./llama-server \
    --model ~/Models/gemma-4-26B-A4B-it-Q4_K_M.gguf \
\
\
    --cache-type-k turbo3 --cache-type-v turbo3 \
    --flash-attn on \
    --no-mmap --mlock \
    --ctx-size 16384
```

```js
prompt eval time =    1035.77 ms /    52 tokens (   19.92 ms/tok,    50.20 tok/s)
       eval time =   67336.21 ms /  1076 tokens (   62.58 ms/tok,    15.98 tok/s)
      total time =   68371.98 ms /  1128 tokens
```

Pushing context out to 128k:

```bash
./llama-server \
    --model ~/Models/gemma-4-26B-A4B-it-Q4_K_M.gguf \
\
\
    --cache-type-k turbo3 --cache-type-v turbo3 \
    --flash-attn on \
    --no-mmap --mlock \
    --ctx-size 128000
```

```js
prompt eval time =     985.96 ms /    52 tokens (   18.96 ms/tok,    52.74 tok/s)
       eval time =   98309.88 ms /  1538 tokens (   63.92 ms/tok,    15.64 tok/s)
      total time =   99295.84 ms /  1590 tokens
```

The memory breakdown is informative (this is what the server prints at startup):

```js
llama_memory_breakdown_print: | memory breakdown [MiB] | total   free     self   model   context   compute    unaccounted |
llama_memory_breakdown_print: |   - CUDA0 (GTX 1080)   |  8107 = 4632 + ( 3299 =  2103 +     664 +     532) +         174 |
llama_memory_breakdown_print: |   - Host               |                 14747 = 14477 +       0 +     270                |
```

i.e. roughly 3.3 GiB on the GPU (model + context + compute scratch), 14.7 GiB on the host. There’s a comfortable ~4.6 GiB free on the GPU, which suggests more layers could come over.

Trying `--n-cpu-moe 20`:

```bash
./llama-server \
    --model ~/Models/gemma-4-26B-A4B-it-Q4_K_M.gguf \
\
\
    --cache-type-k turbo3 --cache-type-v turbo3 \
    --flash-attn on \
    --no-mmap --mlock \
    --ctx-size 128000
```

```js
prompt eval time =     584.25 ms /    29 tokens (   20.15 ms/tok,    49.64 tok/s)
       eval time =   88887.26 ms /  1758 tokens (   50.56 ms/tok,    19.78 tok/s)
      total time =   89471.51 ms /  1787 tokens
```

At `--n-cpu-moe 19` it OOMs. So `20` is the floor for Gemma at 128k context.

### Gemma 4 26B-A4B with MTP speculative decoding

The interesting bit: pair the main model with its small “assistant” MTP head and let `llama.cpp` do speculative decoding:

```bash
./llama-server \
    --model ~/Models/gemma-4-26B-A4B-it-Q4_K_M.gguf \
\
\
    --mtp-head ~/Models/gemma-4-26B-A4B-it-assistant.Q4_K_M.gguf \
\
    --spec-type mtp \
    --draft-block-size 3 --draft-max 8 --draft-min 0 \
    --cache-type-k turbo3 --cache-type-v turbo3 \
    --cache-type-k-draft turbo3 --cache-type-v-draft turbo3 \
    --flash-attn on \
    --no-mmap --mlock \
    --ctx-size 128000
```

```js
prompt eval time =    1041.25 ms /    52 tokens (   20.02 ms/tok,    49.94 tok/s)
       eval time =  102274.86 ms /  1763 tokens (   58.01 ms/tok,    17.24 tok/s)
      total time =  103316.10 ms /  1815 tokens
```

Bringing one more main layer onto GPU (`--n-cpu-moe 20`) so the draft model also fits:

```bash
./llama-server \
    --model ~/Models/gemma-4-26B-A4B-it-Q4_K_M.gguf \
\
\
    --cache-type-k turbo3 --cache-type-v turbo3 \
    --mtp-head ~/Models/gemma-4-26B-A4B-it-assistant.Q4_K_M.gguf \
\
    --spec-type mtp \
    --draft-block-size 3 --draft-max 16 --draft-min 0 \
    --cache-type-k-draft turbo3 --cache-type-v-draft turbo3 \
    --flash-attn on \
    --no-mmap --mlock \
    --ctx-size 128000
```

```js
prompt eval time =     773.84 ms /    52 tokens (   14.88 ms/tok,    67.20 tok/s)
       eval time =   62810.70 ms /  1316 tokens (   47.73 ms/tok,    20.95 tok/s)
      total time =   63584.54 ms /  1368 tokens
draft acceptance rate = 0.76200 (  794 accepted /  1042 generated) (--draft-max 8)
```

A second run, same config (just to get a sense of variance):

```js
prompt eval time =     791.67 ms /    52 tokens (   15.22 ms/tok,    65.68 tok/s)
       eval time =   61002.84 ms /  1221 tokens (   49.96 ms/tok,    20.02 tok/s)
      total time =   61794.51 ms /  1273 tokens
draft acceptance rate = 0.71514 (  718 accepted /  1004 generated) (--draft-max 8)
```

Cranking the draft window up to `--draft-max 16`:

```js
prompt eval time =     770.59 ms /    52 tokens (   14.82 ms/tok,    67.48 tok/s)
       eval time =   55049.45 ms /  1151 tokens (   47.83 ms/tok,    20.91 tok/s)
      total time =   55820.04 ms /  1203 tokens
draft acceptance rate = 0.76096 (  694 accepted /   912 generated)  (--draft-max 16)
```

And with the longer “explain how a server query works” prompt (29 tokens of prompt instead of 52):

```js
prompt eval time =     624.66 ms /    29 tokens (   21.54 ms/tok,    46.43 tok/s)
       eval time =   85882.06 ms /  1651 tokens (   52.02 ms/tok,    19.22 tok/s)
      total time =   86506.72 ms /  1680 tokens
draft acceptance rate = 0.69220 (  958 accepted /  1384 generated)
```

Trying to push one more layer onto the GPU (`--n-cpu-moe 19`) while keeping MTP gives an Out-Of-Memory (OOM).

So with the MTP head also resident, `--n-cpu-moe 20` is the floor at 128k context.

However: This doesn’t seem like a huge MTP win…

#### So where is the MTP head? (skip this section if you’re in a hurry)

The whole premise of the MTP win is that the assistant head is small enough to fit entirely on the GPU — so generating draft tokens with it should be free of PCIe traffic, and the only PCIe cost is the target model’s MoE streaming during the verification pass. If that premise holds, given the bandwidth-limited analysis above, MTP ought to be a bigger win than 20 -> 21 tok/s.

To discover what’s going on, Claude & I dived into the MTP code in the fork itself…

The startup `llama_memory_breakdown_print` line doesn’t help directly here — digging into `src/llama-model.cpp` in the atomic fork:

```cpp
std::map<ggml_backend_buffer_type_t, size_t> llama_model::memory_breakdown() const {
    std::map<ggml_backend_buffer_type_t, size_t> ret;
    for (const auto & [ctx, bufs] : pimpl->ctxs_bufs) {
        // ... walks the target model's buffers only ...
    }
    return ret;
}
```

It walks `pimpl->ctxs_bufs` for the target model only — it does *not* recurse into `tgt->mtp_assistant`, which is loaded as a separate `llama_model` and attached to the target afterwards (see `llama_model_load_mtp_from_file` in `src/llama.cpp`). So the breakdown line we get from `llama-server` startup — the one that said `CUDA0 (GTX 1080) = 4632 free + (3299 used) + 174 unaccounted` — is target-only. Whatever the MTP head consumed is being subtracted from that 4.6 GiB “free” silently.

The better diagnostic is the per-model `load_tensors:` stanza, which fires at INFO level for *every* model load (target and assistant). In `src/llama-model.cpp`:

```cpp
LLAMA_LOG_INFO("%s: offloaded %d/%d layers to GPU\n", ...);
// then, per buffer:
LLAMA_LOG_INFO("%s: %12s model buffer size = %8.2f MiB\n", ...);
```

So in the server’s startup log there are **two** `load_tensors:` stanzas — one for the target Gemma 4, one for the assistant. Here they are, side by side:

```js
# Target Gemma 4 26B-A4B (31 layers):
load_tensors: offloading output layer to GPU
load_tensors: offloading 29 repeating layers to GPU
load_tensors: offloaded 31/31 layers to GPU
load_tensors:          CPU model buffer size =   577.50 MiB
load_tensors:        CUDA0 model buffer size =  6504.39 MiB
load_tensors:    CUDA_Host model buffer size =  9498.51 MiB

# MTP assistant (4 transformer blocks + LM head = 5 layers):
load_tensors: offloading output layer to GPU
load_tensors: offloading 3 repeating layers to GPU
load_tensors: offloaded 5/5 layers to GPU
load_tensors:          CPU model buffer size =   210.00 MiB
load_tensors:        CUDA0 model buffer size =    82.24 MiB
load_tensors:    CUDA_Host model buffer size =     3.09 MiB
```

The target stanza is unsurprising: 6.5 GiB on `CUDA0` (the dense layers and the active slice of the MoE experts), 9.5 GiB in `CUDA_Host` (pinned host RAM holding the `--n-cpu-moe 20` worth of MoE experts that stream over PCIe), 577 MiB in plain `CPU`. The total adds up to ~16.6 GiB, about right for Q4\_K\_M of a 25B model.

The **assistant stanza is the surprise**. Only 82 MiB on `CUDA0`, while **210 MiB — more than twice as much — is sitting on plain CPU**. That’s not what you want at all.

The reason is in `src/llama-model.cpp` at line ~3067:

```cpp
// assign the input layer
// there is very little benefit to offloading the input layer, so always keep it on the CPU
pimpl->dev_input = { cpu_dev, &pimpl->cpu_buft_list };
```

The token embedding table is *unconditionally* pinned to the CPU, regardless of `--n-gpu-layers`. For a regular language model this is fine — the embedding lookup is `get_rows`, which pulls a handful of vocab rows per forward pass, and the table just sits cold on the CPU doing nothing.

But for the Gemma 4 26B-A4B assistant, the LM head is **tied to the token embedding table** (per the assistant tensor inventory in `docs/development/gemma4-assistant-tensor-inventory.md`: “26B-A4B / 31B (dense tied LM head)”). That means every MTP draft step does a full `mul_mat(mtp.tok_embd, h)` against the same table — and from `src/models/gemma4-assistant.cpp`:

```cpp
// dense tied LM head:
cur = gctx.build_lora_mm(mtp.tok_embd, h_inner);
```

So `mtp.tok_embd` is being matmul’d into per-step, but it lives on the CPU. A `262144 × 1024` table at Q4\_K\_M is roughly 150 MiB of weights that get hauled across PCIe **for every draft token generated**. Plus the same is true of the target’s own `tok_embd` — referenced at line 70 of the same file as `target.tok_embd` for the embedding lookup of the just-generated token, which is just a `get_rows` and cheap. But the LM head matmul against the assistant’s own tied embedding table is the killer.

That neatly explains why MTP barely moves the needle on this card: the supposed-to-be-on-GPU draft model is hauling ~150 MiB of weights over PCIe per draft token, on top of the target’s MoE streaming during verification. The bandwidth-limited bottleneck didn’t go away — MTP just added more PCIe traffic.

#### Getting the embedding table onto the GPU: first attempt (didn’t work)

llama.cpp has an `--override-tensor` flag that lets you force a specific tensor onto a chosen buffer type. The obvious-looking invocation:

```bash
./llama-server \
    ...  # all the usual flags
    --override-tensor "mtp\\.tok_embd=CUDA0"
```

But: this doesn’t change the assistant’s `load_tensors` output at all. The second stanza is still 82 MiB CUDA0 / 210 MiB CPU. Two reasons, both worth knowing about:

**1\. Wrong flag for the draft model.** In `common/arg.cpp` there are two parallel flags:

```cpp
{"-ot",  "--override-tensor"},        // pushes into params.tensor_buft_overrides
{"-otd", "--override-tensor-draft"},  // pushes into params.speculative.tensor_buft_overrides
```

`--override-tensor` only touches the target model; the assistant gets its overrides from `params.speculative.tensor_buft_overrides`, populated by `--override-tensor-draft` (or `-otd`). This mirrors the `--cpu-moe` / `--cpu-moe-draft` and `--n-cpu-moe` / `--n-cpu-moe-draft` split.

**2\. Wrong tensor name pattern.** The C++ struct field is `mtp.tok_embd`, but the *tensor name on disk* (which is what the override regex matches against) is `token_embd.weight`. That comes from `tn(LLM_TENSOR_TOKEN_EMBD, "weight")` in `src/llama-model.cpp` line 4747 (in the `LLM_ARCH_GEMMA4_ASSISTANT` case), and `LLM_TENSOR_TOKEN_EMBD` maps to the string `"token_embd"` in `src/llama-arch.cpp` line 350. The override matcher in `src/llama-model-loader.cpp` line 1163 uses `std::regex_search` against this full name, so a simple substring pattern is fine.

There’s a debug log line at the same spot:

```cpp
LLAMA_LOG_DEBUG("tensor %s (%zu MiB %s) buffer type overridden to %s\n", ...);
```

So running with `--verbose` (which enables `LLAMA_LOG_DEBUG`) will print each override that fires — a good way to confirm a pattern is matching before reading the `load_tensors:` summary.

#### Getting the embedding table onto the GPU: second attempt (this worked)

The corrected invocation uses `--override-tensor-draft` and the actual on-disk tensor name. Note that moving 210 MiB into VRAM shrinks the headroom, so `--n-cpu-moe 20` now OOMs - so I increased it until the server started:

**WORKING gemma-4-26B-A4B WITH MTP**

```bash
./llama-server \
    --model ~/Models/gemma-4-26B-A4B-it-Q4_K_M.gguf \
    --mtp-head ~/Models/gemma-4-26B-A4B-it-assistant.Q4_K_M.gguf \
    --spec-type mtp \
    --draft-block-size 3 --draft-max 16 --draft-min 0 \
\
    --n-cpu-moe 21 \            # 20 = OOM; 21 is the floor and the sweet spot
\
\
    --override-tensor-draft "token_embd\.weight=CUDA0" \
    --cache-type-k turbo3 --cache-type-v turbo3 \
    --cache-type-k-draft turbo3 --cache-type-v-draft turbo3 \
    --flash-attn on \
    --no-mmap --mlock \
    --ctx-size 128000
;   --verbose    # add when verifying placement, too noisy during generation
```

With `--verbose`, confirming the override fired:

```js
tensor token_embd.weight (210 MiB q6_K) buffer type overridden to CUDA0
```

And the second `load_tensors:` stanza confirming the move:

```js
load_tensors: offloaded 5/5 layers to GPU
load_tensors:        CUDA0 model buffer size =   292.24 MiB   # was 82 MiB
load_tensors:    CUDA_Host model buffer size =     3.09 MiB
                                                              # CPU line gone
```

82 + 210 = 292. The CPU buffer has disappeared entirely.

Two minor warnings in the log, both benign:

```js
done_getting_tensors: tensor 'token_embd.weight' (q6_K) (and 0 others) cannot be used
    with preferred buffer type CUDA_Host, using CUDA0 instead
```

The loader noted `q6_K` is incompatible with `CUDA_Host` (pinned host memory) and fell back to `CUDA0` — which is exactly what we wanted.

```js
load_all_data: buffer type CUDA_Host is not the default buffer type for device CUDA0
    for async uploads
```

The 3 MiB of `CUDA_Host` content (probably `rope_freqs.weight`) uploads synchronously rather than via async DMA. No meaningful impact.

### Timing results with the MTP Embedding table on the GPU

Sweeping `--n-cpu-moe` from the OOM floor upwards (more CPU layers = less VRAM pressure but more PCIe traffic per target token):

**`--n-cpu-moe 25` (conservative):**

```js
prompt eval time =     938.64 ms /    52 tokens (   18.05 ms/tok,    55.40 tok/s)
       eval time =   66764.55 ms /  1389 tokens (   48.07 ms/tok,    20.80 tok/s)
      total time =   67703.18 ms /  1441 tokens
draft acceptance rate = 0.74150 (829 accepted / 1118 generated)
statistics mtp: #calls(b,g,a) = 1 559 449  dur(b,g,a) = 0.008, 4229.426, 0.087 ms
```

**`--n-cpu-moe 22`:**

```js
prompt eval time =     828.46 ms /    52 tokens (   15.93 ms/tok,    62.77 tok/s)
       eval time =   41976.54 ms /  1025 tokens (   40.95 ms/tok,    24.42 tok/s)
      total time =   42805.00 ms /  1077 tokens
draft acceptance rate = 0.82300 (637 accepted / 774 generated)
statistics mtp: #calls(b,g,a) = 1 387 340  dur(b,g,a) = 0.004, 2631.373, 0.062 ms
```

**`--n-cpu-moe 21` (OOM floor, sweet spot):**

```js
prompt eval time =     799.35 ms /    52 tokens (   15.37 ms/tok,    65.05 tok/s)
       eval time =   47635.24 ms /  1166 tokens (   40.85 ms/tok,    24.48 tok/s)
      total time =   48434.59 ms /  1218 tokens
draft acceptance rate = 0.78587 (712 accepted / 906 generated)
statistics mtp: #calls(b,g,a) = 1 453 389  dur(b,g,a) = 0.004, 3048.331, 0.086 ms
```

*(20 = OOM.)*

`--n-cpu-moe 21` is the tok/s sweet spot at ~24.5 tok/s. The big win is the jump from 25 → 21 (20.80 → 24.48 tok/s); the difference between 22 and 21 is small and the acceptance-rate variation between those two runs is likely noise (different response lengths, single-run variance) rather than a real signal.

The `mtp statistics` lines are revealing. The `dur(b,g,a)` tuple is time spent in each MTP phase: *batch* (prefill), *generation* (drafting), *acceptance* (verification). At `n-cpu-moe 21`: generation takes 3048 ms total over 453 calls (~6.7 ms per draft call), acceptance takes 0.086 ms total. That’s the correct shape for a GPU-resident draft model: generation is CUDA-compute-bound, not PCIe-bound, and acceptance is essentially free.

Compare with the previous runs where the embedding table was on the CPU: generation duration was in the same ballpark (~3000-4000 ms) but with the same or fewer draft calls — meaning each draft call was individually slower. Once the matmul moved to CUDA0, per-call duration dropped and the acceptance rate improved.

**Compared to the no-MTP baseline** (Gemma 4 at `--n-cpu-moe 20`, ~20 tok/s): with MTP properly on the GPU we get **~24.5 tok/s** — a real ~22% improvement, vs the 5% we saw when the embedding table was stranded on the CPU. The speculative decoding is now actually earning its keep.

## Summary of what worked, and what to make of the numbers

Working configurations (all on a single 8 GiB GTX 1080, 128k context):

- **Qwen 3.6 35B-A3B Q4\_K\_M**, `--n-cpu-moe 30`, TurboQuant K=turbo4 V=turbo3: ~24 tok/s generation.
- **Gemma 4 26B-A4B Q4\_K\_M**, `--n-cpu-moe 20`, RotorQuant K=V=turbo3, `--flash-attn on`, no MTP: ~20 tok/s generation.
- **Gemma 4 26B-A4B + MTP assistant, naively configured** (`--n-cpu-moe 20`, `--override-tensor-draft` omitted): ~21 tok/s, ~74% acceptance. The embedding table was silently on the CPU; MTP barely helped.
- **Gemma 4 26B-A4B + MTP assistant, correctly configured** (`--n-cpu-moe 21`, `--override-tensor-draft "token_embd\.weight=CUDA0"`): **~24.5 tok/s, ~79% acceptance**. A real ~22% improvement over the no-MTP baseline.

**MTP works — but only once you force the embedding table onto the GPU.** The key lesson from the debugging: `-ngld 999` is not enough. llama.cpp has a blanket policy of keeping the token embedding table on the CPU, on the grounds that a `get_rows` lookup is cheap and doesn’t need GPU memory. For the Gemma 4 26B-A4B assistant this assumption is violated: the LM head is tied to `token_embd.weight`, so every draft token does a full 262144 × 1024 matmul against that table. At Q4\_K\_M that’s ~150 MiB of weights hauled across PCIe per draft step — which on this bandwidth-saturated card ate almost all of the speculative-decoding win.

The fix is `--override-tensor-draft "token_embd\.weight=CUDA0"`. Once the matmul runs on CUDA0 the draft generation time drops and the acceptance rate improves.

The cost: 210 MiB more in VRAM, which forces `--n-cpu-moe` up from 20 to 21 (two more target-MoE layers stream over PCIe per target token). Net result is still positive — the reduction in draft-step PCIe cost outweighs the slight increase in target-step cost. 21 is now both the OOM floor and the tok/s sweet spot.

**How to tell whether your MTP head is really on the GPU.** The startup `llama_memory_breakdown_print` line is *not* reliable — it covers the target model only, not the assistant. The correct check is the second `load_tensors:` stanza in the startup log. What you want to see:

```js
# Good - no CPU line:
load_tensors:        CUDA0 model buffer size =   292.24 MiB
load_tensors:    CUDA_Host model buffer size =     3.09 MiB

# Bad - embedding table is on the CPU, MTP won't benefit:
load_tensors:          CPU model buffer size =   210.00 MiB
load_tensors:        CUDA0 model buffer size =    82.24 MiB
load_tensors:    CUDA_Host model buffer size =     3.09 MiB
```

If the CPU line is non-zero for the assistant, check whether the model has a tied LM head (as the 26B-A4B does) and add `--override-tensor-draft "token_embd\.weight=CUDA0"`. The `mtp statistics` line in the generation output also tells you whether drafting is on the GPU: `dur(b,g,a)` generation time should be a few milliseconds per call, not tens.

NOTE: **Qwen’s tok/s lead over Gemma has disappeared.** Once MTP is properly configured, Gemma 4 26B-A4B + MTP (~24.5 tok/s) and Qwen 3.6 35B-A3B (~24 tok/s) are essentially neck-and-neck on raw throughput. But Qwen generates substantially more tokens for the same prompt: it’s a more verbose model. End-to-end wall-clock time is dominated by output length, not raw tok/s. So, pick the model based on the answers you want, not the throughput number.

---

## Bottom-Line: The working code (fastest found so far)

**WORKING Qwen3.6-35B-A3B**

```bash
./llama-server \
    --model ~/Models/Qwen_Qwen3.6-35B-A3B-Q4_K_M.gguf \
\
\
    --no-mmap --mlock \
    --cache-type-k turbo4 --cache-type-v turbo3 --ctx-size 128000
```

**WORKING gemma-4-26B-A4B WITH MTP**

```bash
./llama-server \
    --model ~/Models/gemma-4-26B-A4B-it-Q4_K_M.gguf \
\
\
    --mtp-head ~/Models/gemma-4-26B-A4B-it-assistant.Q4_K_M.gguf \
\
\
    --override-tensor-draft "token_embd\.weight=CUDA0" \
    --spec-type mtp \
    --draft-block-size 3 --draft-max 16 --draft-min 0 \
    --cache-type-k turbo3 --cache-type-v turbo3 \
    --cache-type-k-draft turbo3 --cache-type-v-draft turbo3 \
    --flash-attn on \
    --no-mmap --mlock \
    --ctx-size 128000
```

---

All done.

---

## TODOs: More flags to play with

The man-text file dropped in [earlier](#keep-a-local-copy-of-llama-server-help) has plenty of useful flags I haven’t tried yet: `--numa`, `--override-tensor`, `--split-mode`, the various `fim-qwen-*` shortcuts, the speculative-decoding ngram variants, the various `prio` / `cpu-mask` / `cpu-range` options for nailing threads to specific cores, and so on. One for a future post.