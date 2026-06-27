# Rules

Rules are OM Core's higher-level answer to the abstraction gap in spreadsheet-style modeling. Where a spreadsheet forces you to encode business logic as cell syntax scattered across a grid, OM Core lets you express logic as named, inspectable units that operate on semantic relationships.

Rules define how values in a model are calculated. A rule typically maps a target cube and slice to a calculation expressed in terms of other cubes or slices.

Rules are stored separately from the data they produce. This means:

- The same rule can be applied to many slices automatically.
- Rules are visible and auditable in one place.
- Changes to a rule propagate consistently across the model.

Rules are evaluated by the OM Core engine when the model is queried or rendered.
