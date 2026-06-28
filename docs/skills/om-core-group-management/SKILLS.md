# OM Core Group Management

Skillset for organizing dimension items into outline groups in OM Core.

## 1. Overview

OM Core supports hierarchical outlines inside dimensions. Groups let you bundle related items under a labeled parent without changing the underlying dimension items or calculations. Grouping is purely presentational and is useful for:

- Grouping P&L line items into Revenue, COGS, Operating Expenses, etc.
- Grouping balance sheet items into Assets, Liabilities, and Equity.
- Grouping drivers or checks by category.
- Making views and reports easier to read.

## 2. When to Use Groups

- Use groups when the flat dimension item list is too long to scan quickly.
- Use groups to reflect sections in financial statements (e.g., Current Assets, Fixed Assets).
- Do not use groups to store business logic; rules still reference individual items.
- Do not use groups as a substitute for dimensions; if a category is meaningful across rules, consider making it a dimension instead.

## 3. REPL Commands

All group management is done through the `group` REPL command. It is a sub-command dispatcher with the following verbs:

### 3.1. Create a group

```text
group create <dim> <label> [parent=<group>] [item1 item2 ...]
```

Creates a new group node in the outline of `<dim>`. Optionally places it under a parent group and/or nests existing items under it.

```text
om> group create PL "Operating Expenses" Salaries Marketing Rent
om> group create BS "Current Assets" parent=Assets Cash AccountsReceivable Inventory
```

### 3.2. Add items to a group

```text
group add <dim> <group> <item1 item2 ...>
```

Moves existing dimension items into an existing group. Items are detached from their current parent if they have one.

```text
om> group add PL "Operating Expenses" Depreciation Utilities
```

### 3.3. Detach items from their parent group

```text
group detach <dim> <item1 item2 ...>
```

Removes items from their parent group and places them back at the root level of the outline. The items themselves are not deleted.

```text
om> group detach PL Depreciation
```

### 3.4. Delete a group

```text
group delete <dim> <group>
```

Deletes the group node. The child items are not deleted; they are moved to the root level of the outline.

```text
om> group delete PL "Operating Expenses"
```

### 3.5. Rename a group

```text
group rename <dim> <group> <new_label>
```

Renames an existing group node. The label can be referenced by its current text or by its node id.

```text
om> group rename PL "Operating Expenses" "OpEx"
```

### 3.6. List groups

```text
group list <dim>
```

Prints the outline of a dimension, including groups and their nested items.

```text
om> group list PL
```

## 4. Underlying Command Spine

The `group` REPL command dispatches to these engine-level commands via the session:

| REPL action | Engine command | Purpose |
| --- | --- | --- |
| `group create` | `create_group` | Create a group node with optional parent and children |
| `group add` | `move_items_to_group` | Move existing items under a group |
| `group detach` | `ungroup_items` | Move items out of their parent group to root |
| `group delete` | `delete_group` | Delete a group node and promote its children |
| `group rename` | `rename_group_node` | Change a group's label |
| `group list` | `dimension_detail` query | Read and print the outline |

## 5. Examples

### 5.1. Group a P&L dimension

Assume the dimension:

```text
om> dim PLAccount Revenue COGS GrossProfit Salaries Marketing Rent EBITDA
```

Create an outline:

```text
om> group create PLAccount "Revenue Section" Revenue
om> group create PLAccount "COGS Section" COGS
om> group create PLAccount "Operating Expenses" Salaries Marketing Rent
om> group list PLAccount
```

Expected outline:

```text
PLAccount
  Revenue Section
  COGS Section
  Operating Expenses
    Salaries
    Marketing
    Rent
  GrossProfit
  EBITDA
```

Note: `GrossProfit` and `EBITDA` are calculated subtotals and can remain as root-level items or be grouped separately depending on the desired view.

### 5.2. Group a balance sheet dimension

```text
om> dim BSAccount Cash AccountsReceivable Inventory GrossPPE AccumulatedDepreciation NetPPE AccountsPayable Debt Equity TotalAssets TotalLiabilitiesAndEquity
om> group create BSAccount "Current Assets" Cash AccountsReceivable Inventory
om> group create BSAccount "Fixed Assets" GrossPPE AccumulatedDepreciation NetPPE
om> group create BSAccount "Liabilities" AccountsPayable Debt
om> group create BSAccount "Equity Section" Equity
om> group list BSAccount
```

## 6. Best Practices

- Keep group labels short and consistent with the dimension vocabulary.
- Do not create a group label that matches an existing item name in the same dimension; this raises a validation error.
- Create groups after the dimension items are defined, either in a dedicated `08_groups.openm` file or interactively after the model is loaded.
- Groups do not affect rule evaluation; a rule like `PL::PLAccount.Revenue:Year.*:Scenario.*` still works regardless of whether `Revenue` is inside a group.
- Although groups are managed through REPL/session commands, group commands may be recorded in `.openm` scripts and replayed through `source` for reproducibility.
- When deleting a group, its children are promoted to the root. Use `group detach` if you only want to move specific items out while keeping the group.
- Prefer `group list` to verify the outline before saving or sharing the model.

## 7. Limitations

- Groups are not part of the `.openm` declarative language; they are managed through the REPL or session commands.
- Groups are not automatically persisted in the `.openm` source files. Save the model or workspace to preserve them, or record the group commands in a script for reproducibility.
- Nesting is supported through `parent=<group>`. The parent value may be a group label or a group node id.
- Labels with spaces must be quoted when typed at the REPL: `group create Account "Operating Expenses" Salaries Marketing`.
- Numeric-looking item labels are preserved as strings, consistent with dimension item parsing.
- If two groups share the same label, resolution prefers an exact id match; ambiguous labels may be rejected.
