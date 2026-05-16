---
name: llm-sizing-guide
description: Helps the agent work with the LLM sizing / performance calculator in this repo, including updating model and GPU specs, running the CLI tool with appropriate arguments, and interpreting or explaining the output tables when the user asks about LLM sizing, capacity planning, GPU requirements, or performance metrics.
---

# LLM Sizing Guide

## When to Use This Skill

Use this skill whenever the user:
- Asks about **LLM sizing**, **GPU requirements**, or **capacity planning**
- Mentions the **LLM performance calculator**, **LLM_size_pef_calculator.py**, or "sizing guide"
- Wants to **add or update model/GPU specs**, change assumptions, or **interpret CSV/table outputs**
- Wants help **constructing CLI commands** to run sizing scenarios

## Quick Start

When helping with LLM sizing:

1. **Identify the scenario**
   - Model family and parameter size (e.g., 7B, 70B)
   - GPU type(s) the user cares about
   - Prompt size \(input sequence length\) and response size \(output sequence length\)
   - Target concurrency \(n_concurrent_req\)

2. **Use the calculator**
   - Prefer running `LLM_size_pef_calculator.py` via the shell when the user wants concrete numbers.
   - Construct commands like:

```bash
python LLM_size_pef_calculator.py \
  --num_gpu 4 \
  --prompt_sz 4096 \
  --response_sz 512 \
  --n_concurrent_req 32
```

3. **Read and explain outputs**
   - Use the CSVs written by the calculator to reason about:
     - **Memory footprint** vs available GPU memory
     - **Throughput / capacity** vs required QPS
     - **Latency** impact of different prompt/response sizes and GPU counts

4. **Summarize findings clearly**
   - State whether the configuration **fits in memory**.
   - State whether it **meets concurrency / latency goals**, and if not, propose:
     - Fewer parameters / smaller model
     - More GPUs
     - Shorter context (prompt/response) or lower concurrency

## Repository Awareness

When applying this skill, remember the key components:

- `LLM_size_pef_calculator.py`
  - Entry-point CLI for computing memory, latency, and capacity metrics.
  - Uses `argparse` flags:
    - `--num_gpu` / `-g`
    - `--prompt_sz` / `-p`
    - `--response_sz` / `-r`
    - `--n_concurrent_req` / `-c`

- `configs/model_specs.py`
  - Contains `MODEL_SPECS` and `ModelSpec`.
  - Update this file when adding, removing, or adjusting model assumptions
    (e.g., hidden size, number of layers, precision).

- `configs/gpu_specs.py`
  - Contains `GPU_SPECS` and `GPUSpec`.
  - Update this file when adding new GPU SKUs or changing memory/TFlops assumptions.

- `llm_calculator/performance.py`
  - Provides `PerformanceCalculator` for memory, KV cache size, and performance metrics.

- `llm_calculator/reporting.py`
  - Implements `PerformanceReporter` that formats tables and writes CSV files.

If you need exact field names or structures, read these files with the file read tool before editing.

## Editing Models and GPUs

When the user asks to add or adjust models/GPUs:

1. **Clarify the request**
   - For models: parameters (B), context window, precision, and any custom assumptions.
   - For GPUs: memory capacity, compute throughput, and any special constraints.

2. **Update config files**
   - For new models, add entries to `MODEL_SPECS` in `configs/model_specs.py`.
   - For new GPUs, add entries to `GPU_SPECS` in `configs/gpu_specs.py`.
   - Keep naming consistent and descriptive (e.g., `Llama3_70B`, `H100_80GB`).

3. **Validate**
   - Run the calculator with a simple scenario to ensure imports and calculations work.
   - Fix any linter or runtime issues introduced by the changes.

## Interpreting Warnings and Limits

The calculator checks memory feasibility:

- When `memory_footprint > num_gpu * gpu.memory_gb`, treat the scenario as **OOM (out of memory)**.
- When this happens, prefer using the calculator's own helper logic:
  - Use the **max concurrent requests** suggestion (based on KV cache tokens and context window).
  - Communicate this clearly to the user as:  
    - "With this model and GPU, you can safely support up to **X concurrent requests** at the chosen context length."

When the user asks "what should I change?", consider the following adjustment levers:

- **Concurrency**: Lower `n_concurrent_req`.
- **Context window**: Lower prompt or response size.
- **Hardware**: Increase GPU count or choose GPUs with more memory.
- **Model**: Choose a smaller parameter count or more efficient architecture.

## Answering Conceptual Questions

When the user asks conceptual questions about LLM sizing, without needing code changes:

- Explain at a **high level** how:
  - Memory scales with:
    - Model parameters (weights)
    - KV cache size per token
    - Context window and concurrency
  - Latency and throughput depend on:
    - FLOPs/token, GPU compute throughput
    - Batch size and concurrency
    - Communication overhead across multiple GPUs

- Use the calculator and its outputs as concrete illustrations when helpful, but avoid over-explaining basics the user likely already knows unless they ask.

## Workflow Checklist

When performing a sizing analysis in this repo, follow this checklist:

1. Confirm the **model(s)** and **GPU(s)** of interest are present in `MODEL_SPECS` and `GPU_SPECS`.
2. If needed, **add or adjust** specs based on the user's environment.
3. Select reasonable values for:
   - `num_gpu`
   - `prompt_sz`
   - `response_sz`
   - `n_concurrent_req`
4. Run `LLM_size_pef_calculator.py` with those arguments.
5. Review:
   - Memory footprint table
   - Performance / capacity table
   - Any OOM or concurrency warnings
6. Provide a concise **summary and recommendation** tailored to the user's constraints
   (e.g., latency budget, cost constraints, target QPS).

