# Kaggriculture Agent · Round54

A single-file Python agent for Kaggriculture farm-management matches. It coordinates crops, livestock, worker scheduling, transport, and sales under shared action and cash constraints. Version acceptance is based on **full-match outcomes and preserving previously won matches**.

This repository archives **Round54 CARROT**, the last version in this project to pass the full local comparison tests, rather than a later-numbered experimental version that did not pass acceptance. The code is preserved byte for byte from the original delivery; it was not trimmed or refactored for publication.

> This is a research and competition code archive, not a guaranteed winning strategy. Historical local tests from September 2026 do not represent the current online win rate. The 1,024-match suite was not rerun when this archive was published.

## At a Glance

| Item | Details |
| --- | --- |
| Delivered code | `main.py`, including the strategy, embedded data, and model-support logic |
| Final entry point | `macro_cycles_entrypoint54(observation, configuration=None)` |
| Entry-point aliases | `agent` and `kaggle_submission_agent` |
| Execution | The Kaggriculture environment supplies an observation at each step; the agent returns an action dictionary |
| External services | No LLM API, API key, or network-based inference required |
| Learning approach | Rules, offline-collected route data, and in-match state estimation; not online neural-network training |
| Historical acceptance test | 1,024 complete local matches: 858 wins and 166 losses |
| Compared with Round53 | 12 newly won matches, with 0 previously won matches lost |

## Code and Usage

`main.py` is an importable strategy file, not a game program with an interactive command-line interface. Running `python main.py` does not start a complete match.

Check the final entry point from this directory:

```bash
python -c "import main; assert main.agent is main.kaggle_submission_agent; print(main.agent.__name__)"
```

Expected output:

```text
macro_cycles_entrypoint54
```

Call the agent in an environment that supplies a valid observation:

```python
from main import agent

# observation must come from a compatible Kaggriculture environment.
# configuration is supplied by that environment; do not fake observation with {}.
action = agent(observation, configuration)
```

The code uses the Python standard library. Complete matches still require the official game environment and opponents, which must be prepared separately. Load the two agents in isolated processes or module instances to prevent their module-level, in-match state from interfering with each other. Do not skip step 0 initialization or share one strategy module instance across concurrent matches.

For a Kaggle submission, use the original `main.py` in this directory and first check whether the competition is open, what its upload requirements are, and whether the environment version remains compatible. Publishing this archive does not itself submit an entry to the competition.

### File Integrity

SHA-256 of the published version:

```text
6df684d1ceac5f3ba03db799ec590e8985ce5b605aace6370ac8446a28c8b4f1
```

Cross-platform verification:

```bash
python -c "import hashlib,pathlib; print(hashlib.sha256(pathlib.Path('main.py').read_bytes()).hexdigest())"
```

Changing whitespace, line endings, or comments also changes the hash. For further research, create a separate experimental version; do not describe a modified file as the validated snapshot identified above.

## Strategy Architecture

### 1. Preserve the Production Foundation and Coordinate Follow-up Actions

The agent builds on existing production routes and market-control logic, with additional maintenance, inventory, harvesting, transport, and sales fixes. New decisions must account for worker reachability, action availability for later watering and harvesting, and whether produce can be transported back and sold in time.

The code retains wrappers, supporting models, and embedded data from multiple iterations; it is not a minimal framework designed from scratch. Start at the final entry point at the end of the file and trace its call chain. Earlier entry-point definitions alone do not describe the final behavior.

### 2. Plan Production and Logistics Together

The coordination layers handle inventory buffers, cross-day harvest confirmations, remote transport, route recovery, and consistency between predicted actions and actual execution. Predicted and observed states are kept separate so that hypothetical harvests, purchases, or actions are not counted as completed events.

These mechanisms reduce specific failure risks; they do not eliminate every inventory overflow, transport delay, or route failure.

### 3. Account for the Opportunity Cost of Fertilizer and Maintenance

