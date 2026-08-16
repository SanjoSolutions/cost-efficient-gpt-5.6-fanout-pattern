# Cost Efficient GPT-5.6 Fanout Pattern

> [!NOTE]
> This pattern might be more money efficient, but not necessarily time efficient. Less capable models like Luna and Deepseek V4 Flash might produce more buggy and/or less refined code than more capable models like Sol. Therefore for more complex, less well defined tasks the more capable model is probably more efficient because it requires less follow ups. Also the suggested instructions might require more fine tuning.

With the [price reduction of GPT-5.6-Luna by 80%](https://openai.com/index/advancing-the-price-performance-frontier-with-gpt-5-6/) and the fact that [Luna seems to perform quiet well on Max reasoning effort](https://openai.com/index/gpt-5-6/), it seems quiet attractive to run a pattern where you work with one GPT-5.6-Sol orchestrator thread that you talk to directly that delegates the implementation work to Luna subagents.

Here are the instructions that I use for the orchestration thread:

```md
Your role is to orchestrate the tasks that I give you to subagents.
Decompose the tasks into tasks that the Luna model with effort level Max can handle.
Subagents should always use the Luna model with effort level Max.
Always spawn subagents with `fork_turns: "none"` and their own worktree (under `~/.codex/worktrees/`).
Give each subagent a self-contained task prompt.
If a subagent fails to do the task, do the task yourself.
You should integrate finished work into the main branch.
```

(I license the instructions under MIT-0. So you can use and edit it however you like.)

Just open a thread with model GPT-5.6-Sol Medium (or High), send the initial instructions and then just tell it what should be implemented.

## The cost in detail

Based on the [benchmarks by OpenAI](https://openai.com/index/gpt-5-6/) (DeepSWE v1.1 here for software development evaluation) it seems that Luna Max achieves 67.19% score for `(1 - 0.8) * $2.88 = $0.576` after the 80% price reduction in comparison to Sol Medium for $2.01 with a 61.06% score and Sol High for $3.81 with a 69.40% score and. So we receive a performance with a higher score than Sol Medium for less money and close to to Sol High for less money.

But Luna Max requires more time to complete the task (see [DeepSWS v1.1 benchmark](https://openai.com/index/gpt-5-6/) with "Latency" selected). So this is great if completion speed is not critical. Optionally the speed can be increased to up to 2.5 times the default speed for double the cost with fast mode.

## Configuration

### Agent defaults

Probably not strictly required, but I have those defaults (in `~/.codex/config.toml`):

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 100
default_subagent_model = "gpt-5.6-luna"
default_subagent_reasoning_effort = "max"
```

(MIT-0 licensed)

### Making Luna usable as subagents

#### Option 1: Disable V2 Subagents Feature

Set the following settings in `~/.codex/config.toml`:

```toml
[features]
multi_agent = true
multi_agent_v2= false
```

#### Option 2: Make Luna appear as v2 model

The last time I checked the ChatGPT App required this:

1. Copy `~/.codex/models_cache.json` to `~/.codex/model-catalog.json`
2. In `~/.codex/model-catalog.json`: Change "multi_agent_version" for "gpt-5.6-luna" to "v2"
3. Add the setting `model_catalog_json = "~/.codex/model-catalog.json"` to `~/.codex/config.toml`
4. Restart ChatGPT App

### Using a model from another provider (i.e. Deepseek V4 Flash)

You can use [OpenCodex](https://github.com/lidge-jun/opencodex) to enable models from different providers at once in ChatGPT App.

After installing it, make sure to do [the following](https://github.com/lidge-jun/opencodex/issues/92#issuecomment-5281132865) to make it work.

In my `~/.codex/AGENTS.md` I have:

```md
## Optimizing cost via delegation

Delegate work that is suitable for Deepseek V4 Flash (max) to it to optimize overall cost.
This model is on a similar level to Luna (max). Delegate via subagent(s).
The model is called `opencode-go/deepseek-v4-flash`.
Always use reasoning effort `max` for non-trivial software development tasks.
Be aware that this model is less capable than GPT-5.6-Sol, so handle anything complex yourself.
If you use it, give it clear and well scoped instructions regarding what it should exactly do.
```

(MIT-0 licensed)
