# PDF to Editable Word Skill - Conversion PDF vers DOCX avec conservation de la mise en page pour les agents IA

[English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [Français](README.fr.md) | [한국어](README.ko.md) | [日本語](README.ja.md)

[![CI](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/longligooo/pdf-to-editable-word-skill/actions/workflows/ci.yml)
[![GitHub release](https://img.shields.io/github/v/release/longligooo/pdf-to-editable-word-skill)](https://github.com/longligooo/pdf-to-editable-word-skill/releases/latest)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-3776AB)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Une **Agent Skill** portable qui convertit les PDF en documents Word modifiables (`.docx`) **sans sacrifier la mise en page d'origine**. Installez un même fichier `SKILL.md` réutilisable dans **Codex, Claude Code ou d'autres outils compatibles avec Agent Skills** ; l'interface en ligne de commande (CLI) locale incluse assure une conversion déterministe et la validation du résultat.

> Cette Skill de conversion PDF vers Word modifiable fonctionne avec plusieurs agents : elle préserve autant que possible la mise en page d'origine, rend le texte modifiable et permet d'y effectuer des recherches. Elle prend en charge les Skills de Codex, de Claude Code et le format générique Agent Skills, avec un traitement entièrement local.

- **Mise en page préservée :** les bordures de page, tableaux, images, codes QR et la structure visuelle sont conservés dans l'arrière-plan.
- **Texte modifiable :** le texte du PDF est reconstruit sous forme de zones de texte Word modifiables dans lesquelles il est possible d'effectuer des recherches.
- **Confidentialité et traitement local :** votre document n'est jamais envoyé à un service de conversion en ligne.
- **Conçu pour les agents :** un même fichier `SKILL.md` réutilisable pour Codex, Claude Code ou tout agent capable d'exécuter la CLI.

> Logiciel en version alpha. Il convient surtout aux PDF contenant une couche de texte et à la version de bureau de Microsoft Word. Les PDF numérisés nécessitent une reconnaissance optique de caractères (OCR), qui n'est pas encore incluse.

![Démonstration de l'Agent Skill de conversion PDF vers Word modifiable](assets/demo.gif)

La démonstration est générée à partir de données synthétiques redistribuables, avec un rendu effectué dans Microsoft Word. Elle préserve la mise en page sur une seule page, crée `37` zones de texte modifiables, puis remplace `Q2` par `Q3` et `48 hours` par `24 hours` dans le DOCX.

[Télécharger le PDF source](examples/demo-source.pdf) | [Télécharger le DOCX modifiable](examples/demo-output.docx) | [Télécharger le DOCX modifié](examples/demo-edited.docx)

## Installer la Skill en 60 secondes

Installez Python 3.10+, [pipx](https://pipx.pypa.io/) et [Poppler](https://poppler.freedesktop.org/) (`pdftoppm`), puis exécutez :

```bash
pipx install https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl
pdf2word doctor

# Choisissez votre agent
pdf2word skill install --agent codex
pdf2word skill install --agent claude
```

Sous Debian ou Ubuntu, installez Poppler avec `sudo apt-get install poppler-utils`. Sous Windows, ajoutez `pdftoppm.exe` au `PATH` ou définissez la variable d'environnement `PDFTOPPM`.

Sans `pipx`, installez directement depuis GitHub :

```bash
python -m pip install "https://github.com/longligooo/pdf-to-editable-word-skill/releases/download/v0.1.0/pdf_to_editable_word-0.1.0-py3-none-any.whl"
```

Pour un autre outil compatible avec Agent Skills, installez la Skill dans son répertoire de skills :

```bash
pdf2word skill install --destination /path/to/agent/skills
```

Demandez ensuite à votre agent :

```text
Convertis report.pdf en document Word modifiable en préservant la mise en page,
valide le résultat et indique-moi les pages qui nécessitent une vérification visuelle.
```

Les agents qui ne prennent pas en charge les Skills peuvent appeler directement `pdf2word`.

## Utiliser la CLI autonome

Le même moteur de conversion fonctionne sans agent IA :

```bash
pdf2word inspect input.pdf
pdf2word convert input.pdf output.docx
pdf2word validate output.docx --pdf input.pdf
```

## Comment la mise en page est préservée

La plupart des convertisseurs PDF vers Word privilégient soit le texte modifiable, soit la fidélité visuelle. Ce projet combine les deux :

1. Extraire les mots, leurs positions, les tailles de police et les couleurs de la couche de texte du PDF.
2. Effectuer le rendu de chaque page source et effacer les zones des glyphes d'origine dans l'image de la page.
3. Reconstruire le DOCX avec la page nettoyée en arrière-plan et, par-dessus, des zones de texte modifiables positionnées de façon absolue.
4. Valider la structure du DOCX généré avant de le livrer.

Les éléments graphiques non textuels restent visuellement fidèles, car ils demeurent dans l'arrière-plan de la page. Le texte reste modifiable et peut faire l'objet de recherches dans Word.

Pour reprendre une conversion interrompue d'un document long :

```bash
pdf2word convert input.pdf output.docx --work-dir .pdf2word-work/input --resume
```

Le cache n'est réutilisé que si l'empreinte du fichier source, la plage de pages et les paramètres de conversion correspondent.

## Commandes

```text
pdf2word inspect INPUT.pdf [--json]
pdf2word convert INPUT.pdf OUTPUT.docx [--dpi 144] [--resume]
pdf2word validate OUTPUT.docx [--pdf INPUT.pdf] [--json]
pdf2word doctor [--json]
pdf2word skill install [--agent codex|claude] [--scope user|project]
```

Utilisez `--pdftoppm PATH` lorsque Poppler n'est pas dans le `PATH`. Utilisez `--start` et `--end` pour définir une plage de pages dont les indices commencent à zéro.

## Limites connues

- Les PDF sans couche de texte nécessitent un traitement OCR, qui n'est pas inclus dans la version 0.1.
- Le texte est modifiable ; les tableaux, schémas, images et autres éléments graphiques de la page restent intégrés à l'image d'arrière-plan.
- Sur un arrière-plan coloré ou texturé, les zones effacées derrière le texte peuvent être visibles.
- Les documents contenant des pages de tailles différentes sont détectés et rejetés plutôt que de produire silencieusement un résultat incorrect.
- Le DOCX utilise des zones de texte VML pour assurer une large compatibilité avec Microsoft Word. LibreOffice peut les déplacer ou ne pas les afficher.
- Lorsque les polices d'origine ne sont pas disponibles, leur substitution peut modifier la largeur des lignes.

## Développement

```bash
python -m pip install -e ".[dev]"
python scripts/sync_bundled_skill.py
python -m unittest discover -s tests -v
```

Les tests génèrent des PDF synthétiques à l'exécution ; aucun document d'exemple protégé par le droit d'auteur n'est ajouté au dépôt.

## Feuille de route

- Rapports de comparaison visuelle avec un rendu effectué dans Microsoft Word
- Moteurs OCR optionnels pour les documents numérisés
- Meilleure reconstruction de l'arrière-plan des pages colorées
- Documents contenant des pages de tailles et d'orientations différentes
- Serveur MCP asynchrone pour les conversions de longue durée
- Corpus de référence reproductible et tableau de bord de qualité

## Licence

MIT. Poppler est une dépendance externe nécessaire à l'exécution et n'est pas distribué par ce dépôt.
