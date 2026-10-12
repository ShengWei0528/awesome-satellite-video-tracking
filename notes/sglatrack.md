# SGLATrack — 相似度引導的層自適應 ViT 無人機追蹤器

| | |
|---|---|
| **Paper** | C. Xue, B. Zhong, Q. Liang, Y. Zheng, N. Li, Y. Xue, S. Song, *Similarity-Guided Layer-Adaptive Vision Transformer for UAV Tracking* |
| **Venue** | CVPR 2025, pp. 6730–6740 |
| **Links** | [Paper (CVF)](https://openaccess.thecvf.com/content/CVPR2025/html/Xue_Similarity-Guided_Layer-Adaptive_Vision_Transformer_for_UAV_Tracking_CVPR_2025_paper.html) · [arXiv](https://arxiv.org/abs/2503.06625) · [Code](https://github.com/GXNU-ZhongLab/SGLATrack) (MIT) |
| **Task** | UAV single-object tracking (SOT), not MOT |
| **Read for** | Close reading (2026-09-12) and lab-meeting presentation (2026-09-24) |
| **Notes by** | Sheng-Wei Lu (呂聖韡) |

## TL;DR (English)

- **Observation.** In lightweight one-stream ViT trackers, deep layers become highly similar to each other (feature saturation): adjacent-layer cosine similarity of the search features climbs from ~0.6 in shallow layers to ~0.94–0.965 in layers 10–12, while the "stop-at-layer-i" AUC on UAV123 plateaus after about layer 5. Deep layers learn *repetitive*, not *useless*, representations — so cutting all of them hurts accuracy.
- **Method.** Fix a saturation layer l\* = 6. Disable the 6 layers after it and let a tiny 3-layer MLP (hidden 160) pick **one** of them to keep, trained with an L1 "layer-similarity" loss whose one-hot label is the layer most similar to layer l\* (self-generated, no manual labels). Inference always runs 7 layers + the MLP → fixed latency.
- **Results.** SGLATrack-DeiT\*: 224.7 FPS on a GTX 2080Ti, 74.8 FPS on CPU, 33 FPS on Jetson TX2 (no TensorRT). Five-benchmark average AUC / P = 64.7 / 82.6 (AVTrack 63.7 / 81.9). Layer adaptation costs only 0.2–0.4 AUC while giving about +28 % FPS, −27 % parameters and −30 % FLOPs.
- **My take.** What adapts is the *representation*, not the compute — latency is constant, which is actually attractive for on-board scheduling. Training cost is not reduced (all 12 layers run). The "keep one deep layer" trick is backbone-level, so it could speed up a ViT detector or ReID feature extractor inside a MOT pipeline.

---

## 一句話

輕量 ViT 追蹤器的深層彼此高度相似（特徵飽和）；把第 l\* = 6 層之後的 6 層停用、只用一個 3 層 MLP 選出其中一層保留，推論只跑 7 層：速度 224.7 FPS（2080Ti）／74.8 FPS（CPU）／33 FPS（Jetson TX2，無 TensorRT），AUC 只掉 0.2–0.4。

## 為什麼無人機追蹤要特別處理

1. 空拍視角特殊：目標在畫面裡通常很小，高度一變尺度就變，背景又雜。
2. 機上的電力與算力非常有限：追蹤器一定要即時，最好能在 CPU 或嵌入式板子上跑。UAV 追蹤器評比的從來不只是精度，而是**精度與速度的取捨**。

論文把現有追蹤器分成三類：

| 類型 | 優點 | 缺點 |
|---|---|---|
| DCF（相關濾波） | 頻域運算，很快 | 手工特徵，遮擋、形變時不穩 |
| Siamese 網路 | CNN 深度特徵，穩健許多 | 多建立在早期 CNN 上，精度有天花板 |
| ViT | 把影像切成 patch 做 token 化，套用 Transformer | 完整 ViT 太重，無法直接上無人機 |

本文是在第三類中找輕量化的方法。

## 前人做法與作者的批評

- **Aba-ViTrack（token 剪枝）**：token 數隨輸入變動 → 非結構化存取，FLOPs 降了但 FPS 不一定降。
- **AVTrack（層啟用）**：以「層」為單位避開非結構化存取，但幾乎每一層前都要掛分類器判斷要不要跑——分類器本身有成本且冗餘；輸入複雜度難估、分類器輸出也不一定可靠。
- **本文**：不估複雜度，只在「特徵飽和」的那一點做一次選擇。

## 核心觀察：層冗餘（3.1 節，圖 2）

- 單流框架：模板 Z 與搜尋 S 經 patch embedding 串成一條 token 序列 X = [X_z, X_s]，逐層通過 Transformer；預測頭只吃搜尋那一側：B = H(X_s^l)。
- 對 ViT-tiny / DeiT-tiny 逐層計算：
  - 相鄰層搜尋特徵的餘弦相似度 Cos(X_s^i, X_s^{i−1})；
  - 「在第 i 層停止推論、直接接預測頭」的 UAV123 AUC。
- 我從圖上讀到的數值（DeiT-tiny）：
  - 相似度：第 2–3 層約 0.60–0.62，第 4 層 0.70，第 5–6 層約 0.74，第 7–8 層約 0.83，第 9 層 0.92，第 10–12 層 0.94–0.965。
  - AUC：第 3 層約 0.60 → 第 5 層約 0.65 → 第 9 層約 0.66（第 5 層後幾乎持平）。
- 關鍵：深層學到的是「重複的」而非「沒用的」表徵——直接剪掉會掉精度，所以**留一層代表**。

## 方法（3.2–3.3 節，圖 3）

- **目標（式 4）**：在飽和層 l\* 之後的 K = l − l\* 個候選層中，選出輸出與 X^{l\*} 餘弦相似度最高的第 l\* + k 層。挑「最像」而不是「差最多」，是因為 ViT 越深注意力越聚焦在目標上，與飽和層最像的層＝延續並強化這份已聚焦的注意力。
- **選層模組（式 5）**：ŷ = σ(M(z)) ∈ ℝ^K。M 為 3 層 MLP（隱藏維度 160）；輸入 z 是 X^{l\*} 每個 token 的**第 1 個通道**（長度 N，沿用 AVTrack 的作法以節省算力）；σ 為 sigmoid。K = 6。推論時取 argmax 保留一層，其餘停用，X^{l\*} 直接送進被選中的層。
- **層間相似度損失（式 6–7）**：L_sim = (1/K) Σ |ŷ^k − y^k|（L1 / MAE），y 為「與飽和層最相似那一層」的 one-hot 標籤——標籤由網路中間特徵自行產生，不需人工標註；L_sim 只用來訓練選層模組。
- **總損失（式 8）**：L = L_cls + 2·L_iou + 5·L_L1 + 0.2·L_sim（前三項同 OSTrack：focal + GIoU + L1）。
- **訓練 vs. 推論**：訓練時 12 層全跑（才能算標籤）；推論只跑第 1–6 層 + 被選出的第 6 + k 層 → 延遲固定（7 層 + MLP）。

## 實作設定

- 骨幹：ViT-tiny、DeiT-tiny（distilled，記為 DeiT\*）、EVA-tiny → SGLATrack-ViT / -DeiT\* / -EVA。
- 輸入：模板 128²、搜尋 256²。推論時在分類圖上乘 Hanning 窗。
- 訓練資料：LaSOT、COCO、TrackingNet、GOT-10k（去除 1k 序列）；300 epochs × 60k pairs、batch 32、AdamW；lr 4e-4（預測頭）/ 4e-5（骨幹與選層模組），240 epoch 後 ÷10；4 × A100 80GB。
- 速度平台：i7-9700KF + GTX 2080Ti。

## 主結果（AUC / P）

| 資料集 | SGLATrack-DeiT\* | 對照 |
|---|---|---|
| DTB70 | 65.1 / 84.4 | AVTrack 65.0 / 84.3 |
| UAVDT | 59.9 / 81.9 | AUC 最佳其實是 LightTrack 61.2；P 最佳 AVTrack 82.1 |
| UAV123 | 66.9 / 84.9 | AVTrack 66.8 / 84.8 |
| UAVTrack112 | 67.5 / 82.8 | 領先 AVTrack 2.1 / 2.5（差距最大） |
| UAVTrack112_L | 64.0 / 79.2 | AVTrack 62.7；P 最佳 TCTrack++ 82.0 |
| 五集平均 | 64.7 / 82.6 | AVTrack 63.7 / 81.9 |
| UAV123@10fps | 65.5 / 82.6 | 輸 AVTrack 65.8 / 83.2，但快約 27 FPS |

- 速度：GPU 223.2（ViT）/ 224.7（DeiT\*）/ 236.9（EVA）FPS；CPU 74.2 / 74.8 / 77.2 FPS；AVTrack 197.3、HiT-Small 194.6 FPS。
- 換骨幹一樣有效：SGLATrack-ViT 平均 64.7 / 82.8（P 反而最高）；EVA 63.7 / 81.9 但最快。

## 消融實驗（UAV123 / UAVTrack112 AUC）

- **表 3：LA 套到三種骨幹**。DeiT\*：67.1 / 67.8 → 66.9 / 67.5；FPS 175.5 → 224.7；參數 7.98M → 5.81M（−27 %）；FLOPs 2.39G → 1.68G（−30 %）。三種骨幹 FPS 提升：DeiT\* +28.0 %、EVA +27.8 %、ViT +29.2 % → LA 不綁特定骨幹。
- **表 4：飽和層怎麼決定**。自適應門檻 μ = 0.92 → 67.0 / 67.7，但只有 184.1 FPS；固定 l\* = 6 → 66.9 / 67.5、224.7 FPS（採用）；l\* = 5 → 66.1 / 66.4、239.6 FPS；l\* = 7 → 66.9 / 67.7、211.3 FPS。作者理由：省去逐層算相似度、固定 l\* 會逼模型提早飽和、選層模組可補償。
- **表 5：選層模組怎麼監督**。固定 k = 1（永遠留第 7 層）65.4 / 65.8（232.9 FPS）；最大化相似度（本文）66.9 / 67.5（224.7 FPS）；最小化相似度 65.7 / 64.5；隨機選 66.0 / 66.2。
  - 有監督 > 隨機 > 固定 → 層相似度損失有效。
  - 隨機都比固定好 → 候選的第 7–12 層各自帶著不同資訊，不該永遠只用第 7 層。
  - 最小化在 UAVTrack112 上甚至輸給固定層 → 反向證明「維持與飽和層一致的聚焦」才是對的方向。

## 定性結果

- 圖 4：UAV123 三段序列與 AVTrack、TCTrack++、HiFT 比較，停用 5 層仍穩定。
- 圖 5：模板中心對搜尋區域的交叉注意力。等層數下，「有選層模組（最後為 l\* + k 層）」比「無（最後為 l\* + 1 層）」更聚焦在目標上。
- Jetson TX2 4GB 實機：33 FPS，未使用 TensorRT。

## 我的評析

1. **「自適應」的是表徵而不是算力**：推論永遠 7 層，延遲固定，反而適合機載排程；與 AVTrack「簡單樣本少跑幾層」不同。
2. **訓練成本沒降**：12 層全跑，還多算相似度標籤與 L_sim；省的只有推論。
3. **argmax 不可微**：追蹤損失如何回傳到第 7–12 層、被停用的層在訓練時如何更新，論文沒有說明。合理推測是訓練時 12 層全跑、追蹤損失正常更新所有層，L_sim 只更新 MLP——需看官方程式碼確認。
4. **l\* = 6 是折衷**：l\* = 7 在 UAVTrack112 更好且仍有 211 FPS；部署時可依平台重選。
5. **飽和分析只做在 12 層 tiny ViT**：較大的 ViT 飽和位置可能不同，要重做逐層相似度分析才能決定 l\*。
6. **統計意義**：多數差距只有 0.1–0.3，在 50–123 段序列的基準上統計意義有限；應看平均與 2 個百分點以上的差距。

## 原文中前後不一致之處

- 4.2 節文字說 SGLATrack-DeiT\* 在 UAVDT 的 AUC 最佳，但表 1 中 LightTrack 61.2 > 59.9；正確說法是「四個基準最佳」。
- 表 3 與表 4 的標題文字完全相同（排版錯誤）；表 4 應是「飽和層決定方式」的消融。
- 4.3 節「近 22 % 的速度提升」：以 FPS 計算是 +28 %（175.5 → 224.7），以單幀時間縮減計算是 −21.9 %；兩種算法不同。
- 4.1 節推論段先寫 Hamming 窗、後寫 Hanning 窗，前後不一致（應為 Hanning）。
- 式 5 的輸入寫成 e₁ᵀX^{l\*} := z ∈ ℝ^N；若 X ∈ ℝ^{N×D}，則 e₁ᵀX 應在 ℝ^D，維度寫法與文字描述不一致；依文字（每個 token 的第 1 個通道）解讀為 ℝ^N。

## 延伸討論問題

- L_sim 的參考點若改為「與第 12 層最相似」會如何？
- 選層模組只看每個 token 的第 1 個通道，資訊夠嗎？換成平均池化或 CLS token 會不會更好？（論文未比較）
- 同樣 7 層預算，保留兩層（例如 l\* = 5 + 選 2 層）是否更好？（論文未測）
- 與知識蒸餾比較：直接蒸餾出 7 層模型 vs. SGLA 選出的 7 層？前者延遲同樣固定，但沒有依樣本選層的能力。

## 名詞小抄

| 名詞 | 說明 |
|---|---|
| AUC | 成功率曲線下面積：IoU 門檻從 0 掃到 1，畫出成功幀比例曲線後取面積 |
| P（Precision） | 預測框中心與真值中心距離 ≤ 20 像素的幀數比例 |
| 餘弦相似度 | a·b / (‖a‖‖b‖)，只比方向不比大小；1 = 完全同向、0 = 垂直 |
| 飽和層 l\* | 特徵之後幾乎不再改變的那一層；本文固定為第 6 層 |
| 層自適應 LA | 把 l\* 之後的層停用、只依樣本動態保留一層（Layer Adaptation） |
| 單流架構 | 模板與搜尋 token 串成一條序列，特徵萃取與互動在同一 encoder 完成（如 OSTrack、MixFormer） |
| 非結構化存取 | token 剪枝造成 token 數量不固定、記憶體存取不規則，實際 FPS 未必隨 FLOPs 下降 |
| Hanning 窗 | 推論時乘在分類圖上的位置先驗，抑制離中心太遠的響應 |

## 與研究題目（UAV／衛星俯視偵測＋追蹤）的關聯

SGLATrack 是單目標追蹤，不能直接當 MOT 的關聯模組；但「深層冗餘、保留一層」是**骨幹層級**的技巧，可以用來加速 MOT 管線中的 ViT 偵測器或 ReID 特徵抽取。此外，它「用一般資料訓練、在 UAV 基準上零樣本測試」的設定，也符合高空／衛星資料稀缺的現實。
