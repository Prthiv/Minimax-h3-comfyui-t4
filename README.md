# MiniMax H3 — ComfyUI on NVIDIA T4

Run **MiniMax H3 text-to-video with audio** in ComfyUI on a single NVIDIA T4 GPU (~14–15 GB VRAM).

**Works with Google Colab and Kaggle.**

[Open in Colab](https://colab.research.google.com/drive/1IzTmlOrm36stLhIVFBvqlys5O2ou9rMc?usp=sharing)  
[Open in Kaggle](https://www.kaggle.com/code/parthivsm/minmax-h3-comfyui)

---

## How to Run

### 1. Open the Notebook

Open the **Colab** or **Kaggle** notebook above.

### 2. Select a GPU

Use:

- NVIDIA Tesla T4
- ~14–15 GB VRAM

### 3. Run the Notebook

Run the cells from **top to bottom**.

The notebook will:

- Install ComfyUI
- Download the required MiniMax H3 models
- Install the required custom nodes
- Start ComfyUI
- Create a public URL using Cloudflare

Open the generated `trycloudflare.com` URL.

### 4. Load the Workflow

The workflow is included in:

```text
workflows/MiniMax_H3_T4.json
```

In ComfyUI:

**Load → Load Workflow**

or simply drag the JSON file into the ComfyUI window.

---

## Generation Settings

### Width & Height

The **width and height control the output resolution**.

Example:

```text
Width: 384
Height: 224
```

This is a good starting point for a T4.

Higher resolution gives better detail, but requires more VRAM and takes longer.

---

## Video Length

MiniMax H3 uses the frame format:

```text
17n + 5
```

You can increase the number of frames to generate longer videos.

| Frames | Approx. Video Length @ 24 FPS | Approx. Time |
|---:|---:|---:|
| 22 | 0.92 sec | ~0.92 sec |
| 39 | 1.63 sec | ~1.63 sec |
| 56 | 2.33 sec | ~2.33 sec |
| 73 | 3.04 sec | ~3.04 sec |
| 90 | 3.75 sec | ~3.75 sec |
| 107 | 4.46 sec | ~4.46 sec |
| 124 | 5.17 sec | ~5.17 sec |

As the **frame count increases**, generation takes longer.

For a T4, start with **22 frames** and increase gradually.

---

## Recommended Starting Settings

```text
GPU:       NVIDIA Tesla T4
Width:     384
Height:    224
Frames:    22
Steps:     6
FPS:       24
```

Once it works, increase the **frames** or **resolution** depending on the result you want.

---

## Hardware

Tested on:

```text
GPU:       NVIDIA Tesla T4
VRAM:      14.56 GB
PyTorch:   2.11.0+cu130
CUDA:      13.0
ComfyUI:   0.38.0
```

The workflow uses ComfyUI's low-VRAM setup to make MiniMax H3 usable on a single T4.

---

## Models

The notebook automatically downloads the required models.

Approximately **45 GB** of model files are required.

No model weights are included in this repository.

---

## Repository

```text
MiniMax-H3-ComfyUI-T4/
├── MiniMax_H3_ComfyUI_T4.ipynb
├── workflows/
│   └── MiniMax_H3_T4.json
├── README.md
└── .gitignore
```

---

## Credits

- [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- MiniMax H3
- Qwen3-VL
- ComfyUI MiniMax H3 LowVRAM
- ComfyUI Spectrum MiniMax H3

## License

This repository contains setup code and workflow files only.  
Model weights are downloaded from their respective upstream sources.
