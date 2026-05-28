# USER GUIDE

This guide explains how to use `scikgtex-pdflatex` in more detail.

## 1. Purpose

`scikgtex-pdflatex` lets you attach machine-readable annotations to LaTeX documents while keeping compatibility with standard pdfLaTeX-based workflows.

The package is designed so that annotated text can still appear as normal text in the PDF, instead of forcing a special rendering model.

Typical annotation targets include:

- software names
- people
- datasets
- methods

## 2. Installation

Copy `scikgtex-pdflatex.sty` into your LaTeX project directory, or install it somewhere your TeX distribution can find it.

Then load it in your document:

```tex
\usepackage{scikgtex-pdflatex}
```

## 3. Requirements

The package depends on:

- `hyperref`
- `hyperxmp`
- `xparse`

A typical document may also use:

- `fontenc`
- `inputenc`
- `lmodern`

depending on the environment.

## 4. Basic workflow

A normal workflow is:

1. load the package
2. call `\scikgsetup{...}` in the preamble
3. annotate content with `\scikgannotate`
4. compile with `pdflatex`
5. inspect the generated sidecar file

A full example is available in `example.tex`.

## 5. Document setup

Use `\scikgsetup{...}` to define metadata and package options.

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

### Supported setup keys

#### `title`
Document title written into PDF metadata.

#### `author`
Document author written into PDF metadata.

#### `keywords`
Keywords written into PDF metadata.

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

- `true` = validation problems become package errors
- `false` = validation problems are warnings only

#### `format`
Controls the sidecar output format.

Supported values:

- `tsv` — recommended
- `jsonl`

Default:

- `tsv`

## 6. Main annotation command

The main command is:

```tex
\scikgannotate[<options>]{<text>}
```

### Annotation options

#### `mode`
Controls whether the text is printed.

Supported values:

- `inline`
- `silent`

Default:

- `inline`

#### `type`
The semantic category of the annotation.

Examples:

- `software`
- `person`
- `dataset`
- `method`

#### `id`
The external identifier for the annotated entity.

Examples:

- `rrid:SCR_014198`
- `orcid:0000-0002-1825-0097`
- `doi:10.5281/zenodo.1234567`
- `wikidata:Q113145171`

#### `label`
Optional internal label for later reference with `\scikgref`.

## 7. Annotation modes

### Inline mode

Inline annotations print the text in the document and also export metadata.

Example:

```tex
\scikgannotate[
  mode=inline,
  type=software,
  id={rrid:SCR_014198}
]{Cytoscape}
```

Result:

- `Cytoscape` appears in the PDF
- the annotation is written to the sidecar file

### Silent mode

Silent annotations export metadata but do not print the text.

Example:

```tex
\scikgannotate[
  mode=silent,
  type=person,
  id={orcid:0000-0002-1825-0097}
]{Alice Smith}
```

Result:

- nothing is printed in the PDF at that location
- the annotation is still written to the sidecar file

If `silentanchors=true`, silent annotations may also create anchors.

## 8. Labels and references

You can label an annotation:

```tex
\scikgannotate[
  mode=inline,
  type=software,
  id={rrid:SCR_014198},
  label={viz-tool}
]{Cytoscape}
```

and later refer to it with:

```tex
\scikgref{viz-tool}
```

Behavior:

- if the label exists, `\scikgref` creates a hyperlink
- it tries to display the annotated text instead of the raw label
- if the label does not exist, the package prints `??` and emits a warning

### Important
Labels should be unique.

If the same label is reused:

- a warning is emitted by default
- or an error if `strict=true`

## 9. Convenience commands

The package provides shortcuts for common annotation types.

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

These commands are shorthand for `\scikgannotate`.

## 10. Output files

The package writes annotations to a sidecar file named from the TeX job name.

Depending on `format`, the output is:

- `\jobname.scikg.tsv`
- or `\jobname.scikg.jsonl`

## 11. TSV output

TSV is the default and recommended format.

Header:

```text
n	anchor	mode	type	id	label	page	text
```

Example:

```text
1	scikg.1	inline	software	rrid:SCR_014198	viz-tool	1	Cytoscape
2	scikg.2	silent	person	orcid:0000-0002-1825-0097	corr-author	1	Alice Smith
```

## 12. JSONL output

If `format=jsonl` is selected, each annotation is written as one JSON-like line.

Example:

```json
{"n":"1","anchor":"scikg.1","mode":"inline","type":"software","id":"rrid:SCR_014198","label":"viz-tool","page":"1","text":"Cytoscape"}
```

Note that JSONL output is still prototype-level.

## 13. PDF metadata

At the document level, the package writes metadata to the generated PDF using `hyperref` and `hyperxmp`.

Currently this includes:

- title
- author
- keywords
- DOI/license information in the PDF subject field

## 14. Validation behavior

The package checks for:

- missing `type`
- missing `id`
- empty annotation text
- duplicate labels
- undefined `\scikgref` labels

With:

- `strict=false` → warnings
- `strict=true` → package errors for validation failures

## 15. Example document

See `example.tex` for a complete working example.

A typical setup looks like this:

```tex
\documentclass{article}

\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\usepackage{lmodern}
\usepackage{scikgtex-pdflatex}

\scikgsetup{
  title={Example Paper},
  author={Jane Doe},
  keywords={semantic publishing, metadata},
  doi={10.0000/example},
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

## 16. Recommendations

For the most predictable results:

- use `format=tsv`
- keep labels unique
- enable `strict=true` while developing
- start from `example.tex`

## 17. Current limitations

This package is still a prototype. Current limitations include:

- output escaping is still approximate
- JSONL output is less robust than TSV
- sidecar export is the main machine-readable output
- it does not yet provide a rich embedded RDF/XMP annotation model for each inline annotation

## 18. Summary

Use `scikgtex-pdflatex` if you want to:

- annotate entities in LaTeX papers
- keep inline text visible in normal prose
- add silent metadata annotations
- export machine-readable sidecar data
- stay compatible with pdfLaTeX

If you are new to the package, start with `example.tex`.