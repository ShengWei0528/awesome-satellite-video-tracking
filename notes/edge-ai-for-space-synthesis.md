# 衛星／無人機邊緣 AI：三篇論文綜合研讀（感知 → 控制 → 決策）

| | |
|---|---|
| **Papers** | [SR-YOLOv5s](sr-yolov5s.md) (Acta Astronautica 2026) · [LeLaR](lelar.md) (IEEE Access 2026) · [GUIDE](guide.md) (AI4Space @ CVPR 2026) |
| **Written** | Aug 2026, as a paper-reading report |
| **Notes by** | Sheng-Wei Lu (呂聖韡) |

## TL;DR (English)

- The three papers sit at three abstraction levels of the same autonomous-satellite system: **perception** (SR-YOLOv5s, CNN on an FPGA), **control** (LeLaR, deep RL on a Cortex-M4), and **supervisory decision-making** (GUIDE, LLM with an evolving playbook). The closer to perception, the harder the hardware limits but the more mature the technology; the closer to decision-making, the more freedom but the farther from flight.
- Five lessons: (1) the binding constraint of edge AI is the power / latency budget, not accuracy; (2) space AI is about Sim2Real and verifiability (e.g. LeLaR's Safety Cage); (3) the abstraction level dictates the model family; (4) on hardware you cannot retrieve, updatability is a core design constraint; (5) read the "honest" paragraphs — self-reported limitations are often the most informative part of a paper.
- Common prerequisite for all three directions: putting models onto resource-constrained edge hardware (quantization, pruning, FPGA / SoC inference).

---

## 一、三十秒版本

| 論文 | 一句話 | 最關鍵的數字 |
|---|---|---|
| **SR-YOLOv5s** | 與其把偵測模型做大，不如在 YOLOv5s 前面接一個只有 16,960 參數的超解析骨幹，再整條丟進 FPGA。 | 極小目標 mAP50 0.2832 → 0.4658；55 ms/幀、總功耗 2.46 W、mAP50/W 0.366 |
| **LeLaR** | 全球第一個真的在軌道上跑起來的強化學習姿態控制器，成功跨過 Sim2Real gap。 | 穩態誤差各軸 < 1°；7/7 次機動達標（PD 只有 3/7）；推論 38 ms、模型 105 kB |
| **GUIDE** | 讓 LLM 當太空船的決策官，靠「演化一本自然語言規則手冊」變強，完全不動模型權重。 | LG6 分數 2.28 × 10⁷ → 5.07 × 10⁴（p < 0.001）；0 次梯度更新 |

**如果只記一件事**：這三篇處在同一個系統的三個抽象層級（感知 → 控制 → 決策）。越靠近感知端，硬體限制越硬、但技術越成熟；越靠近決策端，越自由、但離真的能上衛星越遠。

## 二、三篇對照表

| 面向 | SR-YOLOv5s | LeLaR | GUIDE |
|---|---|---|---|
| AI 扮演的角色 | 感知（看見目標） | 控制（穩住姿態） | 決策（決定要做什麼） |
| 技術家族 | CNN（YOLOv5s + SR backbone） | Deep RL（SkipPPO，PPO 變體） | LLM + in-context learning |
| 執行位置 | 衛星上的 FPGA（PolarFire SoC） | 衛星上的 ADCS（Cortex-M4, 40 MHz） | 地面／模擬環境（KSPDG） |
| 驗證程度 | 硬體實測（真實影像 + 真實板子） | 真實在軌飛行（InnoCube） | 僅模擬驗證 |
| 即時性需求 | 毫秒級（55 ms/幀） | 秒級（1 Hz 迴路，推論 38 ms） | 線上輕量、學習可離線 |
| 模型大小／功耗 | 總平均 2.46 W | 網路檔 105 kB | 未討論（LLM 需雲端級算力） |
| 「學習」發生在哪 | 地面訓練，星上只推論 | 地面訓練 + post-training，星上只推論 | 離線演化 context，權重完全不動 |
| 如何在軌更新 | 上傳新模型指令流（不必重新合成 FPGA） | 上傳新權重檔（105 kB） | 更新 playbook 文字 |
| 最大的痛點 | 功耗預算與小目標 | Sim2Real gap | 延遲、可靠度、會輸給簡單啟發式 |
| 成熟度 | 高 | 中高（已在軌） | 低（研究早期） |

## 三、學習重點

### 重點 1：邊緣 AI 的真正限制是功耗與延遲預算，不是準確率

在地面做深度學習，遇到準確率不夠的直覺反應是「模型再大一點、資料再多一點」；在衛星上這條路直接被封死：

- SR-YOLO 為了每瓦效率選 FPGA 不選 GPU（2.46 W 對上 Orin Nano 約 6.5 W、A100 約 53 W），並刻意讓偵測器維持 YOLOv5s 的輕量架構，把難題丟給只有 16,960 參數的前處理骨幹。它比 GPU 慢（55 ms vs. 16 ms），換來的是 0.366 vs. 0.127 的 mAP50/W。
- LeLaR 的網路只有 3–4 層 × 64 神經元、105 kB，因為它必須跑在 40 MHz 的 Cortex-M4 上。
- GUIDE 之所以要做 teacher–student 分離，本質也是延遲預算問題——會推理的模型進不了控制迴路。

**推論**：「功耗／延遲／記憶體預算」應該是設計的起點，而不是最後才做的最佳化。模型量化、剪枝、FPGA／SoC 推論是必修的基本功。

### 重點 2：太空 AI 的關鍵字是 Sim2Real 與可驗證性

LeLaR 真正花篇幅的不是 RL 演算法，而是熱真空艙量測感測器雜訊、交叉核對慣量矩、反解殘餘磁偶極，以及一整套 Safety Cage。更關鍵的是態度：真實反作用輪出現三種模擬裡沒有的怪行為，他們刻意不去修模擬，就是要證明 agent 能直接吸收這些偏差。

**推論**：在這個領域，「能證明它不會出事」比「它表現多好」更難、也更有價值。把 AI 當黑盒子、用外部遙測獨立監控的 Safety Cage，是 AI 上太空的入場券，而且被 ECSS Machine Learning Handbook 推薦。

### 重點 3：抽象層級決定技術選型

- **感知層**：輸入是高維影像、輸出是確定性的框，要求毫秒級延遲 → CNN + 硬體加速器。
- **控制層**：輸入是低維狀態向量、輸出是連續動作，要求秒級穩定與可證明的安全邊界 → 小型 MLP + RL。
- **決策層**：輸入是情境與目標、輸出是策略意圖，可容忍延遲、需要彈性推理與人類可讀性 → LLM。

**推論**：「要不要用 LLM」不是品味問題，而是看在哪一層。在控制層硬塞 LLM 是錯的（延遲不允許），在決策層硬塞 RL 也很吃力（獎勵函數難定義）。

### 重點 4：在無法回收的硬體上，可更新性是核心約束

SR-YOLO 選 CoreVectorBlox，是因為換模型不需要重新做 FPGA 合成；LeLaR 把網路做成 105 kB 的檔案，可用遙控指令切換；GUIDE 更極端——連權重都不改，只更新文字。三者都在回答同一個問題：**衛星發射之後就拿不回來了，AI 要怎麼進步？** 這是地面 AI 開發不會遇到、但在太空領域會決定整個系統架構的問題。

### 重點 5：要看得懂論文的「誠實之處」

LeLaR 明說 PD 控制器沒調校過所以比較不公平；GUIDE 明說在兩個情境下輸給最簡單的啟發式。這些自我揭露反而是論文品質的指標。讀論文時要養成習慣去找這些段落——它們通常藏在 Results 的最後一段或 Limitations 裡，而且往往是資訊量最大的部分。

## 四、三個層級的成熟度與開放問題

| 面向 | 感知層（CNN 偵測，如 SR-YOLOv5s） | 控制層（深度強化學習，如 LeLaR） | 決策層（大語言模型，如 GUIDE） |
|---|---|---|---|
| 技術成熟度 | 高：baseline、資料集、FPGA 部署流程都有現成可循 | 中高：LeLaR 已證明可在軌運行 | 低：只在模擬中驗證 |
| 主要挑戰 | 目標或平台高速移動時的運動模糊 + 小目標；三篇都沒處理，SR-YOLO 只處理靜態小目標 | 需要高保真模擬環境與完整的衛星參數校正資料（慣量、感測器雜訊、殘餘磁偶極），前期投入重 | 會輸給簡單啟發式、延遲難控、算力需求與衛星硬體不匹配 |
| 可借力的資源 | YOLO 生態、公開遙測資料集、VectorBlox SDK | LeLaR 的在軌資料已開源；Basilisk / KSPDG 模擬環境 | KSPDG 環境開源，但缺可對照的真實系統 |
| 延伸方向 | 運動模糊下的小目標偵測 | 動量管理、多衛星、故障容錯等方向很多 | 空間大，但沒有前人鋪路 |

**三個層級共同的基本功**：把模型放進資源受限的邊緣硬體——量化、剪枝、FPGA／SoC 推論。

**開放問題**：目標或平台高速移動造成的運動模糊，在三篇中都沒有被處理。[DREB-Net](https://arxiv.org/abs/2410.17822)（高機動 UAV 影像的去模糊 + 偵測）是可參考的起點，可補上 SR-YOLO 缺的那一塊。

## 五、名詞速查表

| 名詞 | 說明 |
|---|---|
| ADCS | Attitude Determination and Control System，姿態決定與控制系統 |
| Reaction Wheel (RW) | 反作用輪：改變飛輪轉速、用角動量守恆讓衛星轉向；力矩大、反應快，但角動量會飽和 |
| Magnetorquer (MT) | 磁力矩器：通電線圈與地磁場作用產生力矩；力矩極小，主要用來替 RW 去飽和與消旋 |
| Momentum management | 動量管理／去飽和：用 MT 把 RW 累積的角動量「倒掉」 |
| Quaternion | 四元數：表示三維旋轉，沒有萬向鎖問題，是姿態控制的標準表示法 |
| Steady-state error | 穩態誤差：機動結束、系統穩定後殘留的指向誤差 |
| Sim2Real gap | 模擬與真實之間的落差，RL 落地的頭號難題 |
| Domain randomization | 領域隨機化：訓練時刻意隨機變動環境參數，讓策略學會容忍不確定性 |
| OOD | Out-of-Distribution，分布外；神經網路擅長內插、不擅長外插 |
| PPO / SkipPPO | 常用的 on-policy RL 演算法／LeLaR 對 PPO 的改造，處理快慢致動器混合時的 credit assignment |
| Credit assignment problem | 回報延遲出現時，難以判斷是先前哪個動作造成的 |
| Safety Cage | 用規則與獨立監控模組包住 AI，違規即接管；把 AI 當黑盒子處理 |
| FDIR / FMECA | 故障偵測、隔離與復原／失效模式、影響與嚴重度分析 |
| Reward hacking | agent 鑽獎勵函數漏洞拿高分，但沒真正解決問題 |
| mAP50 | IoU ≥ 0.5 才算命中的平均精度，物件偵測標準指標 |
| Super-resolution (SR) | 超解析：用神經網路從低解析度影像重建高解析度影像 |
| FPGA / SoC | 可現場程式化邏輯陣列，功耗低、可重組；SoC 版本再整合處理器核心（PolarFire 用 RISC-V） |
| SEU | Single Event Upset，單粒子翻轉；flash-based FPGA 比 SRAM-based 有先天抗性 |
| In-context learning | LLM 不更新權重，只靠 prompt 中的示範與記憶改變行為 |
| Playbook（GUIDE） | 一組帶狀態條件的自然語言決策規則，條件成立時注入 prompt |
| UCB1 | 多臂拉霸機演算法，平衡「利用已知好選項」與「探索新選項」 |
| KSPDG | Kerbal Space Program Differential Games，以 KSP 為基礎的非合作太空作業模擬環境 |
| GNC | Guidance, Navigation, and Control，導引、導航與控制 |
