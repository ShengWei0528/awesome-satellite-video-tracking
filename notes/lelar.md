# LeLaR — 全球首次在軌驗證的 AI 衛星姿態控制器

| | |
|---|---|
| **Paper** | K. Djebko, T. Baumann, E. Dilger, F. Puppe, S. Montenegro, *LeLaR: The First In-Orbit Demonstration of an AI-Based Satellite Attitude Controller* |
| **Venue** | IEEE Access, vol. 14, pp. 49348–49367, 2026 (preprint arXiv:2512.19576; notes written from v5) |
| **Links** | [Paper (DOI)](https://doi.org/10.1109/ACCESS.2026.3678816) · [arXiv](https://arxiv.org/abs/2512.19576) · [In-orbit telemetry data](https://github.com/kdjebko/lelar-in-orbit-data) |
| **Affiliation** | JMU Würzburg × TU Berlin · InnoCube 3U CubeSat |
| **Read for** | Paper-reading report on edge AI for LEO satellites / UAVs, Aug 2026 (control layer) |
| **Notes by** | Sheng-Wei Lu (呂聖韡) |

## TL;DR (English)

- **What.** A deep-RL attitude controller that actually flew: on 2025-10-30 it performed the first in-orbit AI attitude maneuver on the InnoCube 3U CubeSat, reaching < 1° error on all three axes in 114 s.
- **How.** Split policy network (reaction-wheel and magnetorquer sub-networks), trained with *SkipPPO* to handle the two-orders-of-magnitude time-scale gap between actuators; heavy calibration (thermal-vacuum sensor-noise model, cross-checked inertia, residual magnetic dipole estimated from telemetry with PyTorch autograd); a black-box **Safety Cage** monitor recommended by the ECSS ML Handbook; bare-metal MLP inference (~38 ms, ~105 kB network) on a 40 MHz Cortex-M4.
- **Results.** Steady-state error < 1° on every axis; 7/7 repeated maneuvers succeeded vs. 3/7 for the on-board PD controller (6/6 vs. 2/6 on diverse maneuvers); the Safety Cage never triggered. Three real reaction-wheel quirks absent from the simulator were absorbed without retraining.
- **Caveats (stated by the authors).** The PD controller was never tuned after launch; attitude comes from gyro integration only (relative performance); momentum management was not tested in orbit.

---

## 一句話

全球第一個真的在軌道上跑起來的強化學習姿態控制器。真正的貢獻不是 RL 演算法，而是把模擬校準到能匹配真實衛星、外掛一層可證明安全的監控機制，然後讓 agent 直接吸收模擬中根本沒有的硬體怪癖。

## 問題

- 古典 PD 控制器設計耗時，且對模型不確定性與操作邊界條件的變化敏感。
- Deep RL 能在模擬中自行學出控制策略，但真正的難關是 **Sim2Real gap**——神經網路擅長內插、不擅長外插，模擬與真實的偏差會讓 agent 進入分布外（OOD）而失效。
- 在此之前，AI 姿態控制只存在於模擬與實驗室測試台；ESA 在 OPS-SAT 上的嘗試，完整閉迴路 AI 控制最終仍限於模擬與地面測試。

## 方法

### 平台：InnoCube

3U CubeSat，4.2 kg，2025-01-14 發射至 520 km 太陽同步軌道。ADCS 運算單元為 ARM Cortex-M4（40 MHz、1024 kB Flash、256 kB RAM），三軸反作用輪（RW）+ 三軸磁力矩器（MT），控制迴路 1 Hz。對照組為發射後未調校的 PD 控制器。

### 網路與訓練

- **觀測空間 39 維**：姿態誤差四元數、角速率誤差、上次 RW 動作、RW 轉速、磁場向量等；關鍵設計是**給兩個連續時間點的狀態**，讓 agent 能從差分自行推算系統動力學。
- **Split policy network**：RW 子網路（policy 3 × 64、value 2 × 64）與 MT 子網路（皆 4 × 64），SiLU。
- **SkipPPO**：RW 與 MT 的時間尺度差兩個數量級，造成嚴重的 credit assignment 問題。SkipPPO 把 MT 動作凍結 num_skip 步、累積回報後回填給最初的 MT 動作，等效縮短 MT 的 episode 長度。
- **兩階段**：先訓練通用 base-agent，再 post-training 成 flight-agent，以適應部署後才收緊的安全約束。

### 校正（論文花最多力氣的地方）

- **感測器雜訊模型**：在熱真空艙量測陀螺儀與磁力計，模型 = bias（每 episode 取樣）+ 白雜訊（每步取樣），標準差再乘 1.2 倍做 domain randomization；刻意保留未補償的溫度漂移。
- **慣量矩**：CAD + 遙測估計 + 解析計算三方交叉核對（與 CAD 最大偏差約 2.5 %），訓練時每軸每 episode 隨機變動 ±15 %。
- **殘餘磁偶極矩**：以 Euler 旋轉動力學建最佳化問題，用 PyTorch autograd + AdamW 從遙測資料反解磁偶極矩與陀螺 bias。

### Safety Cage

- ECSS Machine Learning Handbook 推薦的架構：以 FMECA 分析關鍵失效模式，實作 (1) 軟體定義的致動器硬性上限（clipping），(2) 與控制迴路平行、獨立執行的監控模組——違規即停用控制器進入 idle。
- **關鍵洞察**：Safety Cage 把系統當黑盒子、只用外部可觀測遙測，因此同樣適用於古典控制器，本質上是傳統 FDIR 的延伸。

### 星上實作

bare-metal MLP 推論引擎，完整前向傳遞約 38 ms（1 Hz 迴路綽綽有餘）；網路檔約 105 kB，可用遙控指令切換不同網路；設計為 PD 控制器的 drop-in replacement。

## 結果

- **里程碑**：2025-10-30 完成全球首次在軌 AI 姿態控制機動，114 秒達成三軸誤差 < 1°。
- **三種模擬中沒有的硬體怪癖**：(1) RW 在低轉速區量測轉速突然跳動（實測 185 rpm）；(2) 零點附近下達力矩指令後數秒毫無反應再突然暴衝；(3) RW 首次啟動後最長約 20 秒的隨機死區。作者**刻意不修改模擬**，證明 agent 能 out-of-the-box 吸收這些偏差。
- **與 PD 正面對決**：

| 實驗 | LeLaR flight-agent | PD 控制器 |
|---|---|---|
| 重複機動 | 7 / 7 次達成三軸 < 1° 穩態 | 3 / 7 次 |
| 多樣化機動 | 6 / 6 次 | 2 / 6 次 |
| 穩態誤差 | 各軸一致 < 1° | Roll 軸最大曾達 2.58° |
| RW 指令軌跡 | 較不平滑 | 較平滑 |
| Safety Cage 介入 | 0 次 | — |

## 限制（作者自己承認）

- **比較不公平**：PD 控制器發射後未經任何調校——這是「AI 打敗未調校的古典控制器」，不是「打敗最佳化過的古典控制器」。
- **姿態只由陀螺積分得到**（無 star tracker）：評估的是相對控制性能，數值含量測雜訊與可能殘留的 bias，應以定性方式解讀。
- **動量管理尚未在軌驗證**：實驗受限於 15 分鐘時窗，MT 子網路的動量管理只在模擬中驗證過。
- 單一衛星、單一構型；任務只是 inertial pointing，沒有處理對地追蹤等更複雜的姿態任務。

## 對研究方向的意義

- **路是通的**：RL 姿態控制器不只能上軌，還能在硬體行為偏離模擬時維持控制——技術風險已有人扛過一次。
- **真正的工作量在校正，不在演算法**：高保真模擬環境、熱真空艙量測、慣量矩三方核對、殘餘磁偶極反解，這些校正資料不一定容易取得，是前期投入最重的部分。
- **Safety Cage 可獨立複用**：與 AI 無關、把系統當黑盒子，可先當作獨立的貢獻，而且有 ECSS 標準背書。
- **算力門檻很低**：105 kB 網路、38 ms 推論、跑在 40 MHz Cortex-M4 上，不需要 FPGA 或 GPU。
- **可延伸方向**：動量管理在軌驗證、多衛星構型的 base-agent 集合、在軌 commissioning 階段的自主微調。
