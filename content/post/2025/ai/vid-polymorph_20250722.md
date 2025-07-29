---
title: Vid-PolyMorph — 多種視覺特效的影片處理
date: 2025-07-21
tags: ["opencv","blog","yolo","ai"]
Description  : "試著不靠 AE 之類的剪輯軟體，用程式＋模組做出各種影片效果"
featured: true
draft: False
---

*A visual and technical showcase of video transformation effects using OpenCV & DL Models.*</br>

![Demo Image](/img/post/vid-polyMorph-demo.gif)

---

#### 目錄 Table of Contents

+ **[為什麼做這個？](#為什麼做這個--why-build-this)**
+ **[安裝與執行方式](#安裝與執行方式)**
+ **[架構設計](#架構設計)**
+ **[功能與技術細節](#功能與技術細節)**
+ **[我學到的](#我學到的)**
+ **[技術資源與參考](#技術資源與參考)**

---

## 為什麼做這個？ | Why build this?
1. 學了些 OpenCV 知識，就來試試不靠 AE 之類的剪輯軟體，用程式＋模組做出各種影片效果。
2. 因為好玩
3. 順便交作業

---

## 安裝與執行方式
### 1. Clone 專案
```bash
git clone https://github.com/AH-DevWorks/vid-polymorph.git
cd vid-polymorph-main
```

### 2. 使用 DevContainer (推薦)
專案內建 `.devcontainer/`，可直接透過 VSCode Remote Container 啟動 GPU 開發環境。

```bash
# [vscode] F1 → Dev Containers: Reopen in Container
# 初次開啟時選擇： Reopen in Container
```
或者也可手動 build Docker：
```bash
docker build -t vidpolymorph .
docker run --gpus all -v ${PWD}:/workspace -w /workspace vidpolymorph python3 vid-poly-morph.py
```

### 3. 或使用 Jupyter Notebook

```bash
# 開啟 notebook
jupyter notebook
# 選擇 `vid-poly-morph.ipynb`
```

---

## 架構設計
影片會依時間序列，套用多種特效，並即時疊加縮圖與提示字樣。整體流程如下：
```text
→ 逐一輸入影片(如video/123.mp4)的影格
   → 動態放大至 1.5x
      → 套用對應特效（時間排程）
         → 疊加縮圖與狀態文字
            → 寫入 video/123_output.mp4
```
時間上的「特效排程」可自訂比例，例如 `[3] 4 in 1 + Flip` 佔整段影片的 10%。

每個效果都對應一個模組函式，如：

```python
(0.10, "[4] 4 in 1 + Flip", lambda canvas, scaled, idx: four_in_one(
    canvas, scaled, effects=[
        None,
        lambda x: cv2.flip(x, 1),
        lambda x: cv2.flip(x, 0),
        lambda x: cv2.flip(x, -1)
    ])),
```

藉此維持主邏輯簡潔，且讓特效模組易於擴充或重組。

---

## 功能與技術細節
影片中的特效依照時間順序依序展示以下處理手法：

| 效果編號 ID  | 特效說明 Effect                                   |
| :--------: | :---------------------------------------------: |
| `[0]`    | 1.0 → 1.5 倍線性放大 |
| `[1]`    | 維持 1.5 倍尺寸 |
| `[2]`    | 旋轉 1080 度（共 3 圈）|
| `[3]`    | 四分割遮罩（圓、矩形、橢圓、三角形）/ 4-in-1 masking |
| `[4]`    | 四分割翻轉效果 / 4-in-1 flip effects     |
| `[5]`    | 16-in-1 彩色映射顯示 / 16-in-1 colormap panel    |
| `[6]`    | 邊緣偵測 (Sobel/Canny/Laplacian) |
| `[7]`    | 高斯差分 / Difference of Gaussian (DoG)           |
| `[8]`    | 形態學處理 / Morphological operations (Dilation, Opening, Gradient) |
| `[9]`    | 特徵點偵測與描述 / Feature detection & descriptors (SIFT, SuperPoint, ORB)   |
| `[10-1]` | RetinaFace 偵測 + 馬賽克 |
| `[10-2]` | 純 RetinaFace 人臉偵測 |
| `[11]`   | YOLOv11 物件偵測  |

---

## 我學到的

* 如何安排、控制特效「時長」與「順序」
* 熟悉程式視覺特效模組化與可擴充性
* 啟用 GPU-ready Docker 容器（TensorFlow 2.15 + OpenCV）

---

## 附錄：技術資源與參考
```markdown
+ OpenCV
+ RetinaFace — 高準確率人臉偵測器  
  → High-accuracy face detector  
  https://github.com/serengil/retinaface
+ SuperPoint — 自監督式特徵點偵測與描述子  
  → Self-supervised keypoint detector & descriptor  
  https://github.com/rpautrat/SuperPoint
+ YOLOv11（官方預訓練版本）— https://docs.ultralytics.com/tasks/detect/
+ tqdm / NumPy / matplotlib 等輔助工具
```

{{< alert_box role="success" title="YOLO 模型" content="假如對於自己訓練 YOLO 模型有興趣，可以看我的另一篇文章：https://ah-devworks.github.io/post/2025/ai/yolo-devcontainer_20250729/" >}}

---

> 若有興趣深入本專案內各項處理邏輯，</br>
> 可至 [GitHub Repo](https://github.com/AH-DevWorks/vid-polymorph) 檢視 `utils/` 中各個模組檔案與主程式流程。

---

> Another Demo video on YouTube: </br>
[![Watch the video](https://i.ytimg.com/vi/FRSJIysQV74/hqdefault.jpg)](https://www.youtube.com/watch?v=FRSJIysQV74)</br>
> Source video: *容疑者Xの献身 (Suspect X, 2008)*</br>