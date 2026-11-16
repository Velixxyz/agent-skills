# Agent Skills

A collection of agent skills: reusable, opinionated playbooks an AI agent
loads when doing a kind of task. Each skill lives in its own directory and
is written to be followed, not just read: checklists, gates, and worked
rules the agent applies before acting.

These skills are generic. They contain no user data, no credentials, no
wallet addresses, and no personal paths. Anything that must be supplied by
the user is written as a `<placeholder>`.

## Skills

| Skill | What it does |
|---|---|
| `deep-research` | Deep, citation-backed research with an evidence ledger, source scoring, and automated validation. Includes the `evidence-kit` sub-system (schemas, scripts, templates, references). |
| `handoff-brief` | Write handoff briefs as contracts: a four-bucket fact survey, verified vs assumed separation, and authorship rules for delegated builder sessions. |
| `leaderboard-strategy-optimizer` | Beat a Top-1 benchmark through measured experiments instead of blind imitation: decision dossiers, run notebooks, and deploy gates. |
| `precision-task-optimizer` | Maximize the probability of the best valid result on challenges, jobs, quests, competitions, and bounty tasks. |
| `research-online` | Research online: query formulation, source evaluation, and tool selection for web research. |
| `skill-effectiveness-bench` | Benchmark whether a skill actually helps: dummy controls, ablation, and audit readiness before trusting a score. |
| `systematic-debugging` | Debug systematically: reproduce first, trace root cause, and verify the fix instead of guessing. |
| `test-driven-development` | Test-driven development: write the failing test first, keep tests honest, and never ship on an untested claim. |
| `verification-before-completion` | Verify before claiming completion: independent checks, evidence-first reporting, and a hard gate on unverified claims. |

## Use

Copy a skill directory into your agent's skills location and follow its
`SKILL.md`. Skills are self-contained; cross-references between skills in
this repo use relative links. Two skills named in
`leaderboard-strategy-optimizer` (Synmerco Agent Bidding, Make.fun Bounty
Hunter) are separate skills and are not included here; they are mentioned
as plain text, not links.

## Language

Skills are written in the language they were authored in — mostly
Indonesian, some English. Commands, identifiers, and quoted source prose
keep their original wording.

## Provenance

Fold-in batch 3 (November 2026) merged patterns from 13 evaluated
open-source projects into these 9 skills, with verbatim prose attributed
inline (source file, line, and license) in each `SKILL.md`. Full
attribution, including which projects were paraphrased-only and which
were rejected, is in [SOURCES.md](SOURCES.md).

## License

MIT — see [LICENSE](LICENSE). Upstream sources keep their own licenses;
see [SOURCES.md](SOURCES.md).
