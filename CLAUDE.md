# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal interview-prep notes (Java, HLD, LLD, DS, behavioral) written in an Obsidian vault (`docs/`) and published as a MkDocs Material site to GitHub Pages (https://shubham721.github.io/engineering-codex/). No application code: the "code" is Markdown plus site config.

## Commands

- `make setup`: create `.venv` and install `requirements.txt`
- `make serve`: live preview at http://127.0.0.1:8000
- `make build`: `mkdocs build --strict`. This is the only check: broken links, missing images, and nav errors fail the build. Run it before committing.

CI (`.github/workflows/deploy.yml`) runs the same strict build on every push to `main` and deploys to Pages. `site/` is build output and is gitignored.

## Structure and conventions

- **Navigation** comes from `.nav.yml` files (mkdocs-awesome-nav), not from `mkdocs.yml`. A directory's `.nav.yml` sets its title and order. Directories without an explicit `nav:` list their pages automatically.
- **Section landing pages** are `index.md` (`navigation.indexes`). Empty placeholder dirs use `.gitkeep`.
- **New notes** follow `docs/templates/note.md` frontmatter: `title`, `tags: [...]`, `status: draft`. `templates/` and `.obsidian/` are excluded from the site build.
- **HLD case studies** go in `docs/system-design/hld/case-studies/<kebab-name>.md`. Name images `<topic>-<what>.png`.
- **Images** go in the flat `docs/assets/` folder (Obsidian's attachment folder). Reference them with standard Markdown, a relative path, and descriptive alt text, e.g. `![Rate limiter architecture](../../../assets/rate-limiter-architecture.png)`. Don't use Obsidian `![[wikilinks]]`: the vault is configured for Markdown links so MkDocs can resolve them.
- **Supported Markdown**: Obsidian callouts (`> [!note]`, via mkdocs-callouts), admonitions, mermaid fences, task lists, tags. Code samples are mostly Java.
- `docs/.obsidian/workspace*.json` and `cache` are gitignored. Other `.obsidian/` config is committed.
