# Building models with an agent

You can use an LLM agent to generate OM Core models as `.openm` script bundles. The agent follows a structured skill so that the output is semantic, runnable, and auditable rather than a prose description or spreadsheet.

This guide explains how to prompt the agent, what output to expect, and how to validate it.

## Skill file

The agent skill lives at:

```text
skills/om-core-financial-modeling/SKILLS.md
```

It defines the agent's workflow, naming conventions, standard dimensions, cube layout, rule-writing conventions, and a validation checklist. When asking an agent to build a model, tell it to read the skill first.

## Windsurf rule (optional)

If you use Windsurf or Cascade, you can add a workspace rule so the agent reads the skill automatically. Create:

```text
.windsurf/rules/om-core-financial-modeling.md
```

with:

```md
# OM Core Financial Modeling Rule

When the user asks for a financial, operational, planning, forecasting, budgeting, or analytical model in OM Core:

1. First read `skills/om-core-financial-modeling/SKILLS.md`.
2. Build models as `.openm` script bundles, not prose-only descriptions.
3. Use dimensions, cubes, rules, views, checks, variables, and `{{...}}` macro expansion.
4. Prefer a multi-file structure:
   - `00_variables.openm`
   - `01_dimensions.openm`
   - `02_cubes.openm`
   - `03_inputs.openm`
   - `04_rules.openm`
   - `05_checks.openm`
   - `06_views.openm`
   - `build.openm`
5. Do not invent unsupported OM Core syntax.
6. Do not rely on hidden first-item defaults.
7. Use `Scenario` as a dimension rather than creating separate scenario files.
8. Return runnable scripts and run instructions.
```

## How to prompt the agent

Describe the business purpose, then ask for the script bundle. A good prompt is:

```text
Read `skills/om-core-financial-modeling/SKILLS.md`.

Using that skill, create an OM Core financial model script bundle for a 3-year SaaS revenue model.

Use variables and `{{...}}` macro expansion where useful.

Create:
- build.openm
- 00_variables.openm
- 01_dimensions.openm
- 02_cubes.openm
- 03_inputs.openm
- 04_rules.openm
- 05_checks.openm
- 06_views.openm

The model should include:
- Revenue
- Customers
- Churn
- ARPU
- COGS
- Gross Profit
- Operating Expenses
- EBITDA
- Base / Upside / Downside scenarios

Make sure the scripts are internally consistent and can be loaded with `source build.openm`.
```

## Expected output structure

The agent should return a file tree and the contents of each `.openm` file. A typical bundle looks like:

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

Each file has a clear responsibility:

| File | Purpose |
| --- | --- |
| `00_variables.openm` | Names, cube names, repeated address fragments |
| `01_dimensions.openm` | Dimensions and items |
| `02_cubes.openm` | Cube definitions |
| `03_inputs.openm` | Hardcoded assumptions and drivers |
| `04_rules.openm` | Calculated outputs |
| `05_checks.openm` | Accounting identities and invariants |
| `06_views.openm` | User-facing views |
| `build.openm` | Sources the files in order and runs `calc` |

## Macros and variables

Variables keep the script readable and easy to change:

```openm
pl_cube="PL"
account_dim="Account"
month_dim="Month"
scenario_dim="Scenario"
```

Macro expansion composes commands from those variables:

```openm
cube {{pl_cube}} {{account_dim}} {{month_dim}} {{scenario_dim}}
view PL_View = {{pl_cube}}::{{account_dim}}:{{month_dim}}
```

Rules should still be readable after expansion. Do not hide business logic inside macros.

## Validation checklist

Before running the model, check that the agent's output:

* References only dimensions and items that are defined.
* Uses cubes that are declared with the right dimensions.
* Avoids hidden first-item defaults by writing ambiguous dimensions explicitly.
* Separates input assumptions from calculated outputs.
* Includes checks for important accounting identities.
* Sources files in the correct order in `build.openm`.
* Runs without errors when you execute `source build.openm`.

## Running the model

Load the bundle in the OM Core REPL or TUI:

```text
om> source build.openm
```

If the model is large, you may want to start the runtime separately and then connect a TUI client:

```bash
# Terminal 1
./start.sh --runtime

# Terminal 2
./start.sh --tui
om> source build.openm
```

## When to use agent-assisted modeling

Agent-assisted modeling works well when:

* The model is new and you want to explore structure quickly.
* The model follows a standard pattern such as P&L, SaaS revenue, or three-statement.
* You want the model stored as source-controlled scripts.
* You need a consistent, repeatable workflow.

For small one-off calculations, a single handwritten `.openm` file is usually faster.
