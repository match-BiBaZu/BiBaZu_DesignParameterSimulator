# BiBaZu Design Parameter Simulator

This repository contains the PyBullet simulation code for studying how prototype settings affect aerodynamic part feeding. The simulator releases workpieces onto a chute, applies a force pulse at the configured nozzle location, and classifies the resulting motion and orientation. The parameter sweep varies the chute angles, entry speed, force application point, nozzle offset and pulse force.

Dasha Shieff, Tobias Recker and Annika Raatz (2025). [Aerodynamic Feeding 4.1: Simulation Based Parameter Selection for a Flexible Part Feeding Prototype](https://doi.org/10.1016/j.procir.2025.08.092). *Procedia CIRP*, 136, 534–539. [BiBaZu project and prototype](https://match-bibazu.github.io/#paper-aerodynamic-feeding)

## Run a PyBullet sweep

Set up a Python 3.11 environment on x86-64 Linux or WSL. Use NumPy 1.26.4, as specified in the simulator source. The sweep uses the OBJ and STL meshes already checked into the repository.

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install numpy==1.26.4 numpy-stl matplotlib pybullet trimesh
mkdir -p Simulation_Data/Bullet_Raw_Data/Logged_Simulations
```

In [Main.py](Main.py), choose `workpiece_names`, `surface_name`, `simulation_number` and the parameter arrays near the top of the file. The script runs every combination. For a short first run, select one workpiece, one value per array and a small trial count. Set the worker count in `Pool(32)` to fit your machine.

```bash
python Main.py
```

The sweep runs in PyBullet's headless mode. Each launch writes a new `Simulation_Data/Bullet_Raw_Data/simulation_outcomes_new.csv` with one row per parameter combination. The row records the inputs, outcome frequencies, sliding distance and orientation changes. Copy any existing output you want to keep before rerunning the sweep. Raw logs are written per workpiece under `Logged_Simulations`, so runs for the same workpiece can overwrite earlier raw logs.

## Code and data

| Path | Contents |
| --- | --- |
| [Main.py](Main.py) | Defines the parameter sweep, runs trials in parallel and writes the summary CSV. |
| [DroptestsFaster.py](DroptestsFaster.py) | Runs the PyBullet trials and records positions, orientations and angular velocities before and after the force pulse. |
| [PoseFinder.py](PoseFinder.py) | Classifies outcomes and computes sliding distance and orientation statistics. |
| [Workpieces](Workpieces) and [Surfaces](Surfaces) | Workpiece and chute geometry, including `Teil_1` through `Teil_5` and the `Slide_Long` chute. |
| [Bullet_Raw_Data](Simulation_Data/Bullet_Raw_Data) | Saved outcome CSVs and sample raw logs. |

## Docker

The [Dockerfile](Dockerfile) installs the repository's [requirements](requirements.txt). From the repository root, build the image and mount the checkout so the simulator can read the meshes and write its outputs.

```bash
docker build -t bibazu-simulator .
docker run --rm --mount "type=bind,source=$PWD,target=/external_repo" bibazu-simulator \
  sh -c 'mkdir -p /external_repo/Simulation_Data/Bullet_Raw_Data/Logged_Simulations && python /external_repo/Main.py'
```

## Earlier Blender workflow

[PoseFinderOriginal](PoseFinderOriginal) contains the earlier Blender scenes, mesh archives and MATLAB pose analysis. [STLtoOBJConverter.py](STLtoOBJConverter.py) uses Blender's `bpy` API to convert new geometry. The PyBullet sweep can use the prepared meshes directly.

The repository is distributed under the [MIT License](LICENSE).
