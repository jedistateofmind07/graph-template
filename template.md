# Graph brief: <name>

<!--
Fill every (required) section. Leave (optional) sections blank and Claude drafts
them at run time and writes the draft back here. Delete the <angle-bracket> hints
as you fill. Then: /graph-template run <name>
-->

## Reason (required)
<Why this matters now, in one or two sentences. What stays broken or missed if it doesn't happen.>

## Goal (required)
<The observable end state, not the activity. "Visitors can X" / "table Y holds Z" — never "add code for".>

## Done when (required — every line must be something Claude can run or look at)
- [ ] <command that proves it, e.g. `npx tsc -b && npm run lint` exit 0>
- [ ] <query that proves it, e.g. `select count(*) from social_posts where status='queued'` > 0>
- [ ] <screen that proves it, e.g. screenshot of /stats at 1280px shows the new chart>

## Not in scope (required)
- <what NOT to build or touch, even if tempting>

## Constraints (required)
- <rules that must hold, e.g. DB writes via service role only; no schema changes; no new deps>
- Must not change: <files / dirs / tables>
- Budget: <max review loops (default 2), time box, max parallel agents>

## Context & pointers (optional)
<@files, tables, URLs, prior decisions, an existing example to copy>
Ledger: [ledger.md](ledger.md#<name>) — every file/doc/code item this graph touches, tagged by node.
Add items here or straight into the ledger; Claude merges both at run time.

## Nodes (required — one job each; if you can't name it in five words it's two nodes)
<!-- One node = one agent in subagents mode. Before launching, Claude prints a Scale line:
     Nodes · Agents · Peak parallel (of the Budget's max) · With loops. Add rows or raise
     the Budget to scale up; merge rows or use inline mode to scale down. -->
| Node | Job (one line) | Depends on | Owns (files / artifacts) | Done when | Model |
|---|---|---|---|---|---|
| <node-name> | <...> | — | <...> | <...> | <fable / opus / sonnet / haiku; blank = session model> |
| <node-name> | <...> | <node> | <...> | <...> | |
| review | Check diff against Done-when | all above | nothing | no correctness gaps | sonnet |

## Edges (optional — derived from "Depends on" if blank)
<sequential / fan-out / fan-in / loop-back with a condition, e.g. `review → fail → implement (max 2 loops)`>

## Shared state (optional)
<what travels between nodes: findings JSON, list of changed files, the diff, a draft>

## Gates (optional — Claude proposes from the project if blank; at least one deterministic gate is mandatory)
- Deterministic: <tsc / lint / tests / build / query>
- Reviewer: fresh context, sees only the diff + Done-when, reports gaps not style
- Stop if: <two nodes need the same file; a prerequisite is missing; budget exceeded; work drifts from Goal>

## Execution (required — delete the options you don't want; if all remain, Claude picks)
- Mode: inline | subagents | workflow
- Before running: ask | go
- Deliverable: commit on branch <name> | PR | files only | report only
