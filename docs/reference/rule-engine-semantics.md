# Rule engine semantics

This page documents internal engine semantics for rule representation, execution, invalidation, and deletion. It is intended for contributors and advanced users who need to understand how OM Core persists and evaluates rules.

For the user-facing syntax, see [Rule syntax](rule-syntax.md).

## Rule representation

Rule display text is user-facing and may contain labels for readability.

Compiled rule representation is engine-facing and resolves references to stable IDs. A dimension item may be renamed without breaking the rule, because the engine stores stable IDs, not labels.

OM Core prefers to persist both display text for editing and compiled references for execution. If only display text is stored, load must re-resolve references deterministically and fail loudly on ambiguity.

## Rule invariants

Rule semantics must preserve:

- explicit reference syntax;
- deterministic rule execution order;
- no bidirectional recurrence;
- no ambiguous writes outside the current target cell;
- stable value precedence rules.

## Rule execution order

Rules execute in the order specified by `workspace.formula_rule_order`. This list of rule IDs is the semantic contract. Do not rely on dictionary insertion order as a semantic execution contract — always use `formula_rule_order`.

## Value precedence

When a cell is evaluated, the engine resolves value precedence from highest to lowest:

1. Hardcoded user override (`user_override_addrs`)
2. Cell rule (single-cell rule)
3. Slice rule (rule targeted at a pattern)
4. Empty cell (`None`)

A more specific rule always wins over a less specific rule. When two rules have the same specificity and match the same cell, the later rule in `workspace.formula_rule_order` wins.

## Rule invalidation

When a graph node, dimension item, dimension, or rule target is deleted, affected rules must be handled in the same semantic mutation that performs the deletion. Acceptable outcomes:

- remove or update affected rule references;
- mark rules invalid with `#REF!`;
- reject the deletion if rules cannot be safely updated.

The system must not expose a durable state where deleted graph or item references remain silently embedded in executable rule logic.

## Rule deletion

Deleting a rule must remove its rule ID from `workspace.formula_rule_order` in the same semantic mutation. The system must not persist dangling rule-order entries.

## Anti-patterns

| Anti-pattern | Why we avoid | Correct approach |
| --- | --- | --- |
| Skipping dependency cleanup during recompute | Leaves stale state | Ensure recompute cleanup and indegree updates always run |
| Rule references stored by label | Renames break rules | Resolve to stable IDs |
| Relying on dict order for rule execution | Serialization drift | Use `workspace.formula_rule_order` |
| Bidirectional recurrence | Ambiguous dependency direction | Use only `PREV` or only `NEXT` |
| Writing to `PREV` / `NEXT` on LHS | Ambiguous mutation target | LHS may target only current cell or explicit slice |
| Persisting error values | Reload corrupts state | Recalculate errors on load |
| Rule cleanup as a separate public command | Partial failure risk | Cleanup inside the semantic mutation |
| Silent rule invalidation | Hidden corrupted logic | Fail loudly, mark `#REF!`, or update references explicitly |
