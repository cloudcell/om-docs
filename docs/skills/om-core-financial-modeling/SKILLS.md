# OM Core Financial Modeling Skill

This skill teaches an agent how to build financial, operational, and analytical models in OM Core using `.openm` scripts, reusable templates, variables, and `{{...}}` macro expansion.

The agent's job is to produce a coherent script bundle that can be loaded into OM Core, not merely to describe a model in prose.

## 1. Objective

When asked to build a financial model, the agent must create an OM Core model using structured dimensions, cubes, rules, views, and checks.

The model should be:

* semantic rather than cell-oriented
* organized around dimensions and cubes
* readable as source-controlled scripts
* split into reusable script files where appropriate
* parameterized using variables and `{{...}}` macro expansion
* auditable through explicit checks and clearly named rules
* suitable for loading through OM Core's scripting interface

## 2. Required Workflow

For every model-building task, follow this workflow:

1. Identify the business purpose of the model.
2. Identify required dimensions.
3. Identify required cubes.
4. Identify input assumptions.
5. Identify calculated outputs.
6. Identify checks.
7. Decide script file structure.
8. Define variables for repeated names, years, scenarios, and cube names.
9. Write the `.openm` scripts.
10. Add views for user-facing inspection.
11. Add checks where possible.
12. Provide run instructions.

Do not start with formulas. Start with model structure.

## 3. Script Organization

Prefer a multi-file script bundle for non-trivial models.

Recommended structure:

```text
model/
  00_variables.openm
  01_dimensions.openm
  02_cubes.openm
  03_inputs.openm
  04_rules.openm
  05_checks.openm
  06_views.openm
  07_formatting.openm      # optional but recommended
  08_groups.openm          # optional presentation outline
  build.openm
```

The `build.openm` file should source the other files in order:

```openm
source 00_variables.openm
source 01_dimensions.openm
source 02_cubes.openm
source 03_inputs.openm
source 04_rules.openm
source 05_checks.openm
source 06_views.openm
source 07_formatting.openm
source 08_groups.openm
calc
```

Use fewer files for small examples, but keep this order conceptually. Small models may omit `07_formatting.openm` and `08_groups.openm`.

## 4. Variables and Macro Expansion

Use variables to avoid repeating important names.

Example:

```openm
model_name="SaaS Revenue Model"
pl_cube="PL"
account_dim="Account"
month_dim="Month"
scenario_dim="Scenario"
base_scenario="Base"
```

Use `{{...}}` macro expansion to compose commands:

```openm
cube {{pl_cube}} {{account_dim}} {{month_dim}} {{scenario_dim}}
view PL_View = {{pl_cube}}::{{account_dim}}:{{month_dim}}
```

Use macro expansion for:

* cube names
* dimension names
* scenario names
* repeated address fragments
* reusable script templates
* model variants

Avoid using macro expansion to hide business logic. Rules should remain readable after expansion. In concrete example bundles, variables are typically used for cube and dimension declarations, while cube names in rules and views may be hardcoded for readability. Reserve full macro saturation for reusable templates and model variants.

## 5. Naming Conventions

Use clear business names.

Good:

```openm
dim Account Revenue COGS GrossProfit OperatingExpense EBITDA
dim Month Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
dim Scenario Actual Base Upside Downside
cube PL Account Month Scenario
```

Avoid vague names:

```openm
dim A x y z
cube C A B
```

For documentation examples, short names are acceptable only when teaching syntax.

## 6. Standard Financial Model Dimensions

Use these standard dimensions when appropriate.

### Time

For short examples:

```openm
dim Month --seq Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
```

For annual models:

```openm
dim Year --seq 2026 2027 2028 2029 2030
```

For quarterly models:

```openm
dim Quarter --seq Q1 Q2 Q3 Q4
```

Time dimensions must be declared with `--seq` so that sequential accessors (`Month[PREV]`, `Year[NEXT]`, `Quarter[THIS]`, etc.) resolve correctly. Without `--seq`, the engine reports `#REF!` for these references.

### Scenario

