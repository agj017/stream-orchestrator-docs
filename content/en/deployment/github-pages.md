# GitHub Pages

This repository can publish documentation with MkDocs Material and GitHub Pages.

## Local Preview

Install the documentation dependencies:

```bash
pip install -r docs/requirements.txt
```

Run a local documentation server:

```bash
mkdocs serve -f docs/mkdocs.yml
```

Build the static site:

```bash
mkdocs build --strict -f docs/mkdocs.yml
```

The generated site is written to `site/`.

## Publishing Flow

```text
docs/en/*.md
docs/ko/*.md
docs/mkdocs.yml
docs/requirements.txt
        |
        v
GitHub Actions
        |
        v
GitHub Pages
```
