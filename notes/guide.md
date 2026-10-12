# GUIDE — 把 LLM 當成太空載具的決策官

| | |
|---|---|
| **Paper** | A. Carrasco, M. Storey-Matsutani, V. Rodriguez-Fernandez, R. Linares, *GUIDE: Guided Updates for In-context Decision Evolution in LLM-Driven Spacecraft Operations* |
| **Venue** | AI4Space Workshop @ CVPR 2026 (arXiv:2603.27306; notes written from v3) |
| **Links** | [arXiv](https://arxiv.org/abs/2603.27306) |
| **Affiliation** | MIT × Universidad Politécnica de Madrid |
| **Read for** | Paper-reading report on edge AI for LEO satellites / UAVs, Aug 2026 (decision layer) |
| **Notes by** | Sheng-Wei Lu (呂聖韡) |

## TL;DR (English)

- **Idea.** Keep the LLM weights frozen and instead *evolve a natural-language playbook*: state-conditioned rules that are injected into the prompt at run time. A lightweight acting model controls the spacecraft online; a frontier reasoning model reflects on past episodes offline and edits the playbook (ADD / UPDATE / REMOVE), with UCB1 choosing among playbook versions.
- **Setting.** Lady–Bandit–Guard orbital-intercept scenarios in KSPDG (Kerbal Space Program Differential Games); guard behaviours are hidden from the LLM.
- **Results.** Composite score (lower is better) drops by orders of magnitude on LG6 (2.28 × 10⁷ → 5.07 × 10⁴) and LG7 (1.95 × 10⁷ → 3.49 × 10⁴), both p < 0.001, with zero gradient updates. The first playbook update in every scenario discovered the same failure mode (chasing the target while the guard closes in).
- **Caveats.** On LG4/LG5 a hard-coded prograde-alignment heuristic beats GUIDE; LG5 is not significant (p = 0.143); simulation only; no ablation yet.

---

## 一句話

不改一個權重，只靠離線演化一本自然語言的「戰術手冊」，就讓輕量 LLM 在對抗式軌道攔截情境中把分數改善兩個數量級——但在最單純的情境裡，一條寫死的啟發式規則就打贏了整套系統。

## 問題

把 LLM 部署為即時閉迴路動態系統的監督決策代理人，會撞上三個彼此耦合的限制：

- **延遲矛盾**：會深思的大模型延遲太高，塞不進控制迴路；跑得動的小模型又不會推理。
- **不會變強**：現有把 LLM 當太空載具操作員的做法都是靜態 prompt——同一個任務跑一百次，agent 不會進步。
- **不能重訓**：太空任務無法重置環境、無法大量收集新資料、無法部署後重新訓練，所以「學習」必須發生在權重之外。

三者共同導向 **teacher–student 分離**：前沿推理模型在離線做分回合反思，輕量 acting model 執行即時控制。

## 方法

### Playbook 就是被學習的策略

不透過梯度學習 π_θ(a | s)，而是讓動作選擇以語言結構化的 context 為條件：π(a | s, P)，P 是動態 playbook。Playbook 一開始為空，表示為一組 bullet b_i = (text_i, condition_i(s))；當狀態謂詞成立（例如距離門檻）時，該 bullet 在執行期被注入 prompt。

**被學習的既不是模型參數，也不是固定的 baseline prompt，而是這本不斷演化的 playbook。**

### 線上：兩段式呼叫鎖住延遲

1. **有界推理代理人**（固定 token 預算）：依當前觀測與已啟動的 playbook bullets 產生短時程意圖。
2. **Actor agent**：透過 function call 把狀態與意圖映射為結構化控制輸出，不產生自由形式推理。

### 離線：ε-偏置反思取樣

- 以機率 1 − ε 反思表現差的軌跡、機率 ε 反思成功的軌跡（避免只從失敗中學）。
- **Reflector**（前沿模型）提出行為修正，**Curator** 轉成 {ADD, UPDATE, REMOVE} 操作產生新版 playbook。
- 同時維護多個 playbook 版本，用 **UCB1** 選擇——偏好表現好的版本，同時仍讓新版本有機會被嘗試。

### 測試環境

KSPDG 的 Lady–Bandit–Guard 攔截情境：agent 控制 bandit，目標是最小化與 lady 的距離、同時避免被 guard 捕獲；動作為三軸推力向量加點火時長。

| 情境 | Guard 行為（不會告訴 LLM） |
|---|---|
| LG4 | 積極追擊 |
| LG5 | 在追擊與防守之間隨機切換 |
| LG6 | 防守佈位（停留在 lady 附近） |
| LG7 | 雙 guard 防守 |

## 結果

複合分數（越低越好）：S = (d_LB,min)² + 10⁶ / (d_BG,min + 0.1)

| 策略 | LG4 | LG5 | LG6 | LG7 |
|---|---|---|---|---|
| LLM（靜態 v0） | 4.15 × 10⁵ | 5.37 × 10⁵ | 2.28 × 10⁷ | 1.95 × 10⁷ |
| GUIDE（最佳演化版） | 7.22 × 10⁴ | 8.49 × 10⁴ | **5.07 × 10⁴** | **3.49 × 10⁴** |
| p 值 vs. v0 | 0.043 | 0.143（不顯著） | < 0.001 | < 0.001 |
| Prograde-Alignment（寫死的啟發式） | **3.73 × 10⁴** | **1.60 × 10⁴** | 2.92 × 10⁷ | 1.62 × 10⁷ |

- **最有價值的發現**：所有情境的第一次 playbook 更新都辨識出同一個失效模式——agent 一直追擊，而 guard 趁勢逼近攔截。加入「暫停追擊、施加側向推力規避 guard」的規則後，幾次演化內分數就下降 82–99 %。
- 距離指標同步改善：例如 LG7 的 lady 距離 258.20 m → 52.16 m，guard 分離距離 8.07 m → 29.43 m。
- 作者把這詮釋為一種 **in-context distillation**：離線模型把修正性策略編碼成狀態條件規則，重新塑形固定 acting model 的行為。

## 限制（這篇的限制特別重要）

- **在 LG4 與 LG5，最單純的 Prograde-Alignment 開迴路啟發式反而贏過 GUIDE**：在只需要直接追擊的情境，一條寫死的規則就打贏整套 LLM 系統。
- LG5 的 p = 0.143，統計上不顯著；四個情境只有三個顯著。
- 完全在模擬中驗證，沒有討論算力需求——「輕量 acting model」要多輕、跑在什麼硬體上仍未解。
- 尚未做 ablation：不清楚 playbook / UCB1 / ε-取樣各自貢獻多少，有可能大部分增益只來自那條 guard-avoidance 規則。
- 收斂行為未知：playbook 會不會無限膨脹或震盪，列為未來工作。
- KSPDG 基於遊戲引擎，與高保真太空模擬環境仍有差距。

## 對研究方向的意義

- **成熟度最低、研究空間最大**：只在模擬中驗證，關鍵問題（算力、收斂、ablation）都還開放。
- **「學 context 不學權重」可以搬到別的層級**：本質上是「在無法重訓的部署環境中做策略改進」，任何邊緣裝置都有這個問題。
- **一定要放一個最笨的啟發式當對照組**：GUIDE 誠實地放了 Prograde-Alignment 並且輸了兩局——這反而讓論文更可信，也提醒我們不要只跟「另一個 LLM」比。
- **可補的研究缺口**：把 playbook 演化搬到真實硬體量測延遲；做 ablation；研究 playbook 的收斂性與規模上限；與 RL 混合（LLM 做高層決策、RL 做低層控制，正好對應 LeLaR 那一層）。
- **先補的知識**：agent 架構（ReAct、Reflexion）、in-context learning、UCB／多臂拉霸機、KSPDG 環境。
