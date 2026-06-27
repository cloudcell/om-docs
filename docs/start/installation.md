# Installation

OM Core is distributed as a Python package. The fastest way to start is to install it into a virtual environment.

```bash
python -m venv .venv
source .venv/bin/activate
pip install om-core
```

After installation, verify the package is available:

```bash
python -c "import om_core; print(om_core.__version__)"
```

For the documentation site specifically, install the doc dependencies from `requirements.txt`:

```bash
pip install -r requirements.txt
```
