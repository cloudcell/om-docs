# Formatting patterns

OM Core uses number and currency formatting patterns based on the Unicode Common
Locale Data Repository (CLDR). This page explains the origin of the syntax and
the pattern types OM Core supports.

This page is about **number and currency display patterns**, not visual styles such
as font color or background fill. Visual styles are set through channels like
`@.fill` and `@.font_color`. See the [Scripting reference](scripting.md) for how
to use style channels.

## Rule-driven formatting

All formatting in OM Core is rule-driven. A format string is attached to a cell
or slice through the `@.format_number` channel using the same `rule` syntax as
value rules:

```openm
rule PL::@.format_number:Account.Revenue:Year.2026 = '#,##0.00'
```

You can also set a hard value directly through the UI or REPL, but under the hood
the engine still stores it as a rule on the format channel. Because formatting
is a rule, it can reference other cells, use wildcards, and respond to the same
dimensional context as any other rule.

## Conditional formatting

Because formatting is rule-driven, it can be conditional. A rule on `@.fill`,
`@.font_color`, or `@.font_weight` can depend on the value of the same cell or
other cells:

```openm
rule Checks::@.fill:Check.Variance:Month.*:Department.* = if(abs(Variance::[VarianceLine.Variance]) > 1000, "#FFCCCC", "#FFFFFF")
rule PL::@.font_color:Account.EBITDA:Year.*:Scenario.* = if(PL::[Account.EBITDA] < 0, "#FF0000", "#000000")
```

The format or style is recomputed automatically when the underlying values change,
so the visual presentation always stays consistent with the model state.

## Source

The formatting syntax in OM Core originates from CLDR number and currency patterns:

[Number and currency patterns — CLDR](https://cldr.unicode.org/translation/number-currency-formats/number-and-currency-patterns)

CLDR defines locale-aware patterns for presenting numbers, currencies,
percentages, and compact numbers. OM Core uses these patterns as the basis for
formatting values in views.

## Pattern types

CLDR supports four general-purpose number patterns:

- **Decimal** — standard numeric values, e.g. `1,234.56`
- **Currency** — monetary values, with standard and accounting variants
- **Percent** — values expressed as percentages
- **Scientific** — values in scientific notation

## Pattern characters

In CLDR patterns, characters such as `.` and `,` are placeholders. The actual
decimal and grouping symbols are determined by the active locale. Literal
characters that are not placeholders must be quoted.

Common pattern characters:

| Character | Meaning |
| --- | --- |
| `0` | Required digit |
| `#` | Optional digit |
| `.` | Decimal placeholder |
| `,` | Grouping placeholder |
| `%` | Percent placeholder |
| `¤` | Currency symbol placeholder |

## Currency variants

CLDR currency patterns may include:

- **standard** — the default currency format
- **accounting** — used for accounting contexts
- **alphaNextToNumber** — variant used when a currency code appears next to a number
- **noCurrency** — variant without an explicit currency symbol

## Compact numbers

CLDR also defines compact patterns for values like `1M` or `1 million`. These
patterns vary significantly by language and may include plural categories.

## OM Core usage

When you set a format string in OM Core, you can write either a CLDR-style
pattern or an OM Core preset expression.

### CLDR-style patterns

A CLDR-style pattern is a locale-aware pattern resolved against the cell or cube
locale:

```openm
rule PL::@.format_number:Account.*:Year.*:Scenario.* = "#,##0.00"
```

This means the same model can be displayed correctly across different locales
without changing the underlying data.

### OM Core preset expressions

A preset expression is a higher-level, readable directive that the engine maps to
a concrete format. Presets are useful when you want explicit control over
decimals, grouping, negative numbers, and zero display without writing a raw CLDR
pattern:

```openm
rule PL::@.format_number:Account.*:Year.*:Scenario.* = 'preset:number(decimals=2; group=true; negative=parentheses; zero=dash)'
```

Common preset parameters include:

| Parameter | Meaning |
| --- | --- |
| `decimals` | Number of decimal places |
| `group` | Use thousands grouping |
| `negative` | Negative number style, e.g. `parentheses` |
| `zero` | Zero display, e.g. `dash` |

## Color maps

OM Core supports color map functions for conditional color formatting. A color map
maps a numeric value to a color along a named gradient.

### Syntax

```text
COLORMAP("name", [min; max])
```

- `name` — the color map name, e.g. `"viridis"` or `"plasma"`.
- `[min; max]` — the value range to map to the gradient.

The function returns a color for the current cell value by interpolating it
between `min` and `max` across the named gradient.

### Examples

Use `COLORMAP` on the `@.fill` or `@.font_color` channel:

```openm
rule PL::@.fill:Account.EBITDA:Year.*:Scenario.* = COLORMAP("viridis", [0; 100000])
rule Variance::@.fill:VarianceLine.VariancePct:Account.*:Month.* = COLORMAP("plasma", [-0.5; 0.5])
```

In the first example, an EBITDA value of `0` maps to one end of the Viridis
gradient and `100000` maps to the other end. Values in between receive an
interpolated color. In the second example, negative and positive variance
percentages are shown on opposite ends of the Plasma gradient.

Color maps are useful for heatmaps, variance highlighting, and any visual
formatting that should scale continuously with a numeric value.

For the full CLDR specification, see:

[Number and currency patterns — CLDR](https://cldr.unicode.org/translation/number-currency-formats/number-and-currency-patterns)
