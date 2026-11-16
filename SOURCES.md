# Sources & Attribution

Fold-in batch 3 (November 2026) evaluated 13 open-source projects and
merged selected patterns into the 9 skills in this repo. Verbatim prose
that was folded is attributed inline in each `SKILL.md` with its source
file, line numbers, and license. This file records the full list.

## Quoted or closely adapted (verbatim prose attributed inline)

MIT licensed:

- `langchain-ai/open_deep_research` — research prompts, stopping rules,
  compression discipline (ODR-1..6).
- `microsoft/autogen` (code only) — fact-survey prompts and termination
  taxonomy from `autogen_agentchat` prompt/condition sources (AG-1..3).
- `stanfordnlp/dspy` — assertions/suggestions concept, applied with a
  hard fence: generation-only steps, never side-effectful ones; the
  Assert/Suggest API itself is deprecated upstream (DSPY-1..6).
- `gepa` — acceptance criterion, candidate pool, batch sampling
  (GEPA-1..5).
- `optuna` — trial and study discipline (OPT-1..4).
- `openai/evals` — evaluation harness patterns, per-sample seeding
  (EV-1..4).
- `AgentTraceLab` — evaluation readiness, evidence handling,
  privacy-reduced snapshots (ATL-1..6).
- `currenjin/skillbench` — benchmark method adapted by
  `skill-effectiveness-bench`.
- `mthines/agent-skills` — handoff pattern adapted by `handoff-brief`.
- `199-biotechnologies/claude-deep-research-skill` — the upstream skill
  vendored as `deep-research/evidence-kit` (MIT, "modify as needed").

Apache 2.0 licensed:

- `confident-ai/deepeval` — `assert_test` evaluation pattern (DV-1..3).
- `AgentHPOBench` — hyperparameter-optimization bench patterns
  (HPO-1..4).
- `mlflow` — run-book / experiment-tracking patterns (ML-1..3).
- `dvc` — data-versioning patterns (DVC-1..2).

## Paraphrased only, never quoted

- `deap` (LGPL v3) — evolutionary-algorithm patterns (DEAP-1..5) are
  custom paraphrases. Verified: 0 shared 8-grams against the DEAP
  sources checked (`algorithms.py`, `tools/support.py`,
  `tools/selection.py`, `README.md`). No DEAP code or prose is copied.

## Listed for completeness, nothing quoted

- `microsoft/autogen` documentation (CC-BY-4.0) — no quotes were taken
  from autogen docs; AG-1..3 come from code prompts (MIT).

## Rejected — nothing included

- `deepresearch-Links` (MIT) — rejected; nothing from it is included,
  including a ban on copying lines from its README (zero-width-space
  contamination).

## External playbooks, selectively adapted

Two skills were adapted from external playbooks pasted by the user and
then evaluated and developed selectively; their generic and unsafe parts
were dropped or bound to real tracks and standing gates. No license is
claimed for them here:

- "Leaderboard Strategy Optimizer v1.0.0" playbook
  (`leaderboard-strategy-optimizer`).
- The source playbook behind `precision-task-optimizer`.

## Repo license

This repo's own content is MIT — see [LICENSE](LICENSE). Upstream
sources above keep their own licenses.
