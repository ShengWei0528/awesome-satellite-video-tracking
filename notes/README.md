# Reading Notes · 論文研讀筆記

Personal reading notes by **Sheng-Wei Lu (呂聖韡)**, written for paper-reading reports and lab-meeting presentations during my M.S. at the Graduate Institute of Space Systems Engineering, National Taipei University of Technology. Each note starts with a short English TL;DR, followed by the full notes in Traditional Chinese.

以下是我在碩士班（國立臺北科技大學 太空系統工程研究所）撰寫論文研讀報告與準備實驗室報告時整理的論文筆記。每份筆記開頭有英文 TL;DR，其後為中文完整筆記。

> These are summaries and critiques in my own words — not translations, and they contain no figures from the papers. Numbers quoted from papers are for reference only; always cite the original publication.
>
> 筆記為個人整理與評析，非逐字翻譯，也不含論文原圖；引用請以原文為準。

| Note | Paper | Venue | Topic |
|---|---|---|---|
| [viso-mmb.md](viso-mmb.md) | Detecting and Tracking Small and Dense Moving Objects in Satellite Videos: A Benchmark | IEEE TGRS 2022 | Satellite-video dataset + moving-object detection baseline |
| [sglatrack.md](sglatrack.md) | Similarity-Guided Layer-Adaptive Vision Transformer for UAV Tracking | CVPR 2025 | Efficient ViT tracker for UAVs |
| [sr-yolov5s.md](sr-yolov5s.md) | Super-Resolution YOLO Object Detection for Maritime Surveillance with Real-Time FPGA Processing Onboard Spacecraft | Acta Astronautica 2026 | On-board small-object detection on FPGA |
| [lelar.md](lelar.md) | LeLaR: The First In-Orbit Demonstration of an AI-Based Satellite Attitude Controller | IEEE Access 2026 | Deep-RL attitude control in orbit |
| [guide.md](guide.md) | GUIDE: Guided Updates for In-context Decision Evolution in LLM-Driven Spacecraft Operations | AI4Space @ CVPR 2026 | LLM decision-making for spacecraft |
| [edge-ai-for-space-synthesis.md](edge-ai-for-space-synthesis.md) | Synthesis of SR-YOLOv5s, LeLaR and GUIDE | — | Perception → control → decision; lessons learned |

## Note template · 筆記格式

1. Header table — paper, venue, official links, why I read it
2. **TL;DR (English)** — 3–5 bullets
3. 一句話 — one-sentence takeaway
4. 問題 / 方法 / 結果 — problem, method, key results
5. 限制 / 評析 — limitations and my critique (including inconsistencies I found in the paper)
6. 與研究題目的關聯／對研究方向的意義 — how it relates to my research (detection & tracking in UAV / satellite imagery)

The synthesis note does not follow this template; it compares several papers side by side. ／ 綜合研讀筆記為多篇對照，不套用此格式。
