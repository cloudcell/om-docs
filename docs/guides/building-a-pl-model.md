# Build a P&L model

This guide walks through building a simple profit and loss model in OM Core.

!!! note "Draft"
    A more detailed, production-style P&L example is coming soon.

## Model outline

A basic P&L model has these dimensions:

- **Time** — months, quarters, years
- **Account** — revenue, cost, expense, and profit lines
- **Scenario** — actual, budget, forecast, variance

## Minimal example

```openm
# Dimensions
dim Month Jan Feb Mar
dim Account Revenue COGS GrossProfit OpEx NetProfit

# Cube
cube PL Month Account

# View
view PLView = PL::Month:Account

# Rules
rule PL::@.value:Month.Jan:Account.Revenue = 1000
rule PL::@.value:Month.*:Account.COGS = PL::[Account.Revenue] * 0.6
rule PL::@.value:Month.*:Account.GrossProfit = PL::[Account.Revenue] - PL::[Account.COGS]
rule PL::@.value:Month.*:Account.OpEx = 200
rule PL::@.value:Month.*:Account.NetProfit = PL::[Account.GrossProfit] - PL::[Account.OpEx]

# Calculate
calc
```

In this example, `PL::[Account.Revenue]` binds the current `Month` from the target cell and reads the revenue for that month. Run `source pl-example.openm` in the REPL or TUI to load it.

## Steps for a full model

1. Create the model and dimensions.
2. Define cubes for revenue, costs, and expenses.
3. Add rules for gross profit, operating profit, and net profit.
4. Build views from the cubes.
