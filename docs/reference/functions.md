# Rule functions reference

This page lists all functions that can be used inside OM Core rule bodies. Function names are case-insensitive in rules but are shown here in uppercase.

## Conditional and logical functions

| Function | Arguments | Behavior |
| --- | --- | --- |
| `IF` | `IF(condition, then_value, else_value)` | Evaluates `condition`; returns `then_value` if truthy, otherwise `else_value`. |
| `AND` | `AND(value1, value2, ...)` | Returns `1` if all arguments are truthy/non-zero, otherwise `0`. Errors propagate. |
| `OR` | `OR(value1, value2, ...)` | Returns `1` if any argument is truthy/non-zero, otherwise `0`. Errors propagate. |
| `NOT` | `NOT(value)` | Returns `1` if `value` is zero or falsy, otherwise `0`. |
| `XOR` | `XOR(value1, value2, ...)` | Returns `1` if an odd number of arguments are truthy/non-zero. |
| `IFERROR` | `IFERROR(value, fallback)` | Returns `fallback` if `value` is an error or raises during evaluation; otherwise returns `value`. |
| `XLS_IF` | `XLS_IF(condition, then_value, else_value)` | Excel-style conditional. |
| `XLS_TRUE` | `XLS_TRUE()` | Returns `1`. |
| `XLS_FALSE` | `XLS_FALSE()` | Returns `0`. |

## Aggregate functions

These functions accept either a single cube reference (e.g., `SUM(Dim.Item)`) or multiple evaluated values.

| Function | Arguments | Behavior |
| --- | --- | --- |
| `SUM` | `SUM(ref)` or `SUM(value1, value2, ...)` | Returns the sum of numeric values. Empty set returns `0`. |
| `MIN` | `MIN(ref)` or `MIN(value1, value2, ...)` | Returns the minimum numeric value. Empty set returns `0`. |
| `MAX` | `MAX(ref)` or `MAX(value1, value2, ...)` | Returns the maximum numeric value. Empty set returns `0`. |
| `AVG` / `AVERAGE` | `AVG(ref)` or `AVG(value1, value2, ...)` | Returns the arithmetic mean. Empty set returns `#DIV/0!`. |
| `COUNT` | `COUNT(ref)` or `COUNT(value1, value2, ...)` | Counts numeric values. |
| `COUNTA` | `COUNTA(ref)` or `COUNTA(value1, value2, ...)` | Counts non-empty values (including text). |
| `COUNTIF` | `COUNTIF(range, criteria)` | Counts values in `range` that match `criteria`. Criteria may be a number, exact text, wildcard (`*`, `?`), or comparison (`>`, `<`, `>=`, `<=`, `=`, `<>`). |
| `COUNTIFS` | `COUNTIFS(range1, criteria1, range2, criteria2, ...)` | Counts positions where all range/criteria pairs match. |
| `XLS_SUM` | `XLS_SUM(value1, value2, ...)` or `XLS_SUM(array_ref)` | Excel-style sum. |
| `XLS_MIN` | `XLS_MIN(value1, value2, ...)` | Returns the minimum numeric value. Empty set returns `0`. |
| `XLS_MAX` | `XLS_MAX(value1, value2, ...)` | Returns the maximum numeric value. Empty set returns `0`. |
| `XLS_AVG` / `XLS_AVERAGE` | `XLS_AVG(value1, value2, ...)` | Returns the arithmetic mean. Empty set returns `0`. |
| `XLS_COUNT` | `XLS_COUNT(value1, value2, ...)` | Counts numeric values. |

## Mathematical functions

