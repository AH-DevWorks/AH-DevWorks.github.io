---
title: YOLOv12/v11 模型訓練與推論
date: 2025-07-29
tags: ["yolo", "object detection", "cli", "docker"]
Description: "使用 Python CLI 與 DevContainer 建立可重複訓練與推論的 YOLOv12/v11 工程環境"
featured: true
draft: false
---

*A clean and reproducible pipeline for YOLO training, validation, testing and prediction using CLI and VSCode DevContainers.*

---

#### 目錄 Table of Contents
- [動機與用途 | Why this setup?](#動機與用途--why-this-setup)
- [執行方式 | How to Run](#執行方式--how-to-run)
  - [1. Clone 專案](#1-clone-專案)
  - [2-1. 使用 VSCode DevContainer (推薦)](#2-1-使用-vscode-devcontainer-推薦)
  - [2-2. 開啟後直接使用 `.py` 執行](#2-2-開啟後直接使用-py-執行)
- [CLI 模組設計 | CLI Script Design](#cli-模組設計--cli-script-design)
- [常見調整點 | Customizable Elements](#常見調整點--customizable-elements)
- [心得與備忘 | Notes \& Learnings](#心得與備忘--notes--learnings)

---

## 動機與用途 | Why this setup?

1. 原本是轉職班小組專題的模型訓練核心程式，經過不少 Trial and Error，到中期才比較完整。整理、分享出來，當作一個歷程紀錄
2. 程式本身提供一套 YOLO（v11、v12為主）的物件偵測流程（Train-Valid-Test）
3. 以 notebook 為主要使用介面，支援互動式開發與視覺化
4. 另提供 Python CLI 腳本，用於簡化批次執行或伺服器自動化
5. 使用 Docker 容器封裝整體環境，確保部署時對跨平台／系統的一致性
6. 對應訓練、驗證、測試、推論流程皆已封裝為獨立模組，可彈性組合使用

---

## 執行方式 | How to Run

### 1. Clone 專案
```bash
git clone https://github.com/AH-DevWorks/yolo-devcontainer.git
cd yolo-devcontainer
``` 

+ **預訓練權重(.pt檔)**可到 [Ultralytics 網站](https://www.ultralytics.com/) 下載
  + [v12](https://docs.ultralytics.com/models/yolo12/#detection-performance-coco-val2017)
  + [v11](https://docs.ultralytics.com/models/yolo11/#performance-metrics)
+ p.s. 權重有分大小（n, s, m, l, x），建議衡量本機性能做挑選
  + 個人使用 32GB RAM | RTX 4070(12G)，頂多到**l**，x 會直接 OOM 爆記憶體

### 2-1. 使用 VSCode DevContainer (推薦)
```bash
# 開啟 VSCode 後選擇
F1 → Dev Containers: Reopen in Container
```

### 2-2. 開啟後直接使用 `.py` 執行
```bash
python train.py --model models/yolo12n.pt --data data.yaml
python valid.py --model runs/train/weights/best.pt
python test.py  --model runs/train/weights/best.pt
python predict.py --model runs/train/weights/best.pt --source dataset/images/test
```

---

## CLI 模組設計 | CLI Script Design
所有 CLI script 使用統一的 `argparse` 介面（定義於 `_common.py`），並在各執行檔中加上必要擴充參數。
例如 `train.py` 支援以下主要參數：
```bash
--model         初始模型權重路徑 (e.g. yolov8n.pt)
--data          訓練資料 yaml 設定檔
--epochs        訓練輪數
--cos_lr        是否使用 cosine lr decay
--multi_scale   是否啟用多尺度訓練
--optimizer     選擇 Optimizer (e.g. SGD, AdamW)
```

---

## 常見調整點 | Customizable Elements

* **資料集路徑**：預設為 `./dataset/`
  * Dataset 除了自己準備、標記以外，也可到 cityscapes、Roboflow 等網站先下載已標記好的來嘗試訓練
  * 關於如何準備＆標記資料集，可能改天會再發一篇簡單的心得文章
  * Ultralytics提供的預訓練權重（如yolov11m.pt、yolov12l.pt）就是用COCO的來訓練，縱使不做遷移學習，也可以有基本幾種類別辨識。如以下影片的最後5秒就是直接拿yolo11來偵測影片，能抓到「person」、「umbrella」、「bicycle」等類別
[![Watch the video](https://i.ytimg.com/vi/FRSJIysQV74/hqdefault.jpg)](https://youtu.be/FRSJIysQV74?t=96)</br>
    * 關於這影片可參考我另一篇 **[OpenCV文章](https://ah-devworks.github.io/post/2025/ai/vid-polymorph_20250722/)**
  * 文末另有我小組專案最後的成果（藥品辨識模型 with YOLOv12）示意圖供參
* **config 檔案**：`data.yaml` 與模型 `*.yaml` 可自己隨資料集替換
* **超參數調整**：可直接修改 `train.py` 呼叫區或透過命令列參數覆寫
* **Notebook 配合**：與 notebook 訓練結果 (如 best.pt) 可相互參考，不衝突

---

## 心得與備忘 | Notes & Learnings
* 使用 DevContainer + Dockerfile，能最小化環境問題，並順利支援 CUDA GPU 訓練
* [！]但整個專案過程裡曾經偶爾出現幾次意外的Error如「'AAttn' object has no attribute 'qkv'...」、「Can't get attribute 'A2C2f' on <module 'ultralytics.nn.modules.block'...>」---> 爬文說只要更新ultralytics套件即可（須注意numpy版本應<2才不衝突）；目前Dockerfile等有做預防措施，但不保證一定不會出現
* CLI 方式可輕鬆整合自動化腳本，部署至伺服器或嵌入設備
* Ultralytics 的 YOLO API 本身支援 `.train()`, `.val()`, `.predict()` 等方法
* 實測YOLOv12 Fine-tune 藥丸辨識（50種左右）每一Class張數約100張左右（使用Copy-Paste Method），Augmentations兩倍，總training set張數約11K；訓練150epoch、batch=8、開啟cos_lr跟multi_scl後成效佳。Inference 結果如下圖
  * 關於 Copy-Paste Method，改天可能會再寫一篇簡介

![Pill-result Image](/img/post/yolo-pills-res.jpg)

---

> GitHub Repo: [yolo-devcontainer](https://github.com/AH-DevWorks/yolo-devcontainer)

> References: [Ultralytics](https://www.ultralytics.com/)