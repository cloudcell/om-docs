# Dimensions

A dimension is an axis along which model data is organized. Common dimensions include Time, Account, Scenario, Department, and Product.

Each dimension contains a list of labels, identifiers, or dimension items. A dimension item is a stable semantic element. Hierarchies and groups organize those items through graph nodes, rather than making the items themselves hierarchical. For example, a Time dimension may contain months, and a hierarchy may roll those months up into quarters and years.

Dimensions are declared once and reused across many cubes. This makes it easy to keep terminology consistent throughout a model.