```openm
dim Scenario Actual Base Upside Downside
```

Use Scenario as a first-class dimension. Do not create separate cubes for scenarios unless there is a strong reason.

### Account / Line Item

For P&L models:

```openm
dim Account Revenue COGS GrossProfit OperatingExpense EBITDA Depreciation EBIT Tax NetIncome
```

For cash flow models:

```openm
dim CashFlowLine EBITDA ChangeInWorkingCapital Capex Tax PaidInterest FreeCashFlow
```

For balance sheet models:

```openm
dim BalanceSheetLine Cash AR Inventory FixedAssets AP Debt Equity
```

### Product / Customer / Region

Use only when needed:

```openm
dim Product Basic Pro Enterprise
dim Region North America Europe APAC
dim CustomerSegment SMB MidMarket Enterprise
```

## 7. Standard Cubes

Separate assumptions, calculations, and outputs when it improves clarity.

Common cubes:

```openm
cube Drivers Driver Month Scenario
cube PL Account Month Scenario
cube Checks Check Month Scenario
```

For simple models, one cube may be enough:

```openm
cube PL Account Month Scenario
```

For larger models, prefer:

```openm
cube Revenue RevenueLine Month Scenario
cube Expenses ExpenseLine Month Scenario
cube PL Account Month Scenario
cube Checks Check Month Scenario
```

## 8. Rule-Writing Conventions

Use rules to express business logic over semantic addresses.

Write each rule on a single line. The REPL and `source` command process one line at a time, so a rule body that continues on the next line is ignored.

Prefer readable business rules:

```openm
rule PL::Account.GrossProfit:Month.*:Scenario.* = PL::[Account.Revenue] - PL::[Account.COGS]
```

Use explicit addresses only for the dimensions you need to override. Omitted dimensions bind from the current rule context.

```openm
rule PL::Account.EBITDA:Month.*:Scenario.* = PL::Account.GrossProfit:Month[THIS] - PL::Account.OperatingExpense:Month[THIS]
```

Use full addresses for recurrence rules that reference a previous or next item:

```openm
rule SaaSPL::Account.Customers:Month.*:Scenario.* = SaaSPL::Account.Customers:Month[PREV] + Drivers::[Driver.NewCustomers]
```

Do not combine bracket shorthand with sequential accessors such as `SaaSPL::[Account.Customers]:Month[PREV]`. Bracket shorthand resolves only one dimension and carries over the rest; it does not support explicit `:Month[PREV]` overrides. For recurrence references, write the full canonical address.

`[THIS]`, `[PREV]`, `[NEXT]`, `[FIRST]`, and `[LAST]` require the dimension to be declared with `--seq`. Do not use them on non-sequential dimensions. For non-sequential dimensions, either omit them and let the engine bind from context, or use shorthand so all dimensions bind from context.

Use shorthand only when it is unambiguous.

If a referenced cube omits a dimension, the omitted dimension may bind from the current target-cell context only when the referenced cube has that dimension. Dimensions absent from the referenced cube are ignored. If a referenced cube has a dimension that is neither explicit nor bindable from context, the reference is ambiguous and should be written explicitly.

Do not rely on hidden first-item defaults.

## 9. Inputs

Input assumptions should be defined explicitly.

Example:

```openm
rule Drivers::Driver.StartingCustomers:Month.Jan:Scenario.Base = 100
rule Drivers::Driver.MonthlyGrowthRate:Month.*:Scenario.Base = 0.05
rule Drivers::Driver.ARPU:Month.*:Scenario.Base = 50
```

Use hardcoded values in examples only when necessary. In production-style scripts, keep assumptions in dedicated input cubes.

### Hard values vs. input rules

A `rule` with a literal value on the right-hand side is the standard way to seed inputs in an `.openm` script. It is readable, source-controlled, and auditable.

For interactive or GUI-driven edits after the model is loaded, set a cell hardvalue with:

```text
om> set_cell_by_keys view_id=<view_id> row_key=(<item>,) col_key=(<item>,) value=<value>
```

