# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MkDocs Material documentation site intended to document the following topics:
- Cardistry
- Rocketry
- Empire of the Sun
- Video Games
- Reading

The site is deployed to GitHub Pages via GitHub Actions, which automatically builds and deploys on push to the main branch.

## Development Commands

### Initial Setup
```bash
# Install dependencies (Python 3.x required)
pip install mkdocs-material

# Or using a virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install mkdocs-material
```

### Local Development
```bash
# Start the development server with live reload
mkdocs serve

# The site will be available at http://127.0.0.1:8000
```

### Building and Deployment
```bash
# Build the static site (outputs to /site directory)
mkdocs build

# GitHub Actions handles automatic deployment to gh-pages branch
# Manual deployment (if needed):
mkdocs gh-deploy
```

## Project Structure

- `mkdocs.yml` - Main configuration file defining site structure, theme settings, and navigation
- `docs/` - Source markdown files for documentation content
- `site/` - Generated static site (gitignored, built by MkDocs)
- `.github/workflows/` - GitHub Actions workflow for automated deployment to GitHub Pages

## Configuration Notes

When modifying `mkdocs.yml`:
- The `nav:` section defines the sidebar navigation structure
- Material theme configuration goes under `theme:` with `name: material`
- Plugins and extensions for enhanced Markdown features are configured under `markdown_extensions:`
- The site builds from the `docs/` directory by default

## Content Organization

Each major topic (Cardistry, Rocketry, Empire of the Sun, Video Games, Reading) should have its own section in the documentation. This can be organized as:
- Individual markdown files in `docs/`
- Subdirectories under `docs/` for topics with multiple pages
- Navigation structure defined in `mkdocs.yml`
