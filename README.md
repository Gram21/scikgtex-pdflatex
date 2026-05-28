# scikgtex-pdflatex

`scikgtex-pdflatex` is a minimal prototype package for adding **machine-readable annotations** to LaTeX documents while remaining compatible with **common LaTeX workflows such as pdfLaTeX**.

The package is inspired by the idea of tools like SciKGTeX, but is designed for users who do **not** want to depend on LuaLaTeX. Its goal is to let you annotate important entities in a paper while keeping the annotated text usable in normal document text.

In particular, the package supports two annotation modes:

- **inline annotations**: the annotated text is printed normally in the PDF and also exported as metadata
- **silent annotations**: the annotation is exported as metadata, but the text is not printed

In addition to inline and silent annotations, the package can also:

- write document-level PDF metadata
- export annotations to a sidecar file
- optionally create invisible PDF anchors
- register labels for annotations and link to them with `\scikgref{...}`

## Status

This package is currently a **prototype**. It is intended as a starting point for experimentation and extension, not yet as a production-ready scholarly metadata framework.

Current implementation level:

- works with standard LaTeX/pdfLaTeX workflows
- supports inline and silent annotations
- supports document metadata export to the PDF
- supports sidecar export in TSV or JSONL-like format
- includes basic warnings for missing required fields

Known limitations include:

- escaping/sanitization is still minimal
- duplicate labels are not yet detected
- JSON output is less robust than TSV
- inline annotations are exported externally rather than embedded as full semantic RDF structures inside the PDF body

## Features

- Compatible with **pdfLaTeX**
- Single-command annotation interface
- `mode=inline` and `mode=silent`
- Annotation fields:
  - `type`
  - `id`
  - `label` (optional)
- Document metadata setup via `\scikgsetup`
- Sidecar output formats:
  - `tsv` (default, recommended)
  - `jsonl`
- Optional invisible anchors for inline annotations
- Cross-reference support via `\scikgref{label}`

## Installation

At this prototype stage, installation is manual.

Put the package file

- `scikgtex-pdflatex.sty`

into the same directory as your `.tex` document, or into a location where your TeX installation can find it.

For the demo, also keep:

- `example.tex`

in the same working directory.

## Requirements

The package currently depends on:

- `hyperref`
- `hyperxmp`
- `keyval`

These are commonly available in standard LaTeX distributions.

A typical document also uses:

- `fontenc`
- `inputenc`
- `lmodern`

depending on your setup.

## Basic idea

The package separates two concerns:

1. **visible document text**
2. **machine-readable annotation output**

This means you can write normal paper text such as:

```tex
This paper uses \scikgannotate[mode=inline,type=software,id={rrid:SCR_014198}]{Cytoscape}.
```

and the word `Cytoscape` will appear normally in the document, while the annotation is also written to the sidecar metadata file.

By contrast, you can also record metadata without printing text:

```tex
\scikgannotate[mode=silent,type=person,id={orcid:0000-0002-1825-0097}]{Alice Smith}
```

This creates the annotation record, but does not print `Alice Smith` into the visible document.

## Quick start

A minimal workflow looks like this:

1. load the package
2. configure document metadata with `\scikgsetup`
3. annotate text with `\scikgannotate`
4. compile with pdfLaTeX
5. inspect the generated sidecar file

### Minimal example

```tex
\documentclass{article}

\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\usepackage{lmodern}

\usepackage{scikgtex-pdflatex}

\scikgsetup{
  title={A Minimal SciKG-like Prototype for pdfLaTeX v5},
  author={Gram21},
  keywords={semantic publishing, metadata, latex, pdfLaTeX},
  doi={10.0000/example-doi},
  license={CC-BY-4.0},
  anchors=true,
  format=tsv
}

\begin{document}

This paper uses
\scikgannotate[mode=inline,type=software,id={rrid:SCR_014198},label={viz-tool}]{Cytoscape}
for visualization.

\scikgannotate[mode=silent,type=person,id={orcid:0000-0002-1825-0097},label={corr-author}]{Alice Smith}

\end{document}
```

