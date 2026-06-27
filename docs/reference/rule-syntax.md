# Rule syntax

This page is a reference for the OM Core rule syntax. It is written for LLMs and users who need to read or generate rules that drive cube calculations.

## What a rule is

A rule is a semantic calculation attached to a cube cell or slice. It describes how a value is derived from other values in the model. Rules are first-class objects in OM Core: they are explicit, stable, and executed in a defined order.

A rule may be:

- a **cell rule** attached to one cell;
- a **slice rule** attached to a target pattern that covers multiple cells;
- a **recurrence rule** that uses ordered dimension context to chain values across time or another ordered dimension.

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

## Reference syntax

A dimension item reference uses dot notation:

```text
DimensionName.ItemName
```

`DimensionName.ItemName` is user-facing syntax. The parser maps names to stable IDs. Renames must not break executable rule identity.

If a user-facing rule reference resolves to multiple stable IDs, the parser rejects it as ambiguous or requires disambiguation. It never chooses one silently.

A full cell or slice reference combines multiple dimension item references according to the active cube or view context:

```text
Cube::@.value:Dim1.Item1:Dim2.Item2
```

## Sequential accessors

Valid sequential accessors in rule references are `[FIRST]`, `[LAST]`, `[PREV]`, `[NEXT]`, and `[THIS]`.

| Reference | Meaning |
| --- | --- |
| `Dim1.Item1` | A single item |
| `Dim1[FIRST]` | First item in the ordered dimension |
| `Dim1[LAST]` | Last item in the ordered dimension |
| `Dim1[PREV]` | Previous item in the ordered dimension |
| `Dim1[NEXT]` | Next item in the ordered dimension |
| `Dim1[THIS]` | Current item during rule evaluation |

Valid examples:

```text
Year.2026
Year[FIRST]
Year[PREV]
Year[THIS]
```

Invalid examples:

```text
Year[2026]     # bracket after dimension name is wrong syntax
```

## LHS and RHS

- The **LHS** (left-hand side) identifies the target cell or slice.
- The **RHS** (right-hand side) is the expression that produces the value.
- RHS references must use explicit syntax; no implicit context inference.
- LHS may target only the current cell or an explicit slice. It may not write to relative cells.

### LHS examples

```text
Revenue[THIS]      # current item of the target dimension
Costs.*            # explicit slice wildcard
```

### Invalid LHS examples

```text
Revenue[PREV]      # relative cell write
Year[NEXT]         # relative cell write
Dim1[Item1]        # wrong bracket syntax
```

`PREV`, `NEXT`, `FIRST`, and `LAST` are invalid on the LHS.

## Wildcards vs sequential placeholders

`*` is an explicit slice wildcard. It declares that the rule applies across a slice.

`THIS` is a rule-template placeholder that binds to the current item during rule evaluation. It is not a wildcard.

For example, `Revenue[THIS]` on the LHS means the rule applies to the current item of the target dimension.

## Recurrence rules

Recurrence rules use ordered dimension context to chain values forward or backward.

- No circular references are allowed. Cycles detected during validation are rejected. Cycles detected during evaluation produce `#CIRC!`.
- A recurrence may be forward-looking or backward-looking, but not both.
- A `PREV` recurrence is evaluated in increasing order of the ordered dimension.
- A `NEXT` recurrence is evaluated in reverse order of the ordered dimension.
- A rule using both `PREV` and `NEXT` is rejected because it implies bidirectional dependency.

Sequential accessors are defined over the active stable item order of the referenced dimension or graph context. The rule resolver must choose exactly one order context. If both dimension order and graph order are possible and the rule does not disambiguate, validation fails.

## Rule precedence

When a cell is evaluated, the engine decides which rule produces the value. Two principles control this: specificity and rule order.

### Specificity

More specific rules override less specific rules. The precedence from highest to lowest is:

