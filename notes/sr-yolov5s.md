# SR-YOLOv5s — 超解析 YOLO：衛星機載即時 FPGA 船舶偵測

| | |
|---|---|
| **Paper** | G. M. Capuano, S. Capuozzo, A. G. M. Strollo, N. Petra, *Super-Resolution YOLO Object Detection for Maritime Surveillance with Real-Time FPGA Processing Onboard Spacecraft* |
| **Venue** | Acta Astronautica, vol. 238, pp. 1291–1310, 2026 (online Sep 2025; special issue "SPAICE2024 – AI in and for Space") |
| **Links** | [Paper (DOI)](https://doi.org/10.1016/j.actaastro.2025.09.018) — open access, CC BY 4.0 |
| **Affiliation** | Università degli Studi di Napoli Federico II × TSD-Space (Italy) |
| **Read for** | Paper-reading report on edge AI for LEO satellites / UAVs, Aug 2026 (perception layer) |
| **Notes by** | Sheng-Wei Lu (呂聖韡) |

## TL;DR (English)

- **Idea.** Instead of making the detector bigger, put a tiny single-image super-resolution backbone (16,960 parameters, ~98 % fewer than EDSR) in front of YOLOv5s and train them **jointly**; then deploy the whole pipeline on a flash-based FPGA SoC (Microchip PolarFire).
- **Accuracy.** On low-resolution ship imagery (208 px input), mAP50 0.8544 → 0.9272 overall and **0.2832 → 0.4658 on very small ships (≤ 7 × 7 px)** (+18.26 pp, +64.48 % relative); p < 0.001 over 100 bootstrap resamples.
- **Hardware.** Microchip PolarFire SoC + CoreVectorBlox overlay, INT8: mAP50 0.916, 55 ms/frame (≈ 18 fps), **2.46 W total average** power → best mAP50/W (0.366) among the compared platforms (A100 0.017, Orin Nano INT8 0.127).
- **Caveats.** Still < 0.5 mAP50 on very small ships; static images only (no motion blur); FPGA is slower than an embedded GPU in absolute terms; CoreVectorBlox radiation tolerance not yet tested; commercial (not RT) PolarFire used.

---

## 一句話

與其把偵測模型做大，不如在 YOLOv5s 前面接一個只有 16,960 個參數的超解析骨幹，先把「看不清楚」解決掉；再整條丟進 FPGA，用 2.46 W 換到每秒約 18 幀的機載船舶偵測。

## 問題

- **延遲**：傳統 EO 流程要把影像下傳地面才處理。LEO 衛星每天產生數百 GB，但對單一地面站每次過境只有約 10 分鐘、每天 4–5 次；從「拍到」到「情報可用」常要數小時，而災難應變類需求要的是 3 小時內。
- **小目標**：一艘 25 m × 5 m 的巡邏艇，GSD 0.5 m 時佔 50 × 10 像素，GSD 5 m 時只剩 5 × 1 像素。深度網路逐層壓縮空間資訊，小物體特徵直接被抹除；baseline YOLOv5s 在極小目標上的 mAP50 只有 0.2832。
- **硬體**：衛星功耗與散熱預算極低，放不下 10–30 W 的嵌入式 GPU；COTS 加速器（Myriad 2、Coral TPU）雖省電，但商用矽製程不符合嚴苛任務的抗輻射要求。

## 方法

### SR-Backbone

整合 FSRCNN 與 ESPCN 的元素：56 個 5×5 特徵萃取 → 12 個 1×1 shrinking → 4 層卷積做非線性映射 → 56 個 1×1 expansion → pixel-shuffle 重建 → RGB 回歸；損失函數 MSE。只有 16,960 個參數。

### SR-YOLOv5s

- 把 SR-Backbone 永久整合進 YOLOv5s 前端並**聯合訓練**（不是獨立前處理）。整體參數 7,029,782（baseline 7,012,822，只增加 0.24 %）。
- 唯一的架構妥協：SiLU 換成 ReLU，因為 CoreVectorBlox v1 不支援 SiLU。
- 三階段遷移學習：凍結 SR 訓練偵測頭 → 凍結偵測頭微調 SR → 端到端聯合訓練（各 50 epochs），搭配 5-fold 分層交叉驗證。

### 資料集

| 資料集 | 內容 | 用途 |
|---|---|---|
| ShipRSImageNet | 3,435 張 RGB，約 930 × 930 px | 來源資料 |
| HR-ShipRSImage | 416 × 416 px | 高解析參考基準 |
| LR-ShipRSImage | 高斯濾波 + 下採樣至 208 × 208 px | 主要實驗 |
| LR-CloudyTest | 999 張，Perlin noise 程序化雲層（覆蓋 20–80 %） | 雲遮擋強健性 |
| MASATI-ship / MASATI-water | 含船／完全無船 | 外部泛化／誤報率 |

另外自訂 very small 類別（面積 ≤ 49 px，即 7 × 7 或更小）。

### FPGA 部署

- Microchip PolarFire SoC（MPFS460T，flash-based FPGA + RISC-V）+ CoreVectorBlox V1000 overlay，INT8 推論；約佔 72 % 邏輯元件，時脈 200 MHz。
- 流程：PyTorch → ONNX → VectorBlox SDK 中間表示 → 動態定點 INT8 量化（含校正）→ Network BLOB → 部署；RISC-V 在 Yocto Linux 下跑 C/C++ 控制推論流程。
- 選 FPGA 而非 GPU 的理由：每瓦效率高；overlay 架構換模型不需重新合成 FPGA；RT PolarFire 支援在軌重新程式化；flash-based 對單粒子翻轉有先天抗性。

## 結果

| 模型 | mAP50 (All) | mAP50:95 | mAP50 (Small) | mAP50 (Very small) |
|---|---|---|---|---|
| YOLOv5s | 0.8544 | 0.6610 | 0.7972 | 0.2832 |
| SR-YOLOv5s | 0.9272 | 0.7604 | 0.9272 | 0.4658 |

- 只用 208 px 輸入的 SR-YOLOv5s（0.927）超越在 416 px HR 上跑的 ReLU 版 YOLOv5s（0.920）。
- 泛化：LR-CloudyTest 0.660 vs. 0.610；MASATI-ship 0.751 vs. 0.529；MASATI-water 誤報 32 vs. 42（加 ResNet18 事後分類器可降到 20）。

| 平台 | 精度 | mAP50 | 推論時間 | 平均功耗 | mAP50/W |
|---|---|---|---|---|---|
| A100 | FP32 | 0.927 | 7.30 ms | ~53.04 W | 0.017 |
| Orin Nano (15 W) | INT8 | 0.843 | 16.00 ms | ~6.51 W | 0.127 |
| PolarFire SoC | INT8 | 0.916 | 55.00 ms | ~2.46 W | **0.366** |

- **功耗別引用錯**：2.46 W 是總平均（靜態估計約 1.3 W + 實測動態約 1.16 W）；只寫 1.16 W 的說法是動態部分。
- **量化工具差異明顯**：VectorBlox 量化只讓 mAP50 從 0.927 掉到 0.916，TensorRT INT8 則掉到 0.843。
- **剪枝**：非結構化剪枝 30 % → 0.885，重訓後可回到 0.923；剪 50 % → 0.511。但非結構化剪枝不改變張量維度，Core VBX 仍會算零值權重，實際沒有加速。

## 限制

- 極小目標仍未被解決：0.4658 雖是大幅改進，但一半以上的極小船隻還是抓不到。
- 靜態影像偵測，完全沒有處理**高速相對運動造成的運動模糊**。
- FPGA 比 GPU 慢（55 ms vs. Orin Nano 16 ms）；它贏在每瓦效率，不是絕對速度。功耗預算寬鬆時嵌入式 GPU 反而划算。
- 尚未對 CoreVectorBlox IP 做輻射測試；TMR 會讓吞吐量掉 35–45 %。實驗用的是商用版而非 RT 版 PolarFire。
- SiLU → ReLU 本身有代價（ReLU 版 YOLOv5s 在 HR 上 0.935 → 0.920）；VectorBlox SDK 不支援剪枝，結構化剪枝留作未來工作。

## 對研究方向的意義

- **可直接沿用**：「輕量 SR 前置 + 聯合訓練」的設計模式；「用每瓦準確率而非絕對 fps 評估邊緣硬體」的評估框架；LR/HR 配對資料集的建構方法。
- **要自己補的**：運動模糊。SR 恢復的是「解析度不足」造成的資訊損失，不是「相對運動」造成的模糊——兩者是不同的退化模型；可搭配 DREB-Net 這類去模糊分支一起看。
- **可談的研究缺口**：把「SR 前置」與「去模糊前置」合併成單一輕量前處理骨幹，在同樣的 FPGA 預算下同時對付低解析與運動模糊。
- **工程必修**：ONNX 轉換、INT8 動態定點量化與校正、overlay 加速器的資源估算。
