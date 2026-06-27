# Why not spreadsheets?

Spreadsheets are fast and flexible, but they become fragile as models grow. Rules live inside cells, references are implicit, and small changes can break large parts of the model silently.

OM Core addresses these problems by making the model explicit:

- Dimensions describe the axes of the model, such as time, account, or scenario.
- Cubes hold data along those dimensions.
- Rules define calculations separately from any single view.
- Groups organize related elements.

This separation makes it easier to reason about the model, test changes, and reuse logic across reports.
