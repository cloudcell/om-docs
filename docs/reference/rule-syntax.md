# Rule syntax

This page is a reference for the OM Core rule syntax. It is written for users and LLMs who need to read or generate `.openm` rules that drive cube calculations.

## What a rule is

A rule is a semantic calculation attached to a cube cell or slice. It describes how a value is derived from other values in the model. Rules are first-class objects in OM Core: they are explicit, stable, and evaluated in a defined order.

A rule may be:

- a **cell rule** attached to one cell;
- a **slice rule** attached to a target pattern that covers multiple cells;
- a **recurrence rule** that uses ordered dimension context to chain values across time or another ordered dimension.

## Semantic address syntax

A rule address has the form:

```text
Cube::@.channel:Dim1.Item1:Dim2.Item2
```

- `Cube` — the target cube.
- `@.channel` — the channel the rule writes to. Use `@.value` for the value channel, `@.fill` for background fill, or `@.font_color` for font color.
- `Dim1.Item1:Dim2.Item2` — dimension item selectors, separated by `:`.

For example:

```text
Sales::@.value:Month.Jan
PL::@.fill:PL.Revenue:Year.2026
```

If you omit the channel, the rule defaults to `@.value`.

## Dimension item references

A single dimension item uses dot notation:

```text
DimensionName.ItemName
```

Valid examples:

```text
Year.2026
Month.Jan
Region.North
```

If a name resolves to multiple items, the parser rejects it as ambiguous. It never chooses one silently.

## Sequential accessors

Sequential accessors refer to the order of an ordered dimension. Valid accessors are `[FIRST]`, `[LAST]`, `[PREV]`, `[NEXT]`, and `[THIS]`.

| Reference | Meaning |
| --- | --- |
| `Dim[FIRST]` | First item in the ordered dimension |
| `Dim[LAST]` | Last item in the ordered dimension |
| `Dim[PREV]` | Previous item in the ordered dimension |
| `Dim[NEXT]` | Next item in the ordered dimension |
| `Dim[THIS]` | Current item during rule evaluation |

Valid examples:

```text
Year[FIRST]
Year[PREV]
Year[THIS]
```

Invalid examples:

```text
Year[2026]     # bracket after dimension name is wrong syntax
```

## Bracket shorthand

Inside a cube reference, you can wrap a dimension selector in brackets to keep it grouped. This is useful when a selector contains multiple parts or when you want to make the cube reference explicit.

```text
Sales::[Month.Jan]
Inputs::[Metric.Cost]
```

This is **not** the same as `Year[2026]`. Bracket shorthand applies to a full cube reference, not to an item lookup after a dimension name. The accessor `Year[2026]` is invalid because `2026` is a dimension item, not a positional accessor.

| Syntax | Valid? | Why |
| --- | --- | --- |
| `Year[PREV]` | Yes | positional accessor on an ordered dimension |
| `Sales::[Month.Jan]` | Yes | bracket shorthand for a cube reference |
| `Year[2026]` | No | `2026` is an item, not a positional accessor |

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

- No circular references are allowed.
- Static cycles detected during validation are rejected.
- Cycles that only become apparent during evaluation are reported as `#CIRC!`.
- A recurrence may be forward-looking or backward-looking, but not both.
- A `PREV` recurrence is evaluated in increasing order of the ordered dimension.
- A `NEXT` recurrence is evaluated in reverse order of the ordered dimension.
- A rule using both `PREV` and `NEXT` is rejected because it implies bidirectional dependency.

Sequential accessors use the active stable order of the referenced dimension. If both dimension order and graph order are possible and the rule does not disambiguate, validation fails.

## Rule precedence

When a cell is evaluated, the engine decides which rule produces the value. Two principles control this: specificity and rule order.

### Specificity

More specific rules override less specific rules. The precedence from highest to lowest is:

1. Hardcoded user override
2. Cell rule (single-cell rule)
3. Slice rule (rule targeted at a pattern)
4. Empty cell

A cell rule that targets `Sales::Month.Jan` wins over a slice rule that targets `Sales::*`. A slice rule wins over an empty cell.

For example, with these rules:

```text
rule Sales::@.value:* = 100
rule Sales::@.value:Month.Jan = 200
```

The cell for `Month.Jan` resolves to `200` because the cell rule is more specific. All other months resolve to `100` from the slice rule.

### Rule order and overriding

When multiple rules have the same specificity and match the same cell, the later rule wins. The rule order is the semantic contract; do not rely on dictionary insertion order.

For example:

```text
rule Sales::@.value:* = 100
rule Sales::@.value:* = 150
```

Both rules are slice rules covering the same cells. The second rule overrides the first, so every cell resolves to `150`.

This behavior lets you define a general rule first and then add targeted overrides later. A later rule with the same target replaces the earlier one for the cells it matches.

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
- **Standard rule (no `$`)**: Unspecified dimensions use wildcards. The rule applies to a slice.

Example with a 3D cube `(Year, Region, Scenario)`:

| Target | Anchored? | Scenario selector | Coverage |
| --- | --- | --- | --- |
| `$[Year.2024, Region.North, Scenario.Actual]` | Yes | `Actual` | One cell: `(2024, North, Actual)` |
| `[Year.2024, Region.North]` | No | wildcard | Slice: `(2024, North, *)` |

Use `$` when entering a rule via the grid or rule panel to bind it to one specific cell. Omit `$` when defining rules that should apply across dimensions.

## Error behavior

Error values (`#CIRC!`, `#DIV/0!`, `#EXPRESSION!`, `#REF!`) are never persisted as cell values. They are always recalculated on load. Diagnostics may be logged, but they are not canonical cell state.

## Examples

### Simple cell rule

```text
rule Inputs::@.value:Asset.Vehicle:Metric.Cost = 50000
```

### Slice rule

```text
rule Drivers::@.value:Driver.PriceGadgets:* = 120
```

### Style rule

```text
rule PL::@.fill:PL.Revenue:Year.2026 = #3B82F6
rule PL::@.font_color:PL.Revenue:Year.2026 = #FFFFFF
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
- Use `Cube::@.channel:Dim.Item:Dim.Item` for rule addresses.
- Use `[THIS]` for the current item, `[PREV]` / `[NEXT]` for recurrence on the RHS only.
- Use `*` for slice wildcards on the LHS.
- Use `$` to anchor a rule to a single cell.
- Avoid `PREV` / `NEXT` / `FIRST` / `LAST` on the LHS.
- Avoid mixing `PREV` and `NEXT` in the same rule.
- Reference cells by stable semantic address, not grid coordinates.
