# Glossary

Common terms used in OM Core.

- **Dimension** — an axis along which data is organized, such as Time or Account.
- **Member** — a single semantic item within a dimension, such as a month name or account code.
- **Node** — an occurrence of a member in a hierarchy or graph context, identified separately from the member itself.
- **Edge** — a parent/child or relation link between two nodes in a graph.
- **Hierarchy** — a tree of nodes that organizes dimension members for display, aggregation, and navigation.
- **Group** — a named collection of related members, cubes, or rules.
- **Cube** — a multidimensional array of data defined by a set of dimensions.
- **Slice** — a subset of a cube, often fixing one or more dimensions to specific members.
- **Rule** — a calculation rule that defines how values are derived.
- **Model** — the complete set of dimensions, cubes, rules, and groups that describe a domain.
- **View** — a report or grid generated from a model, not part of the model itself.
- **Outline** — a read-only projection of a hierarchy for display.
- **Graph primitive** — a low-level operation that mutates hierarchy, grouping, node order, or parentage safely.
