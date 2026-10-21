# Graph-engineering principles the run step must follow

Distilled from what Boris Cherny (Claude Code lead), Anthropic's Claude Code docs,
and the July–September 2026 "graph engineering" writers converged on. Cite by
number when you deviate.

## Decide loop vs graph first
1. **Try the single loop first.** If one agent with a runnable check can finish the
   job, do not build a graph. Wiring three agents together costs more tokens and buys
   nothing. (AI Builder Club guide; Anthropic docs.)
2. **A graph earns its cost only when the job genuinely splits:** distinct
   specialties with handoffs, independent parts that can run in parallel, or a
   verifier that must not share the producer's context.

## Nodes
3. **One job per node.** If you cannot name the node in five words, split it.
4. **Minimal context per node.** Each node gets only the state slice it needs, not
   the conversation history. Structured handoffs (a small JSON or table) beat raw dumps.
5. **Disjoint ownership.** Give each node its own files. If two nodes need the same
   file, assign it to an `integrate` node or stop and ask. Never start a blocked node.

## Edges and state
6. **Routing is explicit and napkin-drawable:** sequential, fan-out/fan-in, or
   loop-back with a written condition and a max loop count (default 2).
7. **Shared state is a named object** that every node reads and writes its own
   section of. Say what travels on each edge.

## Verification (the part everyone agrees is non-negotiable)
8. **At least one node on every critical path must be deterministic**: a compiler,
   type-checker, linter, test suite, build, or query. Three model reviewers that agree
   are a chorus, not a verification. (the-ai-corner; Anthropic docs "Give Claude a
   way to verify its work".)
9. **The reviewer never shares the producer's context.** A fresh subagent sees only
   the diff plus the Done-when list and reports gaps that affect correctness or the
   stated requirements, not style. Chasing every finding leads to over-engineering.
10. **Show evidence, not assertions.** Every finished node hands back:
    ```
    Node:
    Outcome:
    Changed files:
    Validation command:
    Exit code:
    Tests passed:
    Artifacts or IDs produced:
    Known risks:
    ```
    The next node verifies the artifact instead of trusting "done". (aident.ai)

## Stop conditions
11. Stop and ask when: two nodes need the same file; a node discovers an unmet
    prerequisite; a validation step would alter shared state; the combined change
    drifts from the Goal; the budget is exceeded. Do not improvise past a stop.

## Process habits from Boris Cherny
12. **Spec before code.** Write the brief, then execute it in a clean context. The
    most useful specs are self-contained: they name files and interfaces, state what
    is out of scope, and end with an end-to-end verification.
13. **Slash commands for every inner loop** you repeat many times a day. This skill
    is one.
14. **Write corrections down, don't re-prompt.** When a run surfaces a lesson that
    applies beyond this graph, propose one line for CLAUDE.md or a skill.
15. **Lean context.** Give the model the minimum it needs and let it fetch the rest.
    Long briefs and long CLAUDE.md files get ignored in the noise.
16. **Bound the spend.** Parallel nodes multiply tokens. Cap agents, loops, and time
    in the brief, and log anything you drop rather than silently truncating.

## Claude Code primitives that map onto the graph
- Node → a subagent (Agent tool) with a narrow prompt and only its state slice.
- Orchestrator edges → the main session deciding what runs next.
- Fan-out / fan-in / adversarial verify at scale → the Workflow tool (dynamic
  workflows). Invoking this skill with `Mode: workflow` is the user's opt-in.
- Deterministic gate → a Bash command whose exit code decides the edge, or a Stop hook.
- Stop-until-done → the `/goal` condition when the run should keep going unattended.

## Sources (read 2026-09-14)
- Anthropic, Claude Code best practices: https://code.claude.com/docs/en/best-practices
- Anthropic, Introducing dynamic workflows: https://claude.com/blog/introducing-dynamic-workflows-in-claude-code
- Boris Cherny tip collections: https://howborisusesclaudecode.com/ and
  https://github.com/shanraisshan/claude-code-best-practice
- AI Builder Club, Graph engineering with Claude Code:
  https://www.aibuilderclub.com/blog/graph-engineering-with-claude-code and
  https://www.aibuilderclub.com/blog/graph-engineering-guide-2026
- Claude Code Masterclass, graph engineering:
  https://newsletter.claudecodemasterclass.com/p/claude-code-graph-engineering-masterclass
- aident.ai, orchestrate agent teams safely (handoff format):
  https://aident.ai/blog/claude-code-graph-engineering-agent-teams
- the-ai-corner, the verification problem:
  https://www.the-ai-corner.com/p/graph-engineering-verification-problem
- Charlie Hills, the 4-prompt folder map (frame → work → attack → answer):
  https://charliehills.substack.com/p/graph-engineering-claude-code