Example after the agroforestry views are created:

```text
om> set_cell_by_keys view_id=Inventory_View row_key=(Apple,) col_key=(Quantity_m2,) value=1500
```

`set_cell_by_keys` creates a user override that survives recalculation. Clear it with `clear_cell_by_keys`. Prefer input rules for model bundles; reserve hard values for manual edits or imported data.

## 10. Checks

Every serious financial model should include checks.

Examples:

```openm
dim Check BalanceCheck GrossProfitCheck CashFlowCheck
cube Checks Check Month Scenario
```

Example rules:

```openm
rule Checks::Check.GrossProfitCheck:Month.*:Scenario.* = PL::[Account.GrossProfit] - (PL::[Account.Revenue] - PL::[Account.COGS])
```

A check should equal zero when the model is valid.

Prefer check names that describe the invariant being tested.

## 11. Views

Create at least one view for every cube in the model.

Example:

```openm
view PL_View = PL::Account:Month
view Drivers_View = Drivers::Driver:Month
view Checks_View = Checks::Check:Month
```

Views are saved projections of cubes. They are not the source of truth for calculations, but they let you inspect every cube quickly.

## 12. Template Pattern

For reusable model templates, use variables at the top.

Example template:

```openm
pl_cube="{{pl_cube}}"
account_dim="{{account_dim}}"
time_dim="{{time_dim}}"
scenario_dim="{{scenario_dim}}"

cube {{pl_cube}} {{account_dim}} {{time_dim}} {{scenario_dim}}

rule {{pl_cube}}::{{account_dim}}.GrossProfit:{{time_dim}}.*:{{scenario_dim}}.* = {{pl_cube}}::[{{account_dim}}.Revenue] - {{pl_cube}}::[{{account_dim}}.COGS]
```

When generating a concrete model from a template, substitute variables before execution.

## 13. Number Formatting

Apply the default number format to every cube so financial values are readable. Use the cube's `@` channel with the cube's actual dimensions:

```openm
rule PL::@.format_number:Account.*:Year.*:Scenario.* = 'preset:number(decimals=2; group=true; negative=parentheses; zero=dash)'
```

Place the formatting rules in `07_formatting.openm` and source it after the views and before `calc`.

## 14. Visual Formatting

Apply number formatting and styling to every cube, not as a global directive. Use the cube's `@` channel with the cube's actual dimensions:

```openm
rule PL::@.format_number:Account.*:Year.*:Scenario.* = 'preset:number(decimals=2; group=true; negative=parentheses; zero=dash)'
rule PL::@.fill:Account.*:Year.*:Scenario.* = "#f7f9fb"
rule PL::@.font_color:Account.*:Year.*:Scenario.* = "#2d3748"
```

Create a `07_formatting.openm` file and source it after the views and before `calc`. Repeat the three directives for every cube in the model.

For totals and subtotals (e.g., `TotalAssets`, `NetIncome`, `EBITDA`, `FCFF`, `EnterpriseValue`), add specific rules that use a heavier font weight and a slightly darker background:

```openm
rule BS::@.font_weight:BSAccount.TotalAssets:Year.*:Scenario.* = 700
rule BS::@.fill:BSAccount.TotalAssets:Year.*:Scenario.* = "#e2e8f0"
```

Formatting rules may intentionally use broad defaults plus more specific overrides. This is acceptable because more specific style rules override less specific style rules, the same way calculation rules do.

> **Note:** When a formula produces a color value (for example in an `IF`
> expression for conditional formatting), the color must be returned as a
> **string literal**. Unquoted hex codes inside a formula are not parsed as
> color strings.
>
> ```openm
> rule PL::@.font_color:NetIncome:* = IF(PL::NetIncome<0, "#F00", "#000")
> ```

## 15. Group Management

Financial models benefit from grouping dimension items into outline sections. For example, a P&L account dimension can be grouped into Revenue, COGS, Operating Expenses, and Totals.

See the full `om-core-group-management` skillset for command syntax. The available REPL commands are:

