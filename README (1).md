# MiniMax H3 on a 14–15 GB NVIDIA T4 with ComfyUI

Run **MiniMax H3 text-to-video with native audio** in **ComfyUI on a single NVIDIA Tesla T4 (14–15 GB VRAM)** using a low-VRAM workflow.

This repository provides a ready-to-run **Google Colab / Kaggle notebook** that installs ComfyUI, adds the required H3 low-VRAM nodes, downloads the required model files, patches the H3 ClipProj workflow for a T4, launches ComfyUI, and exposes it through a Cloudflare Quick Tunnel.

> **Hardware target:** NVIDIA Tesla T4, ~14–15 GB VRAM  
> **Tested configuration in the notebook:** PyTorch 2.11.0+cu130, CUDA 13.0, ComfyUI 0.38.0, Tesla T4  
> **First test:** 384×224, 22 frames, 6 steps (~0.92 s at 24 FPS)

## ✨ What this project does

The notebook is designed around the **ComfyUI MiniMax H3 workflow**, not standalone DiffSynth.

It uses:

- **MiniMax H3** diffusion model
- **Qwen3-VL 4B FP8** text encoder through ClipProj
- **MiniMax H3 ClipProj** projection
- **MiniMax H3 LowVRAM** node
- H3 video VAE
- H3 audio VAE
- ComfyUI workflow with video + audio output
- `--lowvram` and FP16 VAE settings for T4-class GPUs
- Cloudflare Quick Tunnel for browser access from a hosted notebook

The notebook's ClipProj route avoids the much larger H3 32B text encoder and is intended for a single tight GPU.

## 📁 Repository

```text
.
├── MiniMax_H3_ComfyUI_T4.ipynb
└── README.md
```

## 🚀 Google Colab setup

### 1. Open the notebook

Upload/open:

`MiniMax_H3_ComfyUI_T4.ipynb`

in Google Colab.

### 2. Enable a GPU

In Colab:

**Runtime → Change runtime type → T4 GPU**

Then reconnect/restart the runtime if necessary.

The first notebook cell checks:

- CUDA availability
- GPU name
- VRAM

The notebook is specifically tuned for a **Tesla T4**.

### 3. Run the cells in order

Run the notebook from top to bottom.

The main stages are:

1. GPU check
2. Install ComfyUI
3. Install:
   - `ComfyUI-ClipProj`
   - `ComfyUI-MiniMaxH3-LowVRAM`
   - `ComfyUI-Spectrum-MiniMax-H3`
4. Download H3 model files
5. Fix/normalize the downloaded model paths
6. Download and patch the H3 ClipProj workflow
7. Start ComfyUI on port `8188`
8. Verify the local ComfyUI server
9. Create a Cloudflare Quick Tunnel
10. Open the generated public URL

### 4. Open ComfyUI

The tunnel cell prints something similar to:

```text
🔥 NEW COMFYUI URL
https://xxxxx.trycloudflare.com
```

Open that URL in your browser.

### 5. Load the workflow

Inside ComfyUI, load:

```text
/content/ComfyUI/workflow_minimax_h3_t4.json
```

You can drag the JSON workflow into the ComfyUI canvas.

Then click:

**Queue Prompt**

The generated video is saved under:

```text
/content/ComfyUI/output/video/
```

## 🎬 First generation settings

The notebook starts conservatively:

```text
Resolution: 384 × 224
Frames:     22
Steps:      6
FPS:        24
Seed:       42
```

22 frames is approximately **0.92 seconds** at 24 FPS.

After a successful test, the notebook recommends trying:

```text
384 × 224
73 frames
6 steps
```

Do **not** immediately jump to large resolutions and long videos on a 14–15 GB T4.

## 🧠 H3 frame rule

The workflow uses the H3 `17n + 5` frame pattern.

Examples:

| Frames | Approx. duration @ 24 FPS |
|---:|---:|
| 22 | 0.92 s |
| 39 | 1.63 s |
| 56 | 2.33 s |
| 73 | 3.04 s |
| 90 | 3.75 s |
| 107 | 4.46 s |
| 124 | 5.17 s |

## 📦 Model storage requirements

The current notebook downloads approximately:

| Component | Approx. size |
|---|---:|
| Qwen3-VL 4B FP8 text encoder | 5.2 GB |
| H3 ClipProj | 0.5 GB |
| H3 int8 ConvRot diffusion model | 34 GB |
| H3 video VAE | 5.2 GB |
| H3 audio VAE | 0.6 GB |
| **Total** | **~45.5 GB** |

The exact download size can change as upstream model repositories are updated.

### ⚠️ Important hosted-notebook storage warning

The **34 GB diffusion checkpoint is the major constraint**.

Your runtime must have enough disk space for the complete model set plus ComfyUI and temporary files.

If a hosted runtime does not provide enough persistent/ephemeral disk, the notebook can fail during model download even if the GPU has enough VRAM.

## ☁️ Kaggle setup

### 1. Create a Kaggle Notebook

Create a new Kaggle Notebook and upload:

```text
MiniMax_H3_ComfyUI_T4.ipynb
```

### 2. Enable GPU

In Kaggle Notebook settings:

**Accelerator → GPU**

