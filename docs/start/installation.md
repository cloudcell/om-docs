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
./start.sh --runtime   # headless runtime
./start.sh --gui       # graphical interface
./start.sh --tui       # terminal interface
```
