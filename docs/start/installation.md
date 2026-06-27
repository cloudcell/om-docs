# Installation

OM Core is run from source. Clone the repository and use the provided start script.

## Clone the repository

```bash
git clone https://github.com/cloudcell/om-core.git
cd om-core
```

## Run OM Core

The `start.sh` script sets up the environment and launches OM Core:

```bash
./start.sh
```

You can also start specific runtime modes directly:

```bash
./start --runtime   # headless runtime
./start --gui       # graphical interface
./start --tui       # terminal interface
```

## Build the documentation site

To build or serve this documentation site locally, install the doc dependencies from `requirements.txt`:

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open `http://127.0.0.1:8000` in your browser.
