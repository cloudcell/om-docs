# Formatting patterns

OM Core uses number formatting patterns that are compatible with Excel-style
masks. This page explains the syntax and the patterns OM Core supports.

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
rule Checks::@.fill:Check.Variance:Month.*:Department.* = if(Variance::[VarianceLine.Variance] > 1000, "#FFCCCC", "#FFFFFF")
rule PL::@.font_color:Account.EBITDA:Year.*:Scenario.* = if(PL::[Account.EBITDA] < 0, "#FF0000", "#000000")
```

The format or style is recomputed automatically when the underlying values change,
so the visual presentation always stays consistent with the model state.

## Number format masks

OM Core applies an Excel-compatible number mask to the raw cell value. The mask
controls decimal places, thousands grouping, percentage scaling, and negative/zero
sections.

### Pattern characters

| Character | Meaning |
| --- | --- |
| `0` | Digit placeholder |
| `#` | Digit placeholder (currently treated the same as `0`) |
| `.` | Decimal point position |
| `,` | Thousands grouping marker |
| `%` | Multiply by 100 and add `%` |

### Example masks

| Mask | Value | Display |
| --- | --- | --- |
| `#,##0.00` | `1234.5` | `1,234.50` |
| `#,##0` | `1234.6` | `1,235` |
| `0.00%` | `0.125` | `12.50%` |
| `0;(#,##0)` | `-5` | `(5)` |
| `0;(#,##0);-` | `0` | `-` |

### Sections

A mask can contain up to three semicolon-separated sections:

- **First section** — positive numbers.
- **Second section** — negative numbers.
- **Third section** — zero values.

```openm
rule PL::@.format_number:Account.EBITDA:Year.*:Scenario.* = '#,##0.00;(#,##0.00);-'
```

### Color codes and conditions

Excel-style bracket color codes such as `[Red]` or `[Blue]` are returned as literal
text by the current formatter. They are not interpreted as font colors.

### Rounding

The formatter uses Python's default rounding mode (round-half-to-even). For example,
`1234.5` formatted with `#,##0` produces `1,234`, not `1,235`.

## OM Core preset expressions

A preset expression is a higher-level directive that the engine maps to a concrete
format. Presets are useful when you want explicit control over decimals, grouping,
negative numbers, and zero display without writing a raw mask:

```openm
rule PL::@.format_number:Account.*:Year.*:Scenario.* = 'preset:number(decimals=2; group=true; negative=parentheses; zero=dash)'
```

### Supported preset kinds

| Kind | Required arguments | Notes |
| --- | --- | --- |
| `general` | none | Default, no special formatting |
| `number` | none | `decimals`, `group`, `negative`, `zero` |
| `currency` | `code` | ISO currency code, e.g. `code=USD` |
| `percent` | none | `decimals` |
| `scientific` | none | `decimals` |
| `boolean` | `style` | `true_false`, `yes_no`, or `one_zero` |
| `date` | `pattern` | Quoted pattern string |
| `time` | `pattern` | Quoted pattern string |
| `datetime` | `pattern` | Quoted pattern string |

### Common parameters

| Parameter | Meaning | Example |
| --- | --- | --- |
| `decimals` | Number of decimal places | `decimals=2` |
| `group` | Use thousands grouping | `group=true` |
| `negative` | Negative number style | `negative=parentheses` |
| `zero` | Zero display | `zero=dash` |
| `symbol` | Show currency symbol | `symbol=true` |
| `code` | ISO currency code | `code=USD` |
| `style` | Boolean display style | `style=true_false` |
| `pattern` | Date/time pattern | `pattern="yyyy-mm-dd"` |

### Notes

- Preset arguments are separated by `;` and use `=` for assignment, not `:`.
- `negative` accepts `minus` or `parentheses`.
- `zero` accepts `normal` or `dash`.
- `pattern` must be quoted.
