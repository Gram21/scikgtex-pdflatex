# scikgtex-pdflatex

`scikgtex-pdflatex` is a minimal prototype package for adding machine-readable annotations to LaTeX documents while staying compatible with common LaTeX workflows such as **pdfLaTeX**.

The package is inspired by the general idea of SciKG-style semantic annotation, but is designed for users who do not want to depend on LuaLaTeX. Its goal is to let you annotate important entities in a paper while keeping the annotated text usable in normal document text.

A central design requirement is:

- **inline annotations** should print their text normally in the PDF
- **silent annotations** should record metadata without printing the text

The package also supports:

- document-level PDF metadata
- sidecar export in TSV or JSONL-like format
- optional invisible PDF anchors
- annotation labels and references via `\scikgref{...}`
- optional stricter validation

For a working usage example, see `example.tex`.

## Status

This package is still a **prototype**. It is intended as a foundation for experimentation and extension, not yet as a production-ready semantic publishing framework.

Version 6 improves stability over earlier versions by adding:

- duplicate-label warnings
- optional strict mode
- better sidecar sanitization using `\detokenize`
- improved `\scikgref` output
- optional anchors for silent annotations too

Even so, output escaping is still only prototype-level.

## Features

- Compatible with **pdfLaTeX**
- Single main annotation command
- `mode=inline` and `mode=silent`
- Supported annotation fields:
  - `type`
  - `id`
  - `label` (optional)
- Document metadata setup via `\scikgsetup`
- Sidecar output formats:
  - `tsv` (default, recommended)
  - `jsonl`
- Optional invisible anchors for inline annotations
- Optional invisible anchors for silent annotations
- Cross-reference support via `\scikgref{label}`
- Duplicate-label detection
- Optional strict validation

## Installation

At this stage, installation is manual.

Place the following files in your working directory:

- `scikgtex-pdflatex.sty`
- `example.tex` (optional, but recommended to test usage)

You can then load the package with:

```tex
\usepackage{scikgtex-pdflatex}
```

## Requirements

The package currently depends on:

- `hyperref`
- `hyperxmp`
- `keyval`

These are commonly available in standard LaTeX distributions.

Typical example documents may also use:

- `fontenc`
- `inputenc`
- `lmodern`

depending on the environment.

## Quick start

A minimal workflow is:

1. load the package
2. define document metadata with `\scikgsetup`
3. annotate entities with `\scikgannotate`
4. compile with pdfLaTeX
5. inspect the generated sidecar file

For a complete demonstration, compile `example.tex`.

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
\scikgannotate[mode=inline,type=software,id={rrid:SCR_014198},label={viz-tool}]{Cytoscape}
for visualization.

\scikgannotate[mode=silent,type=person,id={orcid:0000-0002-1825-0097},label={corr-author}]{Alice Smith}

You can refer back to \scikgref{viz-tool}.

