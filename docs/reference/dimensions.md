# Dimensions reference

A dimension is a named axis of items used by one or more cubes. Dimension items
are stable semantic labels, not grid coordinates. They are declared once and
reused across cubes so the same terminology applies everywhere in a model.

## Declaration

Dimensions are declared with the `dim` command:

```openm
dim Region North South East West
dim Year --seq 2026 2027 2028 2029 2030
dim Scenario Actual Base Upside Downside
```

A dimension declaration has the form:

```text
dim <name> [--set | --seq] <item1> <item2> ... <itemN>
```

- `name` — the dimension identifier. Use `PascalCase` or descriptive names.
- `--set` — explicit unordered set. This is the default when no flag is given.
- `--seq` — ordered sequence. Item order matters for navigation, indexing, and
  sequential references.
- Items are separated by whitespace. Multi-word items must be quoted.

### Sequential ranges

For `seq` dimensions, a range expression `start..end` expands to a consecutive
sequence of intermediary items in rules. This is useful for time and index dimensions.

Ranges are only valid on `seq` dimensions. The engine expands the range into a
sequence of items at declaration time, so the resulting items behave exactly like
explicitly listed items.

## Dimension types

### `set` — unordered set

```openm
dim Region North South East West
```

A `set` dimension stores its items as an unordered collection. The order in which
items are listed in the declaration is preserved for display purposes, but the
engine does not treat it as meaningful for calculations. Items are accessed by
name, not by position.

Use `set` for dimensions where items are categories without natural ordering:

- `Region`
- `Department`
- `Scenario`
- `Product`

### `seq` — ordered sequence

```openm
dim Month --seq Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
```

A `seq` dimension is an ordered sequence. Item order is part of the dimension's
semantics. Ordering enables sequential accessors and affects iteration, display,
and recurrence rules.

Use `seq` for dimensions with natural ordering, especially time:

- `Month`, `Quarter`, `Year`
- `Version` when versions follow a progression
- `PlantAge` when age stages follow a sequence

### Why the distinction matters

Sequential references only resolve correctly on `seq` dimensions:

```openm
dim Year --seq 2026 2027 2028 2029 2030

rule Cash::Cash.EndingCash:Year.2027 = Cash::[Cash.EndingCash:Year.PREV] + Cash::[Cash.FreeCashFlow:Year.THIS]
```

If `Year` were declared as a `set`, `Year.PREV` and `Year.NEXT` would produce
`#REF!` because the engine cannot determine the previous or next item without an
explicit order.

The same applies to `FIRST`, `LAST`, and `THIS`:

```openm
dim Month --seq Jan Feb Mar

rule PL::PLLine.Revenue:Month.Feb = PL::[PLLine.Revenue:Month.PREV] * 1.10
rule PL::PLLine.Revenue:Month.Jan = PL::[PLLine.Revenue:Month.FIRST]
```

## Channels and the `@` sigil

Every cell in a cube stores values in named channels. The `@` sigil introduces a
channel selector in a rule address.

### Default value channel

The default channel is `@.value`. It is implied when no channel is written:

```openm
# These two rules are equivalent
rule Sales::Month.Jan = 1000
rule Sales::@.value:Month.Jan = 1000
```

### Style and format channels

Style and formatting rules target non-value channels:

```openm
rule PL::@.fill:PLLine.Revenue:Year.2026 = #3B82F6
rule PL::@.font_color:PLLine.Revenue:Year.2026 = #FFFFFF
rule PL::@.font_weight:PLLine.Revenue:Year.2026 = 700
rule PL::@.format_number:PLLine.Revenue:Year.2026 = 'preset:number(decimals=2; group=true)'
```

Common channels include:

| Channel | Purpose |
| --- | --- |
| `@.value` | Default data channel |
| `@.fill` | Background color |
| `@.font_color` | Text color |
| `@.font_weight` | Text weight, e.g. `700` for bold |
| `@.font_italic` | Italic flag |
| `@.font_size` | Text size |
| `@.format_number` | Number or currency display format |
| `@.text_h_align` | Horizontal alignment |
| `@.text_v_align` | Vertical alignment |
| `@.border_*` | Border style and color |

### Strict and shorthand forms

A full rule address includes the channel explicitly:

```text
Cube::@.channel:Dim1.Item1:Dim2.Item2
```

Most value rules omit the channel for readability:

```text
Cube::Dim1.Item1:Dim2.Item2
```

When the channel is omitted, `@.value` is assumed. Style and format rules must
always include the channel.

### Channel references on the RHS

The `@` sigil is only needed when writing a rule that targets a non-value
channel. RHS references to other cells use the same channel as the target unless
explicitly overridden. For example, a rule on `@.fill` reads the `@.fill` channel
of the referenced cell:

```openm
rule Summary::@.fill:Plant.Apple = Inventory::@.fill:[PlantData.Status]
```

A value-rule reference reads the value channel by default:

```openm
rule PL::PLLine.GrossProfit:Year.2026 = PL::[PLLine.Revenue] - PL::[PLLine.COGS]
```

## Dimension item references

Items are referenced by `DimensionName.ItemName`:

```openm
rule PL::PLLine.Revenue:Year.2026 = 100000
```

Use brackets for shorthand references when the item name contains spaces or
special characters, or when using positional accessors:

```openm
rule PL::PLLine.Revenue:Year.2026 = PL::[PLLine.COGS] * 2.0
rule PL::PLLine.Revenue:Year.2027 = PL::[PLLine.Revenue:Year.PREV] * 1.10
```

## Best practices

- Declare time dimensions with `--seq` so `[PREV]`, `[NEXT]`, `[FIRST]`, and
  `[LAST]` work.
- Use `set` for categorical dimensions. Avoid implying order that does not exist.
- Keep dimension item names stable. Changing an item name may (still) break rules
  that reference it. If this happens, update the rules to use the new item name.
- Omit `@.value` in value rules for readability, but always include the channel
  in style and format rules.
- Use descriptive dimension names. Short names like `A` and `B` are acceptable
  only for syntax examples, not production models.

## See also

- [Concepts: Dimensions](../concepts/dimensions.md)
- [Rule syntax](rule-syntax.md)
- [Scripting reference](scripting.md)
