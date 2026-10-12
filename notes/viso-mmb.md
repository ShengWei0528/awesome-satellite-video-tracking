# VISO & MMB — 衛星影片中小而密集移動目標的偵測與追蹤基準

| | |
|---|---|
| **Paper** | Q. Yin, Q. Hu, H. Liu, F. Zhang, Y. Wang, Z. Lin, W. An, Y. Guo, *Detecting and Tracking Small and Dense Moving Objects in Satellite Videos: A Benchmark* |
| **Venue** | IEEE Transactions on Geoscience and Remote Sensing (TGRS), vol. 60, 2022 |
| **Links** | [Paper (DOI)](https://doi.org/10.1109/TGRS.2021.3130436) · [arXiv](https://arxiv.org/abs/2111.12960) · [Dataset](https://github.com/QingyongHu/VISO) (CC BY-NC-SA 4.0) |
| **Read for** | Lab-meeting paper presentation, Sep 2026 — focus on Section IV (the MMB method) |
| **Notes by** | Sheng-Wei Lu (呂聖韡) |

## TL;DR (English)

- **Problem.** Moving-object detection and tracking in satellite video lags far behind ordinary video because (1) there was no well-annotated public benchmark, (2) targets are tiny (often < 50 px) yet a single frame can hold more than a hundred of them, and (3) backgrounds are whole cities with rivers, buildings and changing illumination.
- **Dataset (VISO).** Jilin-1 videos at 10 fps: 47 videos, frames of 12,000 × 5,000 px processed as 1,000 × 1,000 tiles; about 1.64 M horizontal boxes and 3,711 trajectories, with ~90 % of targets smaller than 50 px. Classes: plane, large vehicle, small car, ship, train. One dataset supports detection, single-object tracking (SOT) and multi-object tracking (MOT).
- **Method (MMB, motion-modeling baseline).** Two parallel branches — **AMFD** (accumulative multi-frame differencing) finds candidate moving pixels / ROIs, and **LRMC** (matrix-completion background modeling) extracts complete foreground objects — fused and then cleaned by a **trajectory-based Pipeline Filter (PF)** that removes false alarms and fills in missed detections.
- **Results.** MMB has the best F1 on all 7 test videos, about 10 pp above the runner-up (D&T) on average; mAP 0.83 vs. 0.73. Ablation (precision): AMFD only 0.66, LRMC only 0.67, without PF 0.75, full MMB 0.84.
- **Tracking is still hard.** Best SOT tracker (ECO) reaches only 61 % of frames within 5 px center error; best MOT method (Kalman on MMB detections) gets MOTA 73.6 % but still has 2,000+ ID switches.

---

## 一句話

這篇建立了第一個衛星影片「偵測＋追蹤」的公開基準 VISO，並提出結合幀差與背景相減的 MMB：偵測效能高、誤警少；但現有追蹤器直接搬到衛星影片上效果仍差，追蹤還有很大空間。

## 研究背景與貢獻

作者認為衛星影片移動目標偵測與追蹤進展落後一般影片，原因有三：

1. 缺乏標註完善的公開資料集與基準，方法之間無法公平比較。
2. 解析度低：目標常常不到 50 像素，但一幀裡可能超過一百個。
3. 背景是整座城市：河流、建築、車道都有，光照還一直在變。

對應的貢獻就是：**VISO 資料集**、**MMB 偵測方法**，以及**第一個公開基準**。

## VISO 資料集

- 來源：吉林一號影片衛星，47 支影片、每秒 10 幀；每幀 12,000 × 5,000，切成 1,000 × 1,000 處理。
- 標註：水平框，逐幀確認目標身分；共約 164 萬個框、3,711 條軌跡，約九成目標小於 50 像素。
- 類別：飛機、大車、小車、船、火車。小車在影像裡幾乎只是一個點。
- 同一份資料可同時用於偵測、單目標追蹤、多目標追蹤。

## MMB 方法（第四節，本次報告重點）

問題定義：目標太小、外觀資訊不足 → 關鍵在於**利用時空資訊偵測移動目標，同時壓住背景造成的誤警**。

整體架構：影片進來分成兩路並行，合流後送進 Pipeline Filter。

### 1. AMFD（累積多幀差分）

- 動機：相鄰兩幀之間目標移動很慢，慢速目標不好偵測，邊緣還會留下鬼影；而車輛、飛機有規則的形狀與運動模式，雜訊沒有。
- 做法：每三個相鄰幀一組，兩兩相減得到三張差分圖，再加總平均成一張「累積響應影像」——雜訊被壓下去，真正在動的目標被突顯。
- 接著二值化、形態學運算，最後用先驗過濾：**面積 5–80 像素、長寬比 1–6** 的區塊才算目標。

### 2. LRMC（背景建模）

- 方向與 AMFD 相反：AMFD 找「在動的像素」，LRMC 是「把背景整個抽掉」。
- 衛星影片背景相對靜止又佔畫面絕大部分；把多幀排在一起，背景就是幾乎不變的共同成分，用矩陣補全類方法抽出來後，當前幀減掉背景就是前景。
- 細節：不對整段影片建模，而是分成小群組各自估背景——群組太多增加計算量又沒有效能增益，太少又描述不了變動的背景。論文參數 L = 4，在 10 fps 下每組約 40 幀。
- 註：架構圖上標示為 VFRMC，內文稱 LRMC，指的是同一個模組。

### 3. PF（基於運動軌跡的誤警濾波器）

- 想法：真實目標的軌跡連續且規則，雜訊軌跡零散不規則。
- 兩路響應影像逐元素相乘得到結果影像，送進一條「管線」：長度 5 幀、大小 7 × 7（比目標的 3 × 3 稍大）。
- 從第一幀找出所有候選點，逐幀檢查下一幀 7 × 7 鄰域內有沒有目標，用匈牙利演算法配對；成功就把出現次數加一並更新位置。
- 5 幀跑完後統計：**3 次以上判為真實目標**，其餘視為誤警；出現 3–4 次者，再依運動特性把目標補回沒偵測到的那一幀。

## 實驗結果

### 移動目標偵測

- 比較 13 個方法：2 個前景偵測、6 個背景建模、4 個低秩分解，以及深度學習的 ClusterNet。
- 評估方式：目標只有 2–20 像素，預測框偏一兩像素 IoU 就劇烈變化，所以改成「與真值框有重疊就算命中」。
- MMB 在 7 支影片的 F1 都最高，平均比第二名 D&T 高約 10 個百分點；mAP 0.83 vs. 0.73。
- 消融（Precision）：只用 AMFD 0.66、只用 LRMC 0.67、拿掉 PF 0.75、三者都用 0.84。
- 質性比較：水面光照變化大的影片中，幀差與背景建模方法誤警滿版；對比極低的影片中，低秩方法誤警少但目標也幾乎抓不到；MMB 在兩種場景都能抓到目標且誤警少。

### 目標追蹤

- **單目標追蹤**：比較 11 個追蹤器（6 個相關濾波、5 個孿生網路），只給第一幀的框。最好的 ECO，中心誤差小於 5 像素的幀只有 61 %；MCCT 的重疊成功率 34.7 %。
- **多目標追蹤**：比較 6 個方法，輸入統一用 MMB 的偵測結果。Kalman 的 MOTA 最高（73.6 %），SORT 的 MOTP 最高（28.6 %）；但 CMOT 與 FairMOT 幾乎整段追丟，表現最好的 Kalman 仍有兩千多次 ID 切換。
- 結論：現有方法無法直接泛化到衛星影片；作者建議把運動與歷史軌跡資訊納入追蹤器，或先做超解析度。

## 讀時注意到的細節（原文前後不一致之處）

- **PF 門檻 h**：論文寫「h 大於 H（H = 3）判為真實目標」，下一句又說 h 等於 3 或 4 要補回漏檢，敘述有出入；我採用「3 次（含）以上」的解讀。
- **LRMC 群組長度**：論文寫 L 是建模所需的幀數並設為 4，但公式 N = M / (L × f) 中 f 是 10 Hz 幀率，且架構圖標到 T = 40，因此每個觀測矩陣實際涵蓋約 40 幀。
- **表 IX 的 mostly lost**：正文說多數序列 mostly lost，但表上 Kalman 的 ML 是 0、MT 是 639；實際上是 CMOT 與 FairMOT 幾乎全失，其餘方法的瓶頸在 ID 切換數千次。
- **MMB 數字**：表 VII（0.85 / 0.84 / 0.85）與表 V 的平均（0.84 / 0.85 / 0.84）不一致，兩張表統計方式不同。
- **追蹤器數量**：正文寫「ten representative trackers」，實際評測了 11 個（表 VIII）。
- **DPR**：中心位置誤差小於門檻的幀數比例，本文門檻 α = 5 像素。

## 與研究題目的關聯

- VISO 提供了衛星影片小目標偵測／追蹤的公開基準，後續方法可以在同一份資料上公平比較（例如 OITrack 即以 VISO 評測）。
- MMB 屬於傳統（非學習式）方法，但「累積多幀差分＋背景建模＋軌跡式誤警濾除」這個思路，對於標註稀少、目標極小的衛星場景仍是重要的基線與偽標籤來源。
- 追蹤結果顯示「偵測好 ≠ 追蹤好」：ID 切換是衛星影片 MOT 的主要瓶頸，這也是之後研讀 OITrack 等關聯策略論文的動機。
