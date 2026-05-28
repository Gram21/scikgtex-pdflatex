# scikgtex-pdflatex

`scikgtex-pdflatex` is a LaTeX package prototype for adding **machine-readable annotations** to papers while staying compatible with **pdfLaTeX** and common LaTeX workflows.

It is inspired by [@Christof93/SciKGTeX](https://github.com/Christof93/SciKGTeX), but is aimed at users who want similar annotation-oriented functionality without requiring LuaLaTeX.

It is designed for cases where you want to annotate entities such as software, people, datasets, or methods, while keeping the text usable in normal LaTeX documents.

## What it does

The package supports two annotation modes:

- **inline annotations**: the text appears normally in the PDF and is also exported as metadata
- **silent annotations**: the metadata is exported, but the text is not printed

It also supports:

- document-level PDF metadata
- sidecar export in TSV or JSONL-like format
- optional annotation anchors
- references to labeled annotations with `\scikgref{...}`

## Quick example

```tex
\usepackage{scikgtex-pdflatex}

\scikgsetup{
  title={Example Paper},
  author={Jane Doe},
  doi={10.0000/example},
  format=tsv,
  anchors=true
}

This paper uses
\scikgannotate[
  mode=inline,
  type=software,
  id={rrid:SCR_014198},
  label={viz-tool}
]{Cytoscape}
for visualization.
```

This prints `Cytoscape` normally in the document and also writes an annotation record to a sidecar file.

## Getting started

1. Put `scikgtex-pdflatex.sty` in your project directory.
2. Look at `example.tex` for a working example.
3. Compile with `pdflatex`.
4. Check the generated sidecar file:
   - `*.scikg.tsv` if `format=tsv`
   - `*.scikg.jsonl` if `format=jsonl`

## Recommended usage

If you are just starting out:

- use `format=tsv`
- keep labels unique
- begin from `example.tex`

## Documentation

For full usage details, see:

- [`USER-GUIDE.md`](USER-GUIDE.md)

## Status

This package is currently a **prototype**. It is suitable for experimentation and small projects, but not yet a fully hardened semantic publishing framework.