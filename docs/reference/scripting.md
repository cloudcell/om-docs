# OM Core scripting

OM Core scripts use the `.openm` extension. They are executed inside the OM Core REPL with the `source` command. Scripts are designed for use with LLMs because they are explicit, semantic, and independent of grid layout.

## Run a script

Start OM Core and source the script file:

```bash
# Start the engine
./start.sh --runtime

# ... run commands in repl mode ...
./start.sh --tui
om> source scripts/build_financial_model.openm

```

You can also run specific runtime modes directly:

```bash
./start --runtime   # headless runtime
./start --gui       # graphical interface
./start --tui       # terminal interface
```

## Script structure

A typical `.openm` script follows this order:

1. Define dimensions
2. Define cubes
3. Define views
4. Define rules
5. Calculate
6. Assert or save

```openm
# Define dimensions
 dim Asset: Vehicle Equipment Building
 dim Year: Y1 Y2 Y3 Y4 Y5
 dim Metric: Cost Salvage Life

# Define cubes
 cube Inputs Asset Metric
 cube AnnualDep Asset Year

# Define views
 view InputsView = Inputs::Asset:Metric

# Define rules
 rule Inputs::@.value:Asset.Vehicle:Metric.Cost = 50000
 rule AnnualDep::@.value:*.* = (Inputs::[Metric.Cost] - Inputs::[Metric.Salvage]) / Inputs::[Metric.Life]

# Calculate and save
 calc
 save model.json
```

## Comments

Lines starting with `#` are comments.

```openm
# This is a comment
 dim Year: 2026 2027 2028
```

## Variables

Variables store strings or command output. Use them for reusing semantic addresses, values, or messages.

### Assignment

```openm
count=42
name="test"
message="hello world"
```

### Expansion

Use `{{name}}` to interpolate a variable into a command.

```openm
selected="C::PL.Revenue:Year.2026"
echo Selected: {{selected}}
set {{selected}} format.bold true
```

### Command capture

Use `exec {{cmd}}` to execute a command and capture its output.

```openm
# Legacy $(selection) is deprecated. Use exec selection instead.
cells=exec selection
```

### Legacy syntax

The `$name`, `${name}`, and `$(command)` syntax is deprecated. Use `{{name}}` and `exec {{cmd}}` instead.

## Commands

### Model definition

#### `dim` — define a dimension

```openm
dim Year: 2026 2027 2028
```

Dimensions can also be created with sequential shorthand:

```openm
dim Year --seq 2026 2027 2028 2029 2030
```

#### `cube` — define a cube

```openm
cube PL Year
```

A cube is a multidimensional array of data defined by its dimensions.

#### `view` — define or activate a view

```openm
view PnL = PL::PL:Year
view PnL
```

The first form defines a view. The second form activates it.

### Rule definition

#### `rule` — define a calculation rule

```openm
rule Cube::@.value:Dim.Item:Dim.Item = expression
```

- `Cube::` is the target cube.
- `@.value:` selects the value channel.
- `Dim.Item:Dim.Item` is the semantic address.
- `*` is a slice wildcard.

Examples:

```openm
rule Drivers::@.value:Driver.PriceGadgets:* = 120
rule PL::@.value:PL.TotalRevenue:* = PL.RevenueGadgets + PL.RevenueWidgets
rule Valuation::@.value:Valuation.TerminalValue:Year.2032 = CF::CF.FreeCashFlow:Year.2032 * (1 + 0.02) / (Drivers::Driver.WACC:Year.2032 - 0.02)
```

### Rule syntax

Rule expressions use semantic addressing.

```openm
Dim.Item               # simple item reference
Dim[FIRST]             # first item in ordered dimension
Dim[LAST]              # last item
Dim[PREV]              # previous item (RHS only)
Dim[NEXT]              # next item (RHS only)
Dim[THIS]              # current item during rule evaluation
*                      # slice wildcard
```

References can also use bracket shorthand:

```openm
rule AnnualDep::@.value:*.* = (Inputs::[Metric.Cost] - Inputs::[Metric.Salvage]) / Inputs::[Metric.Life]
```

### Calculation

#### `calc` — recalculate the model

```openm
calc
```

#### `recalc` — force a full recalculation

```openm
recalc
```

### Persistence

#### `save` — save the workspace

```openm
save model.json
```

#### `load` — load a workspace

```openm
load model.json
```

#### `source` — execute another script

```openm
source scripts/depreciation_schedule.openm
```

### Formatting and style

#### `set` — apply a format or property

```openm
set C::PL.Revenue:Year.2026 format.bold true
set {{selected}} format.bg_color #3B82F6
set {{selected}} format.font_size 14
```

OM Core formatting uses CLDR-style number patterns. See [Formatting](formatting.md).

### Debugging

#### `echo` — print a message

```openm
echo Model built successfully
echo Total is: {{total}}
```

#### `assert` — verify a value

```openm
assert Inputs::@.value:Asset.Vehicle:Metric.Cost == 50000 "Vehicle cost"
assert Inputs::@.value:Asset.Equipment:Metric.Life == 4 "Equipment life"
```

### Selection and navigation

#### `selection` — return the current selection

```openm
selected=exec selection
```

#### `select`, `up`, `down`, `left`, `right` — navigate the grid

```openm
select C::PL.Revenue:Year.2026
up 3
right 2
```

## Semantic addresses

Semantic addresses identify cells or slices without referring to grid coordinates.

```text
Cube::@.value:Dim1.Item1:Dim2.Item2
```

Components:

- `Cube` — the cube name
- `@` or `@.value` — the channel (value, format, etc.)
- `Dim1.Item1` — a dimension item selector
- `Dim2.Item2` — another dimension item selector

Multiple selectors are separated by `:`.

## Examples

### Basic depreciation schedule

```openm
# Dimensions
 dim Asset: Vehicle Equipment Building
 dim Year: Y1 Y2 Y3 Y4 Y5
 dim Metric: Cost Salvage Life

# Cubes
 cube Inputs Asset Metric
 cube AnnualDep Asset Year
 cube AccumDep Asset Year
 cube NBV Asset Year

# Views
 view InputsView = Inputs::Asset:Metric
 view AnnualDepView = AnnualDep::Asset:Year

# Rules
 rule Inputs::@.value:Asset.Vehicle:Metric.Cost = 50000
 rule Inputs::@.value:Asset.Vehicle:Metric.Salvage = 5000
 rule Inputs::@.value:Asset.Vehicle:Metric.Life = 5
 rule AnnualDep::@.value:*.* = (Inputs::[Metric.Cost] - Inputs::[Metric.Salvage]) / Inputs::[Metric.Life]

# Calculate
 calc

# Verify
 assert Inputs::@.value:Asset.Vehicle:Metric.Cost == 50000 "Vehicle cost"

# Save
 save depreciation_schedule.json
```

### Format selected cells

```openm
# User selects cells in the GUI first
selected=exec selection
set {{selected}} format.bold true
set {{selected}} format.bg_color #3B82F6
```

## For LLMs

When generating `.openm` scripts:

- Use `{{name}}` for variable expansion, not `$name` or `${name}`.
- Use `exec {{cmd}}` for command capture, not `$(cmd)`.
- Reference cells by semantic address, not by grid coordinates.
- Place rules after dimensions and cubes.
- Run `calc` after defining rules.
- Use `assert` to verify expected values.

## See also

- [Formatting](formatting.md)
- [Rules](../concepts/rules.md)
- [Groups and Hierarchies](../concepts/groups-and-hierarchies.md)
