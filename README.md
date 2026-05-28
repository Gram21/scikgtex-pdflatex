# scikgtex-pdflatex

`scikgtex-pdflatex` is a minimal prototype LaTeX package for adding machine-readable annotations to documents while remaining compatible with common workflows such as **pdfLaTeX**.

The project is inspired by the idea of SciKG-style semantic annotation, but is aimed at users who want similar functionality without depending on LuaLaTeX.

Its core goals are:

- allow **inline annotations** whose text appears normally in the PDF
- allow **silent annotations** that are exported as metadata without printing visible text
- export annotation data into a machine-readable sidecar file
- remain simple enough to use in ordinary LaTeX documents

For a complete working usage example, see `example.tex`.

## Status

This is still a **prototype**, not yet a production-ready semantic publishing package.

Version 7 is a cleanup and internal redesign built on **`expl3`** and **`xparse`**. It keeps the user-facing model from earlier versions, but improves the implementation by using structured LaTeX3 data types and key parsing.

Version 7 includes:

- single annotation command: `\scikgannotate`
- inline and silent modes
- document-level PDF metadata
- sidecar export in TSV or JSONL-like format
- optional anchors for inline and silent annotations
- annotation labels with `\scikgref`
- duplicate-label detection
- optional strict validation
- `expl3`-based internal implementation

## Features

- Compatible with **pdfLaTeX**
- Single main annotation command
- Annotation modes:
  - `inline`
  - `silent`
- Annotation keys:
  - `type`
  - `id`
  - `label` (optional)
- Document metadata setup via `\scikgsetup`
- Sidecar output formats:
  - `tsv` (default, recommended)
  - `jsonl`
- Optional invisible PDF anchors
- Label-to-anchor and label-to-text mapping
- Cross-reference support with `\scikgref{label}`
- Duplicate-label detection
- Optional strict validation

## Requirements

The package currently depends on:

- `hyperref`
- `hyperxmp`
- `xparse`

The implementation also uses the LaTeX3 programming layer (`expl3`), which is standard in modern LaTeX distributions.

Typical example documents may additionally load:

- `fontenc`
- `inputenc`
- `lmodern`

depending on the TeX setup.

## Installation

At this stage, installation is manual.

Put these files in your working directory:

- `scikgtex-pdflatex.sty`
- `example.tex` (recommended as a demo)

Then load the package in your document:

```tex
\usepackage{scikgtex-pdflatex}
```

## Quick start

A typical workflow is:

1. load the package
2. configure metadata and package behavior with `\scikgsetup`
3. annotate text with `\scikgannotate`
4. compile with pdfLaTeX
5. inspect the generated sidecar file

If you want to see intended usage directly, compile `example.tex`.

## Basic example

```tex
\documentclass{article}

\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\usepackage{lmodern}
\usepackage{scikgtex-pdflatex}

\scikgsetup{
  title={A Minimal SciKG-like Prototype for pdfLaTeX},
  author={Gram21},
  keywords={semantic publishing, metadata, latex, pdfLaTeX},
  doi={10.0000/example-doi},
  license={CC-BY-4.0},
  anchors=true,
  silentanchors=true,
  strict=false,
  format=tsv
}

\begin{document}

This paper uses
\scikgannotate[
  mode=inline,
  type=software,
  id={rrid:SCR_014198},
  label={viz-tool}
]{Cytoscape}
for visualization.

\scikgannotate[
  mode=silent,
  type=person,
  id={orcid:0000-0002-1825-0097},
  label={corr-author}
]{Alice Smith}

You can refer back to \scikgref{viz-tool}.

\end{document}
```

A fuller working example is provided in `example.tex`.

## `example.tex`

The included `example.tex` demonstrates:

- document metadata setup
- inline annotations
- silent annotations
- invisible anchors
- label-based references using `\scikgref{...}`

If you are starting with the package, `example.tex` is the best reference point.

## Compilation

Compile with a standard pdfLaTeX workflow, for example:

```bash
pdflatex example.tex
```

Depending on your document and hyperlink behavior, you may want to compile more than once.

Expected outputs:

- `example.pdf`
- `example.scikg.tsv` if `format=tsv`
- `example.scikg.jsonl` if `format=jsonl`

## Package interface

## 1. Document setup: `\scikgsetup`

Use `\scikgsetup{...}` to define document metadata and package options.

### Syntax

```tex
\scikgsetup{
  title={...},
  author={...},
  keywords={...},
  doi={...},
  license={...},
  anchors=true,
  silentanchors=false,
  strict=false,
  format=tsv
}
```

### Supported keys

#### `title`
Document title written to PDF metadata.

#### `author`
Document author written to PDF metadata.

#### `keywords`
Keywords written to PDF metadata.

#### `doi`
Stored in the PDF subject metadata.

#### `license`
Stored in the PDF subject metadata.

#### `anchors`
Controls whether **inline annotations** get invisible PDF anchors.

Values:

- `true`
- any other value is treated as false

#### `silentanchors`
Controls whether **silent annotations** also get invisible PDF anchors.

Values:

- `true`
- any other value is treated as false

#### `strict`
Controls validation severity.

Values:

- `true` — validation failures become package errors
- `false` — validation failures remain warnings

#### `format`
Controls the sidecar export format.

Supported values:

- `tsv`
- `jsonl`

Default:

- `tsv`

## 2. Annotation command: `\scikgannotate`

The main command is:

```tex
\scikgannotate[<options>]{<text>}
```

### Supported annotation options

#### `mode`
Controls whether the text is printed.

