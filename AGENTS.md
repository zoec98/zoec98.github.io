# AGENTS NOTES

## Purpose
This repository hosts **Book Nook**, a Hugo site meant for long-form stories ("books") plus a lightweight blog for infrequent updates. The build uses the [hugo-book](https://github.com/alex-shpak/hugo-book) theme to get navigation, sidebar table of contents, and built-in lunr-based search.

## Local authoring
- Install [Hugo Extended](https://gohugo.io/getting-started/installing/).
- Fetch theme files: `git submodule update --init --recursive` whenever cloning or updating the repo.
- Run the local server with drafts and future-dated posts enabled:
  ```sh
  hugo serve -D -E -F
  ```
- Source content lives under `content/`:
  - `content/books/<book-slug>/` holds each book. `_index.md` stores book metadata; additional markdown files are chapters ordered by `weight`.
  - `content/blog/` holds standalone blog posts.

## Deployment pipeline
- GitHub Actions workflow: `.github/workflows/github-pages.yml` (name `github pages`).
- Triggered on pushes to `main` and on pull requests for validation builds.
- Steps: checkout with submodules, install latest Hugo extended, run `hugo`, publish the `public/` folder with `peaceiris/actions-gh-pages` when on `main`.
- The site is intended for GitHub Pages. Ensure `GITHUB_TOKEN` is available (it is by default) and Pages is pointed to the `gh-pages` branch created by the action.

## Maintenance reminders
- Update the hugo-book theme when needed: `git submodule update --remote --merge`.
- Keep `hugo.toml` as the single source of truth for menus, base URL, and theme params. Document non-obvious params inline using commit messages or this file.
- When adding automation or content conventions, update both this file and `HUMAN.md` (which is aimed at non-technical editors).