\end{document}
```

A fuller working example is included in `example.tex`.

## `example.tex`

The included `example.tex` demonstrates:

- package loading
- document metadata setup
- inline annotations
- silent annotations
- anchor creation
- annotation references using `\scikgref{...}`

If you want to understand intended usage, start with `example.tex`.

## Compilation

Compile with a standard pdfLaTeX workflow, for example:

```bash
pdflatex example.tex
```

You may want to compile more than once if you rely on hyperlink-related behavior.

Expected outputs include:

- `example.pdf`
- `example.scikg.tsv` if `format=tsv`
- or `example.scikg.jsonl` if `format=jsonl`

## Package interface

## 1. Document setup

Use `\scikgsetup{...}` to configure metadata and package behavior.

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

Allowed values:

- `true`
- any other value is treated as false

#### `silentanchors`
Controls whether **silent annotations** also get invisible PDF anchors.

Allowed values:

- `true`
- any other value is treated as false

#### `strict`
Controls whether annotation validation problems become warnings or package errors.

Allowed values:

- `true` — validation failures raise package errors
- `false` — validation failures raise warnings only

#### `format`
Controls the sidecar export format.

Supported values:

- `tsv` — default and recommended
- `jsonl`

## 2. Annotation command

The main annotation command is:

```tex
\scikgannotate[<options>]{<text>}
```

### Supported annotation options

#### `mode`
Controls whether the annotation text is printed.

Supported values:

- `inline`
- `silent`

Default: `inline`

#### `type`
Semantic class of the annotation.

Examples:

- `software`
- `person`
- `dataset`
- `method`

#### `id`
External identifier for the annotated entity.

Examples:

- `rrid:SCR_014198`
- `orcid:0000-0002-1825-0097`
- `doi:10.5281/zenodo.1234567`
- `wikidata:Q113145171`

#### `label`
Optional internal label for later reference via `\scikgref{...}`.

## Annotation modes

### Inline annotations

Inline annotations print their text normally in the PDF and also export annotation metadata.

Example:

```tex
\scikgannotate[mode=inline,type=software,id={rrid:SCR_014198}]{Cytoscape}
```

Visible result:

- `Cytoscape` appears in the text

Metadata result:

- annotation record is written to the sidecar file

### Silent annotations

Silent annotations export metadata but do not print the annotation text.

Example:

```tex
\scikgannotate[mode=silent,type=person,id={orcid:0000-0002-1825-0097}]{Alice Smith}
```

Visible result:

- nothing is printed

Metadata result:

- annotation record is written to the sidecar file

If `silentanchors=true` is enabled, a silent annotation can still create a PDF anchor.

## Labels and references

If you assign a label:

```tex
\scikgannotate[mode=inline,type=software,id={rrid:SCR_014198},label={viz-tool}]{Cytoscape}
```

you can later refer to it with:

```tex
\scikgref{viz-tool}
```

In version 6, `\scikgref{viz-tool}` tries to display the **annotated text** (`Cytoscape`) instead of the raw label name.

### Undefined labels

If a label is undefined, the package:

- emits a package warning
- prints `??`

### Duplicate labels

If a label is used more than once, the package emits a warning by default, or an error if `strict=true`.

## Convenience commands

The package includes convenience wrappers.

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

## PDF metadata

The package writes document-level metadata into the generated PDF using `hyperref` and `hyperxmp`.

This currently includes:

- title
- author
- keywords
- DOI/license information via the PDF subject metadata

## Sidecar output

Each annotation is written to a sidecar file named from the TeX job name.

Depending on `format`, the output file is either:

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

If `format=jsonl` is selected, each annotation is written as one JSON-like record.

Example:

```json
{"n":"1","anchor":"scikg.1","mode":"inline","type":"software","id":"rrid:SCR_014198","label":"viz-tool","page":"1","text":"Cytoscape"}
{"n":"2","anchor":"scikg.2","mode":"silent","type":"person","id":"orcid:0000-0002-1825-0097","label":"corr-author","page":"1","text":"Alice Smith"}
```

Note: JSONL output remains prototype-level and is not guaranteed to be fully JSON-safe for all TeX input.

## Validation behavior

The package validates:

- missing `type`
- missing `id`
- empty annotation text
- duplicate labels
- undefined `\scikgref{...}` labels

Behavior depends on `strict`:

- `strict=false`: warnings
- `strict=true`: package errors for validation failures during annotation registration

## Sanitization

Version 6 adds coarse sanitization using `\detokenize` before writing sidecar output.

This improves stability for common cases, but is still not a complete escaping strategy.

### Practical recommendation

Use `format=tsv` unless you specifically need JSONL-like output.

## Design rationale

The package is built around one main principle:

> annotated text should remain normal LaTeX text whenever possible

That is why inline annotations print their content directly into the document while separately exporting machine-readable metadata.

This allows writing prose such as:

```tex
We analyzed \scikgsoftware{rrid:SCR_014198}{Cytoscape} output using
\scikgmethod{wikidata:Q113145171}{PageRank}.
```

with natural visible text and parallel annotation output.

## Limitations

This is still a prototype. Current limitations include:

- escaping is improved but still incomplete
- JSONL output is still only approximate
- TSV output can still be affected by unusual content
- no schema validation beyond required fields
- no automatic duplicate-resolution for labels
- annotation metadata is exported to sidecar files rather than embedded as a rich RDF graph in the PDF body

## Suggested usage

Use this package if you want to:

- annotate entities in a paper
- keep inline annotations visible in normal prose
- use silent annotations for hidden metadata
- stay compatible with pdfLaTeX
- export annotations for post-processing

To get started, compile `example.tex` and inspect the generated sidecar file.

## License

Prototype / unspecified in this draft. Add your preferred license here.