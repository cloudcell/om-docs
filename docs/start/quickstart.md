# Quickstart

This quickstart creates a tiny model with a single dimension and a single cube.

## Create a model

```python
from om_core import Model, Dimension, Cube

model = Model(name="hello")
time = Dimension(name="Time", labels=["2024-01", "2024-02"])
cube = Cube(name="Sales", dimensions=[time])
```

## Set values and compute

```python
cube["2024-01"] = 100
cube["2024-02"] = 150
print(cube.total)
```

This is a minimal example. The guides explain how to build full models with multiple dimensions and rules.