Fertilizing competes with watering, feeding, harvesting, and transport for action slots. Round54 inherits Round53's fertilizer opportunity evaluation and related fixes while preserving the existing wheat-growing and feed-supply logic. It does not apply more fertilizer unconditionally just to increase yields.

### 4. Round54's Main Addition: Full-Cycle Carrot Substitution

For selected strawberry-plot plans that have not yet begun execution, the agent compares the full-cycle returns of keeping the original plan versus substituting another crop, rather than comparing only a single sale price:

- Estimate planting, watering, harvesting, and sales using the plot's existing service schedule, while preserving downstream feasibility.
- Include the full cost of newly purchased seeds; do not treat the cost of previously purchased seeds as recoverable cash.
- Consider the market impact of both the agent's future output and the opponent's publicly visible production.
- Check conservative estimates of the agent's revenue and competitive gains across multiple supply and sales scenarios; reject substitutions with insufficient benefit.
- Retain constraints on cash, market order counts, and the total number of substituted plots.
- After buying seeds, wait for the next actual observation to confirm receipt before activating the associated production plan; do not treat predicted purchases as actual inventory.
- Protect existing crops rather than arbitrarily dismantling ongoing production to execute a new plan.

**The new macro-level wheat-substitution branch is disabled.** If that branch wins the comparison, the original plan is retained instead of automatically falling back to the second-best carrot plan. This does not disable the existing wheat-growing or feed-production logic.

### 5. Limits of In-Match Adaptation

The agent adjusts decisions using the current match's visible state, inventory confirmations, and publicly visible opponent production. This file does not automatically update models across matches or retrain weights during real competition matches. Offline testing and iteration remain important to strategy improvement.

## Historical Validation Results

The following results come from the project's acceptance records saved on September 30, 2026. The full local suite covered 16 local opponent types and 32 seeds with both seat assignments, for 1,024 matches in total.

| Version | Wins | Losses | Newly won matches vs. Round53 | Previously won matches lost vs. Round53 |
| --- | ---: | ---: | ---: | ---: |
| Round53 baseline | 846 | 178 | — | — |
| **Round54 (this file)** | **858** | **166** | **12** | **0** |

The overall local win rate was approximately **83.79%**. The 12 newly won matches correspond to both seat assignments for 6 seeds; they should not be treated as 12 independent random observations.

Selected opponent groups:

| Local opponent identifier | Wins / Matches | Notes |
| --- | ---: | --- |
| `archive_mixed2_v2` | 57 / 64 | 89.06%; below the 95% target |
| `archive_sheep2_v2` | 12 / 64 | A significant unresolved weakness |
| `replica` | 31 / 64 | Still below half of the matches |
| `opponent_115414079_v2` | 60 / 64 | 93.75% |
| Six other favorable opponent groups | 64 / 64 each | Performance only under these fixed local conditions |

The six groups are `harvest`, `top10`, `opponent_115415605_v2`, `opponent_115421776_v2`, `opponent_115420280_v2`, and `opponent_115429426_v2`. These names are test identifiers only; `top10` does not establish coverage of the historical or current leaderboard's top ten agents.

Additional checks:

- Of 304 rule and boundary checks, 302 passed. Two checks applied only to a different experimental branch and were explicitly skipped.
- No execution or reporting errors were found in the full-match records. This is not a guarantee of error-free behavior for every input or hardware configuration.
- The recorded maximum single-step time was approximately 0.836 seconds under a local eight-process workload. This does not guarantee that the agent will avoid timeouts on online competition hardware.

### Replaying Two Actual Losses with Fixed Opponent Actions

| Original replay ID | Final cash difference in the local rerun | Result |
| --- | ---: | --- |
| `115949593` | +2,981 | Win under this fixed-replay setup |
| `115951111` | −1,466 | Still a loss; unresolved |

These reruns used fixed opponent actions from actual replays. They were **not new online matches against the original opponent agents responding in real time**.

