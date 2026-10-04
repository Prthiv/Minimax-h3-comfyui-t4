# MiniMax H3 — ComfyUI on NVIDIA T4

Run **MiniMax H3 text-to-video with native audio** in ComfyUI on a single NVIDIA Tesla T4 GPU using a low-VRAM workflow.

This repository provides a ready-to-run setup for **Google Colab and Kaggle**, including the ComfyUI workflow, model setup, low-VRAM configuration, and Cloudflare Quick Tunnel for remote access.

## Quick Start

### Google Colab

[**Open in Google Colab**](YOUR_COLAB_LINK_HERE)

### Kaggle

[**Open in Kaggle**](https://www.kaggle.com/code/parthivsm/minmax-h3-comfyui)

> **Recommended:** Use a Tesla T4 GPU. The notebook is specifically tuned for approximately 14–15 GB of VRAM.

---

## What is this?

This project makes it possible to run the **MiniMax H3 video generation workflow** through ComfyUI on hardware with limited VRAM.

The workflow uses:

- MiniMax H3 diffusion model
- Qwen3-VL 4B FP8 text encoder
- MiniMax H3 ClipProj
- MiniMax H3 LowVRAM optimization
- H3 video VAE
- H3 audio VAE
- ComfyUI
- Cloudflare Quick Tunnel

The ClipProj workflow uses a smaller Qwen3-VL 4B text encoder instead of the much larger original H3 text encoder, making the setup considerably more suitable for a T4-class GPU.

---

## Hardware

| Component | Recommended |
|---|---|
| GPU | NVIDIA Tesla T4 |
| VRAM | ~14–15 GB |
| Platform | Google Colab / Kaggle |
| CUDA | 13.0 |
| PyTorch | 2.11.0+cu130 |
| ComfyUI | 0.38.0 |
| Storage | ~45 GB+ |

The workflow was developed and tested around a **Tesla T4 with 14.56 GB VRAM**.

---

## Model Storage

The required model files are downloaded automatically from their upstream sources.

Approximate storage requirements:

| Component | Size |
|---|---:|
| Qwen3-VL 4B FP8 text encoder | ~5.2 GB |
| H3 diffusion checkpoint | ~34 GB |
| H3 video VAE | ~5.2 GB |
| H3 audio VAE | ~0.6 GB |
| **Total** | **~45 GB+** |

Make sure your runtime has enough disk space before starting.

Model weights are **not redistributed in this repository**. The notebook downloads the required files from their respective upstream sources.

---

## First Generation

The notebook starts with a conservative configuration designed to make the first generation easier on a T4:

```text
Resolution: 384 × 224
Frames:     22
Steps:      6
FPS:        24
```

22 frames produces approximately **0.92 seconds** of video at 24 FPS.

Once the workflow is working, you can increase the number of frames.

### H3 Frame Grid

MiniMax H3 uses the following frame pattern:

```text
17n + 5
```

At 24 FPS:

| Frames | Approx. Duration |
|---:|---:|
| 22 | 0.92 s |
| 39 | 1.63 s |
| 56 | 2.33 s |
| 73 | 3.04 s |
| 90 | 3.75 s |
| 107 | 4.46 s |
| 124 | 5.17 s |

Longer generations require significantly more compute and may require additional optimization depending on the GPU/runtime.

---

## Google Colab

1. Open the Colab notebook.
2. Select:

```text
Runtime → Change runtime type
```

3. Select a **T4 GPU**.
4. Reconnect the runtime.
5. Run the notebook from the beginning.
6. Wait for the model downloads and ComfyUI setup to complete.
7. The notebook starts ComfyUI and creates a Cloudflare Quick Tunnel.
8. Open the generated `trycloudflare.com` URL.

The notebook launches ComfyUI using low-VRAM settings suitable for the T4.

---

## Kaggle

You can run the notebook directly on Kaggle:

**[Open the MiniMax H3 Kaggle Notebook](https://www.kaggle.com/code/parthivsm/minmax-h3-comfyui)**

Before running:

1. Open the notebook.
2. Enable a GPU accelerator.
3. Enable Internet access.
4. Make sure the runtime has enough storage for the model downloads.
5. Run the notebook from the beginning.

> Kaggle storage and GPU availability can change. If the runtime does not provide enough disk space for the required model files, the notebook may fail during model download.

---

## ComfyUI Workflow

The workflow includes the main components required for MiniMax H3 generation:

```text
Text Prompt
     ↓
Qwen3-VL 4B FP8
     ↓
ClipProj
     ↓
MiniMax H3
     ↓
Video VAE + Audio VAE
     ↓
CreateVideo
     ↓
Video + Audio Output
```

The workflow is configured for low-VRAM execution and includes the H3 video/audio generation pipeline.

---

## Low-VRAM Optimization

This setup uses the **MiniMax H3 LowVRAM** workflow to reduce peak memory usage on GPUs such as the Tesla T4.

ComfyUI is launched with low-VRAM options including:

```bash
--lowvram --fp16-vae
```

The low-VRAM optimization helps reduce memory usage during the H3 MLP computation.

It does not eliminate all possible out-of-memory situations. Increasing resolution, frame count, or other settings can increase VRAM requirements.

---

## Cloudflare Quick Tunnel

The notebook automatically creates a temporary Cloudflare Quick Tunnel for the local ComfyUI server.

ComfyUI runs locally on:

```text
http://127.0.0.1:8188
```

The notebook then exposes it through a temporary:

```text
https://xxxxx.trycloudflare.com
```

This allows you to access the ComfyUI interface from your browser without manually configuring port forwarding.

---

## Project Structure

```text
MiniMax-H3-ComfyUI-T4/
│
├── MiniMax_H3_ComfyUI_T4.ipynb
├── README.md
└── .gitignore
```

The notebook handles the majority of the installation and configuration automatically.

---

## Troubleshooting

### CUDA GPU not detected

Make sure the runtime has a GPU enabled.

For Google Colab:

```text
Runtime → Change runtime type → T4 GPU
```

For Kaggle, enable the GPU accelerator in the notebook settings.

---

### Out of memory

Try:

- Lowering the resolution
- Reducing the number of frames
- Keeping the low-VRAM configuration enabled
- Starting with the default 384×224 / 22-frame configuration

Get the first generation working before increasing the workload.

---

### Insufficient storage

The H3 model files require approximately **45 GB or more** of storage.

Check available disk space before starting the model download.

---

### ComfyUI does not open

Check that the ComfyUI server is running on:

```text
127.0.0.1:8188
```

If using the notebook's Cloudflare tunnel, wait for the `trycloudflare.com` URL to appear before opening it.

---

## Upstream Projects

This project builds on the work of the MiniMax H3 and ComfyUI communities.

Relevant components include:

- [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- [ComfyUI-MiniMaxH3-LowVRAM](https://github.com/lericogit/ComfyUI-MiniMaxH3-LowVRAM)
- [ComfyUI-Spectrum-MiniMax-H3](https://github.com/xmarre/ComfyUI-Spectrum-MiniMax-H3)

Please refer to the upstream repositories for their respective licenses and documentation.

---

## Why this project?

Running large video-generation models on consumer or free-tier GPUs can be difficult because of VRAM and storage requirements.

This project focuses on making the MiniMax H3 workflow more accessible by combining:

- Qwen3-VL 4B FP8
- ClipProj
- H3 LowVRAM optimization
- ComfyUI
- Tesla T4-class hardware
- Google Colab
- Kaggle

The goal is a practical, reproducible setup that can be started without requiring a high-end local GPU.

---

## Roadmap

- [x] MiniMax H3 ComfyUI workflow
- [x] T4 low-VRAM configuration
- [x] Qwen3-VL 4B FP8 ClipProj setup
- [x] Google Colab setup
- [x] Kaggle setup
- [x] Cloudflare Quick Tunnel
- [ ] Additional GPU-specific configurations
- [ ] More example workflows
- [ ] Additional optimization experiments
- [ ] Longer-generation presets

---

## Contributing

Issues, improvements, workflow optimizations, and additional hardware configurations are welcome.

If you successfully run the workflow on another GPU, consider sharing:

- GPU model
- VRAM
- Resolution
- Frame count
- Steps
- Generation time
- Any required changes

This helps build a useful hardware compatibility reference for the community.

---

## License

This repository contains setup scripts, notebook code, and workflow configuration.

The underlying models and third-party components remain subject to their respective licenses and terms.

Check the upstream projects before redistributing model files or other third-party assets.

---

## Keywords

`MiniMax H3` · `MiniMax H3 ComfyUI` · `MiniMax H3 T4` · `MiniMax H3 Colab` · `MiniMax H3 Kaggle` · `ComfyUI` · `text-to-video` · `AI video generation` · `video generation with audio` · `low VRAM` · `NVIDIA T4` · `Qwen3-VL` · `ClipProj` · `Google Colab` · `Kaggle`
