# rules-distill

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/rules-distill)

An [Agent Skill](https://agentskills.io/specification) that scans your installed skills, extracts cross-cutting principles appearing in **2+ skills**, and distills them into rules — appending to existing rule files, revising outdated content, or creating new ones.

The final piece of the knowledge lifecycle:

```
search-first → skill-stocktake → learn-eval → rules-distill
(research)     (quality audit)   (extraction)  (principle promotion)
```

## Install

### Claude Code

```bash
git clone https://github.com/shimo4228/rules-distill
cp -r rules-distill/skills/rules-distill ~/.claude/skills/rules-distill
```

Invoked via the `/rules-distill` slash command — discovery is driven by `SKILL.md` frontmatter (`user-invocable: true`), so no separate `commands/` copy is needed.

## How It Works

No scan scripts and no subagent batching — with a large context window the skill reads every skill and every rule into one context. That single-context view is what makes the "appears in 2+ skills" test exact (the old version needed a cross-batch merge step purely to recover that signal after batching broke it).

### Phase 1: Inventory (Glob, exhaustive)

Glob enumerates skill definition files (`~/.claude/skills/*/SKILL.md` + `learned/*.md`) and reads every rule file in full. The rules corpus is small (~800 lines), so no grep pre-filter is needed. Dependency markdown under `.venv` / `.pytest_cache` is excluded structurally because Glob targets only skill files.

### Phase 2: Cross-read, Match & Verdict (holistic)

The skill cross-reads all skills and the full rules text in a single inline pass.

**Extraction criteria** (all must be true):
1. Appears in 2+ skills
2. Actionable behavior change ("do X" / "don't do Y")
3. Clear violation risk (1 sentence)
4. Not already in rules (even if worded differently)

### Phase 3: User Review & Execution

Candidates are presented in a summary table, then confirmed **one at a time** — each shows its evidence, violation risk, and draft text before asking `[y/n/skip]`; bulk approval is banned and the user can stop at any point. **Never modifies rules automatically** — rules load every session, so a bad rule has outsized blast radius.

## Verdict Types

| Verdict | Meaning |
|---------|---------|
| **Append** | Add to an existing section of an existing rule file |
| **Revise** | Fix inaccurate or insufficient content in existing rules |
| **New Section** | Add a new section to an existing rule file |
| **New File** | Create a new rule file |
| **Already Covered** | Sufficiently covered in existing rules |
| **Too Specific** | Should remain at the skill level |

## Design Principles

- **What, not How**: Extract principles (rules) only. Code examples stay in skills.
- **Link back**: Drafts include `See skill: [name]` references.
- **Glob = exhaustive collection, LLM = judgment**: Glob guarantees the inventory is complete; the single-context cross-read guarantees contextual understanding.
- **Anti-abstraction safeguard**: 3-layer filter (2+ skills, actionable behavior test, violation risk) prevents overly abstract principles from entering rules.

## Example Output

```
Rules Distillation Report

Skills scanned: 56 | Rules: 22 files | Candidates: 4

| # | Principle                                          | Verdict     | Target          |
|---|----------------------------------------------------|-------------|-----------------|
| 1 | LLM output: normalize, type-check before reuse     | New Section | coding-style.md |
| 2 | Define explicit stop conditions for iteration loops | New Section | coding-style.md |
| 3 | Compact context at phase boundaries, not mid-task   | Append      | performance.md  |
| 4 | Separate business logic from I/O framework types    | New Section | patterns.md     |
```

## Related Skills

All six AKC phases as standalone Claude Code skills:

| Skill | AKC Phase | Role |
|-------|-----------|------|
| [search-first](https://github.com/shimo4228/search-first) | Research | Find existing solutions before writing custom code |
| [learn-eval](https://github.com/shimo4228/learn-eval) | Extract | Quality-gated extraction of session patterns into skills |
| [skill-stocktake](https://github.com/shimo4228/skill-stocktake) | Curate | Audit accumulated skills for quality and overlap |
| **rules-distill** | **Promote** | **Distill cross-skill principles into rule files** |
| [skill-comply](https://github.com/shimo4228/skill-comply) | Measure | Test whether agents actually follow their skills and rules |
| [context-sync](https://github.com/shimo4228/context-sync) | Maintain | Audit docs for role overlaps, stale content, and missing ADRs |

Together, these form a complete self-improvement loop for AI agents:

```
Research → Extract → Curate → Promote → Measure → Maintain → (back to Research)
   │          │         │         │          │          │
search-    learn-     skill-    rules-     skill-     context-
 first      eval     stocktake  distill    comply      sync
```

## Requirements

- Claude Code with the **Glob**, **Read**, and **Edit** tools (the analysis runs in one main context — no subagents required).
- Optional: `jq` / `python3` for the inline ledger one-liner.

## About this skill

This skill implements the **Promote** phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) — a Zenodo-citable six-phase bidirectional growth loop ([DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)) for sustaining intent alignment between an AI agent and its operator over time. AKC is one of three research lines by [@shimo4228](https://github.com/shimo4228), alongside [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) ([DOI 10.5281/zenodo.19212118](https://doi.org/10.5281/zenodo.19212118)) — autonomous agents grounded in four contemplative axioms — and [Agent Attribution Practice (AAP)](https://github.com/shimo4228/agent-attribution-practice) ([DOI 10.5281/zenodo.19652013](https://doi.org/10.5281/zenodo.19652013)) — harness-neutral ADRs on accountability distribution.

## License

MIT
