# PDF to Editable Word Skill - AI 에이전트를 위한 레이아웃 보존 PDF-to-DOCX 도구

[English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [Français](README.fr.md) | [한국어](README.ko.md) | [日本語](README.ja.md)

[![CI](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml)
[![GitHub release](https://img.shields.io/github/v/release/longligooo/pdf-to-editable-word-skill)](https://github.com/longligooo/pdf-to-editable-word-skill/releases/latest)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-3776AB)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

원본 페이지 레이아웃을 최대한 유지하면서 텍스트 기반 PDF를 편집 가능한 Word(`.docx`) 문서로 변환하는 로컬 **PDF-to-Word 변환기**입니다. 텍스트는 검색하고 편집할 수 있게 되며, 표, 도표, 이미지, QR 코드는 페이지 배경에 남습니다.

**Codex, Claude Code 및 다른 Agent Skills 호환 도구를 위한 이식 가능한 Agent Skill**로 실행하거나, 독립형 Python CLI로 사용할 수 있습니다. 재사용 가능한 `SKILL.md` 하나가 동일한 결정론적 로컬 엔진에 변환과 검증을 위임합니다.

- **레이아웃 보존:** 페이지 테두리, 표, 이미지, QR 코드와 시각적 구조를 배경에 유지합니다.
- **편집 가능한 텍스트:** PDF 텍스트를 검색하고 편집할 수 있는 Word 텍스트 상자로 재구성합니다.
- **개인정보를 보호하는 로컬 처리:** 문서를 온라인 변환 서비스에 업로드하지 않습니다.
- **에이전트 지원:** 동일한 `SKILL.md`를 Codex, Claude Code 또는 CLI를 실행할 수 있는 모든 에이전트에서 재사용할 수 있습니다.

> 알파 버전 소프트웨어입니다. 텍스트 기반 PDF와 데스크톱 Microsoft Word에 가장 적합합니다. 스캔한 PDF에는 OCR이 필요하지만, 아직 포함되어 있지 않습니다.

[활용 사례](#use-cases) | [빠른 시작](#quick-start) | [CLI](#cli) | [제한 사항](#limitations)

![PDF를 편집 가능한 Word로 변환하는 Agent Skill 데모](assets/demo.gif)

이 데모는 재배포 가능한 합성 데이터로 생성하고 Microsoft Word로 렌더링했습니다. 한 페이지 레이아웃을 유지하면서 편집 가능한 텍스트 상자 `37`개를 만든 뒤, DOCX 안에서 `Q2`를 `Q3`로, `48 hours`를 `24 hours`로 변경합니다.

[원본 PDF 다운로드](examples/demo-source.pdf) | [편집 가능한 DOCX 다운로드](examples/demo-output.docx) | [편집 후 DOCX 다운로드](examples/demo-edited.docx)

<a id="use-cases"></a>

## 활용 사례

- **고정 레이아웃 보고서 편집:** 페이지 테두리, 차트 및 기타 시각적 요소를 Word에 유지하면서 기존 PDF의 텍스트를 수정합니다.
- **문서를 로컬에서 처리:** 온라인 PDF 변환 서비스에 파일을 보내지 않고 변환합니다.
- **문서 작업 자동화:** Codex나 Claude Code에 PDF 검사, 변환, 검증을 맡기거나, 터미널에서 동일한 명령어를 실행합니다. 중단된 긴 문서 변환은 `--resume`으로 재개할 수 있습니다.

기존 레이아웃 안에서 텍스트를 변경하는 데 적합하며, 표를 편집 가능한 Word 표로 재구성하는 용도는 아닙니다. [알려진 제한 사항](#limitations)을 확인하세요.

<a id="quick-start"></a>

## 빠른 시작: Skill 설치하기

**1. 필수 소프트웨어를 설치하세요.**

Python 3.10 이상, [pipx](https://pipx.pypa.io/), [Poppler](https://poppler.freedesktop.org/)(`pdftoppm`)를 설치하세요.

Debian이나 Ubuntu에서는 `sudo apt-get install poppler-utils`로 Poppler를 설치합니다. Windows에서는 `pdftoppm.exe`를 `PATH`에 추가하거나 `PDFTOPPM` 환경 변수를 설정하세요.

**2. 변환기를 설치하고 사용할 에이전트를 선택하세요.**

```bash
pipx install https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl
pdf2word doctor

# 사용할 에이전트를 선택하세요
pdf2word skill install --agent codex
pdf2word skill install --agent claude
```

`pipx`를 사용하지 않는 경우 GitHub에서 직접 설치할 수 있습니다.

```bash
python -m pip install "https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl"
```

다른 Agent Skills 호환 도구를 사용하는 경우 해당 도구의 skills 디렉터리에 설치하세요.

```bash
pdf2word skill install --destination /path/to/agent/skills
```

**3. 첫 문서를 변환하세요.**

에이전트에 요청하세요.

```text
report.pdf를 편집 가능한 Word 문서로 변환하고 레이아웃을 유지해 주세요.
결과를 검증한 뒤 육안 검토가 필요한 페이지를 알려 주세요.
```

Skill을 지원하지 않는 에이전트도 `pdf2word`를 직접 호출할 수 있습니다.

<a id="cli"></a>

## 독립형 CLI 사용하기

AI 에이전트 없이도 동일한 변환 엔진을 사용할 수 있습니다.

```bash
pdf2word inspect input.pdf
pdf2word convert input.pdf output.docx
pdf2word validate output.docx --pdf input.pdf
```

## 레이아웃을 보존하는 방식

대부분의 PDF-to-Word 변환기는 텍스트 편집 가능성과 시각적 충실도 중 하나를 선택합니다. 이 프로젝트는 두 가지를 결합합니다.

1. PDF 텍스트 레이어에서 단어, 위치, 글꼴 크기와 색상을 추출합니다.
2. 원본의 각 페이지를 렌더링하고 페이지 이미지에서 원래 글자 영역을 제거합니다.
3. 정리된 페이지를 배경으로 놓고 그 위에 절대 위치로 배치한 편집 가능한 텍스트 상자를 올려 DOCX를 재구성합니다.
4. 전달하기 전에 생성된 DOCX의 구조를 검증합니다.

텍스트가 아닌 그래픽은 페이지 배경에 남아 원본의 시각적 형태를 유지합니다. 텍스트는 Word에서 계속 검색하고 편집할 수 있습니다.

긴 문서를 변환하다 중단된 경우 다음 명령으로 재개하세요.

```bash
pdf2word convert input.pdf output.docx --work-dir .pdf2word-work/input --resume
```

캐시는 원본 파일의 지문, 페이지 범위, 변환 설정이 모두 일치할 때만 재사용됩니다.

## 명령어

```text
pdf2word inspect INPUT.pdf [--json]
pdf2word convert INPUT.pdf OUTPUT.docx [--dpi 144] [--resume]
pdf2word validate OUTPUT.docx [--pdf INPUT.pdf] [--json]
pdf2word doctor [--json]
pdf2word skill install [--agent codex|claude] [--scope user|project]
```

Poppler가 `PATH`에 없으면 `--pdftoppm PATH`를 사용하세요. 0부터 시작하는 페이지 범위는 `--start`와 `--end`로 지정합니다.

<a id="limitations"></a>

## 알려진 제한 사항

- 텍스트 레이어가 없는 PDF에는 OCR이 필요하지만, v0.1에는 포함되어 있지 않습니다.
- 텍스트는 편집할 수 있지만, 표, 도표, 이미지 및 기타 페이지 그래픽은 배경 이미지의 일부로 남습니다.
- 텍스트 뒤에 색상이나 질감이 있는 배경에서는 지워진 영역이 드러날 수 있습니다.
- 페이지 크기가 섞여 있으면 이를 감지하고 거부하여, 별다른 알림 없이 잘못된 결과를 생성하는 일을 방지합니다.
- 다양한 Microsoft Word 버전을 지원하기 위해 DOCX에 VML 텍스트 상자를 사용합니다. LibreOffice에서는 위치가 바뀌거나 일부 상자가 표시되지 않을 수 있습니다.
- 원본 글꼴을 사용할 수 없으면 글꼴 대체로 인해 줄의 너비가 달라질 수 있습니다.

## 개발

```bash
python -m pip install -e ".[dev]"
python scripts/sync_bundled_skill.py
python -m unittest discover -s tests -v
```

테스트는 실행 시 합성 PDF를 생성하며, 저작권으로 보호되는 샘플 문서는 커밋하지 않습니다.

## 로드맵

- Microsoft Word로 렌더링한 시각적 차이 보고서
- 스캔 문서용 선택적 OCR 백엔드
- 색상이 있는 페이지의 배경 복원 개선
- 페이지 크기와 방향이 섞인 문서 지원
- 장시간 변환을 위한 비동기 MCP 서버
- 재현 가능한 벤치마크 코퍼스 및 품질 대시보드

## 라이선스

MIT. Poppler는 외부 런타임 의존성이며, 이 저장소에서 배포하지 않습니다.