Supported values:

- `inline`
- `silent`

Default:

- `inline`

#### `type`
Semantic class of the annotated entity.

Examples:

- `software`
- `person`
- `dataset`
- `method`

#### `id`
External identifier for the entity.

Examples:

- `rrid:SCR_014198`
- `orcid:0000-0002-1825-0097`
- `doi:10.5281/zenodo.1234567`
- `wikidata:Q113145171`

#### `label`
Optional internal label for references via `\scikgref`.

## Annotation modes

### Inline annotations

Inline annotations print their text and also export metadata.

Example:

```tex
\scikgannotate[
  mode=inline,
  type=software,
  id={rrid:SCR_014198}
]{Cytoscape}
```

Result:

- `Cytoscape` is printed normally in the document
- an annotation record is written to the sidecar file

### Silent annotations

Silent annotations export metadata without printing visible text.

Example:

```tex
\scikgannotate[
  mode=silent,
  type=person,
  id={orcid:0000-0002-1825-0097}
]{Alice Smith}
```

Result:

- nothing visible is printed
- an annotation record is written to the sidecar file

If `silentanchors=true` is enabled, silent annotations may still create invisible PDF anchors.

## Labels and references

You can label an annotation and refer to it later:

```tex
\scikgannotate[
  mode=inline,
  type=software,
  id={rrid:SCR_014198},
  label={viz-tool}
]{Cytoscape}
```

Later:

```tex
\scikgref{viz-tool}
```

`\scikgref{viz-tool}` tries to display the annotated text rather than the raw label.

### Undefined labels

If a label is undefined:

- a warning is emitted
- `??` is printed

### Duplicate labels

If a label is reused:

- a warning is emitted by default
- an error is raised if `strict=true`

## Convenience commands

The package also provides shorthand wrappers.

### Software

```tex
\scikgsoftware{rrid:SCR_014198}{Cytoscape}
\scikgsoftwaresilent{rrid:SCR_014198}{Cytoscape}
```

### Person

```tex
\scikgperson{orcid:0000-0002-1825-0097}{Alice Smith}
\scikgpersonsilent{orcid:0000-0002-1825-0097}{Alice Smith}
```

### Dataset

```tex
\scikgdataset{doi:10.5281/zenodo.1234567}{Benchmark Dataset}
\scikgdatasetsilent{doi:10.5281/zenodo.1234567}{Benchmark Dataset}
```

### Method

```tex
\scikgmethod{wikidata:Q113145171}{PageRank}
\scikgmethodsilent{wikidata:Q113145171}{PageRank}
```

## Output files

Each annotation is written to a sidecar file named from the TeX job name.

Depending on `format`, the file is:

- `\jobname.scikg.tsv`
- `\jobname.scikg.jsonl`

## TSV output

TSV is the default and recommended format.

Header:

```text
n	anchor	mode	type	id	label	page	text
```

Example:

```text
1	scikg.1	inline	software	rrid:SCR_014198	viz-tool	1	Cytoscape
2	scikg.2	inline	method	wikidata:Q113145171	ranking-method	1	PageRank
3	scikg.3	silent	person	orcid:0000-0002-1825-0097	corr-author	1	Alice Smith
```

## JSONL output

If `format=jsonl` is selected, each annotation is written as one JSON-like line.

Example:

```json
{"n":"1","anchor":"scikg.1","mode":"inline","type":"software","id":"rrid:SCR_014198","label":"viz-tool","page":"1","text":"Cytoscape"}
{"n":"2","anchor":"scikg.2","mode":"silent","type":"person","id":"orcid:0000-0002-1825-0097","label":"corr-author","page":"1","text":"Alice Smith"}
```

This output is still prototype-level and best treated as a convenience export rather than a formally guaranteed JSON serializer.

## Validation behavior

The package checks:

- missing `type`
- missing `id`
- empty text
- duplicate labels
- undefined references

Behavior depends on `strict`:

- `strict=false` → warnings
- `strict=true` → package errors for validation failures

## PDF metadata

At begin document, the package maps setup metadata into the PDF using `hyperref` and `hyperxmp`.

Currently used fields include:

- title
- author
- keywords
- subject containing DOI and license

## Design rationale

The package is built around the principle that annotated text should remain normal LaTeX text whenever possible.

That means inline annotations behave like ordinary content in prose, for example:

```tex
We analyzed \scikgsoftware{rrid:SCR_014198}{Cytoscape}
using \scikgmethod{wikidata:Q113145171}{PageRank}.
```

The sentence remains natural in the final PDF while still producing machine-readable annotation output.

## Limitations

This is still a prototype. Current limitations include:

- escaping is still only approximate
- JSONL output is not guaranteed to be fully JSON-safe for all TeX inputs
- sidecar export is the main machine-readable output; this is not yet a rich embedded RDF/XMP annotation framework
- validation is basic
- no schema-specific semantics yet

## Recommended usage

Use `format=tsv` unless you specifically need JSONL-like output.

If you want the most predictable current behavior:

- use `format=tsv`
- keep labels unique
- enable `strict=true` while developing documents or tests
- start from `example.tex`

## Summary

`scikgtex-pdflatex` is a prototype package for semantic-ish annotation in standard LaTeX workflows.

Use it when you want to:

- annotate named entities in papers
- keep inline text visible in normal prose
- record silent annotations without printing them
- export machine-readable sidecar metadata
- stay compatible with pdfLaTeX

To get started, compile `example.tex` and inspect the generated sidecar file.

## License

Prototype / unspecified in this draft. Add your preferred license here.