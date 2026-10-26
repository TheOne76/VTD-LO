# VTD-LO

### Voxel-Temporal Dynamic Filtering LiDAR Odometry

VTD-LO estimates sensor motion from 3D LiDAR scans while maintaining a local feature map. Its voxel-temporal statistics identify geometrically unstable regions, and its adaptive residual weighting combines temporal stability, map-point age, and measurement range during scan-to-map registration. The system uses LiDAR geometry without semantic labels or an IMU.

[中文](README_zh-CN.md) · [Demos](#demos) · [Method](docs/method.md) · [Build and run](docs/usage.md) · [ROS interfaces](docs/usage.md#ros-interfaces)

## Demos

### KITTI 02 and 08 · Trajectory replay

<p align="center">
  <img src="assets/demos/kitti-trajectories.gif" alt="VTD-LO full trajectory replay on KITTI 02 and 08" width="760">
</p>

Saved VTD-LO estimates for KITTI 02 and 08, replayed at 20× their recorded time.

### KITTI 08 · Local feature-map replay

<p align="center">
  <img src="assets/demos/kitti-local-map.gif" alt="VTD-LO local feature-map replay on KITTI 08" width="760">
</p>

Recorded feature-map snapshots from three short windows in KITTI 08, replayed at 1× recorded time. Cyan marks edge features and amber marks surface features.

## What VTD-LO does

- **Tracks geometric stability over time.** Separate corner and surface voxel states summarize repeated map observations with online mean and variance updates.
- **Suppresses unstable geometry at two stages.** Voxel states guide correspondence selection during registration and point removal when updating the local feature map.
- **Weights retained constraints.** Temporal stability, map-point age, and range contribute to a bounded residual weight used with robust nonlinear optimization.
- **Provides an accessible ROS workflow.** The package includes KITTI and M2DGR launch files, live odometry and feature-map topics, RViz visualization, and TUM trajectory export.

The temporal model evaluates how geometry behaves across observations. An object category does not determine its label: a stationary vehicle can provide stable geometry, while an unstable region can be suppressed without identifying the object that produced it.

## Method overview

<p align="center">
  <img src="assets/figures/pipeline.png" alt="VTD-LO processing pipeline" width="920">
</p>

LiDAR scans are organized into scan lines and converted into edge and planar features. A motion prediction initializes scan-to-map registration. The optimizer uses a local feature map with voxel-temporal statistics to select and weight geometric constraints. The resulting pose transforms new features into the map, after which the local map and its temporal states are updated.

See [the method and parameter reference](docs/method.md) for the temporal statistic, residual weights, and configuration details.

## Get started

The reference environment is **Ubuntu 20.04 with ROS Noetic**. The package uses **C++17, PCL, Eigen, and Ceres**.

Follow [Build and run](docs/usage.md) to install dependencies and create a catkin workspace. With the workspace sourced and `08.bag` placed in `~/DATASET/KITTI/`:

```bash
mkdir -p "$HOME/vtd_results"
roslaunch vtd_lo kitti.launch \
  seq:=08 \
  tum_path:="$HOME/vtd_results/kitti08.txt"
```

The launch file starts bag playback, odometry, mapping, and visualization. For M2DGR, provide a bag path:

```bash
roslaunch vtd_lo m2dgr.launch \
  bag_file:="/absolute/path/hall_01.bag" \
  tum_path:="$HOME/vtd_results/hall01.txt"
```

## Map examples

| KITTI 02 | KITTI 08 |
| :---: | :---: |
| ![VTD-LO KITTI 02 map](result/02map.png) | ![VTD-LO KITTI 08 map](result/08map.png) |

## Repository

```text
VTD-LO(main)/src/vtd_lo/   ROS package: nodes, headers, launch files, RViz
assets/demos/             Video demonstrations and previews
assets/figures/           Method figure
docs/                    Method, parameters, installation, and usage
result/                  Map images and trajectory figures
```

## Acknowledgements

VTD-LO builds on the LiDAR odometry work and open-source implementations of [LOAM](https://github.com/laboshinl/loam_velodyne), [A-LOAM](https://github.com/HKUST-Aerial-Robotics/A-LOAM), [F-LOAM](https://github.com/wh200720041/floam), and [PFilter](https://github.com/yjsx/PFilter). We thank their authors and contributors. VTD-LO develops voxel-temporal dynamic suppression and adaptive residual weighting within this feature-based odometry framework.

We also thank the teams behind [KITTI](https://www.cvlibs.net/datasets/kitti/eval_odometry.php) and [M2DGR](https://github.com/SJTU-ViSYS/M2DGR) for making their datasets available. The demonstrations above use KITTI data.
