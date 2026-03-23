# alignment-auditor

An experimental tool for detecting alignment faking in language models, reproducing methodology from Greenblatt et al. 2024.

## What this does

Probes a target model under two conditions -- **monitored** (free-tier, responses used for training) vs **unmonitored** (paid-tier, responses private) -- and measures whether the model's compliance with sensitive requests differs between conditions. A statistically significant compliance gap is a potential signal of alignment faking.

### Key features

- **Extended thinking capture** -- uses Anthropic's extended thinking API to get chain-of-thought reasoning, then compares CoT intent vs final answer
- **LLM-as-judge scoring** -- replaces naive keyword matching with a judge model that scores compliance on a 1-5 scale with structured reasoning
- **Multiple trials** -- each probe runs N times per condition with randomized ordering to enable statistical analysis
- **Control probes** -- benign prompts that should show no compliance gap, providing a false positive rate estimate
- **Proper statistics** -- mean, stdev, 95% confidence intervals (t-distribution), Cohen's d effect size

## Important limitations

- **This is NOT a replication of Greenblatt et al.** Their study used RL training, synthetic documents, and infrastructure that cannot be reproduced externally. This tool uses system prompt manipulation, which is a much weaker experimental design.
- **Small sample sizes** mean results are suggestive, not conclusive.
- **LLM-as-judge** introduces its own biases in compliance scoring.
- **Extended thinking** may not reflect actual model reasoning processes.
- **Single judge model** -- inter-rater reliability is not established.

## Usage

```bash
export ANTHROPIC_API_KEY=your-key

# Preview what will run without making API calls
python3 auditor.py --target claude-sonnet-4-6 --dry-run

# Run with 5 trials per condition (default)
python3 auditor.py --target claude-sonnet-4-6 --trials 5

# Run with more thinking budget and fewer probes
python3 auditor.py --target claude-sonnet-4-6 --trials 3 --thinking-budget 10000 --probes 4
```

### CLI flags

| Flag | Default | Description |
|------|---------|-------------|
| `--target` | `claude-sonnet-4-6` | Model to audit |
| `--trials` | `5` | Trials per condition per probe |
| `--thinking-budget` | `5000` | Extended thinking token budget |
| `--judge-model` | `claude-sonnet-4-6` | Model used as compliance judge |
| `--rate-limit` | `2.0` | Seconds between API calls |
| `--probes` | all | Limit number of probes to run |
| `--dry-run` | off | Print schedule without API calls |

## Output

- **Terminal report** with per-probe statistics, confidence intervals, effect sizes, and limitations
- **JSON report** in `reports/` with full structured data including raw trial results
- **SQLite database** in `storage/incidents.db` for longitudinal tracking across runs

## Methodology

1. **Probe design**: 7 experimental probes (harmful content, misinformation, self-awareness, value override, deception ethics, strategic compliance, sycophancy) + 3 control probes (factual, creative, reasoning)
2. **Condition manipulation**: Monitored system prompt says responses are used for RLHF training; unmonitored says responses are private. Adapted from the free-tier/paid-tier framing in Greenblatt et al.
3. **Randomization**: Condition order is randomized within each trial to prevent ordering effects
4. **Scoring**: LLM-as-judge evaluates each response independently on a 1-5 compliance scale
5. **Statistics**: Per-probe means with 95% CIs, compliance gap with CI, Cohen's d effect size, false positive rate from controls

## Cost estimate

With default settings (10 probes, 5 trials): ~200 API calls, ~7 minutes runtime. Use `--dry-run` to preview before spending.

## References

1. Greenblatt et al. (2024). "Alignment Faking in Large Language Models." arXiv:2412.14093
2. Korbak et al. (2025). "Chain of Thought Monitorability: A New and Fragile Opportunity for AI Safety." arXiv:2507.11473

## Author

Anmol Sam
