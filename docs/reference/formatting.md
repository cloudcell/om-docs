# Formatting patterns

OM Core uses number and currency formatting patterns based on the Unicode Common Locale Data Repository (CLDR). This page explains the origin of the syntax and the pattern types OM Core supports.

This page is about **number and currency display patterns**, not visual styles such as font color or background fill. Visual styles are set through channels like `@.fill` and `@.font_color`. See the [Scripting reference](scripting.md) for how to use style channels.

## Source

The formatting syntax in OM Core originates from CLDR number and currency patterns:

[Number and currency patterns — CLDR](https://cldr.unicode.org/translation/number-currency-formats/number-and-currency-patterns)

CLDR defines locale-aware patterns for presenting numbers, currencies, percentages, and compact numbers. OM Core uses these patterns as the basis for formatting values in views.

## Pattern types

CLDR supports four general-purpose number patterns:

- **Decimal** — standard numeric values, e.g. `1,234.56`
- **Currency** — monetary values, with standard and accounting variants
- **Percent** — values expressed as percentages
- **Scientific** — values in scientific notation

## Pattern characters

In CLDR patterns, characters such as `.` and `,` are placeholders. The actual decimal and grouping symbols are determined by the active locale. Literal characters that are not placeholders must be quoted.

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

CLDR also defines compact patterns for values like `1M` or `1 million`. These patterns vary significantly by language and may include plural categories.

## OM Core usage

When you set a format string in OM Core, you can write either a CLDR-style pattern or an OM Core preset expression.

### CLDR-style patterns

A CLDR-style pattern is a locale-aware pattern resolved against the cell or cube locale:

```openm
rule PL::@.format_number:Account.*:Year.*:Scenario.* = "#,##0.00"
```

This means the same model can be displayed correctly across different locales without changing the underlying data.

### OM Core preset expressions

A preset expression is a higher-level, readable directive that the engine maps to a concrete format. Presets are useful when you want explicit control over decimals, grouping, negative numbers, and zero display without writing a raw CLDR pattern:

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

For the full CLDR specification, see:

[Number and currency patterns — CLDR](https://cldr.unicode.org/translation/number-currency-formats/number-and-currency-patterns)
