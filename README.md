# Arsenal Documentation

This directory contains the documentation for Arsenal. you can find examples, guides, and other resources here.

## Setup

1. Install the required packages:

```bash
# Using uv (recommended)
uv install mkdocs-material

# Alternatively, using pip
pip install mkdocs-material
```

MkDocs Material will automatically install MkDocs and other dependencies.

## Local Development

To preview the documentation locally with live reloading:

```bash
# Navigate to the docs directory
cd docs

# Start the development server
mkdocs serve
```

This will start a local development server at [http://127.0.0.1:8000/](http://127.0.0.1:8000/).

## Building the Documentation

To build the static site:

```bash
# Navigate to the docs directory
cd docs

# Build the site
mkdocs build
```

This will create a `site` directory with the compiled HTML files.

## Documentation Structure

- `docs/`: The main documentation content as Markdown files
  - `index.md`: The home page
  - Various directories for different sections
- `mkdocs.yml`: The MkDocs configuration file
- `theme/`: Custom theme overrides (if any)

## Writing Documentation

- Add new Markdown files to the appropriate directory under `docs/`
- Update `mkdocs.yml` to include new pages in the navigation
- Use standard Markdown syntax with MkDocs-specific extensions

## Configuration

The configuration is defined in `mkdocs.yml`. Key settings include:

- Theme configuration
- Navigation structure
- Plugins
- Markdown extensions

## Deployment

The documentation is automatically built and deployed when changes are pushed to the main branch.