### Why Round55 Was Not Published

The final two experiments each screened 100 matches on the same selected panel, reusing the Round54 control results:

| Experiment | Experimental wins | Round54 wins | Decision |
| --- | ---: | ---: | --- |
| Increase the number of macro-level plots considered each day | 88 / 100 | 88 / 100 | No newly won matches; not merged |
| Select up to 3 new substitution plots at a time | 86 / 100 | 88 / 100 | Lost 2 previously won matches; not merged |

Improving the cash difference in some matches does not necessarily improve the win rate. Round54 therefore remained the final recommended version.

## Scope and Limitations of the Test Evidence

1. The opponent pool includes local strategies and opponents reconstructed from replays; it is not equivalent to the original programs submitted by leaderboard participants.
2. The test panel was used repeatedly during iteration and is not an independent holdout set. The 83.79% result should not be interpreted as a prediction of online win rate.
3. "No previously won matches lost" applies only to the panel described above. It does not imply no regression against every opponent, map, or random seed.
4. In the historical environment, changes to plot layouts could change random-number consumption and subsequently affect the shop. The same seed does not fully fix future market conditions. Not every change in cash can be attributed solely to crop returns.
5. This directory archives the submission code and its documentation. It does not include the complete original replays, opponent files, or full test runner; these two files alone cannot reproduce all 1,024 matches.
6. No claim is made of covering the leaderboard's top 500 agents, achieving online winning streaks or a 95% overall win rate, or resolving all known losses.

For further improvements, use the original snapshot as the control. First validate mechanics and action boundaries, then run complete matches with the same seeds and both seat assignments. Maintain a separate test set that is not used for tuning.

## Environment and Upstream Sources

This version corresponds to the historical Kaggriculture environment. The rule source identified in the code is `kaggriculture.py` from Kaggle's `kaggle-environments` **1.32.7**, with this checksum:

```text
bc8a54879ef02c7ea64b8b333d6a976f0ea65c4949149d01f463f23bccee653e
```

Source references:

- [Kaggriculture competition](https://www.kaggle.com/competitions/kaggriculture)
- [Official Kaggle environments repository](https://github.com/Kaggle/kaggle-environments)
- [Thomas Tschinkel's public router version](https://www.kaggle.com/code/thomastschinkel/kaggriculture-93-8-win-rate-public-state-router?scriptVersionId=347936183)
- [yhay81's public shop-router](https://www.kaggle.com/code/yhay81/shop-router-0913)

These links support source tracing; their current contents are not guaranteed to match the historical snapshots. Check versions and source hashes when validating historical mechanics.

## Attribution and Licensing

The repository's existing MIT `LICENSE` does not replace the Apache-2.0 licensing and attribution requirements of third-party code within `main.py`. Do not relabel the entire agent as MIT-only. Retain the original license notices applicable to each part when using or distributing the code.

This project was not implemented independently from scratch. `main.py` retains upstream Apache-2.0 license text, attribution, and modification records, including references to Kaggle / kaggle-environments contributors and the contributors named in the source: thomastschinkel, yhay81, destbreso, aurax7, tetsutani, prvsiyan, Dmitrii Gluzdov, and Ahmed Berat Ozer. The original notices in the file determine the specific attribution; this archive does not imply endorsement by those contributors.

The project's additions primarily concern strategy integration, in-match coordination and safeguards, fixes for confirmation and prediction consistency, and the Round54 macro-level adjustments that passed the historical comparison tests. Upstream license notices were not removed, rewritten, or replaced during publication.

**Data provenance limitation:** Existing source notes indicate that some public route data lacks a complete, replay-by-replay provenance manifest. An upstream notebook's license statement does not establish the complete rights provenance of the underlying replay data. This archive preserves that limitation and does not claim that data with incomplete provenance is wholly original or that all ownership rights have been verified. Review the relevant terms and data sources before further distribution or commercial use.
