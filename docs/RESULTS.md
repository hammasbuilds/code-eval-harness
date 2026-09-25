# Results

[<- back to README](../README.md)

All 164 HumanEval problems, greedy decoding (temperature 0, fixed seed), one generation per
(model, problem), cached on disk.

## pass@1 by model and extraction strategy

| Model | `raw` | `prompt+body` | `first_fence` | `all_fences` | `smart` |
|---|---:|---:|---:|---:|---:|
| qwen2.5-coder:14b | **0.0%** | **0.0%** | 85.4% | 85.4% | 85.4% |
| qwen2.5:14b-instruct | **0.0%** | **0.0%** | 77.3% | 77.3% | 77.3% |

**Spread: 94 percentage points**, with the generations held identical.

### The 14B changed nothing, and that is the result

`qwen2.5-coder:14b` is **4.8x the parameters of the 3B** and scores **exactly the same**
under every strategy. The extraction method moves the number by 94 points; going from 3B
to 14B moves it by zero.

The aggregate hides that the models are not behaving identically. They fail *different*
problems:

| Model | failed (under `smart`) |
|---|---|
| qwen2.5-coder:3b | HumanEval/26, HumanEval/32, HumanEval/38 |
| qwen2.5:7b-instruct | HumanEval/19, HumanEval/26, HumanEval/32 |
| qwen2.5-coder:14b | HumanEval/10, HumanEval/32, HumanEval/38 |

Only `HumanEval/32` defeats all three. The generations differ in length and content -
the models genuinely disagree - and a single pass@1 number is insensitive to all of it.

## Why the two zeros are one finding

| Strategy | Failures | Cause |
|---|---:|---|
| `raw` | 150 / 150 | `SyntaxError` |
| `prompt+body` | 150 / 150 | `SyntaxError` |

Every single failure in both strategies is a syntax error, and the cause is the same: the
models wrap their answers in ```` ```python ```` fences, and backticks are not Python.

So the table shows **one finding with two faces**, not five independent results:

1. **`raw`** - using the output verbatim fails whenever the model writes markdown.
2. **`prompt+body`** - *the original HumanEval protocol.* It treats the output as a
   continuation of the function signature, which is correct for **completion** models. A
   chat-tuned model restates the whole function inside a fence, so the concatenation is
   invalid Python.

The second is the substantive point. A harness written for completion models scores modern
chat models at zero - not because they cannot code, but because the protocol assumes an
output shape they no longer produce. **The failure is silent: it looks like a model result,
not a harness result.**

## Model comparison

Two models of the same size and family, differing only in tuning, over all 164 problems:

```
qwen2.5-coder:14b     85.4%
qwen2.5:14b-instruct  77.3%
```

**8.1 points for code tuning at 14B.** That is consistent with the +10.0 points the same
pair shows on MBPP in
[code-llm-lab](https://github.com/hammasbuilds/code-llm-lab/tree/main/projects/16_coder_vs_generalist).

An earlier version of this page compared a 3B, a 7B and a 14B over the first 50 problems
and reported all three at 94.0%, concluding that 2.3x the parameters bought nothing
measurable. That is withdrawn. Fifty problems was too small a denominator to separate them,
and three models spanning two families and three sizes is not a controlled comparison.

## Raw data

`results/scores.json` contains one record per (model, problem) with the pass/fail and the
failure reason under every strategy. Cached generations are in `generations/`, one JSON per
(model, problem), so any new strategy can be scored without re-running a model.

## Reproducing

```bash
python src/run_eval.py 50
streamlit run ui/app.py
```

Generations are cached, so a second run scores instantly.
