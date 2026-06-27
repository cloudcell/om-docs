# Groups and Hierarchies

OM Core separates dimension members from the graphs that organize them into hierarchies, groups, and outlines.

- A **member** is a stable semantic item in a dimension, identified by its member ID.
- A **node** is an occurrence of that member in a graph context, identified by a node ID.
- A single member may appear in many graph contexts through different nodes.

This separation lets the same member participate in multiple hierarchies without duplicating the member itself.

## Terms

| Term | Meaning |
| --- | --- |
| Member | Stable semantic dimension item, identified by a member ID |
| Node | Graph occurrence or hierarchy node, identified by a node ID |
| Edge | Parent/child or relation link between nodes |
| Hierarchy | A tree of nodes built from parent/child edges |
| Group | A named collection of members or nodes used in rules and views |
| Outline | A read-only projection of a hierarchy for display |

## Graph primitives

Only graph primitives may change hierarchy, grouping, node order, or node parentage.

Graph primitives operate on node IDs, not member IDs. Higher-level commands may request changes, but the actual mutation is delegated to graph primitives.

Examples of graph primitives:

- `create_node`
- `delete_node`
- `move_node`
- `rename_node_label`
- `set_node_order`
- `create_edge`
- `delete_edge`

Higher-level operations, such as group deletion or drag-and-drop, compose these primitives instead of mutating the outline directly.

## Node identity

A node ID is stable. The label is mutable. Renaming changes only the label; edges continue to point to the same node ID.

Node labels are display values. Edges, parentage, ordering, and deletion resolve through node IDs, not labels.

## Outline mutation

An outline is a read-only projection of a hierarchy. Do not mutate the outline directly. Instead, route changes through graph primitives.

## Group deletion

Group deletion is one semantic mutation. Node deletion and dependent rule cleanup must succeed or fail together. The system must not expose a state where the group is deleted but affected rules are not cleaned up.

Rule cleanup is an internal effect of `delete_group_node`, not a separate public command.

Correct:

```text
delete_group_node
  -> delete node
  -> update affected rules internally
  -> emit one command lifecycle
```

Incorrect:

```text
delete_group_node
update_rules_after_group_deletion
```

## Anti-patterns

| Anti-pattern | Why we avoid it | Correct approach |
| --- | --- | --- |
| Direct outline mutation from a view | Breaks caching and the audit trail | Route through graph primitives |
| Separate public rule cleanup command after deletion | Partial failure risk | Cleanup internally inside `delete_group_node` |
| Using member ID as graph identity | One member may appear in multiple graph contexts | Use node ID for graph mutation and member ID for semantic dimension identity |