A fuller working example is included in `example.tex`.

## `example.tex`

The repository includes an `example.tex` file that demonstrates the intended usage of the package.

It shows:

- document metadata setup
- inline annotations
- silent annotations
- optional anchor generation
- label-based annotation references using `\scikgref{...}`

If you want to see how the package is meant to be used in practice, start by compiling `example.tex`.

## Compilation

Compile with a standard pdfLaTeX workflow, for example:

```bash
pdflatex example.tex
```

Depending on your setup, you may run pdfLaTeX more than once to stabilize references.

After compilation, you should get:

- `example.pdf`
- a sidecar annotation file:
  - `example.scikg.tsv` if `format=tsv`
  - or `example.scikg.jsonl` if `format=jsonl`

## Package interface

## 1. Document metadata setup

Use `\scikgsetup{...}` to configure package-level metadata and output behavior.

### Syntax

```tex
\scikgsetup{
  title={...},
  author={...},
  keywords={...},
  doi={...},
  license={...},
  anchors=true,
  format=tsv
}
```

### Supported keys

#### `title`
Document title written into PDF metadata.

#### `author`
Document author written into PDF metadata.

#### `keywords`
Keywords written into PDF metadata.

#### `doi`
DOI stored in the PDF subject metadata.

#### `license`
License information stored in the PDF subject metadata.

#### `anchors`
Controls whether inline annotations receive invisible PDF anchors.

Allowed values:

- `true`
- anything else is treated as false

Example:

```tex
anchors=true
```

#### `format`
Controls the sidecar export format.

Supported values:

- `tsv` — recommended and default
- `jsonl`

Example:

```tex
format=tsv
```

## 2. Annotations

The main command is:

```tex
\scikgannotate[<key=value options>]{<text>}
```

### Supported annotation keys

#### `mode`
Controls whether the annotation text is printed.

Supported values:

- `inline`
- `silent`

`inline` is the default.

#### `type`
Describes the semantic class of the annotation.

Examples:

- `software`
- `person`
- `dataset`
- `method`

#### `id`
The external identifier for the entity.

Examples:

- `rrid:SCR_014198`
- `orcid:0000-0002-1825-0097`
- `doi:10.5281/zenodo.1234567`
- `wikidata:Q113145171`

#### `label`
Optional internal label for cross-referencing the annotation.

Example:

```tex
label={viz-tool}
```

## Annotation modes

### Inline annotations

Inline annotations print the annotated text in the document and also export the annotation.

Example:

```tex
\scikgannotate[mode=inline,type=software,id={rrid:SCR_014198}]{Cytoscape}
```

Visible result in the PDF:

- `Cytoscape`

Metadata result:

- annotation record written to the sidecar file

### Silent annotations

Silent annotations export the annotation without printing the text.

Example:

```tex
\scikgannotate[mode=silent,type=person,id={orcid:0000-0002-1825-0097}]{Alice Smith}
```

Visible result in the PDF:

- nothing is printed

Metadata result:

- annotation record written to the sidecar file

## Convenience commands

The package also provides convenience wrappers for common annotation types.

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

These wrappers are just shorthand for `\scikgannotate[...]`.

## Cross-references with labels

If you assign a `label` to an annotation, you can reference it later with:

```tex
\scikgref{viz-tool}
```

Example:

```tex
\scikgannotate[mode=inline,type=software,id={rrid:SCR_014198},label={viz-tool}]{Cytoscape}

See annotation \scikgref{viz-tool}.
```

### Notes about `\scikgref`

- If anchors are enabled, the reference becomes a hyperlink to the annotation anchor.
- The visible text of the reference is currently the label itself.
- If the label is undefined, the package prints `??` and emits a package warning.