| Function | Arguments | Behavior |
| --- | --- | --- |
| `ABS` | `ABS(x)` | Absolute value. |
| `ROUND` | `ROUND(x, places=0)` | Rounds `x` to `places` decimal places. |
| `ROUNDUP` | `ROUNDUP(x, places=0)` | Rounds away from zero. |
| `ROUNDDOWN` | `ROUNDDOWN(x, places=0)` | Rounds toward zero. |
| `PI` | `PI()` | Returns π. |
| `LN` | `LN(x)` | Natural logarithm. Returns `#NUM!` for `x <= 0`. |
| `LOG` | `LOG(x, base=10)` | Logarithm of `x` with optional base. Default is base 10. |
| `LOG10` | `LOG10(x)` | Base-10 logarithm. Returns `#NUM!` for `x <= 0`. |
| `EXP` | `EXP(x)` | e raised to `x`. Returns `#NUM!` on overflow. |
| `SQRT` | `SQRT(x)` | Square root. Returns `#NUM!` for `x < 0`. |
| `POWER` | `POWER(base, exponent)` | Returns `base ** exponent`. |
| `SIN` | `SIN(x)` | Sine of `x` (radians). |
| `COS` | `COS(x)` | Cosine of `x` (radians). |
| `TAN` | `TAN(x)` | Tangent of `x` (radians). |
| `ASIN` | `ASIN(x)` | Arcsine. Returns `#NUM!` if `\|x\| > 1`. |
| `ACOS` | `ACOS(x)` | Arccosine. Returns `#NUM!` if `\|x\| > 1`. |
| `ATAN` | `ATAN(x)` | Arctangent. |
| `ATAN2` | `ATAN2(y, x)` | Arctangent of `y/x` with quadrant. |
| `RADIANS` | `RADIANS(degrees)` | Converts degrees to radians. |
| `DEGREES` | `DEGREES(radians)` | Converts radians to degrees. |
| `SIGN` | `SIGN(x)` | Returns `1`, `-1`, or `0` based on the sign of `x`. |
| `INT` | `INT(x)` | Truncates `x` to an integer (returns float). |
| `MOD` | `MOD(a, b)` | Returns `a % b`. Raises `#DIV/0!` if `b == 0`. |
| `QUOTIENT` | `QUOTIENT(a, b)` | Integer division `a // b`. Raises `#DIV/0!` if `b == 0`. |
| `XLS_ABS` | `XLS_ABS(x)` | Excel-style absolute value. |
| `XLS_ROUND` | `XLS_ROUND(x, places=0)` | Excel-style rounding. |
| `XLS_VALUE` | `XLS_VALUE(x)` | Converts text/number to a float. |
| `XLS_RAND` | `XLS_RAND()` | Returns a random float in `[0, 1)`. |
| `XLS_RANDBETWEEN` | `XLS_RANDBETWEEN(bottom, top)` | Returns a random integer between `bottom` and `top` inclusive. |

## String functions

| Function | Arguments | Behavior |
| --- | --- | --- |
| `LEN` | `LEN(text)` | Returns the length of `text` as a number. |
| `TRIM` | `TRIM(text)` | Removes leading/trailing whitespace. |
| `LEFT` | `LEFT(text, num_chars=1)` | Returns the first `num_chars` characters. |
| `RIGHT` | `RIGHT(text, num_chars=1)` | Returns the last `num_chars` characters. |
| `REPT` | `REPT(text, num_times)` | Repeats `text` `num_times`. Truncated to 1024 characters. |
| `CODE` | `CODE(text)` | Returns the ASCII code of the first character, or `0` if empty. |
| `CHAR` | `CHAR(code)` | Returns the character for ASCII code `1`–`255`. |
| `JOIN` | `JOIN(list, delimiter)` | Joins a list of values into a string with `delimiter`. |
| `XLS_CONCATENATE` | `XLS_CONCATENATE(text1, text2, ...)` | Concatenates arguments into one string. |
| `XLS_UPPER` | `XLS_UPPER(text)` | Converts text to uppercase. |
| `XLS_REPT` | `XLS_REPT(text, num_times)` | Excel-style repeat. Truncated to 1024 characters. |
| `XLS_CODE` | `XLS_CODE(text)` | Excel-style code of first character. |
| `XLS_CHAR` | `XLS_CHAR(code)` | Excel-style character. |
| `XLS_TEXT` | `XLS_TEXT(value, format)` | Formats a number using an Excel-style mask. |

## Date and time functions

Excel serial dates are used internally. Dates are represented as numbers relative to the Excel epoch (`1899-12-30`).

| Function | Arguments | Behavior |
| --- | --- | --- |
| `XLS_DATE` | `XLS_DATE(year, month, day)` | Returns the serial date for the given date. |
| `XLS_TODAY` | `XLS_TODAY()` | Returns the serial date for today. |
| `XLS_NOW` | `XLS_NOW()` | Returns the serial date/time for the current moment. |
| `XLS_YEAR` | `XLS_YEAR(date)` | Returns the year of a date value. |
| `XLS_EOMONTH` | `XLS_EOMONTH(start_date, months)` | Returns the serial date for the last day of the month offset by `months`. |

## Financial functions

| Function | Arguments | Behavior |
| --- | --- | --- |
| `XLS_NPV` | `XLS_NPV(rate, cashflow1, cashflow2, ...)` or `XLS_NPV(rate, array_ref)` | Net present value. Requires `rate > -1`. |
| `XLS_IRR` | `XLS_IRR(array_ref, guess=0.1)` | Internal rate of return. Requires at least one positive and one negative cash flow. |
| `XLS_XIRR` | `XLS_XIRR(values_ref, dates_ref, guess=0.1)` | IRR with irregular dates. Requires equal non-empty arrays and mixed cash flows. |

## Lookup and reference functions

