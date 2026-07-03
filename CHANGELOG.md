# Changelog

All notable changes to this project will be documented in this file.

## [2.1.0] - 2026-07-03

### Changed

- **Confirm-each flow in Phase 3.** Candidates are now confirmed one at a time — evidence, violation risk, and draft text first, then `[y/n/skip]` — with bulk approval explicitly banned (design borrowed from config-gc). Skipped candidates go to the ledger with `status: skipped`.
- Related section now cross-references `rules-stocktake`, the inverse audit over the same skill/rule boundary.

### Added

- `scripts/sync-from-local.sh` — one-way export from the live Claude Code harness (vendored byte-identical across shimo4228 skill repos); the harness copy is canonical, this repository is the publication mirror.

## [2.0.0] - 2026-06-24

Major redesign for the large-context era — same modernization as the sibling
`skill-stocktake`. The skill dropped its shell-script scanning and subagent
batching in favor of inline Glob enumeration and a single-context cross-read.

### Changed

- **Removed scan scripts** (`scripts/scan-skills.sh`, `scripts/scan-rules.sh`). Inventory is now a Glob over `~/.claude/skills/*/SKILL.md` + `learned/*.md`, and all rule files are read in full inline.
- **Removed subagent batching and the cross-batch merge.** The whole corpus (skills + rules) is read into one context. The cross-batch merge existed only to recover the "appears in 2+ skills" signal that batching broke; with one context that test is exact from the start.
- **`results.json` is now a lean ledger** — `{distilled_at, candidates}`. Dropped the `skills_scanned` / `rules_scanned` counters.
- Frontmatter gained `license: MIT` and `user-invocable: true`.

### Fixed

- The old `find -name "*.md"` could count dependency markdown under `.venv` / `.pytest_cache` as skills. Glob over skill-definition files excludes that noise structurally.

### Requirements

- Claude Code with the Glob / Read / Edit tools (no subagents required).
- Optional: `jq` / `python3` for the inline ledger one-liner.

## [1.0.0]

Initial release: deterministic shell scan (`scan-skills.sh`, `scan-rules.sh`) +
thematically-batched subagent cross-read, with a `results.json` cache. Layout
normalized to nested `skills/rules-distill/` to match the sibling skill repos.

---

## About

A Claude Code Agent Skill that scans all installed skills, identifies cross-cutting
principles appearing in 2+ skills, and distills them into rules — appending to,
revising, or creating rule files. It is the **Promote** phase of the
[Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle):
`search-first → skill-stocktake → learn-eval → rules-distill` (research → quality
audit → pattern extraction → principle promotion). Never edits rules without user
approval.

### Relationship to companion skills

| Skill | Role | When |
|---|---|---|
| [`search-first`](https://github.com/shimo4228/search-first) | Research before coding | First phase of the lifecycle |
| [`skill-stocktake`](https://github.com/shimo4228/skill-stocktake) | Quality audit of existing skills | Second phase — keep/retire before distilling |
| [`learn-eval`](https://github.com/shimo4228/learn-eval) | Per-session pattern extraction with quality gate | Third phase — feeds candidate patterns into this skill |