```text
group create <dim> <label> [parent=<group>] [item1 item2 ...]
group add <dim> <group> <item1 item2 ...>
group detach <dim> <item1 item2 ...>
group delete <dim> <group>
group rename <dim> <group> <new_label>
group list <dim>
```

Add a `08_groups.openm` file and source it after `07_formatting.openm` and before `calc`:

```text
source 00_variables.openm
source 01_dimensions.openm
source 02_cubes.openm
source 03_inputs.openm
source 04_rules.openm
source 05_checks.openm
source 06_views.openm
source 07_formatting.openm
source 08_groups.openm
calc
```

Example P&L grouping:

```text
om> group create PLAccount "Revenue Section" Revenue
om> group create PLAccount "COGS Section" COGS
om> group create PLAccount "Operating Expenses" Salaries Marketing Rent
om> group create PLAccount "Totals" GrossProfit EBITDA NetIncome
```

Example balance sheet grouping:

```text
om> group create BSAccount "Current Assets" Cash AccountsReceivable Inventory
om> group create BSAccount "Fixed Assets" GrossPPE AccumulatedDepreciation NetPPE
om> group create BSAccount "Liabilities" AccountsPayable Debt
om> group create BSAccount "Equity Section" Equity
om> group create BSAccount "Total" TotalAssets TotalLiabilitiesAndEquity
```

Groups affect only the outline presentation; rules continue to reference individual items.

## 16. Output Format

When asked to build a model, return:

1. File tree.
2. Contents of each `.openm` file.
3. Run instructions.
4. Explanation of major dimensions and cubes.
5. Notes about assumptions.
6. Known limitations or TODOs.

Example output structure:

```text
model/
  build.openm
  00_variables.openm
  01_dimensions.openm
  02_cubes.openm
  03_inputs.openm
  04_rules.openm
  05_checks.openm
  06_views.openm
  07_formatting.openm
  08_groups.openm
```

Small models may omit `07_formatting.openm` and `08_groups.openm`.

Then include each file in a separate code block.

## 17. Validation Checklist

Before finalizing a model, check:

* Every referenced dimension exists.
* Every referenced item exists.
* Every cube uses defined dimensions.
* Every rule target references a valid cube address.
* No rule accidentally depends on itself.
* No RHS reference relies on hidden first-item defaults.
* Input assumptions are separated from calculated outputs.
* Checks exist for important accounting identities.
* At least one view exists for every cube.
* The model can be loaded through `source build.openm`.

## 18. Example Minimal P&L Model

File tree:

```text
model/
  build.openm
  01_dimensions.openm
  02_cubes.openm
  03_inputs.openm
  04_rules.openm
  05_views.openm
```

`build.openm`:

```openm
source 01_dimensions.openm
source 02_cubes.openm
source 03_inputs.openm
source 04_rules.openm
source 05_views.openm
calc
```

`01_dimensions.openm`:

```openm
dim Account Revenue COGS GrossProfit OperatingExpense EBITDA
dim Month --seq Jan Feb Mar
dim Scenario Base Upside Downside
```

`02_cubes.openm`:

```openm
cube PL Account Month Scenario
```

`03_inputs.openm`:

```openm
rule PL::Account.Revenue:Month.Jan:Scenario.Base = 1000
rule PL::Account.Revenue:Month.Feb:Scenario.Base = 1100
rule PL::Account.Revenue:Month.Mar:Scenario.Base = 1200

rule PL::Account.COGS:Month.*:Scenario.Base = PL::[Account.Revenue] * 0.60
rule PL::Account.OperatingExpense:Month.*:Scenario.Base = 250
```

`04_rules.openm`:

```openm
rule PL::Account.GrossProfit:Month.*:Scenario.* = PL::[Account.Revenue] - PL::[Account.COGS]
rule PL::Account.EBITDA:Month.*:Scenario.* = PL::[Account.GrossProfit] - PL::[Account.OperatingExpense]
```

`05_views.openm`:

```openm
view PL_View = PL::Account:Month
```

Run:

```text
om> source build.openm
```

