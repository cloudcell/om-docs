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
- `@.channel` — the channel the rule writes to. If omitted, the value channel `@.value` is implied. Explicit channels are required only for style or format rules, such as `@.fill` or `@.font_color`.
- `Dim1.Item1:Dim2.Item2` — dimension item selectors, separated by `:`.

For example:

```text
Sales::Month.Jan                 # value channel implied
PL::@.fill:PL.Revenue:Year.2026 # style channel explicit
```

If you omit the channel, the reference defaults to `@.value`. This applies to both rule targets and RHS cube references.

### Strict form vs shorthand

| Style | Form | Example |
| --- | --- | --- |
| Strict | `Cube::@.value:Dim.Item:Dim.Item` | `Sales::@.value:Month.Jan` |
| Shorthand | `Cube::Dim.Item:Dim.Item` | `Sales::Month.Jan` |

Most examples omit `@.value` for readability. Style and formatting rules must specify their channel explicitly, for example `@.fill` or `@.font_color`.

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

If a name resolves to multiple items, the parser rejects it as ambiguous.

## Range references

For sequential (`seq`) dimensions, the range syntax `start..end` selects a
contiguous sequence of items. It is valid only on the RHS of a rule or inside an
aggregate function.

```text
SUM(Cube::Dim.2026..2030)
Cube::[Dim.2026..2030]
```

The bounds are matched case-insensitively against item names. Using `..` on a
non-sequential dimension raises a validation error.

## Sequential accessors

Sequential accessors refer to the order of an ordered dimension. Valid accessors
are `[FIRST]`, `[LAST]`, `[PREV]`, `[NEXT]`, and `[THIS]`.

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

## Bracket shorthand and cube-relative references

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

### Cube-relative shorthand on the RHS

RHS references may use cube-relative shorthand. OM Core resolves a shorthand reference in this order:

1. Explicit selectors in the RHS reference are applied first.
2. For any remaining dimensions in the referenced cube, OM Core carries over matching dimensions from the current target-cell context, provided the binding is unambiguous.
3. Dimensions that exist in the target cube but not in the referenced cube are ignored.
4. Any dimension that exists in the referenced cube but is neither explicitly selected nor available from the current context is ambiguous and should be written explicitly.

A fully explicit address is always safer and required when the shorthand would be ambiguous.

For example, if the target cell is `AnnualDep::Asset.Vehicle:Year.2026`, then this shorthand:

```text
Inputs::[Metric.Cost]
```

carries over the shared `Asset.Vehicle` context to resolve `Inputs::Asset.Vehicle:Metric.Cost`. The `Year` dimension is not carried over because `Inputs` does not have a `Year` dimension.

## LHS and RHS

- The **LHS** (left-hand side) identifies the target cell or slice.
- The **RHS** (right-hand side) is the expression that produces the value.
- RHS references may use explicit semantic addresses or documented cube-relative shorthand.
- LHS may target only the current cell or an explicit slice. It may not write to relative cells.

### LHS examples

```text
Revenue::Account.Revenue:Year.*   # slice across all years
Costs::Account.Costs:Year.*       # explicit slice wildcard
```

### Invalid LHS examples

```text
Revenue::Account.Revenue:Year[PREV]   # relative cell write
Costs::Account.Costs:Year[NEXT]       # relative cell write
Costs::Account.Costs:Year[THIS]      # relative cell write
Cube::Dim1[Item1]                     # wrong bracket syntax
```

`THIS`, `PREV`, `NEXT`, `FIRST`, and `LAST` are invalid on the LHS.

## Wildcards vs sequential placeholders

`*` is an explicit slice wildcard. It declares that the rule applies across a slice.

`THIS` is a rule-template placeholder that binds to the current item during rule evaluation. It is not a wildcard. It is valid only on the RHS, where it reads the current item of the referenced dimension in the rule's evaluation context. Do not use `THIS` on the LHS.

## Recurrence rules

Recurrence rules use ordered dimension context to chain values forward or backward.

- No circular references are allowed.
- Static cycles detected during validation are rejected.
- Cycles that only become apparent during evaluation are reported as `#CIRC!`.
- A recurrence may be forward-looking or backward-looking, but not both.
- A `PREV` recurrence is evaluated in increasing order of the ordered dimension.
- A `NEXT` recurrence is evaluated in reverse order of the ordered dimension.
- A rule using both `PREV` and `NEXT` is rejected because it implies bidirectional
  dependency.