1. Hardcoded value (`user_override_addrs`)
2. Cell rule (single-cell rule)
3. Slice rule (rule targeted at a pattern)
4. Empty (`None`)

A cell rule that targets `Sales::Month.Jan` wins over a slice rule that targets `Sales::*`. A slice rule wins over an empty cell.

For example, with these rules:

```text
rule Sales::@.value:* = 100
rule Sales::@.value:Month.Jan = 200
```

The cell for `Month.Jan` resolves to `200` because the cell rule is more specific. All other months resolve to `100` from the slice rule.

### Rule order and overriding

When multiple rules have the same specificity and match the same cell, the later rule in `workspace.formula_rule_order` wins. The order is the semantic contract; do not rely on dictionary insertion order.

For example:

```text
rule Sales::@.value:* = 100
rule Sales::@.value:* = 150
```

Both rules are slice rules covering the same cells. The second rule overrides the first, so every cell resolves to `150`.

This behavior lets you define a general rule first and then add targeted overrides later. A later rule with the same target replaces the earlier one for the cells it matches.

## Rule execution order

Rules execute in the order specified by `workspace.formula_rule_order`. This list of rule IDs is the semantic contract. Do not rely on dictionary insertion order as a semantic execution contract — always use `formula_rule_order`.

## Anchored rules

An **anchored rule** targets exactly one cell. The user opts in by prefixing the rule target with `$`.

```text
$[Year.2024, Region.North] = Revenue[THIS] * 1.1   # anchored: one cell
 [Year.2024, Region.North] = Revenue[THIS] * 1.1   # standard: slice (wildcarded)
```

Syntax:

- `$` immediately before the target address marks the rule as anchored.
- The `$` is stripped before parsing the target; it carries no semantic meaning in the expression.

Semantics:

- **Anchored rule (`$`)**: Unspecified dimensions use their default items. The rule applies to exactly one address.
- **Standard rule (no `$`)**: Unspecified dimensions use wildcards (`None` in the mask). The rule applies to a slice.

Example with a 3D cube `(Year, Region, Scenario)`:

| Target | Anchored? | Scenario mask | Coverage |
| --- | --- | --- | --- |
| `$[Year.2024, Region.North, Scenario.Actual]` | Yes | `Actual` | One cell: `(2024, North, Actual)` |
| `[Year.2024, Region.North]` | No | `None` (wildcard) | Slice: `(2024, North, *)` |

Use cases:

- Use `$` when entering a rule via the grid or rule panel to bind it to a specific cell.
- Omit `$` when defining rules that should apply across dimensions.

## Error behavior

Error values (`#CIRC!`, `#DIV/0!`, `#EXPRESSION!`, `#REF!`) are never persisted as cell values. They are always recalculated on load. Diagnostics may be logged, but they are not canonical cell state.

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

## Examples

### Simple cell rule

```text
rule Inputs::@.value:Asset.Vehicle:Metric.Cost = 50000
```

### Slice rule

```text
rule Drivers::@.value:Driver.PriceGadgets:* = 120
```

### Recurrence rule

```text
rule BS::@.value:BS.Cash:* = IF(POS(Year)=1, Drivers::Driver.OpeningCash:Year[THIS], BS::BS.Cash:Year[PREV] + CF::CF.FreeCashFlow:Year[THIS])
```

### Anchored rule

```text
rule $[Year.2024, Region.North] = Revenue[THIS] * 1.1
```

## For LLMs

When generating rule syntax:

- Use `DimensionName.ItemName` for item references.
- Use `[THIS]` for the current item, `[PREV]` / `[NEXT]` for recurrence on the RHS only.
- Use `*` for slice wildcards on the LHS.
- Use `$` to anchor a rule to a single cell.
- Avoid `PREV` / `NEXT` / `FIRST` / `LAST` on the LHS.
- Avoid mixing `PREV` and `NEXT` in the same rule.
- Reference cells by stable semantic address, not grid coordinates.
