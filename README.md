# 🦾 Offline RL with Behavioral Cloning & RLPD for MuJoCo-UR5e Environments

This repository demonstrates how to:

1. Record expert demonstration trajectories.
2. Pretrain a Behavioral Cloning (BC) policy from offline demos.
3. Fine-tune or train a Reinforcement Learning agent (SAC) with demo data via RLPD.

---

## 📂 Directory Structure Overview

```bash
mujoco_sim/
├── examples/
│   ├── record_demos.py       # Script to collect and save demonstrations
│   ├── train_bc.py           # Script to train behavioral cloning policy
│   └── train_rlpd.py         # Script to train SAC/RLPD agent using BC data
```

---

## 1️⃣ Record Expert Demonstrations

Run this script to collect successful expert demonstrations using human or intervention policy:

```bash
python mujoco_sim/examples/record_demos.py \
    --exp_name peg_in_hole_demos \
    --successes_needed 5
```

- 📁 Demos are saved to: `./demo_data/{exp_name}_*_demos_TIMESTAMP.pkl`
- 🎥 Videos of successful trajectories are saved as `.mp4` in the same folder.

### Options

| Flag | Description |
|------|-------------|
| `--exp_name` | Name to prefix demo files |
| `--successes_needed` | Number of successful episodes to collect |

---

## 2️⃣ Train Behavioral Cloning (BC) Agent

Train a BC agent using the saved demonstration `.pkl` files:

```bash
python mujoco_sim/examples/train_bc.py \
    --exp_name PegInHoleFixed \
    --bc_checkpoint_path ./checkpoints/bc/ \
    --train_steps 200000
```

- ✅ BC checkpoints will be saved periodically in `bc_checkpoint_path`
- 📈 Optionally supports Weights & Biases logging

### Options

| Flag | Description |
|------|-------------|
| `--train_steps` | Number of BC training steps |
| `--bc_checkpoint_path` | Where to save BC checkpoints |
| `--eval_n_trajs` | If set > 0, runs evaluation instead of training |
| `--save_video` | Save eval rollout videos |
| `--debug` | Disable WandB logging if true |

---

## 3️⃣ Train RL Agent with Demos (RLPD)

Train a SAC agent from scratch or fine-tune using BC demos + online data:

```bash
# Learner process
python mujoco_sim/examples/train_rlpd.py \
    --learner \
    --exp_name ram_insertion \
    --checkpoint_path ./checkpoints/rlpd/ \
    --demo_path ./demo_data/converted_demo.pkl

# Actor process (in parallel terminal)
python mujoco_sim/examples/train_rlpd.py \
    --actor \
    --exp_name ram_insertion \
    --checkpoint_path ./checkpoints/rlpd/
```

### Options

| Flag | Description |
|------|-------------|
| `--learner` / `--actor` | Choose mode: learner or actor |
| `--exp_name` | Experiment name for loading config and logging |
| `--checkpoint_path` | Path to save or resume from checkpoints |
| `--demo_path` | Path(s) to demonstration `.pkl` file(s) |
| `--save_video` | Save video during evaluation |
| `--eval_n_trajs` | Run evaluation at a given checkpoint step |
| `--eval_checkpoint_step` | Step number to load for evaluation |

---

# Installation:

- run `pip install -e .` to install this package.
- run `pip install -r requirements.txt` to install sim dependencies.

# Explore the Environments

- Run `python mujoco_sim/test/test_gym_env_human.py` to launch a display window and visualize the task.

# Notes:

- Error due to `egl` when running on a CPU machine:

```bash
export MUJOCO_GL=egl
conda install -c conda-forge libstdcxx-ng
```


## 📦 Example Workflow

```bash
# Step 1: Record Demos
python mujoco_sim/examples/record_demos.py --exp_name peg_in_hole_demos --successes_needed 10

# Step 2: Train BC
python mujoco_sim/examples/train_bc.py --exp_name PegInHoleFixed --bc_checkpoint_path ./checkpoints/bc/

# Step 3: Train RLPD (Learner and Actor run in parallel)
python mujoco_sim/examples/train_rlpd.py --learner --exp_name ram_insertion --checkpoint_path ./checkpoints/rlpd/
python mujoco_sim/examples/train_rlpd.py --actor --exp_name ram_insertion --checkpoint_path ./checkpoints/rlpd/
```

---

## 🧠 Tips

- Use `--debug` to skip WandB logging during testing.
- If your MuJoCo rendering doesn't work, verify `render_mode="human"` and your system GUI settings.
- All `.pkl` demo files should contain lists of transition dictionaries.

---

## 📬 Questions?

Feel free to open an issue or reach out via your organization channels if you get stuck.


