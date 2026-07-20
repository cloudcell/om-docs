# OM Script Language Reference

This page is the formal reference for the OM Core scripting language (`.openm` files). It covers syntax grammar, all commands, types, control flow, and error codes. For a tutorial-style introduction, see [Scripting](scripting.md).

## File format

OM Core scripts use the `.openm` extension. Each line is a statement. Lines are executed sequentially by the REPL/TUI command interpreter.

## Lexical rules

| Element | Rule |
| --- | --- |
| Comments | Lines starting with `#` are comments and ignored. |
| Whitespace | Spaces and tabs separate tokens. Leading/trailing whitespace is stripped. |
| Strings | Double-quoted `"..."` or single-quoted `'...'` literals. |
| Numbers | Integer (`42`) or float (`3.14`). |
| Booleans | `true`/`yes` and `false`/`no` (case-insensitive). |
| Tuples | `(item1, item2)` or `(item1,)` — used for key parameters. |
| Identifiers | `[a-zA-Z_][a-zA-Z0-9_]*` — dimension, cube, view, variable, and item names. |
| Semantic address | `Cube::Dim.Item:Dim.Item` — see [Semantic addresses](#semantic-addresses). |

## Variables

### Assignment

Three forms are accepted. The canonical form is `var` (or `set`):

```openm
var count = 42
set name = "test"
var message = "hello world"
```

Bash-style assignment also works when there are no spaces around `=`:

```openm
count=42
name="test"
```

Use `var -g` (or `set -g`) to create a global variable that persists across macro playback:

```openm
var -g theme_color = "#3B82F6"
```

### Expansion

The canonical expansion syntax is `{{name}}`:

```openm
selected="PL::Account.Revenue:Year.2026"
echo Selected: {{selected}}
rule {{selected}} = 100000
```

### Command capture

The canonical command capture syntax is `exec {{cmd}}`:

```openm
cmd = "timestamp %Y%m%d_%H%M%S"
ts = exec {{cmd}}
```

You can also capture directly in an assignment:

```openm
ts = exec timestamp %Y%m%d_%H%M%S
```

### Legacy syntax (deprecated)

The `$name`, `${name}`, and `$(command)` syntax still works but emits a warning. Use `{{name}}` and `exec {{cmd}}` instead.

### Variable commands

| Command | Syntax | Description |
| --- | --- | --- |
| `var` | `var <name> = <value>` | Create a variable. Use `var -g` for global. |
| `set` | `set <name> = <value>` | Alias for `var`. Use `set -g` for global. |
| `vars` | `vars [name]` / `vars -g` | List all variables, or show one by name, or list globals. |
| `unset` | `unset <name>` | Delete a variable. |

## Control flow

### `if` / `then` / `else` / `elseif` / `end`

Conditional execution with comparison operators and boolean logic.

```openm
if Revenue > 1000 then echo "High revenue" end
if Quarter == "Q1" then set Target 100 else set Target 200 end
if A > B then echo "A wins" elseif A == B then echo "Tie" else echo "B wins" end
```

**Comparison operators:** `==`, `!=`, `<`, `>`, `<=`, `>=`

**Boolean operators:** `and`, `or`, `not`

### `run`

Execute an OpenM script file with control flow support.

```openm
run myscript.openm
```

Script files support all REPL commands, `if/then/else/elseif/end` blocks, variables, and conditions.

## Commands

### Model definition

#### `dim` — define a dimension

```openm
dim Region North South East West          # unordered by default
dim Region --set North South East West     # explicit unordered
dim Year --seq 2026 2027 2028 2029 2030   # ordered sequence
```

#### `dimension` — dimension sub-commands

```openm
dimension create <name> [--set|--seq] [item1 item2 ...]
dimension rename <old_name> <new_name>
dimension delete <name>
```

#### `cube` — define or manage a cube

```openm
cube PL Year                              # create (backward-compatible)
cube create <name> [dim1 dim2 ...]
cube delete <name>
cube attach <cube_name> <dim_name>
cube detach <cube_name> <dim_name>
```

#### `view` — define or activate a view

```openm
view PnL                                   # activate existing view
view PnL = Financials::Account:Year        # create simple 2D view
view PnL = Sales rows: Account cols: Month page: Scenario Version
view create V = Sales::Region:Product
view rename <old_name> <new_name>
view delete <name>
```

**Supported axes:** `rows` (vertical), `cols` (horizontal), `page` (higher-level).

#### `use` — set the active cube context

```openm
use Sales
rule Revenue = Cost * 1.15                 # resolves to Sales::Revenue
```

`use` sets the default cube for subsequent `rule` commands that omit a `Cube::` prefix.

### Rule definition

#### `rule` — define a calculation rule

```openm
rule Cube::@.channel:Dim.Item:Dim.Item = expression
rule set <target> = <expression>
rule delete <rule_id>
rule set-anchored <view_id> <cell_ref> = <expression>
rule delete-anchored <view_id> <cell_ref>
```

- `Cube::` — the target cube.
- `@.channel` — the channel the rule writes to. If omitted, `@.value` is implied. Use `@.fill`, `@.font_color`, etc. for style/format rules.
- `Dim.Item:Dim.Item` — the semantic address.
- `*` — slice wildcard.
- `$` prefix — anchored rule (attached to a specific cell).

Examples:

```openm
rule Drivers::Driver.PriceGadgets:* = 120
rule PnL::Account.TotalRevenue:* = PnL::[Account.RevenueGadgets] + PnL::[Account.RevenueWidgets]
rule $Sales::Years.2023:Products.A = 100   # anchored rule
```

#### `delete_rule` — delete a rule by ID

```openm
delete_rule rule_abc123
```

#### `set_rule_order` — set rule execution order

```openm
set_rule_order rule_a rule_b rule_c
```

#### `rules` — list rules for a cube

```openm
rules <cube_name>
```

### Cell values

#### `hval` / `hardvalue` — set a cell hardvalue

Set a user hardvalue that overrides rule computation. This is the preferred way to input data without creating per-cell rules.

```openm
hval view_id=<id> row=<n> col=<n> value=<value>
hval view_id=<id> row_key=(<id>,) col_key=(<id>,) value=<value>
```

`hardvalue` is an alias for `hval`. Both delegate to the canonical `set_cell_hardvalue` command.

Examples:

```openm
hval view_id=view_1 row=0 col=0 value=42
hval view_id=view_1 row_key=(item1,) col_key=(item2,) value=3.14
hardvalue view_id=view_1 row=0 col=0 value="hello"
```

#### `set_cell` — set a cell value by indices

```openm
set_cell view_id=<id> row=<n> col=<n> value=<value>
```

Alias for `hval` with row/col index addressing.

#### `set_cell_by_keys` — set a cell value by dimension keys

```openm
set_cell_by_keys view_id=<id> row_key=(<id>,) col_key=(<id>,) value=<value>
```

Alias for `hval` with dimension item key addressing.

#### `clear_cell` — clear a cell hardvalue

```openm
clear_cell view_id=<id> row=<n> col=<n>
```

Removes the direct stored value/override. Does NOT delete anchored rules.

#### `clear_cell_by_keys` — clear a cell hardvalue by keys

```openm
clear_cell_by_keys view_id=<id> row_key=(<id>,) col_key=(<id>,)
```

### Calculation

#### `calc` — recalculate the model

```openm
calc
calc all
```

#### `recalc` — force full recalculation

```openm
recalc
```

### Groups and outlines

#### `group` — group/outline operations

```openm
group create <dim> <label> [parent=<group>] [item1 item2 ...]
group add <dim> <group> <item1 item2 ...>
group detach <dim> <item1 item2 ...>
group delete <dim> <group>
group rename <dim> <group> <new_label>
group list <dim>
```

### Persistence

#### `save` — save the workspace

```openm
save model.json
save model ~/models/finance.openm
save macro format_blue ~/format_blue.json
```

#### `load` — load data, macros, or models

```openm
load model ~/models/finance.openm
load macro my_macro.json
load macro ~/macros/format.json --play
load data ~/data/sales.xlsx
```

#### `source` — execute another script

```openm
source scripts/depreciation_schedule.openm
```

Commands executed via `source` are not saved to command history. Nested `source` paths resolve relative to the containing script. Circular sources are detected and rejected.

### Style channels

Visual styling is applied through rule channels, not a separate formatting command.

| Channel | Description |
| --- | --- |
| `@.value` | Default value channel (implied when no channel is given) |
| `@.format_number` | Number or currency display format |
| `@.fill` | Background fill color |
| `@.font_color` | Font color |
| `@.font_weight` | Font weight (e.g. `700` for bold) |

```openm
rule C::@.fill:PL.Revenue:Year.2026 = #3B82F6
rule C::@.font_color:PL.Revenue:Year.2026 = #FFFFFF
rule C::@.font_weight:PL.Revenue:Year.2026 = 700
rule C::@.format_number:PL.Revenue:Year.2026 = 'preset:number(decimals=2; group=true)'
```

See [Formatting](formatting.md) for number format patterns.

### Debugging

#### `echo` — print a message

```openm
echo Model built successfully
echo Total is: {{total}}
```

#### `assert` — verify a condition

```openm
assert <condition> [message]
```

Halts script execution on failure. Supports numeric comparisons and OpenM rule references.

```openm
assert 5 > 3
assert count == 10 "Count should be 10"
assert Inputs::Asset.Vehicle:Metric.Cost == 50000 "Vehicle cost"
```

**Comparison operators:** `==`, `!=`, `<`, `>`, `<=`, `>=`

#### `timestamp` — print a timestamp

```openm
timestamp                          # 2026-05-03T11:42:00+12:00
timestamp %Y%m%d_%H%M%S            # 20260503_114200
```

### Selection and navigation

| Command | Syntax | Description |
| --- | --- | --- |
| `selection` | `selection` | Show current cursor position as `(row, col)`. |
| `select` | `select <row> <col>` or `select <r1> <c1> <r2> <c2>` | Select a cell or range. |
| `up` | `up [steps]` | Move selection up. |
| `down` | `down [steps]` | Move selection down. |
| `left` | `left [steps]` | Move selection left. |
| `right` | `right [steps]` | Move selection right. |

### Introspection

| Command | Syntax | Description |
| --- | --- | --- |
| `info` | `info` or `info <command_id>` | Workspace summary or detailed command info. |
| `views` | `views` | List all views. |
| `cubes` | `cubes` | List all cubes. |
| `dimensions` | `dimensions` | List all dimensions. |
| `engine` | `engine [version]` | Show engine backend type and version. |
| `list` | `list [category] [search]` | List available commands. |
| `categories` | `categories` | List all command categories. |
| `search` | `search <pattern>` | Search for commands. |
| `exec` | `exec <command_id> [key=value ...]` | Execute a registered command. |

### Macros

| Command | Syntax | Description |
| --- | --- | --- |
| `record` | `record start [--expand] <name> [description]` | Start recording a macro. |
| `record` | `record stop` | Stop recording. |
| `play` | `play <name> [--preserve-vars]` | Play back a recorded macro. |

### Engine settings

| Command | Syntax | Description |
| --- | --- |
| `set_dependency_tracking` | `set_dependency_tracking on\|off` | Toggle dependency tracking. |
| `set_multithread_recompute` | `set_multithread_recompute on\|off` | Toggle multithreaded recalculation. |

### Session

| Command | Syntax | Description |
| --- | --- | --- |
| `clear` | `clear` | Clear the screen. |
| `quit` / `exit` | `quit` | Exit the REPL. |
| `restart` | `restart` | Quit and relaunch OpenM. |

### Aliases

| Alias | Canonical |
| --- | --- |
| `cat` | `categories` |
| `ls` | `list` |
| `run` | `exec` (in command context) or `run <filename>` (script execution) |
| `recalc` | `calc` |
| `hval` | `set_cell_hardvalue` |
| `hardvalue` | `set_cell_hardvalue` |
| `set_cell` | `set_cell_hardvalue` |
| `set_cell_by_keys` | `set_cell_hardvalue` |
| `exit` | `quit` |

## Semantic addresses

Semantic addresses identify cells or slices without referring to grid coordinates.

```text
Cube::Dim1.Item1:Dim2.Item2
```

Components:

- `Cube` — the cube name.
- `Dim1.Item1` — a dimension item selector.
- `Dim2.Item2` — another dimension item selector.
- `@.channel` — optional channel specifier (before the selectors). Implied `@.value` if omitted.
- `:` — separates multiple selectors.
- `*` — slice wildcard (matches all items in that dimension).

### Sequential accessors

Sequential keywords are valid only in bracket notation on the right-hand side of rules:

```openm
Dim[FIRST]             # first item in ordered dimension
Dim[LAST]              # last item
Dim[PREV]              # previous item (RHS only)
Dim[NEXT]              # next item (RHS only)
Dim[THIS]              # current item during rule evaluation
```

Regular item references use dot notation: `DimensionName.ItemName`, not `DimensionName[ItemName]`.

### Reference resolution

OM Core resolves a shorthand reference in this order:

1. Explicit selectors in the RHS reference are applied first.
2. For any remaining dimensions in the referenced cube, OM Core carries over matching dimensions from the current target-cell context, provided the binding is unambiguous.
3. Dimensions that exist in the target cube but not in the referenced cube are ignored.
4. Any dimension that exists in the referenced cube but is neither explicitly selected nor available from the current context is ambiguous and should be written explicitly.

## Script structure

A `.openm` script typically follows this order:

1. Define dimensions
2. Define cubes
3. Define views
4. Define rules (or set hardvalues)
5. Calculate
6. Assert or save

For model bundles, use the numbered structure:

```text
00_variables
01_dimensions
02_cubes
03_inputs
04_rules
05_checks
06_views
07_formatting
08_groups
build.openm
```

`build.openm` sources the other files in order and then runs `calc`.

## Error codes

OM Core uses cell-level error codes consistent with spreadsheet conventions:

| Code | Meaning |
| --- | --- |
| `#DIV/0!` | Division by zero |
| `#NAME?` | Unrecognized or deleted name |
| `#N/A` | Value not available (reserved) |
| `#NULL!` | Intersection of ranges produced zero cells (reserved) |
| `#NUM!` | Failed to meet domain constraints |
| `#REF!` | Reference to an invalid cell |
| `#VALUE!` | Parameter is wrong type |
| `#CIRC!` | Circular dependency detected |
| `#EXPRESSION!` | Expression evaluation failure |
| `#SYNTAX!` | Syntax or reload parse error (reserved) |

Error propagation rules:

- Arithmetic operators propagate errors from any operand.
- Functions propagate errors from arguments (unless `IFERROR` is used).
- Aggregates (`SUM`, `AVERAGE`, etc.) skip error cells by default.
- `IFERROR(expr, fallback)` returns the fallback when the expression errors.

## See also

- [Scripting](scripting.md) — tutorial-style introduction
- [Rule syntax](rule-syntax.md) — detailed rule syntax reference
- [Formatting](formatting.md) — number format patterns
- [Rule engine semantics](rule-engine-semantics.md) — evaluation order and dependency tracking
- [Functions](functions.md) — built-in function reference
