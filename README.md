# Unsloth AMD Finetuning Workshop

A step-by-step guide to set up your environment and run the finetuning script.

---

## Prerequisites

- Ubuntu 22.04 / 24.04 machine with an AMD GPU
- Internet access

---

## Step 1 — Install ROCm (skip if already installed)

> Full reference: [Install Ryzen Software for Linux with ROCm](https://rocm.docs.amd.com/projects/radeon-ryzen/en/latest/docs/install/installryz/native_linux/install-ryzen.html)

ROCm must be installed on the **host machine** so the Docker container can access your AMD GPU.

Check if ROCm is already installed:

```bash
rocminfo
```

If the command is not found, install from [Install Ryzen Software for Linux with ROCm](https://rocm.docs.amd.com/projects/radeon-ryzen/en/latest/docs/install/installryz/native_linux/install-ryzen.html).


## Step 2 — Pull the Docker image

```bash
docker pull rocm/vllm-dev:rocm7.2.1_navi_ubuntu24.04_py3.12_pytorch_2.9_vllm_0.16.0
```

> This image includes Python 3.12, PyTorch 2.9, Triton 3.5, and ROCm support for AMD GPUs.
> The download may take several minutes depending on your connection.

---

## Step 3 — Start the container

Run the container with your home directory mounted so files persist between sessions:

```bash
docker run -it \
  --name workshop-env \
  --ipc=host \
  --device=/dev/kfd \
  --device=/dev/dri \
  --group-add video \
  --group-add render \
  --security-opt seccomp=unconfined \
  -v $HOME:/root/home \
  -v $PWD:/workspace \
  -v $HOME/.cache/huggingface:/root/.cache/huggingface \
  -w /workspace \
  -p 8888:8888 \
  -e PYTHONUNBUFFERED=1 \
  -e HF_HOME=/root/.cache/huggingface \
  rocm/vllm-dev:rocm7.2.1_navi_ubuntu24.04_py3.12_pytorch_2.9_vllm_0.16.0 \
  bash
```

> **What the flags do:**
> - `--device=/dev/kfd` and `--device=/dev/dri` — exposes your AMD GPU to the container
> - `--group-add video --group-add render` — grants GPU access permissions
> - `-v $HOME:/root/home` — mounts your home directory so your files are accessible
> - `-v $PWD:/workspace` — mounts the current folder (where `finetune.py` lives) into `/workspace`
> - `-w /workspace` — sets the working directory inside the container

You should now be inside the container shell at `/workspace`.

---

## Step 4 — Install Unsloth (inside the container)

Run these three commands in order:

```bash
pip install notebook
```

```bash
pip install --no-deps unsloth unsloth-zoo
```

```bash
pip install --no-deps git+https://github.com/unslothai/unsloth-zoo.git
```

```bash
pip install "unsloth[amd] @ git+https://github.com/unslothai/unsloth"
```

```bash
pip install bitsandbytes
```

Verify the install:

```bash
python -c "import unsloth; print('Unsloth OK')"
```

---

## Step 6 — Configure your finetuning run

```bash
cd /workspace

curl -L -o "Gemma4_(E2B)_Reinforcement_Learning_Sudoku_Game.ipynb" "https://raw.githubusercontent.com/iswaryaalex/Unsloth-RL-Workshop-on-Radeon/main/Gemma4_(E2B)_Reinforcement_Learning_Sudoku_Game.ipynb"

jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser --allow-root

```

Key settings to review before running:

| Setting | What to change |
|---|---|
| `MODEL_NAME` | The base model to finetune |
| `DATASET_NAME` / `DATASET_SPLIT` | Your training data |
| `MAX_STEPS` / `NUM_TRAIN_EPOCHS` | Training duration |
| `LORA_R` | LoRA rank — higher = more capacity |
| `OUTPUT_DIR` | Where the trained model is saved |
| `HF_TOKEN` / `HF_REPO_ID` | Fill in to push to HuggingFace Hub |

---



---

## Tips

- **Full training run:** in `finetune.py` set `MAX_STEPS = None` and `NUM_TRAIN_EPOCHS = 1`
- **Save VRAM:** keep `LOAD_IN_4BIT = True` and `FULL_FINETUNING = False`
- **Push to Hub:** set `HF_TOKEN` and `HF_REPO_ID` in the config — the script will push automatically after training
- **Cache:** the container mounts `$HOME` so HuggingFace model downloads are cached across runs