These functions operate on cube references and treat dimensions named `Row` and `Column` as axes when present.

| Function | Arguments | Behavior |
| --- | --- | --- |
| `XLS_INDEX` | `XLS_INDEX(ref, row_num, col_num=1)` | Returns the cell at `row_num`/`col_num` within the referenced range. |
| `XLS_OFFSET` | `XLS_OFFSET(ref, rows, cols, height=1, width=1)` | Returns a cell or array offset from the reference by `rows`/`cols`. Supports negative dimensions. Returns a scalar for 1x1 or a list otherwise. |
| `XLS_MATCH` | `XLS_MATCH(lookup_value, lookup_array, match_type=0)` | Returns the 1-based position of an exact match. Currently only `match_type = 0` is supported. |
| `XLS_ROWS` | `XLS_ROWS(ref)` | Returns the number of rows in the referenced range. |
| `XLS_COLUMNS` | `XLS_COLUMNS(ref)` | Returns the number of columns in the referenced range. |
| `XLS_HLOOKUP` | `XLS_HLOOKUP(lookup_value, table_array, row_index, range_lookup=TRUE)` | Horizontal lookup. Searches the first row for `lookup_value` and returns the value from `row_index`. |
| `XLS_VLOOKUP` | `XLS_VLOOKUP(lookup_value, table_array, col_index, range_lookup=TRUE)` | Vertical lookup. Searches the first column for `lookup_value` and returns the value from `col_index`. |
| `XLS_CHOOSE` | `XLS_CHOOSE(index, value1, value2, ...)` | Returns the argument selected by `index` (1-based). |

## Hierarchy navigation functions

These functions return lists of item references that can be passed to aggregate functions like `SUM()`.

| Function | Arguments | Behavior |
| --- | --- | --- |
| `DESC` | `DESC(Dim.Item)` | Returns descendant leaf item references under `Dim.Item`. |
| `ANCE` | `ANCE(Dim.Item)` | Returns ancestor item references (parent chain) for `Dim.Item`. |
| `PEER` | `PEER(Dim.Item)` | Returns peer (same-level) item references. |
| `SIBL` | `SIBL(Dim.Item)` | Returns sibling item references. |
| `CHIL` | `CHIL(Dim.Item)` | Returns immediate child item references. |
| `PARE` | `PARE(Dim.Item)` | Returns the parent item reference (single-item list). |

## Metadata functions

| Function | Arguments | Behavior |
| --- | --- | --- |
| `LABEL` | `LABEL()` or `LABEL(Dim)` | Returns the label for the current cell address, or the label for the given dimension item. |
| `POS` | `POS(Dim)` | Returns the 1-based position of the current item in `Dim`. |
| `POSMAX` | `POSMAX(Dim)` | Returns the number of items in `Dim`. |

## Array and reference inspection functions

| Function | Arguments | Behavior |
| --- | --- | --- |
| `SLICE` | `SLICE(ref1, ref2, ...)` | Returns a list of cell values from the Cartesian product of the given references. Supports dynamic references returned by navigation functions. |
| `REF` | `REF(ref1, ref2, ...)` | Same syntax as `SLICE` but returns a list of coordinate tuples (`Dim.Item`, ...) for debugging. |
| `VALUE` | `VALUE(x)` | Converts `x` to a number; text that cannot be parsed returns `#VALUE!`. |

## Color functions

These functions return hex color strings (`#RRGGBB`) for use in format/style rules.

| Function | Arguments | Behavior |
| --- | --- | --- |
| `COLORMAP` | `COLORMAP(palette_name, position)` | Returns a color from a named palette. Available palettes: `viridis`, `plasma`, `coolwarm`, `rdylgn`, `blues`, `greens`, `grayscale`. `position` is clamped to `[0, 1]`. |
| `HSV2RGB` | `HSV2RGB(hue, saturation, value)` | Converts HSV (`hue` 0–360, `saturation`/`value` 0–1) to a hex color. |
| `RGB` | `RGB(red, green, blue)` | Converts RGB values (0–255) to a hex color. |

## Volatile functions

These functions are recalculated on every recompute and may be cached per render.

| Function | Arguments | Behavior |
| --- | --- | --- |
| `RAND` | `RAND()` | Random float in `[0, 1)`. |
| `RANDBETWEEN` | `RANDBETWEEN(bottom, top)` | Random integer between `bottom` and `top` inclusive. |

## User-defined functions (UDFs)

UDFs are workspace-scoped functions defined in `.openm` script files located at:

- `~/.om/udf/*.openm` — global defaults
- `<workspace>/.udf/*.openm` — workspace-specific functions

A UDF is defined with the syntax:

```text
UDFNAME(param1, param2) = expression
```

