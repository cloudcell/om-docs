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
rule PL::Month.Jan:Account.Revenue = 1000
rule PL::Month.Feb:Account.Revenue = 1100
rule PL::Month.Mar:Account.Revenue = 1200
rule PL::Month.*:Account.COGS = PL::[Account.Revenue] * 0.6
rule PL::Month.*:Account.GrossProfit = PL::[Account.Revenue] - PL::[Account.COGS]
rule PL::Month.*:Account.OpEx = 200
rule PL::Month.*:Account.NetProfit = PL::[Account.GrossProfit] - PL::[Account.OpEx]

# Calculate
calc
```

Save the example above as `pl-example.openm`, then run it in the REPL or TUI:

```bash
om> source pl-example.openm
```

In this example, `PL::[Account.Revenue]` binds the current `Month` from the target cell and reads the revenue for that month.

## Steps for a full model

1. Create the model and dimensions.
2. Define cubes for revenue, costs, and expenses.
3. Add rules for gross profit, operating profit, and net profit.
4. Build views from the cubes.
