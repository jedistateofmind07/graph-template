---
name: graph-template
description: >-
  Graph-engineering brief: `/graph-template new <name>` writes a fill-in template
  (reason, goal, done-when, not-in-scope, constraints, nodes, edges, state, gates,
  execution) to .claude/graphs/<name>.md, registers it in the project ledger
  (.claude/graphs/ledger.md: a project index of docs/skills/key files, plus files/docs/
  code/skills per graph, tagged by task), and prints it. `/graph-template run <name>`
  validates the filled brief, resolves the graph, executes it node by node (inline,
  subagents, or a dynamic workflow), passes it through deterministic gates and a
  fresh-context reviewer, and reports handoff evidence per node. Trigger only on
  /graph-template or when the user says "graph template", "new graph", "run the graph".
disable-model-invocation: true
---

# Graph Template

Files live next to this skill: `template.md` (the blank brief), `ledger-template.md`
(the project ledger) and `references/principles.md` (the rules the run step follows; read
it before running). Filled briefs live in the current project at `.claude/graphs/<name>.md`;
one shared ledger lives at `.claude/graphs/ledger.md` and indexes every graph's files,
docs and code by task, so any agent can load only what its task needs.

**Ledger section format** (one per graph, heading = the slug so `ledger.md#<slug>` links):
```
## <slug>
Brief: [<slug>.md](<slug>.md) · Status: draft | running | done | stopped
| Item | Kind | Use for | Why it matters |
|---|---|---|---|
| [lib/foo.ts](../../lib/foo.ts) | code | implement, review | owns the merge logic |
```
Kind is one of: code, doc, skill, table, migration, url, artifact, brief. A skill item is
written as `/<skill-name>` and links to its SKILL.md when it lives in the repo. Paths are relative to
`.claude/graphs/`. One line per item; no duplicates (an item used by two graphs goes in
"Shared across graphs").

Pick the mode from `$ARGUMENTS`:

| Invocation | Mode |
|---|---|
| `/graph-template`, `/graph-template new`, `/graph-template new <name>` | **new** |
| `/graph-template run <name>`, `/graph-template <name>` (file exists) | **run** |
| A filled brief pasted in chat, or "run the graph" | **run** (from the pasted text / most recent brief) |

## Mode: new — write the template, then stop

1. Slug the name to kebab-case. No name given → `graph-<YYYY-MM-DD>` using `date +%F`.
2. Create `.claude/graphs/` if missing and copy `template.md` to `.claude/graphs/<slug>.md`,
   replacing every `<name>` with the slug. If the file already exists, do not overwrite; say
   so and print the existing one instead.
3. Ledger: if `.claude/graphs/ledger.md` is missing, copy `ledger-template.md` there and
   fill its **Project index** (one line per item, link + one-line description): CLAUDE.md,
   AGENTS.md, READMEs, `docs/`, other top-level `.md` docs, project skills
   (`.claude/skills/*/SKILL.md`), user-level skills (`~/.claude/skills/*`, listed as
   `/<name>`, no link) and entry-point files (package.json, main app/server files). Read
   only frontmatter or the first heading for descriptions. Add a
   Contents row (`[<slug>](#<slug>) | draft | — | [brief](<slug>.md)`) and an empty
   `## <slug>` section in the format above. Skip if the section already exists.
4. Print the whole brief in one code block so the user can fill it in place or paste it back.
5. Say in one line: fill the (required) sections, leave (optional) ones for Claude, add
   any known files/docs to the ledger section, then run `/graph-template run <slug>`.
6. Do nothing else. No exploration (beyond the one-time Project index scan in step 3), no
   code, no drafting of the optional sections yet.

## Mode: run — validate, resolve, execute, report

**Step 1 — Validate.** Read the brief. Required: Reason, Goal, Done when (at least one line
Claude can run or look at), Not in scope, Constraints, Nodes (at least one), Execution.
If any required section still holds an `<angle-bracket>` hint or is empty, list the missing
sections and stop. This is the only blocking question in the skill.

