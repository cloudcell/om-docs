# Why not spreadsheets?

Spreadsheets are fast and flexible, but they become fragile as models grow. Rules live inside cells, references are implicit, and small changes can break large parts of the model silently.

This is not because spreadsheets are bad tools. It is because they force you to express business logic at the wrong level of abstraction.

## Every spreadsheet is a program

When you write `=A1*B1`, copy it down a column, and link cells across tabs, you are writing logic, executing logic, and changing logic. The question is not whether you are programming. The question is at what level.

## The spreadsheet as assembly language

A spreadsheet is the closest thing to assembly language that most business people will ever write.

In assembly, every instruction maps to a machine operation. It is direct, but the abstraction gap is enormous. You spend most of your energy managing memory addresses, registers, and control flow instead of solving the actual problem.

A spreadsheet works the same way. Cells are memory addresses. Cell syntax is operations. The grid layout is the control flow. When you write `=B2*C2` in cell `D2`, you are encoding a calculation at the level of individual memory locations. You are not defining relationships between rows, columns, or sheets explicitly. The structure and relationships live only in your head, and the software does not enforce them.

This is fine for five calculations. It is catastrophic for five thousand.

## The abstraction gap

A business rule stated in natural language is clear and self-documenting:

```text
Gross_Margin = Revenue - Cost_of_Sales
```

The same idea expressed in a spreadsheet is buried in coordinates:

```text
=B2-C3
```

Anyone reading the spreadsheet must trace `B2` and `C3` back to their meaning, hope the naming conventions are consistent, and hope nobody has overwritten a calculation with a hardcoded number.

This is the abstraction gap. Spreadsheets force you to express high-level business logic at a low-level addressing scheme. You are programming in coordinates instead of concepts.

## Rules as a higher-level paradigm

OM Core closes that gap by treating rules as first-class citizens of the model.

A rule says what, not where:

```text
Gross Margin = Revenue - Cost
```

A rule has an identity. You can trace its dependencies. You can change its expression without moving data around. The model understands it as a relationship, not as a combination of cell syntax and grid layout.

Rules operate on semantic relationships, not coordinates. Add a new dimension, reorganize a hierarchy, or flip rows and columns in a view: the rules do not change.

## What changes when rules are first-class

- **Auditing becomes reading.** You inspect the rules directly instead of tracing cell references through a maze of addresses.
- **Structure changes do not break logic.** Rules are independent of layout and hierarchy ordering.
- **The model and the view are separated.** The model defines dimensions, cubes, rules, and groups. The grid is just one way to look at it.
- **Logic is expressed once.** The same rule applies to every slice it targets, rather than being copied into hundreds of cells.

## The tradeoff

OM Core is not a spreadsheet with extra features. It is a different level of abstraction.

You trade the spreadsheet's infinite flexibility for structure. You must define dimensions, dimension items, cubes, and rules explicitly. In return, you get predictability, auditability, and the ability to scale a model without it becoming a maze of references.

This is the same tradeoff that structured programming made over machine code, that relational databases made over flat files, and that high-level languages made over assembly.

## OM Core's approach

OM Core addresses these problems by making the model explicit:

- **Dimensions** describe the axes of the model, such as time, account, or scenario.
- **Cubes** hold data along those dimensions.
- **Rules** define calculations separately from any single view.
- **Groups** organize related elements.

This separation makes it easier to reason about the model, test changes, and reuse logic across views.

For more background on rules as a higher-level paradigm, see [Programming with Rules](https://openmodeling.substack.com/p/programming-with-rules).
