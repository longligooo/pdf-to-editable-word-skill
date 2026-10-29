# PDF to Editable Word Skill - AI エージェント向けのレイアウトを保持する PDF-to-DOCX ツール

[English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [Français](README.fr.md) | [한국어](README.ko.md) | [日本語](README.ja.md)

[![CI](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml)
[![GitHub release](https://img.shields.io/github/v/release/longligooo/pdf-to-editable-word-skill)](https://github.com/longligooo/pdf-to-editable-word-skill/releases/latest)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-3776AB)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

元のページレイアウトをできる限り保持しながら、テキストベースの PDF を編集可能な Word（`.docx`）文書に変換する、ローカルの **PDF-to-Word コンバーター**です。テキストは検索・編集できるようになり、表、図、画像、QR コードはページの背景に残ります。

**Codex、Claude Code、その他の Agent Skills 対応ツール向けのポータブルな Agent Skill** として、またはスタンドアロンの Python CLI として利用できます。再利用可能な 1 つの `SKILL.md` が、変換と検証を同じ決定的なローカルエンジンに委ねます。

- **レイアウトを保持：** ページの枠線、表、画像、QR コード、視覚的な構造を背景に残します。
- **編集可能なテキスト：** PDF のテキストを、検索・編集できる Word のテキストボックスとして再構築します。
- **ローカル処理でプライバシーを保護：** 文書をオンライン変換サービスにアップロードすることはありません。
- **エージェントに対応：** 同じ `SKILL.md` を Codex、Claude Code、または CLI を実行できる任意のエージェントで再利用できます。

> アルファ版のソフトウェアです。テキストベースの PDF とデスクトップ版 Microsoft Word に最適です。スキャンした PDF には OCR が必要ですが、現時点では含まれていません。

[活用例](#use-cases) | [クイックスタート](#quick-start) | [CLI](#cli) | [制限](#limitations)

![PDF を編集可能な Word に変換する Agent Skill のデモ](assets/demo.gif)

このデモは再配布可能な合成データから生成し、Microsoft Word でレンダリングしています。1 ページのレイアウトを保持したまま、編集可能なテキストボックスを `37` 個作成し、その後 DOCX 内で `Q2` を `Q3` に、`48 hours` を `24 hours` に変更しています。

[元の PDF をダウンロード](examples/demo-source.pdf) | [編集可能な DOCX をダウンロード](examples/demo-output.docx) | [編集後の DOCX をダウンロード](examples/demo-edited.docx)

<a id="use-cases"></a>

## 活用例

- **固定レイアウトのレポートを編集：** ページの枠線、グラフなどの視覚要素を Word 内に残しながら、既存の PDF のテキストを修正します。
- **文書をローカルで処理：** オンラインの PDF 変換サービスに送信せずにファイルを変換します。
- **文書処理を自動化：** Codex や Claude Code に PDF の調査・変換・検証を依頼するか、同じコマンドをターミナルで実行します。中断した長い文書の変換は `--resume` で再開できます。

既存のレイアウト内でテキストを変更する用途に適しています。表を編集可能な Word の表として再構築するためのものではありません。[既知の制限](#limitations)を参照してください。

<a id="quick-start"></a>

## クイックスタート：Skill のインストール

**1. 必要なソフトウェアをインストールします。**

Python 3.10 以降、[pipx](https://pipx.pypa.io/)、[Poppler](https://poppler.freedesktop.org/)（`pdftoppm`）をインストールします。

Debian または Ubuntu では、`sudo apt-get install poppler-utils` で Poppler をインストールします。Windows では、`pdftoppm.exe` を `PATH` に追加するか、環境変数 `PDFTOPPM` を設定してください。

**2. コンバーターをインストールし、エージェントを選択します。**

```bash
pipx install https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl
pdf2word doctor

# 使用するエージェントを選択
pdf2word skill install --agent codex
pdf2word skill install --agent claude
```

`pipx` を使わない場合は、GitHub から直接インストールできます。

```bash
python -m pip install "https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl"
```

その他の Agent Skills 対応ツールでは、そのツールの skills ディレクトリにインストールします。

```bash
pdf2word skill install --destination /path/to/agent/skills
```

**3. 最初の文書を変換します。**

エージェントに次のように依頼します。

```text
report.pdf を編集可能な Word 文書に変換し、レイアウトを保持してください。
結果を検証し、目視確認が必要なページを教えてください。
```

Skill に対応していないエージェントでも、`pdf2word` を直接呼び出せます。

<a id="cli"></a>

## スタンドアロン CLI を使う

同じ変換エンジンを、AI エージェントなしでも利用できます。

```bash
pdf2word inspect input.pdf
pdf2word convert input.pdf output.docx
pdf2word validate output.docx --pdf input.pdf
```

## レイアウトを保持する仕組み

多くの PDF-to-Word コンバーターでは、テキストの編集可能性と見た目の忠実さのどちらかを選ぶ必要があります。このプロジェクトは両方を組み合わせます。

1. PDF のテキストレイヤーから単語、位置、フォントサイズ、色を抽出します。
2. 元の各ページをレンダリングし、ページ画像から元の文字の領域を除去します。
3. 処理済みのページを背景にし、その上に絶対位置指定の編集可能なテキストボックスを配置して DOCX を再構築します。
4. 生成した DOCX の構造を、引き渡し前に検証します。

テキスト以外の図形などはページの背景に残るため、元の見た目を保持します。テキストは Word 内で引き続き検索・編集できます。

長い文書の変換が中断した場合は、次のコマンドで再開します。

```bash
pdf2word convert input.pdf output.docx --work-dir .pdf2word-work/input --resume
```

キャッシュは、元ファイルのフィンガープリント、ページ範囲、変換設定が一致する場合にのみ再利用されます。

## コマンド

```text
pdf2word inspect INPUT.pdf [--json]
pdf2word convert INPUT.pdf OUTPUT.docx [--dpi 144] [--resume]
pdf2word validate OUTPUT.docx [--pdf INPUT.pdf] [--json]
pdf2word doctor [--json]
pdf2word skill install [--agent codex|claude] [--scope user|project]
```

Poppler が `PATH` にない場合は `--pdftoppm PATH` を使います。0 始まりのページ範囲を指定するには、`--start` と `--end` を使います。

<a id="limitations"></a>

## 既知の制限

- テキストレイヤーのない PDF には OCR が必要ですが、v0.1 には含まれていません。
- テキストは編集できますが、表、図、画像などのページ上のグラフィックは背景画像の一部として残ります。
- テキストの背景に色や模様がある場合、文字を消去した領域が目立つことがあります。
- ページサイズの混在は検出して拒否し、誤った出力を通知なしに生成することを防ぎます。
- Microsoft Word で幅広く利用できるよう、DOCX には VML テキストボックスを使います。LibreOffice では位置がずれたり、表示されなかったりする場合があります。
- 元のフォントが利用できない場合、フォントの置き換えによって行の幅が変わることがあります。

## 開発

```bash
python -m pip install -e ".[dev]"
python scripts/sync_bundled_skill.py
python -m unittest discover -s tests -v
```

テストでは実行時に合成 PDF を生成します。著作権で保護されたサンプル文書はコミットしていません。

## ロードマップ

- Microsoft Word でレンダリングした視覚的な差分レポート
- スキャン文書向けのオプションの OCR バックエンド
- カラーのページに対する背景再構築の改善
- サイズや向きが混在する文書への対応
- 長時間の変換に対応する非同期 MCP サーバー
- 再現可能なベンチマーク用コーパスと品質ダッシュボード

## ライセンス

MIT。Poppler は外部の実行時依存関係であり、このリポジトリでは配布していません。
