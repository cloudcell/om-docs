# Glossary

Common terms used in OM Core.

- **Dimension** — an axis along which data is organized, such as Time or Account.
- **Dimension item** — a single semantic element within a dimension, such as a month name or account code.
- **Node** — an occurrence of a dimension item in a hierarchy or graph context, identified separately from the dimension item itself.
- **Edge** — a parent/child or relation link between two nodes in a graph.
- **Hierarchy** — a tree of nodes that organizes dimension items for display, aggregation, and navigation.
- **Group** — a named collection or hierarchy of dimension items, represented through graph nodes.
- **Cube** — a multidimensional array of data defined by a set of dimensions.
- **Slice** — a subset of a cube, often fixing one or more dimensions to specific dimension items.
- **Rule** — a calculation rule that defines how values are derived.
- **Model** — the complete set of dimensions, cubes, rules, and groups that describe a domain.
- **View** — a saved projection of a cube into rows, columns, and filters. A view is part of the workspace metadata, but it is not the source of truth for values or calculations.
- **Outline** — a read-only projection of a hierarchy for display.
- **Graph primitive** — a low-level operation that mutates hierarchy, grouping, node order, or parentage safely.