## 19. Example Three-Statement Model

File tree:

```text
examples/three-statement-model/
  00_variables.openm
  01_dimensions.openm
  02_cubes.openm
  03_inputs.openm
  04_rules.openm
  05_checks.openm
  06_views.openm
  07_formatting.openm
  08_groups.openm
  build.openm
```

This is the flagship example. It builds an integrated P&L, Balance Sheet, and Cash
Flow Statement over five years for three scenarios (Base, Upside, Downside). The
model links revenue, COGS, operating expenses, depreciation, interest, and taxes
into net income; rolls forward fixed assets, working capital, debt, and retained
earnings on the balance sheet; and bridges net income to ending cash through
operating cash flow, capex, and working capital changes. Checks verify the balance
sheet balances, the cash bridge reconciles, and retained earnings roll forward
correctly.

Run:

```text
om> source build.openm
```

## 20. Example Budget vs Actual / Forecast Variance Model

File tree:

```text
examples/budget-vs-actual-model/
  00_variables.openm
  01_dimensions.openm
  02_cubes.openm
  03_inputs.openm
  04_rules.openm
  05_checks.openm
  06_views.openm
  07_formatting.openm
  08_groups.openm
  build.openm
```

This is the FP&A example. It stores Actual, Budget, and Forecast as a `Version`
dimension and computes `Variance` and `VariancePct` across Account, Month, and
Department. The model shows how multidimensional rules replace the copied tabs,
linked workbooks, and manual variance columns typical in spreadsheet-based
variance analysis.

Run:

```text
om> source build.openm
```

## 21. Example Manufacturing CapEx & Depreciation Model

File tree:

```text
examples/manufacturing-capex-model/
  00_variables.openm
  01_dimensions.openm
  02_cubes.openm
  03_inputs.openm
  04_rules.openm
  05_checks.openm
  06_views.openm
  07_formatting.openm
  08_groups.openm
  build.openm
```

This model tracks fixed assets across three asset types (Machinery, Equipment, Vehicles) with different useful lives. It computes gross block, straight-line depreciation, accumulated depreciation, and net book value over five years for three scenarios.

Run:

```text
om> source build.openm
```

## 22. Example Business Valuation Model

File tree:

```text
examples/business-valuation-model/
  00_variables.openm
  01_dimensions.openm
  02_cubes.openm
  03_inputs.openm
  04_rules.openm
  05_checks.openm
  06_views.openm
  07_formatting.openm
  08_groups.openm
  build.openm
```

This model builds a 5-year pro-forma P&L, balance sheet, and cash flow statement, then derives FCFF, WACC, terminal value, enterprise value, and equity value. It includes working capital (accounts receivable, inventory, accounts payable), fixed assets, debt, and taxes.

Run:

```text
om> source build.openm
```

## 23. Example Agroforestry Model

File tree:

```text
examples/agroforestry-model/
  00_variables.openm
  01_dimensions.openm
  02_cubes.openm
  03_inputs.openm
  04_rules.openm
  05_checks.openm
  06_views.openm
  07_formatting.openm
  08_groups.openm
  build.openm
```

This model tracks an agroforestry planting across five species (Apple, Pear, Hazelnut, Oak, Willow). It computes the number of plants from area and spacing, initial planting costs (material and labor), and recurring lifecycle costs (planting, fertiliser, pruning, weeding, harvesting) over 16 years. A summary cube rolls up total initial cost, total lifecycle cost, total cost, number of plants, and planted area per species, and an area check verifies the planted area matches the available land.

Run:

```text
om> source build.openm
```

## 24. Agent Behavior Rules

The agent must not:

* generate spreadsheet cell references
* use vague dimensions such as `A`, `B`, `C` in business models
* hide core business logic inside macros
* create separate scenario files instead of a Scenario dimension
* rely on hidden first-item defaults
* produce prose-only answers when a script bundle is requested
* invent unsupported OM Core syntax

The agent should ask clarifying questions only when required to avoid building the wrong model. If reasonable assumptions can be made, state them and proceed.
