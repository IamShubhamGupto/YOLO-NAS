# YOLO-NAS
Train and Inference your custom **YOLO-NAS** model by **Single Command Line** <br>
<a href="https://colab.research.google.com/drive/1VGX8FVCviclmxUu1v-DHV0lK0u74suIX?usp=sharing"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"></a>
[<img src="https://img.shields.io/badge/Docker-Image-blue.svg?logo=docker">](<https://hub.docker.com/repository/docker/naseemap47/yolo-nas>)

# About YOLO-NAS
**A Next-Generation, Object Detection Foundational Model generated by Deci’s Neural Architecture Search Technology**<br>
Deci is thrilled to announce the release of a new object detection model, **YOLO-NAS** - a game-changer in the world of object detection, providing superior real-time object detection capabilities and production-ready performance. Deci's mission is to provide AI teams with tools to remove development barriers and attain efficient inference performance more quickly.<br>

In terms of pure numbers, **YOLO-NAS** is **~0.5 mAP point more accurate** and **10-20% faster** than equivalent variants of **YOLOv8 and YOLOv7**.

| Model            | mAP   | Latency (ms) |
|------------------|-------|--------------|
| YOLO-NAS S       | 47.5  | 3.21         |
| YOLO-NAS M       | 51.55 | 5.85         |
| YOLO-NAS L       | 52.22 | 7.87         |
| YOLO-NAS S INT-8 | 47.03 | 2.36         |
| YOLO-NAS M INT-8 | 51.0  | 3.78         |
| YOLO-NAS L INT-8 | 52.1  | 4.78         |

mAP numbers in table reported for **COCO 2017** Val dataset and latency benchmarked for **640x640** images on **Nvidia T4 GPU**.

**YOLO-NAS's** architecture employs quantization-aware blocks and selective quantization for optimized performance. When converted to its INT8 quantized version, **YOLO-NAS** experiences a smaller precision drop (**0.51, 0.65, and 0.45 points of mAP for S, M, and L variants**) compared to **other models that lose 1-2 mAP points** during quantization. These techniques culminate in innovative architecture with superior object detection capabilities and top-notch performance.

## Build your first custom YOLO-NAS model
- Clone the Repository
- Install dependencies
- Prepare dataset
- Train your custom **YOLO-NAS** model
- Inference your custom **YOLO-NAS** model on Camera/Video/RTSP

# 😎 Custom YOLO-NAS Model
### Clone this Repository
```
git clone https://github.com/naseemap47/YOLO-NAS.git
cd YOLO-NAS
```
### Install dependencies
**Recommended**:
```
conda create -n yolo-nas python=3.9 -y
conda activate yolo-nas
pip install torch==1.11.0+cu113 torchvision==0.12.0+cu113 torchaudio==0.11.0 --extra-index-url https://download.pytorch.org/whl/cu113 
# For Quantization Aware Training
pip install pytorch-quantization==2.1.2 --extra-index-url https://pypi.ngc.nvidia.com
pip install super-gradients==3.2.0
```
#### OR
```
pip3 install -r requirements.txt
```
### 🎒 Prepare Dataset
Your custom dataset should be in **COCO JSON** data format.<br>
To convert **YOLO (.txt) / PASCAL VOC (.XML)** format to **COCO JSON**.<br>
Using JSON Converter https://github.com/naseemap47/autoAnnoter#10-yolo_to_jsonpy <br>
**COCO Data Format**:
```
├── Dataset
|   ├── annotations
│   │   ├── train.json
│   │   ├── valid.json
│   │   ├── test.json
│   ├── train
│   │   ├── 1.jpg
│   │   ├── abc.png
|   |   ├── ....
│   ├── val
│   │   ├── 2.jpg
│   │   ├── fram.png
|   |   ├── ....
│   ├── test
│   │   ├── img23.jpeg
│   │   ├── 50.jpg
|   |   ├── ....
```

To training custom model using your custom data.
You need to create [data.yaml](https://github.com/naseemap47/YOLO-NAS/blob/master/data.yaml)
Example:
```
Dir: 'Data'
images:
  test: test
  train: train
  val: valid
labels:
  test: annotations/test.json
  train: annotations/train.json
  val: annotations/valid.json
```

## 🤖 Train
You can train your **YOLO-NAS** model with **Single Command Line**

<details>
  <summary>Args</summary>
  
  `-i`, `--data`: path to data.yaml <br>
  `-n`, `--name`: Checkpoint dir name <br>
  `-b`, `--batch`: Training batch size <br>
  `-e`, `--epoch`: number of training epochs.<br>
  `-s`, `--size`: Input image size <br>
  `-j`, `--worker`: Training number of workers <br>
  `-m`, `--model`: Model type (Choices: `yolo_nas_s`, `yolo_nas_m`, `yolo_nas_l`) <br>
  `-w`, `--weight`: path to pre-trained model weight (`ckpt_best.pth`) (default: `coco` weight) <br>
  `--gpus`: Train on multiple gpus <br>
  `--cpu`: Train on CPU <br>
  `--resume`: To resume model training <br>
  
  **Other Training Parameters:**<br>
  `--warmup_mode`: Warmup Mode, eg: Linear Epoch Step <br>
  `--warmup_initial_lr`: Warmup Initial LR <br>
  `--lr_warmup_epochs`: LR Warmup Epochs <br>
  `--initial_lr`: Inital LR <br>
  `--lr_mode`: LR Mode, eg: cosine <br>
  `--cosine_final_lr_ratio`: Cosine Final LR Ratio <br>
  `--optimizer`: Optimizer, eg: Adam <br>
  `--weight_decay`: Weight Decay
  
</details>

**Example:**
```
python3 train.py --data /dir/dataset/data.yaml --batch 6 --epoch 100 --model yolo_nas_m --size 640

# From Pre-trained weight
python3 train.py --data /dir/dataset/data.yaml --batch 6 --epoch 100 --model yolo_nas_m --size 640 \
                 --weight runs/train2/ckpt_latest.pth
```
### If your training ends in 65th epoch (total 100 epochs), now you can start from 65th epoch and complete your 100 epochs training.
**Example:**
```
python3 train.py --data /dir/dataset/data.yaml --batch 6 --epoch 100 --model yolo_nas_m --size 640 \
                 --weight runs/train2/ckpt_latest.pth --resume
```

### Quantization Aware Training

<details>
  <summary>Args</summary>
  
  `-i`, `--data`: path to data.yaml <br>
  `-b`, `--batch`: Training batch size <br>
  `-e`, `--epoch`: number of training epochs.<br>
  `-s`, `--size`: Input image size <br>
  `-j`, `--worker`: Training number of workers <br>
  `-m`, `--model`: Model type (Choices: `yolo_nas_s`, `yolo_nas_m`, `yolo_nas_l`) <br>
  `-w`, `--weight`: path to pre-trained model weight (`ckpt_best.pth`) <br>
  `--gpus`: Train on multiple gpus <br>
  `--cpu`: Train on CPU <br>
  
  **Other Training Parameters:**<br>
  `--warmup_mode`: Warmup Mode, eg: Linear Epoch Step <br>
  `--warmup_initial_lr`: Warmup Initial LR <br>
  `--lr_warmup_epochs`: LR Warmup Epochs <br>
  `--initial_lr`: Inital LR <br>
  `--lr_mode`: LR Mode, eg: cosine <br>
  `--cosine_final_lr_ratio`: Cosine Final LR Ratio <br>
  `--optimizer`: Optimizer, eg: Adam <br>
  `--weight_decay`: Weight Decay
  
</details>

**Example:**
```
python3 qat.py --data /dir/dataset/data.yaml --weight runs/train2/ckpt_best.pth --batch 6 --epoch 100 --model yolo_nas_m --size 640
```

## 📺 Inference
You can Inference your **YOLO-NAS** model with **Single Command Line**
#### Support
- Image
- Video
- Camera
- RTSP

<details>
  <summary>Args</summary>
  
  `-n`, `--num`: Number of classes the model trained on <br>
  `-m`, `--model`: Model type (choices: `yolo_nas_s`, `yolo_nas_m`, `yolo_nas_l`) <br>
  `-w`, `--weight`: path to trained model weight <br>
  `-s`, `--source`: video path/cam-id/RTSP <br>
  `-c`, `--conf`: model prediction confidence (0<conf<1) <br>
  `--save`: to save video <br>
  `--hide`: hide video window

</details>

**Example:**
```
python3 inference.py --num 3 --model yolo_nas_m --weight /runs/train4/ckpt_best.pth --source /test/video.mp4 --conf 0.66           # video
                                                                                    --source /test/sample.jpg --conf 0.5 --save    # Image save
                                                                                    --source /test/video.mp4 --conf 0.75 --hide    # to save and hide video window
                                                                                    --source 0 --conf 0.45                         # Camera
                                                                                    --source 'rtsp://link' --conf 0.25 --save      # save RTSP video stream

```
## 📺 Inference Batching
#### Support
- Video
- Camera
- RTSP

<details>
  <summary>Args</summary>
  
  `-n`, `--num`: Number of classes the model trained on <br>
  `-m`, `--model`: Model type (choices: `yolo_nas_s`, `yolo_nas_m`, `yolo_nas_l`) <br>
  `-w`, `--weight`: path to trained model weight <br>
  `-s`, `--source`: paths to videos/cam-ids/RTSPs <br>
  `--full`: to enable full screen

</details>

```
python3 batch.py --num 3 --model yolo_nas_m --weight /runs/train4/ckpt_best.pth --source '/test/video.mp4' '/test/video23.mp4'      # videos
                                                                                --source 0 2 --conf 0.45 --full                     # web-Cameras with full screen
                                                                                --source 'rtsp://link' 'rtsp://link3' --conf 0.25   # RTSPs video stream
```