Sequential accessors use the active stable order of the referenced dimension. If both
dimension order and graph order are possible and the rule does not disambiguate,
validation fails.

## Rule precedence

When a cell is evaluated, the engine decides which rule produces the value. Two
principles control this: specificity and rule order.

### Specificity

More specific rules override less specific rules. The precedence from highest to
lowest is:

1. Hardcoded user override
2. Cell rule (single-cell rule)
3. Slice rule (rule targeted at a pattern)
4. Empty cell

A cell rule that targets `Sales::Month.Jan` wins over a slice rule that targets
`Sales::*`. A slice rule wins over an empty cell.

For example, with these rules:

```text
rule Sales::* = 100
rule Sales::Month.Jan = 200
```

The cell for `Month.Jan` resolves to `200` because the cell rule is more specific.
All other months resolve to `100` from the slice rule.

### Rule order and overriding

When multiple rules have the same specificity and match the same cell, the later rule
wins. The rule order is the semantic contract; do not rely on dictionary insertion
order.

For example:

```text
rule Sales::* = 100
rule Sales::* = 150
```

Both rules are slice rules covering the same cells. The second rule overrides the
first, so every cell resolves to `150`.

This behavior lets you define a general rule first and then add targeted overrides
later. A later rule with the same target replaces the earlier one for the cells it
matches.

## Anchored rules

An **anchored rule** targets exactly one cell. The user opts in by prefixing the rule
target with `$`.

If `Sales` has dimensions `(Year, Region, Scenario)`, then:

```text
rule $Sales::Year.2024:Region.North:Scenario.Actual = 1100   # anchored: exactly one cell
rule  Sales::Year.2024:Region.North:Scenario.Actual = 1100   # standard: same single cell, but not anchored
rule  Sales::Year.2024:Region.North = 1100                   # standard: slice across all Scenario items
```

The `$` prefix is the only anchored shorthand available in standalone scripts. The
bracket form `$[...]` is **not** accepted as a rule target; it is only valid in
contextual input such as the grid or rule panel. In scripts, prefer the explicit
form:

```text
rule Cube::Dim.Item:Dim.Item = expression
```

Syntax:

- `$` immediately before the target address marks the rule as anchored.
- The `$` is stripped before parsing the target; it carries no semantic meaning in
  the expression.

Semantics:

- **Anchored rules must resolve to exactly one cell**. If any cube dimension is
  unspecified and cannot be resolved explicitly, the rule is invalid.
- **Standard rule (no `$`)**: Unspecified dimensions use wildcards. The rule applies
  to a slice only when the cube has dimensions that are not specified in the
  target.
- **Dimension addition protection**: When a new dimension is added to a cube, an
  anchored rule does not spread across all items of that new dimension. It stays
  pinned to the address it was given; if the new dimension makes the address
  ambiguous, the rule becomes invalid until the new dimension is specified
  explicitly.

Example with a 3D cube `(Year, Region, Scenario)`:

| Target | Anchored? | Scenario selector | Coverage |
| --- | --- | --- | --- |
| `$Sales::Year.2024:Region.North:Scenario.Actual` | Yes | `Actual` | One cell: `(2024, North, Actual)` |
| `Sales::Year.2024:Region.North` | No | wildcard | Slice: `(2024, North, *)` |

In scripts, use `$` only as a prefix on an explicit cube-qualified address. In the
grid or rule panel, the contextual `$[...]` form may be used to bind a rule to one
specific cell.

### Why scripts require an explicit cube-qualified address

Standalone `.openm` files have no implicit context. The parser must know which cube,
channel, and dimension items a rule refers to without relying on the current UI
selection. The prefix form supplies that context explicitly:

```text
rule $Sales::Year.2024:Region.North:Scenario.Actual = 1100
```

Here `$Sales::Year.2024:Region.North:Scenario.Actual` is unambiguous: the rule is
anchored to exactly one cell in the `Sales` cube.

The `$[...]` bracket form is shorthand for "the currently selected cell in the active
cube, channel, and view." It only makes sense where those active selections exist,
such as the grid or rule panel. In a script there is no active selection, so `$[...]`
is undefined and should be avoided.

