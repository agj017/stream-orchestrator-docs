# GitHub Pages

This repository publishes documentation with MkDocs Material and GitHub Pages.

## Local Preview

Install the documentation dependencies:

```bash
pip install -r requirements-docs.txt
```

Run a local documentation server:

```bash
mkdocs serve
```

Build the static site:

```bash
mkdocs build --strict
```

The generated site is written to `site/`.

## GitHub Pages Setup

In the GitHub repository settings:

1. Open **Settings**.
2. Open **Pages**.
3. Set **Build and deployment** to **GitHub Actions**.

After that, every push to `main` that changes documentation files will run `.github/workflows/docs.yml` and deploy the generated site to GitHub Pages.

## Publishing Flow

```text
docs/*.md
docs/*.svg
mkdocs.yml
        |
        v
GitHub Actions
        |
        v
mkdocs build --strict
        |
        v
GitHub Pages
```

## Notes

- Keep documentation source files under `docs/`.
- Add public documentation pages to the `nav` section in `mkdocs.yml`.
- Keep generated or private artifacts out of the navigation unless they are intentionally part of the published site.
