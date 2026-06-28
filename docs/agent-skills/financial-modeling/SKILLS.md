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
calc
```

Use fewer files for small examples, but keep this order conceptually.

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

Avoid using macro expansion to hide business logic. Rules should remain readable after expansion.

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

Use full addresses when clarity matters:

```openm
rule PL::Account.EBITDA:Month.*:Scenario.* = PL::Account.GrossProfit:Month[THIS]:Scenario[THIS] - PL::Account.OperatingExpense:Month[THIS]:Scenario[THIS]
```

Use full addresses for recurrence rules that reference a previous or next item:

```openm
rule SaaSPL::Account.Customers:Month.*:Scenario.* = SaaSPL::Account.Customers:Month[PREV]:Scenario[THIS] + Drivers::[Driver.NewCustomers]
```

Do not combine bracket shorthand with sequential accessors such as `SaaSPL::[Account.Customers]:Month[PREV]`. Bracket shorthand resolves only one dimension and carries over the rest; it does not support explicit `:Month[PREV]` overrides. For recurrence references, write the full canonical address.

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

## 10. Checks

Every serious financial model should include checks.

Examples:

```openm
dim Check BalanceCheck GrossProfitCheck CashFlowCheck
cube Checks Check Month Scenario
```

Example rules:

```openm
rule Checks::Check.GrossProfitCheck:Month.*:Scenario.* =
  PL::[Account.GrossProfit]
  - (PL::[Account.Revenue] - PL::[Account.COGS])
```

A check should equal zero when the model is valid.

Prefer check names that describe the invariant being tested.

## 11. Views

Create views for important cubes.

Example:

```openm
view PL_View = PL::Account:Month
view Drivers_View = Drivers::Driver:Month
view Checks_View = Checks::Check:Month
```

Views are saved projections of cubes. They are not the source of truth for calculations.

## 12. Template Pattern

For reusable model templates, use variables at the top.

Example template:

```openm
pl_cube="{{pl_cube}}"
account_dim="{{account_dim}}"
time_dim="{{time_dim}}"
scenario_dim="{{scenario_dim}}"

cube {{pl_cube}} {{account_dim}} {{time_dim}} {{scenario_dim}}

rule {{pl_cube}}::{{account_dim}}.GrossProfit:{{time_dim}}.*:{{scenario_dim}}.* =
  {{pl_cube}}::[{{account_dim}}.Revenue]
  - {{pl_cube}}::[{{account_dim}}.COGS]
```

When generating a concrete model from a template, substitute variables before execution.

## 13. Output Format

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
```

Then include each file in a separate code block.

## 14. Validation Checklist

Before finalizing a model, check:

* Every referenced dimension exists.
* Every referenced item exists.
* Every cube uses defined dimensions.
* Every rule target references a valid cube address.
* No rule accidentally depends on itself.
* No RHS reference relies on hidden first-item defaults.
* Input assumptions are separated from calculated outputs.
* Checks exist for important accounting identities.
* Views exist for the main output cubes.
* The model can be loaded through `source build.openm`.

## 15. Example Minimal P&L Model

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

## 16. Agent Behavior Rules

The agent must not:

* generate spreadsheet cell references
* use vague dimensions such as `A`, `B`, `C` in business models
* hide core business logic inside macros
* create separate scenario files instead of a Scenario dimension
* rely on hidden first-item defaults
* produce prose-only answers when a script bundle is requested
* invent unsupported OM Core syntax

The agent should ask clarifying questions only when required to avoid building the wrong model. If reasonable assumptions can be made, state them and proceed.
