# PDF 轉可編輯 Word Skill - 為 AI Agent 保留版面的 PDF-to-DOCX 工具

[English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [Français](README.fr.md) | [한국어](README.ko.md) | [日本語](README.ja.md)

[![CI](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml)
[![GitHub release](https://img.shields.io/github/v/release/longligooo/pdf-to-editable-word-skill)](https://github.com/longligooo/pdf-to-editable-word-skill/releases/latest)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-3776AB)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

這是一個可跨 Agent 使用的 **Agent Skill**：將 PDF 轉為文字可搜尋、可選取、可編輯的 Word（`.docx`），同時**盡可能保留原始頁面版面**。將同一份可重複使用的 `SKILL.md` 安裝到 **Codex、Claude Code 或其他相容於 Agent Skills 的工具**中；隨附的本機 CLI 負責執行確定性的轉換與驗證，也可以脫離 Agent 獨立使用。

> 這是一個跨 Agent 的 PDF 轉可編輯 Word Skill：盡可能保留原始版面，文字可搜尋、可編輯，支援 Codex Skill、Claude Code Skill 與通用 Agent Skills，全程在本機處理。

- **版面保真：** 頁面框線、表格、圖片、QR Code 與視覺結構保留在背景層。
- **文字可編輯：** PDF 文字依原座標重建為可搜尋、可編輯的 Word 文字方塊。
- **本機隱私：** 文件不會上傳至線上轉換服務。
- **跨 Agent：** 同一份 `SKILL.md` 可用於 Codex、Claude Code 或任何能執行 CLI 的 Agent。

> 目前為 Alpha 版本，適合具有文字層的 PDF，並優先相容於桌面版 Microsoft Word。掃描版 PDF 需要 OCR，目前尚未包含此功能。

![PDF 轉可編輯 Word Agent Skill 示範](assets/demo.gif)

示範完全使用可公開散布的合成資料，並透過 Microsoft Word 實際轉譯呈現。轉換後保留單頁版面，產生 `37` 個可編輯文字方塊，隨後直接在 DOCX 中將 `Q2` 改為 `Q3`、將 `48 hours` 改為 `24 hours`。

[下載來源 PDF](examples/demo-source.pdf) | [下載可編輯 DOCX](examples/demo-output.docx) | [下載編輯後的 DOCX](examples/demo-edited.docx)

## 60 秒安裝 Skill

先安裝 Python 3.10+、[pipx](https://pipx.pypa.io/) 與 [Poppler](https://poppler.freedesktop.org/)（`pdftoppm`），然後執行：

```bash
pipx install https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl
pdf2word doctor

# 依使用的 Agent 選擇一個
pdf2word skill install --agent codex
pdf2word skill install --agent claude
```

在 Debian 或 Ubuntu 上，使用 `sudo apt-get install poppler-utils` 安裝 Poppler。在 Windows 上，將 `pdftoppm.exe` 加入 `PATH`，或設定 `PDFTOPPM` 環境變數。

沒有 `pipx` 時，可以直接從 GitHub 安裝：

```bash
python -m pip install "https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl"
```

其他相容於 Agent Skills 的工具可以指定 skills 目錄：

```bash
pdf2word skill install --destination /path/to/agent/skills
```

然後直接告訴 Agent：

```text
將 report.pdf 轉為可編輯的 Word 文件，保留原始版面，完成後驗證結果，
並告訴我哪些頁面需要人工檢查。
```

不支援 Skill 的 Agent 也可以直接呼叫 `pdf2word`。

## 直接使用 CLI

同一套轉換引擎也可以在不使用 AI Agent 的情況下執行：

```bash
pdf2word inspect input.pdf
pdf2word convert input.pdf output.docx
pdf2word validate output.docx --pdf input.pdf
```

## 如何保留版面

大多數 PDF 轉 Word 工具需要在文字可編輯性與視覺保真度之間取捨。本專案將兩者結合：

1. 從 PDF 文字層擷取文字、座標、字型大小與顏色。
2. 轉譯每個原始頁面，並清除頁面影像中的原文字區域。
3. 將處理後的頁面作為背景，在其上疊加絕對定位的可編輯文字方塊，重建 DOCX。
4. 交付前驗證產生的 DOCX 結構，包括頁數、背景圖、文字方塊與可編輯文字數量。

非文字圖形保留在背景圖中，因此視覺結構穩定；文字則在 Word 中保持可搜尋、可編輯。

長文件轉換中斷後可以繼續執行：

```bash
pdf2word convert input.pdf output.docx --work-dir .pdf2word-work/input --resume
```

只有來源檔案指紋、頁面範圍與轉換設定皆相符時，才會重複使用快取。

## 命令

```text
pdf2word inspect INPUT.pdf [--json]
pdf2word convert INPUT.pdf OUTPUT.docx [--dpi 144] [--resume]
pdf2word validate OUTPUT.docx [--pdf INPUT.pdf] [--json]
pdf2word doctor [--json]
pdf2word skill install [--agent codex|claude] [--scope user|project]
```

如果 Poppler 不在 `PATH` 中，使用 `--pdftoppm PATH` 指定路徑。使用 `--start` 與 `--end` 指定從零開始計數的頁面範圍。

## 已知限制

- 不含文字層的 PDF 需要 OCR，v0.1 尚未包含此功能。
- 文字可編輯；表格、圖表、圖片與其他頁面圖形仍屬於背景圖，並非可依語意編輯的 Word 物件。
- 彩色或紋理背景上的文字清除區域可能可見。
- 混合頁面尺寸會被偵測並主動拒絕，避免在未提示的情況下產生錯誤文件。
- DOCX 使用 VML 文字方塊，以廣泛相容於 Microsoft Word；LibreOffice 可能重新定位或遺漏這些文字方塊。
- 檢視端缺少來源字型時，字型替代可能改變文字行寬。

## 開發與測試

```bash
python -m pip install -e ".[dev]"
python scripts/sync_bundled_skill.py
python -m unittest discover -s tests -v
```

測試 PDF 全部在執行時使用合成資料產生，儲存庫不包含受著作權保護或私密的範例文件。

## 路線圖

- 透過 Microsoft Word 轉譯呈現的視覺差異報告
- 用於掃描文件的可選 OCR 後端
- 更完善的彩色頁面背景重建
- 混合頁面尺寸與方向的文件
- 用於長時間轉換的非同步 MCP 伺服器
- 可重現的基準測試語料庫與品質儀表板

## 授權條款

MIT。Poppler 是外部執行階段相依套件，本儲存庫不散布 Poppler。