| Form | Where it is valid | Why |
| --- | --- | --- |
| `rule $Cube::Dim.Item:Dim.Item = ...` | Scripts, grid, rule panel | Explicit cube and items |
| `rule $[...] = ...` | Grid, rule panel only | Depends on active context |
| `rule $[...] = ...` | **Not valid** in `.openm` files | No context |

If you write a script that needs to anchor a rule, always use the full
`$Cube::Dim.Item:...` address. If the target would be ambiguous without the active
UI context, rewrite it as an explicit address before saving it to a file.

## Aggregate functions

OM Core supports aggregate functions in rule expressions. Each takes a single
slice reference as its argument.

| Function | Result |
| --- | --- |
| `SUM(address)` | Sum of all values in the slice |
| `MIN(address)` | Smallest value in the slice |
| `MAX(address)` | Largest value in the slice |
| `AVG(address)` | Average of the values in the slice |
| `COUNT(address)` | Number of numeric values in the slice |
| `COUNTA(address)` | Number of non-empty values in the slice |

Use wildcards (`*`) to select all items of one or more dimensions:

```text
SUM(Cube::Dim.Item1:Dim.*)
SUM(Cube::[Dim1.*, Dim2.*])
```

The address inside the aggregate must resolve to a slice. Wildcards are aggregated
over the referenced dimensions; any dimensions not wildcarded are bound from the
current evaluation context or selected explicitly.

Example:

```text
rule Sum::Plant.Apple:Metric.Total = SUM(Costs::[Age.*, Type.*]) * Inv::[Qty]
```

Here `SUM(Costs::[Age.*, Type.*])` adds every cost value across all ages and all
cost types for the current plant, because `Plant` is not wildcarded and binds from
the current context.

## Conditional functions

| Function | Result |
| --- | --- |
| `IF(cond, then, else)` | Returns `then` if `cond` is true, otherwise `else` |
| `IFERROR(value, fallback)` | Returns `value`, or `fallback` on error |

## Position and metadata functions

| Function | Result |
| --- | --- |
| `POS(Dim)` | 1-based position of the current item in `Dim` |
| `POSMAX(Dim)` | Number of items in `Dim` |
| `LABEL(Dim)` | Display label for the current item in `Dim` |

## Error behavior

Error values (`#CIRC!`, `#DIV/0!`, `#EXPRESSION!`, `#REF!`) are never persisted
as cell values. They are always recalculated on load. Diagnostics may be logged,
but they are not canonical cell state.

## Examples

### Simple cell rule

```text
rule Inputs::Asset.Vehicle:Metric.Cost = 50000
```

### Slice rule

```text
rule Drivers::Driver.PriceGadgets:* = 120
```

### Style rule

```text
rule PL::@.fill:PL.Revenue:Year.2026 = #3B82F6
rule PL::@.font_color:PL.Revenue:Year.2026 = #FFFFFF
```

### Recurrence rule

```text
rule PL::Revenue:* = IF(POS(Year)=1, 1000, PL::Revenue:Year[PREV] * 1.05)
```

### Anchored rule

```text
rule $Sales::Year.2024:Region.North:Scenario.Actual = 1100
```

Prefer the full script form in standalone `.openm` files:

```text
rule $Sales::Year.2024:Region.North:Scenario.Actual = Actuals::[Sales] * 1.1
```

## For LLMs

When generating rule syntax:

- Use `DimensionName.ItemName` for item references.
- Use `Cube::Dim.Item:Dim.Item` for rule addresses in standalone scripts. Add
  `@.channel` only when targeting a non-value channel.
- Use `[THIS]` for the current item, `[PREV]` / `[NEXT]` for recurrence on the RHS
  only.
- Use `*` for slice wildcards on the LHS.
- Use cube-relative shorthand (`Cube::[Dim.Item]`) only when context binding is
  unambiguous.
- Use `$` only as a prefix on an explicit cube-qualified address in scripts. Avoid
  the `$[...]` bracket form in standalone `.openm` files.
- Avoid `PREV` / `NEXT` / `FIRST` / `LAST` on the LHS.
- Avoid mixing `PREV` and `NEXT` in the same rule.
- Reference cells by stable semantic address, not grid coordinates.
