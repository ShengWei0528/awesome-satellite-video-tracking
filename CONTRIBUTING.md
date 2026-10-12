# Contributing · 貢獻指南

Thanks for helping improve this list! ／ 感謝協助改進這份清單！

## What belongs here · 收錄範圍

- Object detection and tracking in **satellite video** or **remote-sensing image sequences** (single- or multi-object, small / dense targets).
- Datasets and benchmarks for the above.
- **On-board / edge / FPGA** processing of Earth-observation or tracking workloads.
- Closely related **UAV-view** detection and tracking, and **AI for spacecraft operations**.

衛星影片或遙測序列影像中的目標偵測與追蹤、相關資料集與基準、星上／邊緣／FPGA 處理，以及密切相關的無人機視角追蹤與太空載具 AI。

## Entry format · 條目格式

```markdown
- **ShortName** — *Full Paper Title.* F. Author et al. *Venue*, vol. X, year. [[Paper]](https://doi.org/...) [[arXiv]](https://arxiv.org/abs/...) [[Code]](https://github.com/...)  
  One factual English sentence. ／ 一句中文說明。
```

- Use the **official** paper link (DOI, publisher, CVF, PMLR, OpenReview or arXiv). ／ 論文連結請用官方來源。
- Add **[Code]** / **[Dataset]** only for the authors' own repository or official data page. ／ 程式碼與資料集只連作者官方 repo 或官方頁面。
- Mark preprints as `arXiv preprint, YEAR`. ／ 尚未正式發表者請標明 arXiv preprint。
- Numbers in descriptions must be traceable to the paper (abstract, tables or text) or the official repository. ／ 描述中的數字須能在論文（摘要、表格或內文）或官方 repo 中查到。
- Newest first within each section. Each paper has one main entry; the Datasets tables may cross-reference papers listed elsewhere. ／ 各分類內由新到舊；每篇只有一個主要條目，資料集表格可交叉引用其他分類的論文。

## Reading notes · 研讀筆記

Notes in [`notes/`](notes/) follow the template in [`notes/README.md`](notes/README.md): an English TL;DR, then notes in Traditional Chinese. Notes must be written in your own words — do not paste paper text or figures, and do not upload paper PDFs.

筆記請依 [`notes/README.md`](notes/README.md) 的格式：英文 TL;DR ＋ 中文筆記；請用自己的話撰寫，勿貼上論文原文、原圖或上傳論文 PDF。

## Pull request checklist · PR 檢查清單

- [ ] Title, authors, venue and year match the official page ／ 題目、作者、出處、年份與官方頁一致
- [ ] All links open and point to official sources ／ 所有連結可開啟且為官方來源
- [ ] Entry placed in the right section, newest first ／ 放在正確分類並依年份排序
- [ ] No copyrighted figures, full texts or PDFs added ／ 未加入受版權保護的圖、全文或 PDF