## Output files

## PDF output

The package writes document metadata into the produced PDF using `hyperref` and `hyperxmp`.

Currently this includes:

- title
- author
- keywords
- DOI/license information via the PDF subject metadata

## Sidecar output

Each annotation is exported into a sidecar file.

The output file name is based on the TeX job name:

- `\jobname.scikg.tsv`
- or `\jobname.scikg.jsonl`

### TSV format

TSV is the default and recommended output format.

Header:

```text
n	anchor	mode	type	id	label	page	text
```

Example rows:

```text
1	scikg.1	inline	software	rrid:SCR_014198	viz-tool	1	Cytoscape
2	scikg.2	inline	method	wikidata:Q113145171	ranking-method	1	PageRank
3	scikg.3	silent	person	orcid:0000-0002-1825-0097	corr-author	1	Alice Smith
```

### JSONL format

If `format=jsonl` is selected, each annotation is written as one JSON-like line.

Example:

```json
{"n":"1","anchor":"scikg.1","mode":"inline","type":"software","id":"rrid:SCR_014198","label":"viz-tool","page":"1","text":"Cytoscape"}
{"n":"2","anchor":"scikg.2","mode":"silent","type":"person","id":"orcid:0000-0002-1825-0097","label":"corr-author","page":"1","text":"Alice Smith"}
```

Note that JSONL output is still prototype-level and not fully escaped.

## Warnings and validation

The package currently emits warnings for:

- missing `type`
- missing `id`
- empty annotation text
- undefined `\scikgref{...}` labels

These warnings do not stop compilation; they are diagnostic only.

## Example workflow

A typical usage pattern might look like this:

1. define document metadata in `\scikgsetup`
2. annotate entities inline where they appear in the text
3. use silent annotations for metadata that should not appear visibly
4. compile the document
5. post-process the sidecar file if needed

For a complete demonstration, see `example.tex`.

## Design rationale

This prototype is designed around a simple principle:

> annotated text should remain usable as normal LaTeX content

That is why inline annotations print their text directly instead of replacing it with a special visual object.

This makes the package easy to use in ordinary prose, for example:

```tex
We analyzed \scikgsoftware{rrid:SCR_014198}{Cytoscape} output using
\scikgmethod{wikidata:Q113145171}{PageRank}.
```

The sentence reads normally in the PDF, while still emitting annotation metadata.

## Limitations

This is still a prototype. Important limitations include:

- escaping of special characters is incomplete
- tabs/newlines inside annotation text may break TSV output
- JSONL output is not fully JSON-safe
- duplicate annotation labels are not yet detected
- silent annotations do not create visible text
- silent annotations currently do not create physical PDF anchors in the same way inline annotations do
- `\scikgref{...}` displays the label rather than the original annotation text

## When to use TSV vs JSONL

### Use TSV if:
- you want the most robust current output mode
- you plan to post-process annotations with scripts
- your data is mostly simple text without complex escaping needs

### Use JSONL if:
- you want a more structured export shape
- you are comfortable treating the current output as prototype-level
- you plan to improve escaping in a future version

## Future directions

Possible future improvements include:

- better escaping/sanitization
- duplicate-label detection
- strict validation mode
- richer reference output
- exporting annotation content together with labels
- richer PDF embedding beyond sidecar export
- schema-aware output formats such as JSON-LD or RDF

## Summary

`scikgtex-pdflatex` provides a simple way to experiment with **semantic annotations in ordinary LaTeX documents** without requiring LuaLaTeX.

Use it when you want to:

- keep annotated text visible in the document
- attach structured metadata to entities in the paper
- stay in a pdfLaTeX-friendly workflow
- generate a machine-readable sidecar export for further processing

To get started, compile `example.tex` and inspect the generated `.scikg.tsv` or `.scikg.jsonl` file.

## License

Prototype / unspecified in this draft. Add your preferred license here.