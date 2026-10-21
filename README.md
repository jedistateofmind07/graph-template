# /graph-template

A Claude Code skill for graph engineering: write a short brief (reason, goal, done-when,
nodes, gates), then let Claude run it node by node — inline, with subagents, or as a
dynamic workflow — with deterministic gates and a fresh-context reviewer.

Each project also gets a **ledger** (`.claude/graphs/ledger.md`): a table of contents of
the project's docs, skills and key files with one-line descriptions, plus a section per
graph listing exactly which files, docs and skills each node needs. Agents load only
their own rows instead of the whole repo.

## Install

```bash
git clone https://github.com/jedistateofmind07/graph-template.git ~/.claude/skills/graph-template
```

Restart Claude Code (or start a new session). The skill is user-level, so it works in
every project. To update later:

```bash
git -C ~/.claude/skills/graph-template pull
```

## Use

| Command | What it does |
|---|---|
| `/graph-template new <name>` | Writes a blank brief to `.claude/graphs/<name>.md`, creates/updates the ledger, prints the brief |
| `/graph-template run <name>` | Validates the brief, drafts blank optional sections, executes the graph, runs gates, reports evidence per node |

Fill the `(required)` sections of the brief; leave `(optional)` ones for Claude.

## Files

| File | What it is |
|---|---|
| [SKILL.md](SKILL.md) | The skill instructions Claude follows |
| [template.md](template.md) | The blank brief |
| [ledger-template.md](ledger-template.md) | The blank project ledger |
| [references/principles.md](references/principles.md) | The graph-engineering rules the run step follows, with sources |
