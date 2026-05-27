# Nanonis_AutoSTM_reactions_TPM

**AI-Driven Autonomous Scanning Tunneling Microscopy (STM) for On-Surface Reaction Experiments on Triphenylmethane (TPM) Molecules**

---

## Overview

This repository provides a full-stack framework for automating STM-based on-surface reaction experiments with **triphenylmethane (TPM)** molecules. Building upon our earlier [C-Br bond selectivity project](https://github.com/gggg0034/Nanonis_AutoSTM_C-Br_bond_selectivity), this system extends autonomous STM control to more complex molecular systems and lattice geometries.

Key capabilities:

- **Autonomous tip positioning** on TPM molecules using AI agents (SAC reinforcement learning, fixed-pattern, or manual control)
- **Configurable lattice construction**: supports triangle, kagome, honeycomb, kagome_lattice_all, honeycomb_lattice_all, and custom lattice shapes
- **Real-time molecular registry** with drift correction for long-duration experiments
- **On-surface reaction triggering** via bias pulse, constant-current (CC), or constant-height (CH) modes
- **Simulation environment** (`env_TPM.py`) for offline training and testing
- **Molecular visualization** (`mol_vis.py`) for keypoint heatmaps and spatial analysis
- **YAML-based configuration** (`config.yaml`) for fully reproducible experiments

This is the experimental platform behind our published works on deep-learning-driven molecular reaction automation and self-driving STM laboratories.

---

## Top-Level Structure

```
.
├── AI_Tip_TPM_Main.py           # Main entry point for autonomous STM experiment
├── AI_Tip_TPM_Main_fixed.py     # Fixed-pattern variant (no RL agent)
├── Auto_scan_class.py           # Autonomous scanning & image processing class
├── core.py                      # Core Mustard_AI_Nanonis class (TCP interface to Nanonis)
├── interface.py                 # High-level instrument interface abstractions
├── config.yaml                  # Centralized YAML configuration file
├── env_TPM.py                   # TPM molecule simulation environment
├── env_game.py                  # Interactive GUI simulation for testing
├── SAC.py                       # Standalone Soft Actor-Critic implementation
├── SAC_H_nanonis.py             # Nanonis-integrated SAC agent (hardware-ready)
├── molecule_registry.py         # Molecule registration & drift correction
├── mol_vis.py                   # Molecular visualization (keypoint heatmaps)
├── tip_para_vis.py              # Tip parameter visualization & analysis
├── utils.py                     # Shared utility functions
├── models/                      # Saved neural network model checkpoints
├── log/                         # Experiment log output directory
├── pre_buffer/                  # Pre-collected experience replay buffer
├── keypoint/                    # Keypoint detection data & training
├── ini/                         # Nanonis Initialization configuration files
├── SAC/                         # Additional SAC agent variants
├── STM_img_simu/                # STM image simulation tools
├── LICENSE                      # License file
└── README.md                    # This file
```

---

## Detailed File / Folder Description

### Core Control

#### `AI_Tip_TPM_Main.py`
Main experiment script. Orchestrates the full autonomous workflow:

1. Loads `config.yaml` and initializes the `Mustard_AI_Nanonis` core controller
2. Performs initial wide-area scan to locate molecules
3. Constructs the specified lattice (triangle / kagome / honeycomb / custom) over detected molecules
4. Iterates over each lattice site:
   - Moves tip to the target molecule
   - Zooms in for high-resolution imaging
   - Detects keypoints (center + vertices for triangle-shaped molecules)
   - Executes tip manipulation (SAC / human / fixed mode)
   - Triggers on-surface reaction (pulse / CC / CH mode)
   - Re-scans to verify reaction outcome
   - Updates molecular registry
5. Logs all experimental parameters and outcomes to `experiment.log`

**Key parameters (set via `config.yaml`):**
- `tip_agent_mode`: `"SAC"` | `"human"` | `"fixed"` — tip control strategy
- `tip_induce_mode`: `"pulse"` | `"CC"` | `"CH"` — reaction triggering method
- `lattice_shape`: `"all"` | `"triangle"` | `"kagome"` | `"honeycomb"` | `"kagome_lattice_all"` | `"honeycomb_lattice_all"` | `"costom"`
- `simulation_mode`: `true` | `false` — run in simulation vs. real hardware mode

#### `AI_Tip_TPM_Main_fixed.py`
Simplified variant that uses a pre-defined fixed tip trajectory instead of an RL agent. Useful for:
- Baseline experiments (no learning component)
- Debugging and hardware validation
- Reproducibility testing

#### `Auto_scan_class.py`
Manages autonomous scanning workflows:
- Wide-area survey scans
- Zoom-in high-resolution scans
- Image analysis and molecule detection
- Drift estimation between successive scans
- Scan parameter auto-optimization

#### `core.py`
The heart of the system — `Mustard_AI_Nanonis` class:
- **TCP/IP communication** with the Nanonis SPM controller using Nanonis' programming interface
- Tip motion control (coarse/fine approach, lateral movement, Z-height control)
- Bias voltage and tunneling current control
- Scan acquisition (topography, current, spectroscopy)
- Tip manipulation (pulse generation, CC feedback, CH approach)
- Safety interlocks and error recovery

#### `interface.py`
Higher-level abstractions built on top of `core.py`:
- Simplified scan acquisition with region-of-interest
- Molecule search and centering routines
- Tip condition monitoring and conditioning
- Workflow state machine for experiment orchestration

---

### Configuration

#### `config.yaml`
Centralized YAML configuration file. All experimental parameters are defined here — no hardcoded values.

```yaml
# --- Simulation / Hardware Mode ---
simulation_mode: false          # true = simulation, false = real Nanonis hardware

# --- Scanning Parameters ---
Scan_edge: 50.0e-9              # Wide-area scan edge length (meters)
zoom_in_tip_speed: 50.0e-9      # Zoom-in scan size (meters)
bias_rise_factor: 1.5           # Bias multiplicative factor for rise detection
current_rise_factor: 1.5        # Current multiplicative factor for rise detection

# --- Tip Manipulation Parameters ---
tip_manipulate_speed: 1.0e-9    # Tip manipulation step size (meters)
voltage: 1.0                    # Bias voltage (V)
current: 100.0e-12              # Tunneling current setpoint (A)
threshold: 0.5                  # Keypoint detection confidence threshold

# --- Lattice Configuration ---
lattice_shape: "kagome"         # Lattice type
tri_size: 3                     # Number of molecules per triangle edge

# --- Tip Control ---
tip_init_mode: "auto"           # Tip initialization mode
tip_agent_mode: "SAC"           # SAC / human / fixed
tip_induce_mode: "pulse"        # pulse / CC / CH

# --- Iteration Control ---
max_iterations: 100             # Max reaction attempts per site
max_rescan_times: 3             # Max re-scan attempts per site

# --- Keypoint Detection ---
key_point_ditection_factor: 0.8 # Detection sensitivity factor (typo preserved from original codebase)
```

---

### Reinforcement Learning

#### `SAC.py`
Clean, standalone implementation of the **Soft Actor-Critic (SAC)** algorithm:
- Policy network (actor) and twin Q-networks (critics)
- Automatic entropy tuning (temperature parameter alpha)
- Experience replay buffer
- Training loop with target network soft updates

#### `SAC_H_nanonis.py`
Nanonis-integrated SAC agent. Extends `SAC.py` with:
- **PolicyNet**: 256-unit hidden layer, outputs tip displacement (dx, dy, dz) as continuous actions
- **CriticNet**: 256-unit hidden layer, estimates Q-values for state-action pairs
- Nanonis-specific state representation (STM image patches, tip position, bias/current readings)
- Action space translation from normalized SAC outputs to physical tip displacements
- Safety constraints on tip movement (tip crash prevention, scan range limits)

#### `SAC/` (directory)
Additional SAC agent variants and experimental implementations.

#### `models/`
Saved neural network model checkpoints (PyTorch `.pt` files):
- Trained SAC policy and critic networks
- DQN model weights
- Pre-trained keypoint detection models

#### `pre_buffer/`
Pre-collected experience replay buffer data for offline RL training. Contains state-action-reward-next_state tuples collected from real or simulated experiments.

---

### Simulation & Environment

#### `env_TPM.py`
Simulation environment for TPM molecules. Implements a Gym-style interface:

- **Molecule class**: Supports `circle`, `triangle`, and `square` molecular shapes
  - Triangle molecules feature 4 keypoints: 1 center + 3 vertices
  - Each keypoint has a reaction probability model
  - Molecular orientation and position are tracked in continuous space
- **State representation**: Simulated STM topography image + keypoint positions
- **Action space**: Continuous tip displacement (dx, dy)
- **Reward function**: Based on tip proximity to target keypoint and reaction success
- **Reaction model**: Probabilistic on-surface reaction triggered when tip is within threshold distance of a keypoint

Use this environment for:
- Offline RL agent training without hardware
- Algorithm development and hyperparameter tuning
- Reproducible benchmarking

#### `env_game.py`
Interactive GUI version of `env_TPM.py` for manual experimentation:
- Real-time visualization of the molecular environment
- Mouse-driven tip positioning
- Visual feedback for keypoint detection and reaction events
- Useful for demonstrations, debugging, and intuition-building

---

### Molecule Detection & Registration

#### `molecule_registry.py`
Maintains a persistent registry of all detected molecules during an experiment:
- **Molecule matching**: Associates newly detected molecules with previously registered ones using spatial proximity and feature similarity
- **Drift correction**: Estimates and compensates for thermal/mechanical drift between successive scans by tracking molecule displacement
- **Persistent IDs**: Each molecule receives a unique ID preserved across re-scans
- **State tracking**: Tracks reaction status (unreacted / reacted / uncertain) for each molecule
- **Spatial indexing**: Efficient nearest-neighbor queries for lattice construction

#### `keypoint/` (directory)
Keypoint detection data and training pipeline:
- Training datasets (molecule images with labeled keypoints)
- Pre-trained detection models
- Data augmentation scripts
- Detection evaluation metrics

---

### Visualization & Analysis

#### `mol_vis.py`
Molecular visualization toolkit:
- `triangle_norm()`: Draws an equilateral triangle with 2D normal distribution heatmaps at each keypoint (center + 3 vertices)
- Generates publication-quality figures of molecular geometry and keypoint distributions
- Supports overlay of detected keypoints on STM images

#### `tip_para_vis.py`
Tip parameter visualization & analysis:
- Plots tip trajectory, bias/current time series, and reaction outcomes
- Statistical analysis of tip manipulation performance
- Comparison plots across different experimental conditions

#### `STM_img_simu/` (directory)
STM image simulation tools:
- Generate synthetic STM images from molecular models
- Convolution with tip geometry (tip convolution effects)
- Noise models for realistic simulation
- Useful for training keypoint detectors with synthetic data

---

### Utilities & Configuration

#### `utils.py`
Shared utility functions:
- Image processing (filtering, normalization, registration)
- Coordinate transformations (sample frame ↔ tip frame ↔ image frame)
- File I/O helpers (log writing, data serialization)
- Mathematical utilities (vector operations, statistics)

#### `ini/` (directory)
Initialization configuration files for Nanonis hardware:
- Scanner calibration parameters
- Tip approach settings
- Feedback loop PID parameters
- Communication port settings

#### `log/`
Experiment log output directory. Each run generates a timestamped subdirectory containing:
- `experiment.log`: Human-readable experiment log
- Raw STM images (topography + current channels)
- Keypoint detection results
- Reaction outcome records

---

## Quick Start

### Prerequisites

- **Python** >= 3.8
- **Nanonis SPM controller** (for hardware mode) or simulation mode
- **Git** (for cloning)

### Installation

```powershell
# 1. Clone the repository
git clone https://github.com/gggg0034/Nanonis_AutoSTM_reactions_TPM.git
cd Nanonis_AutoSTM_reactions_TPM

# 2. Create and activate a virtual environment (recommended)
python -m venv venv
venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt
```

### Configuration

```powershell
# Edit config.yaml to match your experimental setup
# Key parameters to review:
#   - simulation_mode: set to true for testing without hardware
#   - lattice_shape: choose your target lattice
#   - voltage / current: match your typical STM imaging conditions
#   - tip_agent_mode: "SAC" for RL, "fixed" for baseline, "human" for manual
notepad config.yaml
```

### Running the Experiment

```powershell
# Standard autonomous experiment (hardware mode with SAC agent)
python AI_Tip_TPM_Main.py

# Simulation mode (no hardware required)
# Set simulation_mode: true in config.yaml, then:
python AI_Tip_TPM_Main.py

# Fixed-pattern baseline (no RL)
python AI_Tip_TPM_Main_fixed.py

# Interactive simulation GUI
python env_game.py
```

### Training the RL Agent

```powershell
# Train SAC agent in simulation
cd SAC
python train.py --env tpm --episodes 1000

# The trained model will be saved to ../models/
# Update config.yaml tip_agent_mode to "SAC" and point to the model path
```

---

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| Python | >= 3.8 | Runtime |
| PyTorch | >= 1.10 | Deep learning (SAC, DQN, keypoint detection) |
| NumPy | >= 1.20 | Numerical computation |
| SciPy | >= 1.7 | Scientific computing (signal processing, spatial) |
| OpenCV (cv2) | >= 4.5 | Image processing & keypoint detection |
| Matplotlib | >= 3.4 | Visualization |
| PyYAML | >= 5.4 | YAML configuration parsing |
| scikit-image | >= 0.18 | Advanced image processing |
| scikit-learn | >= 1.0 | Machine learning utilities |
| pandas | >= 1.3 | Data manipulation & logging |
| Gym | >= 0.21 | RL environment interface |
| TCP/IP (Nanonis) | N/A | Nanonis programming interface (hardware mode only) |

> **Note**: For hardware mode, ensure your Nanonis controller is accessible via TCP/IP and the programming interface is enabled.

---

## Developer Notes

### Project Philosophy

This project follows the **configuration-over-code** principle: all experimental parameters live in `config.yaml`, not in Python source files. This ensures:
- **Reproducibility**: A single YAML file fully captures an experiment configuration
- **Version control**: Config changes are tracked separately from code changes
- **Batch experiments**: Script multiple runs by swapping config files

### Key Design Decisions

1. **SAC over DQN for tip control**: Tip manipulation requires continuous action spaces (smooth tip displacements). SAC natively handles continuous actions, while DQN requires discretization which can lead to jerky motion and tip crashes.

2. **Molecule registry with drift correction**: In UHV-STM experiments, thermal drift can shift molecule positions by nanometers over hours. The registry tracks and corrects for this drift, maintaining spatial correspondence across hundreds of scan-manipulate cycles.

3. **Modular agent architecture**: The `tip_agent_mode` parameter cleanly separates the RL agent (`SAC`), baseline (`fixed`), and manual (`human`) control modes, enabling direct A/B comparisons between learning-based and rule-based approaches.

4. **Simulation-first development**: The `env_TPM.py` simulation environment allows rapid algorithm iteration without consuming precious UHV instrument time. Models trained in simulation transfer reasonably well to real hardware with minimal fine-tuning.

### Adding New Lattice Shapes

To add a new lattice shape:

1. Define the lattice geometry in `AI_Tip_TPM_Main.py` (add a new case in the lattice construction function)
2. Add the shape name to the `lattice_shape` enum in `config.yaml`
3. Add a corresponding reaction sequence definition

### Adding New Tip Induce Modes

To add a new reaction triggering method:

1. Implement the triggering logic in `core.py`
2. Register the mode name in `AI_Tip_TPM_Main.py`
3. Add to the `tip_induce_mode` options in `config.yaml`

---

## Key File Index

| File | Lines (approx.) | Role |
|------|-----------------|------|
| `AI_Tip_TPM_Main.py` | ~500 | Main experiment workflow |
| `core.py` | ~600 | Nanonis TCP communication |
| `env_TPM.py` | ~400 | TPM simulation environment |
| `SAC.py` | ~300 | SAC algorithm implementation |
| `SAC_H_nanonis.py` | ~350 | Hardware-integrated SAC agent |
| `Auto_scan_class.py` | ~300 | Autonomous scanning routines |
| `interface.py` | ~250 | Instrument interface abstractions |
| `molecule_registry.py` | ~200 | Molecular registration & drift |
| `mol_vis.py` | ~150 | Molecular visualization |
| `utils.py` | ~150 | Shared utilities |
| `config.yaml` | ~50 | Configuration parameters |

---

## Gameplay — Interactive Simulation (`env_game.py`)

For a quick demonstration and intuition-building, launch the interactive simulation:

```powershell
python env_game.py
```

### What You See

The simulation opens a **single resizable window** (title: *"Interactive STM Environment"*, default size 500 × 500 pixels) rendered with Pygame. Everything happens in this one window — there are no side panels or split views.

**Default Setup (1 molecule):**

By default, the surface is initialized with `grid_size = (1, 1)`, meaning **one equilateral triangle molecule** is placed at the window center `(250, 250)`. The molecule is drawn at size 200 pixels (representing ~1 nm in STM scale). The spacing between neighboring molecules in multi-molecule grids is 138 pixels (= 1.38 nm). The surface is rotated 0 degrees by default.

### Color Legend — What Each Color Means

The simulation uses exactly three colors to indicate molecular states. **All colors are rendered via `pygame.draw.polygon()` as filled triangles.**

| Molecule Color | Hex Code | BGR Value | State Value | Scientific Meaning | Reward (SAC training) |
|:---|:---|:---|:---|:---|:---|
| **Gray** | `#7F7F7F` | `(127, 127, 127)` | `state = 0` | **Unreacted** — molecule has not yet been triggered, or has been reset | Baseline state |
| **Deep Blue** | `#0643F0` | `(240, 67, 6)` | `state = 1` | **Reacted — Success** — the desired on-surface reaction occurred | **+15** (minus 0.2 × reaction_time) |
| **Bright Red** | `#FF0000` | `(0, 0, 255)` | `state = 2` | **Reacted — Failure** — the tip induced an undesired/indeterminate outcome | **-5** (minus 0.2 × reaction_time) |

> **Note**: In the source code, the deep blue color variable is named `bad_react_color` and the red is named `suc_react_color`, but the actual state-to-reward mapping is the reverse: state 1 (blue) = success, state 2 (red) = failure. The variable names are a known code-level inconsistency — the table above reflects the **actual visual behavior you see on screen**.

### Tip & Keypoint Markers

- **Tip**: A bright **green circle** (`(0, 255, 0)`, radius 5 pixels) — this is your STM tip. Its position follows your mouse cursor in real time.
- **Keypoints**: Four **black dots** (`(0, 0, 0)`, radius 3 pixels) overlaid on the triangle molecule:
  - **3 vertices** — one at the top, one at the bottom-left, one at the bottom-right of the triangle
  - **1 center point** — at the geometric center of the triangle
  - These are the interaction hotspots where a reaction can be triggered. The reaction probability is highest when the tip is directly on a keypoint.

### HUD (On-Screen Display)

The window shows real-time information overlaid as white-background text:

| HUD Element | Position | Shows |
|:---|:---|:---|
| **Bias Voltage & Current** | Top-left corner | `Bias Voltage: -4.00V  Current: 0.20nA` — the current tip bias and tunneling current setpoint |
| **Reaction Probabilities** | Vertical center, left side | `Suc: 0.XX  Non: 0.XX  Fail: 0.XX` — **real-time probabilities** of success / no reaction / failure given the current tip position, bias, and current. These update on every mouse movement. |
| **Tip Coordinates** | Floating near the tip | `Tip: (xxx, yyy)` — pixel coordinates of the tip, shown just to the upper-right of the green dot |

### Complete Keyboard & Mouse Controls

**Every key binding is hardcoded in the `run()` method's event loop.** There are no configurable keybindings.

#### Mouse Controls

<div style="overflow-x: auto;">

| Input | Code (`event.type`) | Behavior |
|:---|:---|:---|
| **Move mouse** | `pygame.MOUSEMOTION` | Tip (green dot) follows cursor position. Reaction probabilities (`Suc/Non/Fail`) are **recalculated and updated in real time** via `tip.culculate_reaction_factor(molecule)` for every pixel movement. No reaction is triggered — this is a preview mode. |
| **Left-click** | `pygame.MOUSEBUTTONDOWN` | **Triggers a reaction** at the current tip position. Calls `tip.interact(molecule)` on every molecule in the surface. The reaction outcome is determined by the probability model (see below). The molecule's state and color will change immediately if a reaction occurs. |

</div>

> **No right-click or middle-click is handled** — those events are ignored.

#### Keyboard Controls

<div style="overflow-x: auto;">

| Key | Code (`event.key`) | Behavior | Console Output |
|:---|:---|:---|:---|
| **R** | `pygame.K_r` | **Reset all molecules** to state 0 (gray / unreacted). Equivalent to "wipe the surface clean." This is the only way to undo a reaction without restarting the program. | *(none)* |
| **Space** | `pygame.K_SPACE` | **Print molecule geometry** to the terminal. Outputs the `render_points` (3 vertex coordinates) of every molecule on the surface. Useful for debugging lattice geometry. | `[(x1,y1), (x2,y2), (x3,y3)]` per molecule |
| **Right Arrow →** | `pygame.K_RIGHT` | **Rotate all molecules +1 degree** clockwise. Applies `molecule.rotate(1)` to every molecule. | `angle: <new_angle>` (e.g. `1.0`, `2.0`, ...) |
| **Left Arrow ←** | `pygame.K_LEFT` | **Rotate all molecules -1 degree** counter-clockwise. Applies `molecule.rotate(-1)` to every molecule. | `angle: <new_angle>` |
| **W** | `pygame.K_w` | **Increase bias voltage by +0.05 V**. Calls `tip.change_bias(0.05)`. | `bias: <new_voltage>` |
| **S** | `pygame.K_s` | **Decrease bias voltage by -0.05 V**. Calls `tip.change_bias(-0.05)`. | `bias: <new_voltage>` |
| **A** | `pygame.K_a` | **Increase tunneling current by +0.05 nA**. Calls `tip.change_current(0.05)`. | `current: <new_current>` |
| **D** | `pygame.K_d` | **Decrease tunneling current by -0.05 nA**. Calls `tip.change_current(-0.05)`. | `current: <new_current>` |

</div>

> **WASD does NOT move the tip.** WASD adjusts the bias voltage (W/S) and tunneling current (A/D). The tip is moved exclusively by the mouse. This is consistent with real STM operation where bias and current are set electronically while tip positioning is a separate control.

### Reaction Probability Model — The Math Behind the Colors

When you click to trigger a reaction, the outcome is determined by a two-factor probability model. You can preview these probabilities at any time by hovering the tip — the `Suc/Non/Fail` display updates live.

#### Factor 1: Position Factor (`molecule.interact_area()`)

The reaction probability depends on the tip's **distance to the four keypoints**. Each keypoint has a 2D Gaussian distribution centered on it:

| Keypoint | Standard Deviation (σ) | Weight | Location |
|:---|:---|:---|:---|
| **Center** | 18 pixels | **1.0** (highest) | Molecule geometric center |
| **Top vertex** | 15 pixels | 0.6 | ~115 px above center |
| **Left vertex** | 15 pixels | 0.6 | ~100 px left, ~58 px below center |
| **Right vertex** | 15 pixels | 0.6 | ~100 px right, ~58 px below center |

The four Gaussians are summed and normalized by the maximum value. **The center keypoint has the highest weight (1.0 vs 0.6 for edges)**, meaning the tip is most effective when placed directly at the molecular center.

> The 2D Gaussian formula used is: `(1 / (2π·σx·σy)) · exp(-(dx²/(2σx²) + dy²/(2σy²)) · weight)`

#### Factor 2: Bias/Current Parameter Factor (`molecule.interact_parameters()`)

The reaction success also depends on how close the **bias voltage** and **tunneling current** are to the molecule's optimal values.

**Optimal (best) parameters for TPM triangle molecules:**

| Parameter | Optimal Value | Defined In |
|:---|:---|:---|
| **Bias Voltage** | **-4.0 V** | `molecule.optimal_bias` (line 68) |
| **Tunneling Current** | **0.2 nA** | `molecule.optimal_current` (line 69) |

The parameter factor is computed using the `suc()` and `ind()` functions from `utils.py`. The scaled deltas are:

- `x = |bias - (-4.0)| × 3` (bias deviation scaled by factor 3)
- `y = (current - 0.2) × 12` (current deviation scaled by factor 12)

The success contribution from parameters:

```
parameters_suc_factor = normal_distribution_2d(x, y, 0, 0, 1, 1) × 0.96
```

where `normal_distribution_2d` is a standard 2D Gaussian with σx=σy=1, centered at (0,0), and normalized so the peak value is 1. This means: **at exactly bias=-4.0V and current=0.2nA, the parameter success factor reaches its maximum of 0.96**.

#### Combined Final Probabilities

When you click (trigger `tip.interact()`):

```
position_factor  = interact_area(tip_position)       # [0, 1] — from Gaussian keypoint model
weight_suc       = 0.95                               # fixed success weight
weight_fail      = 0.05                               # fixed base failure weight

suc_factor  = position_factor × parameters_suc_factor × 0.95
fail_factor = position_factor × 0.05 + parameters_ind_factor
non_factor  = 1 - suc_factor - fail_factor
```

The reaction outcome is then decided by a random roll (`random.random()`) against these probabilities:

- If `random() < suc_factor` → **state becomes 1** (blue, success)
- If `random() > 1 - fail_factor` → **state becomes 2** (red, failure)
- Otherwise → **state stays 0** (gray, no reaction), and the reaction time is set to either the computed time or a maximum of 8 seconds

#### Reaction Time

After each interaction, a simulated reaction time is computed:

```
reaction_time = max((non_factor + parameters_non_factor) × 6 + Gaussian_noise(0,1), 0.01)
```

This simulates the time it takes for the on-surface reaction to complete, with a minimum of 0.01 seconds and no strict upper bound (capped at 8 seconds if no reaction occurs). This reaction time is used in the SAC reward function: successful reactions get a higher reward when they happen faster.

### Best / Optimal Performance — How to Maximize Success

To achieve the **highest possible success rate** in a single click:

1. **Position**: Move the tip (green dot) exactly onto the **center keypoint** (the black dot at the geometric center of the gray triangle). The center has weight=1.0, the highest among all keypoints. You can verify you're on target by watching the `Suc` value in the HUD — it will peak when the tip is at the center.

2. **Bias**: Press **W** or **S** to tune the bias voltage to exactly **-4.00 V**. The display updates in real time at top-left.

3. **Current**: Press **A** or **D** to tune the tunneling current to exactly **0.20 nA**. The display updates in real time at top-left.

4. **Click** to trigger the reaction.

When all three conditions are perfectly met, the `Suc` rate will show its maximum possible value, the `Fail` and `Non` rates will be minimal, and the molecule will most likely turn **deep blue** (state 1 = success). If the result is red (state 2 = failure) or gray (state 0 = no reaction), adjust and try again after pressing **R** to reset.

### Batch / Multi-Molecule Experiments

Although the default setup has only 1 molecule, you can change `grid_size` in the `Surface.__init__()` call (line 210 of `env_game.py`) to create larger grids. For example, setting `grid_size = (3, 3)` creates 9 molecules arranged in an equilateral triangle grid. The spacing between molecules is hardcoded to `138 pixels` (1.38 nm equivalent).

When multiple molecules are present:
- The tip **interacts with all molecules simultaneously** on click (the `run()` loop calls `tip.interact(molecule)` for every molecule)
- The reaction probability display (`Suc/Non/Fail`) shows values for the **last molecule's calculation** (since `culculate_reaction_factor()` is called on each molecule in sequence during mouse movement)
- All molecules rotate together with Left/Right arrow keys
- **R** resets all molecules to state 0

### Typical Workflow

A typical interactive session goes like this:

```
1. python env_game.py                          # Launch
2. [Observe] 1 gray triangle at center         # Default setup
3. [Hover mouse] Watch Suc/Non/Fail change     # Preview probabilities
4. [W/S/A/D] Tune bias to -4.00V, current to 0.20nA  # Set optimal parameters
5. [Move to center keypoint] Suc rate peaks    # Target center
6. [Left-click] Trigger reaction               # Execute
7. [Observe color change] Blue or red          # See result
8. [Press R] Reset to try again                # Repeat
```

This interactive mode is excellent for:
- **Understanding the tip manipulation workflow** before running on real hardware — the same probability model and state transitions are used in the actual SAC training environment (`env_TPM.py`)
- **Developing intuition for SAC reward design** — you can see exactly what state transitions yield what rewards (success=+15, failure=-5, no reaction=-0.1 minus time penalty)
- **Debugging lattice construction logic** — press Space to inspect molecular geometry in the terminal
- **Demonstrating the on-surface reaction concept** to collaborators without needing UHV equipment
- **Tuning bias/current sensitivity** — the WASD controls let you feel how sensitive the success rate is to parameter deviations

---

## License

See [LICENSE](./LICENSE) for details.

---

## Contact

- **Author**: Zhiwen Zhu 
- **email**:zhiwenzhu@shu.edu.cn
- **Institution**: Shanghai University

For questions, bug reports, or collaboration inquiries, please open a GitHub issue or contact the author directly.
