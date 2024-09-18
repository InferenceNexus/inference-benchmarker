# Text Generation Inference benchmarking tool

A lightweight benchmarking tool for LLM inference servers.
Benchmarks using constant arrival rate or constant virtual user count.



![ui.png](assets%2Fui.png)

## Table of contents

<!-- TOC -->
* [Text Generation Inference benchmarking tool](#text-generation-inference-benchmarking-tool)
  * [Table of contents](#table-of-contents)
  * [TODO](#todo)
  * [Running a benchmark](#running-a-benchmark)
  * [Development](#development)
  * [Frequently Asked Questions](#frequently-asked-questions)
<!-- TOC -->

## TODO
- [X] Customizable token count and variance
- [ ] Check results
- [X] Allow for system prompts for prefix caching
- [ ] Allow for multi-turn prompts
- [ ] Push results to Optimum benchmark backend
- [X] Script to generate plots from results


## Get started

### Run a benchmark

Run a benchmark using Docker image:

```shell
# start a TGI/vLLM server somewhere, then run benchmark...
# ... we mount results to the current directory
$ docker run \
    --rm \
    -it \
    --net host \
    -v $(pwd):/opt/text-generation-inference-benchmark/results \
    registry.internal.huggingface.tech/api-inference/text-generation-inference-benchmark:latest \
    text-generation-inference-benchmark \
    --tokenizer-name "Qwen/Qwen2-7B" \
    --max-vus 800 \
    --url http:/localhost:8080 \
    --warmup 20s \
    --num-rates 10 \
    --prompt-options "num_tokens=50,max_tokens=60,min_tokens=40,variance=10" \
    --decode-options "num_tokens=50,max_tokens=60,min_tokens=40,variance=10"
```

Results will be saved in `results.json` in current directory.


### Configure your benchmark

#### Benchmark mode

In default mode, tool runs a `sweep` benchmark. It first runs a throughput test to find the maximum throughput, then
sweeps on QPS values up to the maximum throughput.

Available modes:
- `sweep`: runs a sweep benchmark
- `rate`: runs a benchmark at a fixed request rate
- `throughput`: runs a benchmark at a fixed throughput (constant VUs)


#### Dataset configuration

Prompts are sampled for a Hugging Face dataset file, using a [subset of ShareGPT
as default](https://huggingface.co/datasets/hlarcher/share_gpt_small). You can specify a different dataset file using the
`--dataset` and `--dataset-file` option.

Dataset is expected to be JSON with the following format:
```json
[
  {
    "conversations": [
      {
        "role": "user",
        "content": "rewrite that entire paragraph in the same style like this one: "
      }
    ]
  }
]
```

To benchmark with prefix caching, you can use a system prompt that will be sent with each request from a discussion.
```json
[
  {
    "conversations": [
      {
        "role": "system",
        "content": "You are a helpful assistant that makes jokes at each response."
      },
      {
        "role": "user",
        "content": "rewrite that entire paragraph in the same style like this one:"
      }
    ]
  }
]
```


#### Prompt configuration
For consistent results you can configure the token count and variance. The tool will sample prompts with the specified
values, sampling token counts from a normal distribution with the specified variance.

```shell
--prompt-options "num_tokens=50,max_tokens=60,min_tokens=40,variance=10"
```


## Development

You need [Rust](https://rustup.rs/) installed to build the benchmarking tool.
```shell
$ make build
```


## Frequently Asked Questions
* **What's the difference between constant arrival rate and constant virtual user count?**
  * **Constant virtual user count** means that the number of virtual users is fixed. Each virtual user can send a single requests and waits for server response. It's basically simulating a fixed number of users querying the server.
  * **Constant arrival rate** means that the rate of requests is fixed and the number of virtual users is adjusted to maintain that rate. Queries hit the server independently of responses performances.

  **Constant virtual user count** is a closed loop model where the server's response time dictates the number of iterations. **Constant arrival rate** is an open-loop model more representative of real-life workloads.
