# ServiceNow Notes

[![Python](https://img.shields.io/badge/Python-3.x-blue)](https://www.python.org/)
[![MkDocs](https://img.shields.io/badge/MkDocs-1.5.0-green)](https://www.mkdocs.org/)
[![Material Theme](https://img.shields.io/badge/Material%20Theme-7.3.0-orange)](https://squidfunk.github.io/mkdocs-material/)

A **personal ServiceNow reference site** built with [MkDocs](https://www.mkdocs.org/) and organized into fundamentals, scripting, flows, integrations, UI, reports, and API references. Navigate topics via the sidebar in the MkDocs site.

---

## Project Structure

.
├── App_Dev_fundamentals/
├── App_engine_Studio/
├── Flows/
├── Integration_Hub/
├── Reports/
├── Scripting/
├── Studio/
├── UI/
├── docs/ # Markdown files for MkDocs
├── mkdocs.yml # MkDocs configuration
└── index.md # Home page for the documentation

- `docs/` contains all Markdown files for the site.
- Images and diagrams are currently stored alongside the Markdown files; may move to `docs/images/` later.

---

## Getting Started

### Prerequisites

- Python 3.x
- Pip

### Install MkDocs

\`\`\`bash
pip install mkdocs
\`\`\`

Optional: Install the Material theme for a modern look:
\`\`\`bash
pip install mkdocs-material
\`\`\`

### Serve Locally

\`\`\`bash
mkdocs serve
\`\`\`

If you want it to refresh with your changes run
\`\`\`bash
mkdocs serve --livereload
\`\`\`

- Opens a local server at [http://127.0.0.1:8000](http://127.0.0.1:8000)
- Navigate notes via the sidebar.

### Build Static Site

\`\`\`bash
mkdocs build
\`\`\`

- Creates static HTML site in the `site/` folder.

### Deploy to GitHub Pages

\`\`\`bash
mkdocs gh-deploy
\`\`\`

- Publishes documentation to GitHub Pages automatically.

---

## Navigation & Conventions

- Keep notes organized by folder topic.
- Add new Markdown files in the appropriate folder.
- Update `mkdocs.yml` to add pages to the navigation sidebar.
- Use `placeholder.md` for folders with no content yet to avoid MkDocs errors.

---

## Quick Links (Local / Future Live Site)

- [Home](index.md)
- [ServiceNow Fundamentals](ServiceNow_Fundamentals.md)
- [Flows](Flows/flow_fundamentals.md)
- [Reports](Reports/Report_fundamentals.md)
- [Scripting Overview](Scripting/Client_Scripts.md)
- [API Reference](api/gliderecord.md)

---

## Notes

- Images, diagrams, and Mermaid charts are supported in Markdown.
- Sections can be expanded over time as your notes grow.
- Keep links in Markdown files relative for portability.

---

## License

Personal use and learning purposes only.
EOF
