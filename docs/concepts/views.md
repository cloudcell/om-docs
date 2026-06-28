# Views

A view is a projection of a cube into a grid. It defines which dimensions appear as rows, which appear as columns, and which are fixed or filtered. The same cube can have many views.

## What a view is

In OM Core, the model and the view are separate. A cube stores the data. A view decides how to display that data.

- A cube is a multidimensional array.
- A view is a projection of a cube into rows, columns, and page filters.
- Multiple views can point to the same cube.

For example, a cube with dimensions `Product`, `Region`, and `Time` can be viewed as:

- Product by Region for a fixed year
- Product by Time for a fixed region
- Region by Time for a fixed product

Each of these is a different view of the same cube.

## Creating a view

Views are defined with the `view` command.

```openm
view SalesByMonth = Sales::Month
```

This creates a view named `SalesByMonth` that projects the `Sales` cube with `Month` as the column dimension.

A view with two dimensions:

```openm
view SalesByProductAndMonth = Sales::Product:Month
```

Here, `Product` is the row dimension and `Month` is the column dimension.

A view with more than two dimensions:

```openm
view SalesByProductAndMonthAndRegion = Sales::Product:Month:Region
```

The first dimension is used for rows, the second for columns, and remaining dimensions become page or filter axes. The UI displays the selected page values and lets you switch between them. If you need a different layout, create a separate view with the dimensions in a different order.

## Activating a view

You can activate a view by giving its name without the cube assignment:

```openm
view SalesByMonth
```

When a view is active, the Matrix Grid displays that layout.

## Multiple views from one cube

A single cube can have any number of views. This is useful because different audiences need different cuts of the same data.

```openm
# One cube
cube Sales Product Month

# Three views of the same cube
view ProductByMonth = Sales::Product:Month
view MonthByProduct = Sales::Month:Product
view ProductOnly = Sales::Product
```

Changing the data in the cube updates every view. Changing a view does not change the cube.

## Views vs cubes

| | Cube | View |
| --- | --- | --- |
| Stores data | Yes | No |
| Has dimensions | Yes | Yes, selected from the cube |
| Can be multiple per model | Yes | Yes, many views per cube |
| Used by rules for calculation | Yes | No |
| Defines display | No | Yes |

## Views in scripts

Views are saved workspace objects in `.openm` scripts. You can define them, activate them, and switch between them. A view is part of the workspace metadata, but it is not the source of truth for values or calculations.

```openm
cube Revenue Product Region Month

view RevenueByProduct = Revenue::Product
view RevenueByRegion = Revenue::Region
view RevenueByMonth = Revenue::Month

view RevenueByProduct
```

## Views in the UI

In the GUI and TUI, views appear under the Views section of the Model Browser. Selecting a view opens it in the Matrix Grid. You can switch between views to see the same cube from different angles.

## See also

- [Cubes](cubes.md)
- [Dimensions](dimensions.md)
- [Scripting reference](../reference/scripting.md)
- [Interface overview](../ui-ux/interface.md)
