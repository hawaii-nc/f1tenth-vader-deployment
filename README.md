# F1Tenth VADER — RMA Deployment

> **Superseded.** This is a backup of the on-car code as it ran in July 2026. The maintained deployment node, trained models and install checklist are in [f1tenth-rma](https://github.com/hawaii-nc/f1tenth-rma) (`src/rma_deploy_pkg/`, `runs/`, `deploy/`).
>
> Do not deploy the checkpoints in this repo for new experiments: this node runs at 50 Hz while the policies were trained at 100 Hz, it has no active grip probe, and the July pipeline had training bugs (see `docs/BUGS_AND_LESSONS.md` in f1tenth-rma). Keep this repo for reproducing the July pilot only.

ROS2 deployment package that runs a trained **Rapid Motor Adaptation** policy on the physical F1Tenth car ("VADER", Jetson, JetPack 5.4.1 / Python 3.8).

The policy drives autonomously using **only LiDAR (108 beams) + odometry** — no map, GPS, or raceline required — and works on any track shape, including our physical cardboard track.

## How It Works

- **Observation (113D):** 5 base signals (velocity / steering feedback) + 108 LiDAR ranges.
- **Phase 1 checkpoint:** Base PPO policy — the most thoroughly validated option for first on-car tests.
- **Phase 2 checkpoint:** Adaptation module that infers tire grip online. Included for testing once Phase 1 is confirmed on the car.
- **Safety:** Policy caps speed at **3.0 m/s**. Hold **R2** to engage autonomous control; release R2 (or hold L1 for manual override) immediately if anything looks wrong. Always secure the car (wheels off ground) for bench tests.

## Repo Layout

- `src/rma_deploy_pkg/` — ROS2 package: deployment node, `veh_launch.py` (vehicle) and `sim_launch.py` (simulation), and trained checkpoints (`phase1_*.pt`, `phase2_*.pt`).
- `start_rma.sh` / `start_rma_noadapt.sh` — Launch helpers with the car's ROS environment preconfigured.
- `joy_manager.py` — Joystick trial manager: logs trial start / pass events with timestamps for experiment tracking.
- `TA_INSTALL_CHECKLIST.md` — Full install checklist (PyTorch wheel for Jetson, PYTHONPATH, colcon build, test launch).

## Quick Start (on the car)

cd ~/ws && colcon build --packages-select rma_deploy_pkg
source install/local_setup.bash
ros2 launch rma_deploy_pkg veh_launch.py checkpoint:=/home/$USER/ws/src/rma_deploy_pkg/checkpoints/phase1_v28.pt

See `TA_INSTALL_CHECKLIST.md` for the complete setup, including the Jetson PyTorch wheel install.

## Context

Deployment half of my UNLV NSF REU project on Rapid Motor Adaptation. Training code: https://github.com/hawaii-nc/research_ws · Project overview & results: https://github.com/hawaii-nc/NSF-REU-UNLV-Smart-Cities-2026
