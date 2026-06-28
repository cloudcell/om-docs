# Formatting patterns

OM Core formats numbers through **preset expressions**. A preset is a compact,
structured directive such as `preset:number(decimals=2)` or
`preset:currency(code=USD)`. The syntax and terminology are guided by the
[Unicode Common Locale Data Repository (CLDR)](https://cldr.unicode.org/),
which defines international formatting patterns for numbers, currencies, dates,
and times. This page explains the preset syntax and the options OM Core
supports.

This page is about **number and currency display patterns**, not visual styles
such as font color or background fill. Visual styles are set through channels
like `@.fill` and `@.font_color`. See the [Scripting reference](scripting.md)
for how to use style channels.

## Rule-driven formatting

All formatting in OM Core is rule-driven. A format preset is attached to a cell
or slice through the `@.format_number` channel using the same `rule` syntax as
value rules:

```openm
rule PL::@.format_number:Account.Revenue:Year.2026 = 'preset:number(decimals=2)'
```

You can also set a hard value directly through the UI or REPL, but under the
hood the engine still stores it as a rule on the format channel. Because
formatting is a rule, it can reference other cells, use wildcards, and respond
to the same dimensional context as any other rule.

## Conditional formatting

Because formatting is rule-driven, it can be conditional. A rule on `@.fill`,
`@.font_color`, or `@.font_weight` can depend on the value of the same cell or
other cells:

```openm
rule Checks::@.fill:Check.Variance:* = if(V::[Variance] > 100, "#FCC", "#FFF")
rule PL::@.font_color:Acct.EBITDA:* = if(PL::[Acct.EBITDA] < 0, "red", "black")
```

The format or style is recomputed automatically when the underlying values
change, so the visual presentation always stays consistent with the model state.

## Color maps

`COLORMAP(palette, position)` returns a hex color string interpolated from a
built-in palette. It is useful for conditional formatting rules that map a cell
value to a color gradient.

```openm
rule PL::@.fill:Account.Revenue:Year.* = COLORMAP("viridis", @.value / 100)
rule Checks::@.fill:Check.Variance:* = COLORMAP("coolwarm", @.value / 2000)
```

- `palette` — one of the built-in palette names below.
- `position` — a number in the range `[0, 1]`. Values outside this range are
  clamped.
- Returns a hex color string such as `"#RRGGBB"`.

### Available palettes

| Palette | Type |
| --- | --- |
| `viridis` | Perceptually uniform, colorblind-friendly |
| `plasma` | Perceptually uniform, warmer |
| `coolwarm` | Diverging blue-to-red |
| `rdylgn` | Diverging red-yellow-green |
| `blues` | Sequential blue |
| `greens` | Sequential green |
| `grayscale` | Black-to-white |

## OM Core preset expressions

A preset expression is a higher-level directive that the engine maps to a
concrete format. It gives explicit control over decimals, grouping, negative
numbers, and zero display:

```openm
rule PL::@.format_number:Acct.*:Year.* = 'preset:number(decimals=2; group=true)'
```

The `@.format_number` channel accepts only `general` or `preset:...`
expressions. Raw Excel-style masks such as `#,##0.00` are not supported. The
`pattern:...` prefix is also not supported in the current version.

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