UDF names must be valid identifiers, normalized to uppercase, and must not conflict with built-in function names. Once loaded, a UDF can be called inside any rule body like a built-in function.

## Rule errors

Rule evaluation uses a single `CellError` type that carries a string code. The codes are grouped into families that mirror the order in which a rule is processed: parsing first, reference resolution next, arithmetic near the end, and a final catch-all for anything that cannot be classified.

### Error family order

| Family | Codes | Stage |
| --- | --- | --- |
| 0 | `#SYNTAX!` | Pre-evaluation gate. |
| 1 | `#NAME!`, `#REF!` | Name and reference resolution. |
| 2 | `#SHAPE!` | Shape / dimensionality checks. |
| 3 | `#VALUE!` | Type and value-kind errors. |
| 4 | `#N/A` | Lookup / availability failures. |
| 5 | `#DIV/0!`, `#NUM!` | Arithmetic and numeric errors. |
| 6 | `#CIRC!` | Circular dependency detection. |
| 7 | `#EXPRESSION!` | Expression fallback for unhandled failures. |

### Why the order matters

A rule is evaluated in a pipeline. The earlier a problem is detected, the more specific the error code can be. Later families are broader or depend on earlier stages having already succeeded:

1. **Syntax** (`#SYNTAX!`) is the first gate: the expression must tokenize and parse into a valid AST before anything else happens.
2. **Name / reference** (`#NAME!`, `#REF!`) comes after parsing but before computation. If a referenced dimension or item does not exist, the engine cannot proceed.
3. **Shape** (`#SHAPE!`) covers dimensionality mismatches, for example a multi-dimension reference that does not fit the cube it is applied to.
4. **Value** (`#VALUE!`) covers type-kind problems: text where a number is expected, invalid conversions, or malformed function arguments.
5. **Lookup / availability** (`#N/A`) covers cases where a value is structurally valid but not available in the current context.
6. **Arithmetic** (`#DIV/0!`, `#NUM!`) covers errors that only appear during numeric calculation: division by zero, square roots of negatives, overflow, etc.
7. **Circular dependency** (`#CIRC!`) is detected after the engine has started resolving references; it means the rule eventually depends on itself.
8. **Expression fallback** (`#EXPRESSION!`) is the last resort: any unexpected exception that is not mapped to a more specific family is reported here.

### Error code reference

| Code | Family | When it is raised |
| --- | --- | --- |
| `#SYNTAX!` | 0 | Reserved for pre-evaluation syntax failures. Currently defined but not raised by the rule engine. |
| `#NAME!` | 1 | A dimension, item, cube, or function name cannot be resolved. |
| `#REF!` | 1 | A cell reference is invalid or cannot be looked up. Raised when `resolve_ref` or `resolve_multi_ref` fails with `KeyError` or `ValueError`. |
| `#SHAPE!` | 2 | Reserved for shape / dimensionality mismatches. Currently defined but not raised by the rule engine. |
| `#VALUE!` | 3 | A value is the wrong kind for the operation, for example text that cannot be converted to a number in `VALUE()` or `XLS_VALUE()`. |
| `#N/A` | 4 | Reserved for lookup / availability failures. Currently defined but not raised by the rule engine. |
| `#DIV/0!` | 5 | Division by zero, for example `MOD(a, 0)` or `QUOTIENT(a, 0)`. Also raised by Python `ZeroDivisionError` during rule evaluation. |
| `#NUM!` | 5 | A numeric operation is invalid, for example `SQRT(-1)`, `LN(0)`, or overflow. Also raised by Python `OverflowError` during rule evaluation. |
| `#CIRC!` | 6 | A circular dependency is detected while resolving references. Also raised by `CircularReferenceError` during rule evaluation. |
| `#EXPRESSION!` | 7 | Any unexpected runtime exception that is not mapped to a more specific code. |

### Exceptions vs. cell errors

The engine distinguishes between two kinds of failures:

- **`CellError`** — returned as the value of a cell. The user sees the code (`#DIV/0!`, etc.) in the model.
- **`RuleValidationError`** — raised to the caller, not stored in a cell. It is used for validation failures that should be fixed before the rule can run, such as invalid dynamic bounds or malformed references.

During rule evaluation, the engine maps Python exceptions to `CellError` codes as follows:

| Python exception | Cell error |
| --- | --- |
| `ZeroDivisionError` | `#DIV/0!` |
| `OverflowError` | `#NUM!` |
| `CircularReferenceError` | `#CIRC!` |
| `RuleValidationError` | re-raised (not stored in cell) |
| any other `Exception` | `#EXPRESSION!` |

Errors propagate through function calls: if any argument evaluates to a `CellError`, most functions return that error immediately rather than computing a result.
