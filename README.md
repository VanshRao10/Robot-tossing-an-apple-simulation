# Unitree G1 Hand-to-Hand Apple Toss (MuJoCo)

A physics simulation and reinforcement learning environment where a **Unitree G1 humanoid robot** (29 DoF) tosses an apple from its left hand to its right hand in **MuJoCo**.

## Overview

The project consists of a standalone physics simulation (`toss_sim.py`) and a Gymnasium-compatible RL environment (`toss_env.py`), wrapped inside a Jupyter/Colab notebook workflow.

### Key Features
* **Runtime Model Editing (`MjSpec`)**: Dynamically modifies the stock Unitree G1 scene at load time by adding a free-flying apple (`0.15 kg`, `r = 4 cm`) and attaching custom shallow "cup" palms (cylinder pad + 10-segment box rim) to each `wrist_yaw_link`. Collision bitmasks (`contype`/`conaffinity`) ensure the cups interact exclusively with the apple and never self-collide with the robot.
* **Control Architecture**:
  * **Base & Legs**: Pelvis is pinned to its standing pose (`pin_base=True`) with PD-held legs so control focuses purely on upper-body manipulation.
  * **Inverse Kinematics & Torque Control**: Damped-least-squares IK drives each palm site (matching both 3D position and upward-facing palm orientation) with a nullspace pull toward the home posture, tracked via joint PD + velocity feedforward + gravity compensation.
  * **Left Hand (Scripted Toss)**: Executes a constant-acceleration launch stroke calculated to hit the right palm's catch plane, followed by a rapid deceleration, an outward/downward retreat to avoid hitting the apple, and a minimum-jerk trajectory back to the ready pose.
  * **Right Hand (Closed-Loop / RL)**: In `toss_sim.py`, uses ballistic trajectory prediction to intercept the apple on the catch plane and applies an exponential downward "cushion" upon contact to absorb impact. In `toss_env.py`, replaces the closed-loop catch with a 3D palm velocity action space for PPO training.

## Project Structure

* **`toss_sim.py`**: Core MuJoCo simulation (`TossSim`), ballistic trajectory planner, closed-loop catching controller, plotting utilities (`plot_log`), video rendering with live synchronized telemetry (`make_video`), and parameter sweep tools (`success_sweep`).
* **`toss_env.py`**: `AppleCatchEnv`, a `gymnasium.Env` wrapper around `TossSim` designed for training RL policies (e.g., via `stable-baselines3` PPO) with randomized launch velocities and stroke lengths.
* **Notebook (`.ipynb`)**: Installs dependencies, generates the Python modules, verifies palm/cup alignment, and runs experiments.

## Installation & Requirements

Requires **Python 3.9+** and **MuJoCo >= 3.2** (for `MjSpec` support).

```bash
pip install mujoco mediapy matplotlib numpy gymnasium