**Step 2 — Resolve the graph.** Read `references/principles.md`. Then:
- Loop-or-graph check (principle 1): if one node plus a deterministic gate would finish the
  job, say so in one line and run it as `inline` regardless of the node table.
- Draft every blank optional section: Edges from "Depends on", Shared state from what each
  node produces, Gates from the project's own commands (CLAUDE.md, package.json) plus the
  default reviewer node and default stop conditions.
- Enforce: five-word node names, disjoint file ownership (shared file → `integrate` node),
  at least one deterministic gate, reviewer sees only diff + Done-when, max loops from Budget
  (default 2), parallel agents capped by the Budget's max.
- Model per node: read the Nodes table's Model column (fable, opus, sonnet or haiku). Blank
  means the session's model. Default the `review` node to sonnet when its cell is blank.
- Write the drafted sections back into the brief under a `<!-- drafted by Claude -->` marker
  so the file is the record.
- Fill the ledger section for this graph: merge items from Context & pointers, every
  node's Owns column, the CLAUDE.md/docs the Gates rely on, and anything the user already
  listed, plus any skills (from the Project index or the session's skill list) a node will
  invoke. Tag each item's "Use for" with node names. Verify every path exists; drop or
  flag dead ones. Set the Contents row's Status to `running` and fill its Goal.

**Step 3 — Choose execution and announce.** Take Mode from the Execution section; if all
options remain, pick: `inline` for ≤2 nodes or strictly sequential work, `subagents` when
nodes need isolated context or a fresh reviewer, `workflow` when there is real fan-out
(many independent items, adversarial verification, or a sweep). Print the resolved node
table (one line per node, model included), the mode, and one **Scale** line computed from
the edges: `Nodes: N · Agents: A · Peak parallel: P of budget B · With loops: up to L`,
where A is one per node in `subagents` (0 in `inline`; the planned agent() call count in
`workflow`), P is the largest set of nodes with no unmet dependency at one moment, capped
at B, and L is A plus the nodes a review loop can relaunch times the max loops. If "Before running" is `ask`, stop and wait for a
yes. If `go`, continue without asking.

**Step 4 — Execute per node.**
- `inline`: do the work yourself, one node at a time, in dependency order.
- `subagents`: one Agent call per node with a narrow prompt containing only that node's
  job, its inputs from shared state, its owned files, its Done-when, and the ledger rows
  tagged with that node (or `all`) — not the whole ledger. Pass the node's Model cell as
  the Agent call's `model` parameter; omit it when the cell is blank. Run independent
  nodes in the same message so they execute concurrently, no more at once than the
  Budget's max parallel agents.
- `workflow`: call the Workflow tool. This skill's Mode line is the user's explicit opt-in.
  Default to `pipeline()`; add adversarial verify for any node that produces findings.
  Pass each node's Model cell as that `agent()` call's `model` option; the Budget's max
  parallel agents overrides the session's default workflow size.
- Every node returns the handoff block from principle 10. The next node verifies the
  artifact before using it.
- Honor the stop conditions (principle 11). On a stop, report what happened and ask.

**Step 5 — Gates.** Run every deterministic gate and paste the command plus exit code.
Run the reviewer node in a fresh context with only the diff and the Done-when list. Loop
back along the edges up to the max; if still failing, stop and report rather than widen scope.

**Step 6 — Report and record.**
- Deliver what the Execution section asked for (branch commit, PR, files, or report).
  Follow the project's git rules; show the diff before any push.
- Final message: the Done-when list with a proof line under each item, the handoff block
  per node, and anything skipped with the reason.
- Update the ledger: add rows for files changed and artifacts/IDs produced (from the
  handoff blocks), set Status to `done` or `stopped`, and move any item now used by
  2+ graphs into "Shared across graphs". If the run created a new doc, skill or
  entry-point file, add it to the Project index with a one-line description.
- Append a `## Run log` section to the brief: date, mode, outcome, the Scale line with
  actual agents launched and peak parallel, loops used, lessons.
  If a lesson applies beyond this graph, propose one line for CLAUDE.md (principle 14);
  do not add it yourself.
