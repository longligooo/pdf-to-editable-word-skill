# PDF 转可编辑 Word Skill - 为 AI Agent 保留排版的 PDF-to-DOCX 工具

[English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [Français](README.fr.md) | [한국어](README.ko.md) | [日本語](README.ja.md)

[![CI](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml)
[![GitHub release](https://img.shields.io/github/v/release/longligooo/pdf-to-editable-word-skill)](https://github.com/longligooo/pdf-to-editable-word-skill/releases/latest)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-3776AB)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

这是一款本地 **PDF 转 Word 工具**：将带有文字层的 PDF 转为可编辑的 Word（`.docx`）文档，同时尽可能保留原始页面排版。文字可搜索、可选择、可编辑；表格、图表、图片和二维码则保留在页面背景中。

既可以作为 **Codex、Claude Code 及其他兼容 Agent Skills 工具的通用 Skill** 使用，也可以直接运行独立的 Python CLI。同一份可复用的 `SKILL.md` 将转换与校验交给确定性的本地引擎完成。

- **排版保真：** 页面边框、表格、图片、二维码和视觉结构保留在背景层。
- **文字可编辑：** PDF 文字按原坐标重建为可搜索、可编辑的 Word 文本框。
- **本地隐私：** 文档不会上传到在线转换服务。
- **跨 Agent：** 同一份 `SKILL.md` 可用于 Codex、Claude Code 或任何能运行 CLI 的 Agent。

> 当前为 Alpha 版本，适合自带文字层的 PDF，并优先兼容桌面版 Microsoft Word。扫描版 PDF 需要 OCR，当前尚未包含该功能。

[适用场景](#use-cases) | [快速开始](#quick-start) | [CLI](#cli) | [已知限制](#limitations)

![PDF 转可编辑 Word Agent Skill 演示](assets/demo.gif)

演示完全使用可公开分发的合成数据，并通过 Microsoft Word 真实渲染。转换后保留单页布局，生成 `37` 个可编辑文本框，随后直接在 DOCX 中把 `Q2` 改为 `Q3`、把 `48 hours` 改为 `24 hours`。

[下载源 PDF](examples/demo-source.pdf) | [下载可编辑 DOCX](examples/demo-output.docx) | [下载编辑后的 DOCX](examples/demo-edited.docx)

<a id="use-cases"></a>

## 适用场景

- **编辑固定版面的报告：** 修改现有 PDF 中的文字，同时在 Word 中保留页面边框、图表及其他视觉元素。
- **让文档留在本地：** 转换文件，无需将其发送至在线 PDF 转换服务。
- **自动化文档处理：** 让 Codex 或 Claude Code 检查、转换并校验 PDF，也可以在终端中运行相同命令。长文档转换中断后，可用 `--resume` 继续执行。

适合修改现有版面中的文字，而不是将表格重建为可编辑的 Word 表格。详见[已知限制](#limitations)。

<a id="60-秒安装-skill"></a>
<a id="quick-start"></a>

## 快速开始：安装 Skill

**1. 安装前置依赖。**

先安装 Python 3.10+、[pipx](https://pipx.pypa.io/) 和 [Poppler](https://poppler.freedesktop.org/)（`pdftoppm`）。

在 Debian 或 Ubuntu 上，使用 `sudo apt-get install poppler-utils` 安装 Poppler。在 Windows 上，将 `pdftoppm.exe` 加入 `PATH`，或设置 `PDFTOPPM` 环境变量。

**2. 安装转换器并选择 Agent。**

```bash
pipx install https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl
pdf2word doctor

# 按使用的 Agent 选择一个
pdf2word skill install --agent codex
pdf2word skill install --agent claude
```

没有 `pipx` 时，可以直接从 GitHub 安装：

```bash
python -m pip install "https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl"
```

其他兼容 Agent Skills 的工具可以指定 skills 目录：

```bash
pdf2word skill install --destination /path/to/agent/skills
```

**3. 转换第一份文档。**

告诉 Agent：

```text
把 report.pdf 转为可编辑 Word，保持原始排版，完成后校验结果，
并告诉我哪些页面需要人工检查。
```

不支持 Skill 的 Agent 也可以直接调用 `pdf2word`。

<a id="cli"></a>

## 直接使用 CLI

同一套转换引擎也可以在不使用 AI Agent 的情况下运行：

```bash
pdf2word inspect input.pdf
pdf2word convert input.pdf output.docx
pdf2word validate output.docx --pdf input.pdf
```

## 如何保留排版

大多数 PDF 转 Word 工具需要在文字可编辑性与视觉保真度之间取舍。本项目将两者结合：

1. 从 PDF 文字层提取文字、坐标、字号和颜色。
2. 渲染每个原始页面，并擦除页面图像中的原文字区域。
3. 将处理后的页面作为背景，在其上叠加绝对定位的可编辑文本框，重建 DOCX。
4. 交付前校验生成的 DOCX 结构，包括页数、背景图、文本框和可编辑文字数量。

非文字图形保留在背景图中，因此视觉结构稳定；文字则在 Word 中保持可搜索、可编辑。

长文档转换中断后可以续跑：

```bash
pdf2word convert input.pdf output.docx --work-dir .pdf2word-work/input --resume
```

只有源文件指纹、页面范围和转换设置均匹配时，才会复用缓存。

## 命令

```text
pdf2word inspect INPUT.pdf [--json]
pdf2word convert INPUT.pdf OUTPUT.docx [--dpi 144] [--resume]
pdf2word validate OUTPUT.docx [--pdf INPUT.pdf] [--json]
pdf2word doctor [--json]
pdf2word skill install [--agent codex|claude] [--scope user|project]
```

如果 Poppler 不在 `PATH` 中，使用 `--pdftoppm PATH` 指定路径。使用 `--start` 和 `--end` 指定从零开始计数的页面范围。

<a id="limitations"></a>

## 已知限制

- 不带文字层的 PDF 需要 OCR，v0.1 尚未包含该功能。
- 文字可编辑；表格、图表、图片和其他页面图形仍属于背景图，并非可按语义编辑的 Word 对象。
- 彩色或纹理背景上的文字擦除区域可能可见。
- 混合页面尺寸会被检测并主动拒绝，避免静默生成错误文档。
- DOCX 使用 VML 文本框，以广泛兼容 Microsoft Word；LibreOffice 可能将其重新定位或遗漏。
- 查看端缺少源字体时，字体替换可能改变文本行宽。

## 开发与测试

```bash
python -m pip install -e ".[dev]"
python scripts/sync_bundled_skill.py
python -m unittest discover -s tests -v
```

测试 PDF 全部在运行时使用合成数据生成，仓库不包含有版权或私密的示例文档。

## 路线图

- 通过 Microsoft Word 渲染的视觉差异报告
- 用于扫描文档的可选 OCR 后端
- 更完善的彩色页面背景重建
- 混合页面尺寸和方向的文档
- 用于长时间转换的异步 MCP 服务器
- 可复现的基准测试语料库与质量仪表板

## 许可证

MIT。Poppler 是外部运行依赖，本仓库不分发 Poppler。
