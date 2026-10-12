# Awesome Satellite Video Tracking

**A curated reading list on object detection and tracking in satellite video and remote-sensing imagery — plus on-board / FPGA processing and AI for spacecraft operations — with my own paper reading notes.**

**衛星影片與遙測影像之目標偵測、追蹤論文清單（含星上／FPGA 處理與太空載具 AI），附個人論文研讀筆記。**

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
![Topic](https://img.shields.io/badge/topic-satellite%20video%20%7C%20remote%20sensing-0b5394)
![Languages](https://img.shields.io/badge/lang-English%20%7C%20%E4%B8%AD%E6%96%87-blue)
![Links checked](https://img.shields.io/badge/links%20checked-2026--10-brightgreen)

---

## Overview · 簡介

Satellite video (e.g. from Jilin-1) lets us watch vehicles, ships and aircraft over a whole city, but the targets are only a few pixels wide, densely packed, and embedded in cluttered, changing backgrounds — so detectors and trackers designed for ordinary video transfer poorly. This list collects key surveys, benchmarks and methods in this niche. It starts from the papers I read for my research on deep-learning detection and tracking for UAV and satellite imagery, plus the "edge AI in space" papers I read on running such models on board (entries with a reading note are marked **[Notes]**), and extends them with well-known related work.

- Every entry links to the **official** paper page (DOI, publisher, CVF / PMLR, or arXiv) and to **official** code or data when it exists. Links were checked in October 2026.
- Entries marked **[Notes]** have a reading note in [`notes/`](notes/) — written in my own words (English TL;DR + Traditional Chinese notes).

衛星影片（例如吉林一號）能讓我們觀察整座城市的車輛、船隻與飛機，但目標往往只有幾個像素、分布密集、背景複雜且持續變化，為一般影片設計的偵測與追蹤方法很難直接套用。本清單整理此領域的重要綜述、資料集與方法：以我在碩士研究（UAV／衛星影像的深度學習偵測與追蹤）中研讀過的論文，以及「太空邊緣 AI」相關論文為起點（有研讀筆記者標有 **[Notes]**），並補充此領域常見的代表性工作。

- 每一筆都連到**官方**論文頁（DOI、出版社、CVF／PMLR 或 arXiv），有官方程式碼或資料時一併附上；連結於 2026 年 10 月逐一確認。
- 標有 **[Notes]** 的條目在 [`notes/`](notes/) 中有我的研讀筆記（英文 TL;DR ＋ 中文筆記）。

<a id="layout"></a>
## Repository layout · 檔案結構

```text
awesome-satellite-video-tracking/
├── README.md          # the curated list (this page) / 論文清單
├── notes/             # reading notes: English TL;DR + 中文筆記 (index: notes/README.md)
│   ├── README.md
│   ├── viso-mmb.md
│   ├── sglatrack.md
│   ├── sr-yolov5s.md
│   ├── lelar.md
│   ├── guide.md
│   └── edge-ai-for-space-synthesis.md
├── CONTRIBUTING.md    # entry format and PR checklist / 貢獻指南
└── LICENSE            # CC BY 4.0
```

## Contents · 目錄

- [Repository layout · 檔案結構](#layout)
- [Reading notes · 研讀筆記](#reading-notes)
- [Surveys · 綜述](#surveys)
- [Datasets & benchmarks · 資料集與基準](#datasets)
- [Moving / small object detection in satellite video · 衛星影片移動與小目標偵測](#detection)
- [Satellite video single-object tracking · 衛星影片單目標追蹤](#sot)
- [Satellite video multi-object tracking · 衛星影片多目標追蹤](#mot)
- [On-board / FPGA processing · 星上與 FPGA 處理](#onboard)
- [UAV-view tracking & detection (related) · 無人機視角追蹤與偵測（相關）](#uav)
- [Tracking-by-detection baselines · 常用多目標追蹤基線](#baselines)
- [AI for spacecraft operations · 太空載具 AI 運作](#spacecraft)
- [Contributing · 貢獻方式](#contributing)
- [Context · 背景](#context)
- [License · 授權](#license)

---

<a id="reading-notes"></a>
## Reading notes · 研讀筆記

| Paper | Venue | Note |
|---|---|---|
| VISO & MMB — small and dense moving objects in satellite videos | IEEE TGRS 2022 | [viso-mmb.md](notes/viso-mmb.md) |
| SGLATrack — similarity-guided layer-adaptive ViT for UAV tracking | CVPR 2025 | [sglatrack.md](notes/sglatrack.md) |
| SR-YOLOv5s — super-resolution YOLO on an on-board FPGA | Acta Astronautica 2026 | [sr-yolov5s.md](notes/sr-yolov5s.md) |
| LeLaR — first in-orbit AI attitude controller | IEEE Access 2026 | [lelar.md](notes/lelar.md) |
| GUIDE — in-context decision evolution for LLM spacecraft operations | AI4Space @ CVPR 2026 | [guide.md](notes/guide.md) |
| Synthesis: perception → control → decision for edge AI in space | — | [edge-ai-for-space-synthesis.md](notes/edge-ai-for-space-synthesis.md) |

<a id="surveys"></a>
## Surveys · 綜述

- **Satellite video SOT review + OOTB** — *Satellite Video Single Object Tracking: A Systematic Review and an Oriented Object Tracking Benchmark.* Y. Chen et al. *ISPRS Journal of Photogrammetry and Remote Sensing*, vol. 210, 2024. [[Paper]](https://doi.org/10.1016/j.isprsjprs.2024.03.013) [[Dataset & toolkit]](https://github.com/YZCU/OOTB)  
  Systematic review of satellite-video single-object tracking, released together with the oriented-box OOTB benchmark. ／ 衛星影片單目標追蹤的系統性回顧，並發布以旋轉框標註的 OOTB 基準。
- **Recent Advances in Intelligent Processing of Satellite Video** — *Recent Advances in Intelligent Processing of Satellite Video: Challenges, Methods, and Applications.* S. Li et al. *IEEE JSTARS*, vol. 16, pp. 6776–6798, 2023. [[Paper]](https://doi.org/10.1109/JSTARS.2023.3296451)  
  Reviews a decade of satellite-video processing — detection, tracking, scene classification and segmentation — with the public datasets and benchmarks for each task. ／ 回顧近十年衛星影片智慧處理（偵測、追蹤、場景分類、分割），並整理各任務的公開資料集與評測基準。
- **DL-based tracking in satellite videos + SV248S** — *Deep Learning-Based Object Tracking in Satellite Videos: A Comprehensive Survey With a New Dataset.* Y. Li et al. *IEEE Geoscience and Remote Sensing Magazine*, vol. 10, no. 4, pp. 181–212, 2022. [[Paper]](https://doi.org/10.1109/MGRS.2022.3198643) [[Dataset]](https://github.com/xdai-dlgvv/SV248S)  
  Survey of deep-learning trackers for satellite video; introduces the SV248S single-object tracking dataset. ／ 衛星影片深度學習追蹤綜述，並提出 SV248S 單目標追蹤資料集。

<a id="datasets"></a>
## Datasets & benchmarks · 資料集與基準

**Satellite video · 衛星影片**

| Dataset | Tasks | Paper | Data | Notes |
|---|---|---|---|---|
| **SDM-Car-SU** | Vehicle detection & MOT in *unstabilized* satellite video | DeTracker, IEEE TGRS 2026 [[DOI]](https://doi.org/10.1109/TGRS.2026.3694908) | [GitHub](https://github.com/alex-chenjiajun/DeTracker) (CC BY-NC 4.0, academic research only) | Four subsets (U1, U2, U3, Merge) ／ 未穩像衛星影片車輛資料集 |
| **OOTB** | SOT with oriented boxes | ISPRS J. 2024 [[DOI]](https://doi.org/10.1016/j.isprsjprs.2024.03.013) | [GitHub](https://github.com/YZCU/OOTB) | 110 sequences, 29,890 frames; car / ship / plane / train ／ 旋轉框單目標追蹤 |
| **SAT-MTB** | Detection, tracking, segmentation | IEEE TGRS 2023 [[DOI]](https://doi.org/10.1109/TGRS.2023.3278075) | [Official page](http://www.csu.cas.cn/gb/kybm/sjlyzx/gcxx_sjj/sjj_wxxl/202211/t20221121_6551405.html) | 249 videos, 50,000+ frames ／ 多任務衛星影片基準 |
| **VISO** | Moving-object detection, SOT, MOT | IEEE TGRS 2022 [[DOI]](https://doi.org/10.1109/TGRS.2021.3130436) [[arXiv]](https://arxiv.org/abs/2111.12960) | [GitHub](https://github.com/QingyongHu/VISO) (CC BY-NC-SA 4.0) | Jilin-1; tiny, dense targets ／ 吉林一號小而密集目標 **[[Notes]](notes/viso-mmb.md)** |
| **SatSOT** | SOT | IEEE TGRS 2022 [[DOI]](https://doi.org/10.1109/TGRS.2022.3140809) | [Official page](http://www.csu.cas.cn/gb/kybm/sjlyzx/gcxx_sjj/sjj_wxxl/202106/t20210607_6080256.html) | 105 sequences, 27,664 frames ／ 衛星影片單目標追蹤 |
| **SV248S** | SOT | IEEE GRSM 2022 [[DOI]](https://doi.org/10.1109/MGRS.2022.3198643) | [GitHub](https://github.com/xdai-dlgvv/SV248S) (non-commercial, on request) | 248 targets from six Jilin-1 videos ／ 吉林一號 248 個目標 |

**UAV view (related) · 無人機視角（相關）**

| Dataset | Tasks | Paper | Data |
|---|---|---|---|
| **VisDrone** | Image / video detection, SOT, MOT, crowd counting | *Detection and Tracking Meet Drones Challenge*, IEEE TPAMI 2022 [[DOI]](https://doi.org/10.1109/TPAMI.2021.3119563) | [GitHub](https://github.com/VisDrone/VisDrone-Dataset) |
| **UAVDT** | Detection, SOT, MOT (vehicles) | *The Unmanned Aerial Vehicle Benchmark: Object Detection and Tracking*, ECCV 2018 [[DOI]](https://doi.org/10.1007/978-3-030-01249-6_23) | [Project page](https://sites.google.com/view/grli-uavdt) (research use only) |
| **UAV123** | SOT | *A Benchmark and Simulator for UAV Tracking*, ECCV 2016 [[DOI]](https://doi.org/10.1007/978-3-319-46448-0_27) | [KAUST IVUL page](https://ivul.kaust.edu.sa/benchmark-and-simulator-uav-tracking-dataset) |

> Dataset licenses differ (several are non-commercial). Check each dataset's terms before use. ／ 各資料集授權不同（部分限非商業用途），使用前請確認。

<a id="detection"></a>
## Moving / small object detection in satellite video · 衛星影片移動與小目標偵測

- **HiEUM** — *Highly Efficient and Unsupervised Framework for Moving Object Detection in Satellite Videos.* C. Xiao et al. *IEEE TPAMI*, vol. 46, no. 12, 2024. [[Paper]](https://doi.org/10.1109/TPAMI.2024.3409824) [[arXiv]](https://arxiv.org/abs/2411.15895) [[Code]](https://github.com/ChaoXiao12/Moving-object-detection-in-satellite-videos-HiEUM)  
  Unsupervised training via self-evolving pseudo-labels plus a sparse-convolution anchor-free detector that only processes candidate regions; the paper reports 98.8 fps on 1024 × 1024 frames. ／ 以偽標籤自我演化做無監督訓練，搭配只處理候選區域的稀疏卷積 anchor-free 偵測器，論文報告 1024 × 1024 影像可達 98.8 fps。
- **MMB (in VISO)** — motion-modeling baseline combining accumulative multi-frame differencing, matrix-completion background modeling and a trajectory-based false-alarm filter. See VISO in [Datasets](#datasets). **[[Notes]](notes/viso-mmb.md)**  
  結合累積多幀差分、背景建模與軌跡式誤警濾除的傳統基線方法，詳見 VISO。
- **DSFNet** — *DSFNet: Dynamic and Static Fusion Network for Moving Object Detection in Satellite Videos.* C. Xiao et al. *IEEE GRSL*, vol. 19, 2022. [[Paper]](https://doi.org/10.1109/LGRS.2021.3124222) [[Code]](https://github.com/ChaoXiao12/Moving-object-detection-DSFNet)  
  Fuses a dynamic (temporal) stream and a static (appearance) stream to detect tiny moving objects. ／ 融合動態（時序）與靜態（外觀）兩路特徵偵測微小移動目標。
- **ClusterNet** — *ClusterNet: Detecting Small Objects in Large Scenes by Exploiting Spatio-Temporal Information.* R. LaLonde, D. Zhang, M. Shah. *CVPR* 2018. [[Paper]](https://openaccess.thecvf.com/content_cvpr_2018/papers/LaLonde_ClusterNet_Detecting_Small_CVPR_2018_paper.pdf) [[arXiv]](https://arxiv.org/abs/1704.02694)  
  Two-stage spatio-temporal CNN (ClusterNet proposes regions, FoveaNet localizes centroids) for tiny objects in wide-area motion imagery; the deep-learning method compared in the VISO detection benchmark. ／ 針對廣域動態影像（WAMI）微小目標的兩階段時空 CNN，VISO 偵測基準中比較的深度學習方法。

<a id="sot"></a>
## Satellite video single-object tracking · 衛星影片單目標追蹤

- **HTTrack** — *HTTrack: Learning to Perceive Targets via Historical Trajectories in Satellite Video Tracking.* J. Wang et al. *AAAI* 2026. [[Paper]](https://doi.org/10.1609/aaai.v40i12.37950)  
  Uses a target's historical trajectory to help perceive it in later frames. ／ 利用目標歷史軌跡輔助後續幀的目標感知。
- **SatSAM2** — *SatSAM2: Motion-Constrained Video Object Tracking in Satellite Imagery using Promptable SAM2 and Kalman Priors.* R. Fan et al. arXiv preprint, 2025. [[arXiv]](https://arxiv.org/abs/2511.18264)  
  Zero-shot satellite tracker built on SAM2 with Kalman-filter motion priors; evaluated on OOTB, SatSOT and SAT-MTB. ／ 以 SAM2 為基礎、加入 Kalman 運動先驗的零樣本衛星追蹤器。
- **HRSiam** — *HRSiam: High-Resolution Siamese Network, Towards Space-Borne Satellite Video Tracking.* J. Shao et al. *IEEE TIP*, vol. 30, pp. 3056–3068, 2021. [[Paper]](https://doi.org/10.1109/TIP.2020.3045634)  
  High-resolution Siamese tracker designed for satellite video. ／ 針對衛星影片設計的高解析孿生網路追蹤器。
- **Multiframe optical-flow tracker** — *Object Tracking in Satellite Videos Based on a Multiframe Optical Flow Tracker.* B. Du, S. Cai, C. Wu. *IEEE JSTARS*, vol. 12, no. 8, pp. 3043–3055, 2019. [[Paper]](https://doi.org/10.1109/JSTARS.2019.2917703)  
  Multi-frame optical-flow tracker for satellite video. ／ 以多幀光流追蹤衛星影片中的目標。
- **Velocity-feature correlation filter** — *Tracking Objects From Satellite Videos: A Velocity Feature Based Correlation Filter.* J. Shao, B. Du, C. Wu, L. Zhang. *IEEE TGRS*, vol. 57, no. 10, pp. 7860–7871, 2019. [[Paper]](https://doi.org/10.1109/TGRS.2019.2916953)  
  Correlation-filter tracker that exploits velocity features of satellite-video targets. ／ 在相關濾波追蹤器中利用目標的速度特徵。

<a id="mot"></a>
## Satellite video multi-object tracking · 衛星影片多目標追蹤

Small, dense targets make data association hard — on the VISO benchmark even the best baseline tracker still had 2,000+ ID switches. ／ 目標小且密集使關聯困難——在 VISO 基準上，表現最好的基線追蹤器仍有兩千多次 ID 切換。

- **DeTracker** — *DeTracker: Motion-Decoupled Vehicle Detection and Tracking in Unstabilized Satellite Videos.* J. Chen et al. *IEEE TGRS*, vol. 64, 2026. [[Paper]](https://doi.org/10.1109/TGRS.2026.3694908) [[arXiv]](https://arxiv.org/abs/2601.09240) [[Dataset / code]](https://github.com/alex-chenjiajun/DeTracker)  
  Decouples platform jitter (global motion) from vehicle motion and fuses multi-frame features for tiny vehicles; releases the SDM-Car-SU dataset (core network code announced as forthcoming). ／ 將衛星平台抖動（全域運動）與車輛自身運動解耦，並以跨幀特徵強化微小車輛；釋出 SDM-Car-SU 資料集（核心程式碼預告後續公開）。
- **FMA-Net** — *FMA-Net: Flow-Driven Motion-Aware Network for Multiobject Tracking in Satellite Videos.* W. Lu et al. *IEEE TGRS*, vol. 63, 2025. [[Paper]](https://doi.org/10.1109/TGRS.2025.3624205)  
  Optical-flow-driven, motion-aware network for satellite-video MOT. ／ 以光流驅動的運動感知多目標追蹤網路。
- **OITrack** — *OITrack: Multi-Object Tracking for Small Targets in Satellite Video via Online Trajectory Completion and Iterative Expansion over Union.* W. Lu et al. *Remote Sensing*, 17(12), 2042, 2025 (open access). [[Paper]](https://doi.org/10.3390/rs17122042)  
  Trajectory-completion module + adaptive Kalman filter + iterative expansion-IoU association for tiny targets; evaluated on VISO. ／ 軌跡補全模組、自適應 Kalman 濾波與迭代擴張 IoU 關聯，針對極小目標，於 VISO 上評測。
- **MP2Net** — *MP²Net: Mask Propagation and Motion Prediction Network for Multiobject Tracking in Satellite Videos.* M. Zhao et al. *IEEE TGRS*, vol. 62, 2024. [[Paper]](https://doi.org/10.1109/TGRS.2024.3385406) [[Code]](https://github.com/DonDominic/MP2Net)  
  Joint detection-and-tracking with mask propagation and motion prediction for tiny targets. ／ 以遮罩傳播與運動預測做聯合偵測追蹤。
- **TGraM** — *Multi-Object Tracking in Satellite Videos With Graph-Based Multitask Modeling.* Q. He et al. *IEEE TGRS*, vol. 60, 2022. [[Paper]](https://doi.org/10.1109/TGRS.2022.3152250)  
  Graph-based multitask modeling for multi-object tracking in satellite video. ／ 以圖結構多任務建模處理衛星影片多目標追蹤。
- **Needles in a Haystack** — *Needles in a Haystack: Tracking City-Scale Moving Vehicles From Continuously Moving Satellite.* W. Ao et al. *IEEE TIP*, vol. 29, pp. 1944–1957, 2020. [[Paper]](https://doi.org/10.1109/TIP.2019.2944097)  
  Tracks city-scale moving vehicles from a continuously moving satellite platform. ／ 從持續移動的衛星平台追蹤城市尺度的移動車輛。

<a id="onboard"></a>
## On-board / FPGA processing · 星上與 FPGA 處理

- **SR-YOLOv5s** — *Super-Resolution YOLO Object Detection for Maritime Surveillance with Real-Time FPGA Processing Onboard Spacecraft.* G. M. Capuano et al. *Acta Astronautica*, vol. 238, pp. 1291–1310, 2026 (open access). [[Paper]](https://doi.org/10.1016/j.actaastro.2025.09.018) **[[Notes]](notes/sr-yolov5s.md)**  
  A 16,960-parameter super-resolution front-end trained jointly with YOLOv5s raises very-small-ship mAP50 from 0.2832 to 0.4658 (full-precision model); the INT8 build on a PolarFire SoC keeps overall mAP50 at 0.916 at ~55 ms/frame and 2.46 W. ／ 在 YOLOv5s 前接 16,960 參數的超解析骨幹並聯合訓練，使極小船隻 mAP50 由 0.2832 提升到 0.4658（全精度模型）；部署到 PolarFire SoC 的 INT8 版本整體 mAP50 為 0.916，約 55 ms/幀、2.46 W。
- **On-satellite real-time MOT** — *On-Satellite Implementation of Real-Time Multi-Object Moving Vehicle Tracking with Complex Moving Backgrounds.* J. Yu et al. *Remote Sensing*, 17(3), 418, 2025 (open access). [[Paper]](https://doi.org/10.3390/rs17030418)  
  Cancels background motion by feature matching and detects small vehicles by neighborhood pixel differencing, with the whole pipeline implemented on an FPGA designed for on-satellite processing; real-time at 1024 × 1024 px @ 47 fps. ／ 以特徵匹配抵消背景運動、以鄰域像素差分偵測小型車輛，整條流程實作於為星上處理設計的 FPGA，1024 × 1024 可達 47 fps 即時多車輛追蹤。
- **Φ-Sat-1** — *The Φ-Sat-1 Mission: The First On-Board Deep Neural Network Demonstrator for Satellite Earth Observation.* G. Giuffrida et al. *IEEE TGRS*, vol. 60, 2022. [[Paper]](https://doi.org/10.1109/TGRS.2021.3125567)  
  First in-orbit demonstrator of a deep neural network for Earth-observation data on board. ／ 第一個在軌執行深度神經網路處理地球觀測資料的示範任務。
- **CloudScout** — *CloudScout: A Deep Neural Network for On-Board Cloud Detection on Hyperspectral Images.* G. Giuffrida et al. *Remote Sensing*, 12(14), 2205, 2020 (open access). [[Paper]](https://doi.org/10.3390/rs12142205)  
  On-board cloud detection network for hyperspectral images. ／ 星上高光譜影像雲層偵測網路。

<a id="uav"></a>
## UAV-view tracking & detection (related) · 無人機視角追蹤與偵測（相關）

Aerial views share the small-target and limited-compute problems of satellite video. ／ 空拍視角同樣面臨小目標與算力受限的問題。

- **SGLATrack** — *Similarity-Guided Layer-Adaptive Vision Transformer for UAV Tracking.* C. Xue et al. *CVPR* 2025. [[Paper]](https://openaccess.thecvf.com/content/CVPR2025/html/Xue_Similarity-Guided_Layer-Adaptive_Vision_Transformer_for_UAV_Tracking_CVPR_2025_paper.html) [[arXiv]](https://arxiv.org/abs/2503.06625) [[Code]](https://github.com/GXNU-ZhongLab/SGLATrack) **[[Notes]](notes/sglatrack.md)**  
  Keeps only one of the redundant deep ViT layers, chosen by a tiny MLP; ~225 FPS on GPU and 33 FPS on Jetson TX2 with only 0.2–0.4 AUC loss. ／ 只保留一層冗餘深層、由小型 MLP 選擇；GPU 約 225 FPS、Jetson TX2 33 FPS，AUC 僅降 0.2–0.4。
- **DREB-Net** — *DREB-Net: Dual-stream Restoration Embedding Blur-feature Fusion Network for High-mobility UAV Object Detection.* Q. Li et al. arXiv preprint, 2024. [[arXiv]](https://arxiv.org/abs/2410.17822) [[Code]](https://github.com/EEIC-Lab/DREB-Net)  
  Detection under motion blur from high-mobility UAVs, with a restoration branch fused into the detector. ／ 高機動 UAV 運動模糊下的物件偵測，將影像復原分支融合進偵測器。
- **AVTrack** — *Learning Adaptive and View-Invariant Vision Transformer for Real-Time UAV Tracking.* Y. Li et al. *ICML* 2024 (PMLR 235). [[Paper]](https://proceedings.mlr.press/v235/li24ax.html) [[Code]](https://github.com/wuyou3474/AVTrack)  
  Adaptive layer activation and view-invariant features for real-time UAV tracking; the main baseline SGLATrack improves on. ／ 自適應層啟用與視角不變特徵的即時 UAV 追蹤器，也是 SGLATrack 的主要對手。
- **BDTrack** — *Learning Motion Blur Robust Vision Transformers for Real-Time UAV Tracking.* Y. Wu et al. arXiv preprint, 2024. [[arXiv]](https://arxiv.org/abs/2407.05383)  
  Motion-blur-robust ViT tracker for real-time UAV tracking (code repository announced, not yet released). ／ 對運動模糊具強健性的即時 UAV 追蹤器（程式碼尚未公開）。
- **From Blurry to Brilliant Detection** — *From Blurry to Brilliant Detection: YOLO-Based Aerial Object Detection with Super Resolution.* R. A. Nihal et al. arXiv preprint, 2024. [[arXiv]](https://arxiv.org/abs/2401.14661)  
  Combines super-resolution with YOLO for small objects in aerial imagery. ／ 將超解析與 YOLO 結合，偵測空拍影像中的小目標。

<a id="baselines"></a>
## Tracking-by-detection baselines · 常用多目標追蹤基線

General-purpose MOT methods that satellite-video papers commonly compare against. ／ 衛星影片 MOT 論文常用的通用比較基線。

| Method | Paper | Venue | Code |
|---|---|---|---|
| **OC-SORT** | *Observation-Centric SORT: Rethinking SORT for Robust Multi-Object Tracking* [[arXiv]](https://arxiv.org/abs/2203.14360) | CVPR 2023 | [GitHub](https://github.com/noahcao/OC_SORT) |
| **ByteTrack** | *ByteTrack: Multi-Object Tracking by Associating Every Detection Box* [[arXiv]](https://arxiv.org/abs/2110.06864) | ECCV 2022 | [GitHub](https://github.com/FoundationVision/ByteTrack) |
| **BoT-SORT** | *BoT-SORT: Robust Associations Multi-Pedestrian Tracking* [[arXiv]](https://arxiv.org/abs/2206.14651) | arXiv 2022 | [GitHub](https://github.com/NirAharon/BoT-SORT) |
| **FairMOT** | *FairMOT: On the Fairness of Detection and Re-Identification in Multiple Object Tracking* [[DOI]](https://doi.org/10.1007/s11263-021-01513-4) [[arXiv]](https://arxiv.org/abs/2004.01888) | IJCV 2021 | [GitHub](https://github.com/ifzhang/FairMOT) |
| **JDE** | *Towards Real-Time Multi-Object Tracking* [[arXiv]](https://arxiv.org/abs/1909.12605) | ECCV 2020 | [GitHub](https://github.com/Zhongdao/Towards-Realtime-MOT) |
| **DeepSORT** | *Simple Online and Realtime Tracking with a Deep Association Metric* [[DOI]](https://doi.org/10.1109/ICIP.2017.8296962) [[arXiv]](https://arxiv.org/abs/1703.07402) | ICIP 2017 | [GitHub](https://github.com/nwojke/deep_sort) |
| **SORT** | *Simple Online and Realtime Tracking* [[DOI]](https://doi.org/10.1109/ICIP.2016.7533003) [[arXiv]](https://arxiv.org/abs/1602.00763) | ICIP 2016 | [GitHub](https://github.com/abewley/sort) |

<a id="spacecraft"></a>
## AI for spacecraft operations · 太空載具 AI 運作

- **LeLaR** — *LeLaR: The First In-Orbit Demonstration of an AI-Based Satellite Attitude Controller.* K. Djebko et al. *IEEE Access*, vol. 14, pp. 49348–49367, 2026. [[Paper]](https://doi.org/10.1109/ACCESS.2026.3678816) [[arXiv]](https://arxiv.org/abs/2512.19576) [[In-orbit data]](https://github.com/kdjebko/lelar-in-orbit-data) **[[Notes]](notes/lelar.md)**  
  Deep-RL attitude controller flown on the InnoCube 3U CubeSat; < 1° steady-state error per axis, with a black-box Safety Cage monitor. ／ 在 InnoCube 3U 立方衛星上實際飛行的深度強化學習姿態控制器，各軸穩態誤差 < 1°，並以 Safety Cage 獨立監控。
- **GUIDE** — *GUIDE: Guided Updates for In-context Decision Evolution in LLM-Driven Spacecraft Operations.* A. Carrasco et al. AI4Space Workshop @ CVPR 2026. [[arXiv]](https://arxiv.org/abs/2603.27306) **[[Notes]](notes/guide.md)**  
  Evolves a state-conditioned natural-language playbook offline instead of updating weights; evaluated on KSPDG intercept scenarios. ／ 不更新權重、改為離線演化帶狀態條件的自然語言 playbook，於 KSPDG 攔截情境評測。
- **LLMs as autonomous spacecraft operators in KSP** — *Large Language Models as Autonomous Spacecraft Operators in Kerbal Space Program.* A. Carrasco, V. Rodriguez-Fernandez, R. Linares. *Advances in Space Research*, vol. 76, no. 6, pp. 3480–3497, 2025. [[Paper]](https://doi.org/10.1016/j.asr.2025.06.034) [[arXiv]](https://arxiv.org/abs/2505.19896) [[Code]](https://github.com/ARCLab-MIT/kspdg)  
  Few-shot prompting and fine-tuned LLM agents that operate spacecraft in KSPDG. ／ 以 few-shot prompting 與微調 LLM 代理人在 KSPDG 中操作太空載具。
- **Language Models are Spacecraft Operators** — V. Rodriguez-Fernandez et al. arXiv preprint, 2024. [[arXiv]](https://arxiv.org/abs/2404.00413) [[Code]](https://github.com/ARCLab-MIT/kspdg)  
  Early study of LLM agents as operators in the KSPDG challenge. ／ 早期以 LLM 代理人擔任 KSPDG 太空載具操作員的研究。
- **LLMSat** — *LLMSat: A Large Language Model-Based Goal-Oriented Agent for Autonomous Space Exploration.* D. Maranto. arXiv preprint (B.A.Sc. thesis), 2024. [[arXiv]](https://arxiv.org/abs/2405.01392)  
  LLM-based goal-oriented agent for autonomous space exploration. ／ 以 LLM 為核心、目標導向的自主太空探索代理人。
- **SpaceGym / KSPDG** — *SpaceGym: Discrete and Differential Games in Non-Cooperative Space Operations.* R. E. Allen et al. *IEEE Aerospace Conference* 2023. [[Paper]](https://doi.org/10.1109/AERO55745.2023.10115968) [[Code]](https://github.com/mit-ll/spacegym-kspdg) (MIT)  
  Non-cooperative satellite-operations challenge problems in the Kerbal Space Program engine — the environment used by GUIDE. ／ 以 Kerbal Space Program 引擎實作的非合作衛星作業挑戰環境，即 GUIDE 使用的環境。

---

<a id="contributing"></a>
## Contributing · 貢獻方式

Suggestions are welcome — please open an issue or a pull request. See [CONTRIBUTING.md](CONTRIBUTING.md) for the entry format. In short:

1. **Official links only** — DOI / publisher / CVF / PMLR / arXiv for papers; the authors' own repository for code and data. No mirrors or paywalled PDF uploads.
2. **Verify before adding** — title, authors, venue and year must match the official page; mark preprints as such.
3. **Keep descriptions short and factual** — one English sentence and one Chinese sentence; every number must be traceable to the paper (abstract, tables or text) or the official repository.
4. **One main entry per paper**, newest first within each section; the Datasets tables may cross-reference papers listed elsewhere.

歡迎透過 issue 或 pull request 建議新條目，格式請見 [CONTRIBUTING.md](CONTRIBUTING.md)。原則：只放官方連結、加入前先核對題目／作者／出處／年份、說明保持一句英文＋一句中文且數字須能在論文（摘要、表格或內文）或官方 repo 中查到、每篇只有一個主要條目（資料集表格可交叉引用其他分類的論文），各分類內依年份由新到舊排列。

<a id="context"></a>
## Context · 背景

Maintained by **Sheng-Wei Lu (呂聖韡)** — M.S. student, Graduate Institute of Space Systems Engineering, National Taipei University of Technology (國立臺北科技大學 太空系統工程研究所); B.S. in Electrical Engineering, Yuan Ze University (元智大學). My research focuses on deep-learning detection, classification and tracking for UAV and satellite imagery.

The notes in [`notes/`](notes/) were written in Aug–Sep 2026 for paper-reading reports and lab-meeting presentations. They are personal summaries and critiques — not translations — and include no figures from the papers.

本清單由呂聖韡維護（國立臺北科技大學 太空系統工程研究所 碩士生，元智大學電機工程學系學士），研究方向為無人機與衛星影像的深度學習目標偵測、分類與追蹤。[`notes/`](notes/) 內的筆記寫於 2026 年 8–9 月撰寫論文研讀報告與準備實驗室報告期間，為個人整理與評析，非翻譯，亦不含論文原圖。

**Credits · 致謝** — All papers, code and datasets belong to their respective authors; this repository only links to them. Dataset licenses (e.g. VISO: CC BY-NC-SA 4.0; SDM-Car-SU: CC BY-NC 4.0, academic research only; SV248S: non-commercial use) are set by their owners. ／ 所有論文、程式碼與資料集的權利屬於原作者，本 repo 僅提供連結；資料集授權以各擁有者公告為準。

<a id="license"></a>
## License · 授權

Copyright (c) 2026 Sheng-Wei Lu. The curated list (README.md), the contributing guide and the reading notes in this repository are licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/); the full legal code is in [LICENSE](LICENSE). This license covers only the original content of this repository. The papers, code and datasets linked from it belong to their respective authors and are subject to their own licenses.

本 repo 的清單、貢獻指南與研讀筆記以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 授權釋出（完整條文見 [LICENSE](LICENSE)）。此授權僅涵蓋本 repo 的原創內容；所連結的論文、程式碼與資料集屬於各自作者，並依其各自的授權條款。