Use a **Tesla T4** if available.

This workflow is designed around a **single ~14–15 GB T4**, not by combining two GPUs.

### 3. Enable Internet

Kaggle Notebook:

**Settings → Internet → On**

The notebook needs internet access to clone GitHub repositories and download the model files.

### 4. Run the notebook

Run the cells in order.

The same workflow is used:

```text
GPU
 ↓
ComfyUI
 ↓
H3 custom nodes
 ↓
H3 model downloads
 ↓
Workflow patch
 ↓
ComfyUI server
 ↓
Cloudflare Tunnel
 ↓
Browser
```

### ⚠️ Kaggle storage

Before starting a full model download, check the available disk space:

```bash
!df -h /content
```

or:

```bash
!df -h
```

The complete H3 setup needs roughly **45+ GB** for the model files alone.

Kaggle's available notebook storage can vary by runtime/account/environment. If the available disk is below the required space, use a storage approach that provides enough capacity or use Colab/a runtime with sufficient disk.

## 🔧 Useful commands

Check GPU:

```bash
!nvidia-smi
```

Check disk:

```bash
!df -h
```

Check ComfyUI:

```python
import requests
print(requests.get("http://127.0.0.1:8188").status_code)
```

Check generated videos:

```bash
!find /content/ComfyUI/output -type f \( -iname '*.mp4' -o -iname '*.webm' \) -printf '%T@ %p\n' | sort -nr | head
```

## 🛠️ How the low-VRAM setup works

### ClipProj

Instead of loading the much larger H3 32B text encoder, the workflow uses:

```text
Qwen3-VL 4B FP8
        +
MiniMax H3 ClipProj
```

This significantly reduces the text-encoder memory requirement.

### H3 LowVRAM

The workflow inserts the `MiniMaxH3LowVRAM` node between the diffusion model loader and the H3 sigma-shift node.

The notebook uses:

```text
memory profile: low_vram
chunk tokens: 4096
```

This is intended to reduce peak MLP memory usage.

### ComfyUI

ComfyUI is launched with:

```bash
--lowvram
--fp16-vae
```

and listens on:

```text
0.0.0.0:8188
```

## 🌐 Cloudflare Quick Tunnel

The notebook automatically downloads `cloudflared` and creates a temporary:

```text
https://xxxxx.trycloudflare.com
```

URL pointing to:

```text
http://127.0.0.1:8188
```

Quick Tunnels are intended for temporary/testing use and do **not** provide a production uptime guarantee.

## ❗ Troubleshooting

### `No CUDA GPU is attached`

Enable the GPU accelerator and restart/reconnect the runtime.

### `No space left on device`

The H3 model set is large. Check:

```bash
!df -h
```

The diffusion checkpoint alone is about 34 GB.

### CUDA out-of-memory

Start with:

```text
384 × 224
22 frames
6 steps
```

Do not immediately increase resolution, frame count, or other memory-heavy settings.

### ComfyUI URL is not accessible

Check the local server first:

```python
import requests
print(requests.get("http://127.0.0.1:8188", timeout=10).status_code)
```

Then rerun the Cloudflare Tunnel cell.

### Workflow cannot find a model

The notebook includes a path-normalization step because Hugging Face downloads can create nested directories.

Run the model verification cell and make sure the required files exist under:

```text
/content/ComfyUI/models/
```

## ⚖️ Model and third-party components

This repository contains the notebook/workflow setup and installation instructions. It does **not** redistribute the large MiniMax H3 model weights.

The notebook downloads model files from their respective upstream Hugging Face repositories.

You are responsible for complying with the licenses and terms of:

- MiniMax H3
- ComfyUI
- ComfyUI-ClipProj
- ComfyUI-MiniMaxH3-LowVRAM
- ComfyUI-Spectrum-MiniMax-H3
- Qwen3-VL
- Cloudflare `cloudflared`

## 🔗 Upstream projects

- [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- [ComfyUI-ClipProj](https://github.com/nicolab28/ComfyUI-ClipProj)
- [ComfyUI-MiniMaxH3-LowVRAM](https://github.com/lericogit/ComfyUI-MiniMaxH3-LowVRAM)
- [ComfyUI-Spectrum-MiniMax-H3](https://github.com/xmarre/ComfyUI-Spectrum-MiniMax-H3)

## 🔎 SEO / GitHub keywords

`MiniMax H3`, `MiniMax H3 ComfyUI`, `MiniMax H3 T4`, `MiniMax H3 14GB VRAM`, `MiniMax H3 15GB VRAM`, `MiniMax H3 Colab`, `MiniMax H3 Kaggle`, `ComfyUI T4`, `ComfyUI low VRAM`, `text to video`, `AI video generation`, `video generation with audio`, `Qwen3-VL`, `ClipProj`, `Tesla T4`, `Google Colab AI video`, `Kaggle AI video`, `open source video generation`

## ⭐ Star the repo

If this notebook helps you run MiniMax H3 on a low-VRAM T4, consider starring the repository and sharing your results.

---

### Disclaimer

This notebook is an experimental community setup for running MiniMax H3 through ComfyUI on constrained GPU hardware. Upstream repositories, model files, APIs, hosted notebook environments, and download paths can change over time.
