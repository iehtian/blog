# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hexo 7 personal blog (zh-CN) using the Butterfly theme (v5.3.5, installed via npm). Site URL: `iehtian.com`.

## Commands

- `hexo server` / `npm run server` — local preview at localhost:4000
- `hexo generate` / `npm run build` — generate static files
- `hexo clean` / `npm run clean` — clear cache and generated files
- `hexo deploy` / `npm run deploy` — deploy (no deploy target currently configured in _config.yml)
- `hexo new post "title"` — create a new post from scaffold

Always run `hexo clean && hexo generate` after modifying `_config.yml` or `_config.butterfly.yml`.

## Architecture

- **Theme**: Butterfly, loaded as npm dependency (`hexo-theme-butterfly`). The `themes/` directory is empty. All theme customization goes in `_config.butterfly.yml` (1100+ lines), which Hexo 7 merges with the theme defaults.
- **Posts**: `source/_posts/` — Chinese filenames, flat structure with some subdirectories (`linux/`, `stm32f103c8_debug/`, `html&css/`). Post asset folders enabled (`post_asset_folder: true`), so each post can have a same-named directory for images/resources.
- **Scaffolds**: `scaffolds/post.md` defines the front matter schema with many Butterfly-specific fields (toc, copyright, mathjax, katex, aplayer, etc.).
- **Permalink pattern**: `:year/:month/:day/:title/`
- **Renderers**: marked (Markdown), ejs, pug, stylus

## Content Formatting

Follow [FORMAT_GUIDE.md](FORMAT_GUIDE.md) when editing or creating posts. Key rules:
- Chinese punctuation in body text, spaces between Chinese and English
- Code blocks must specify language (```bash, ```js, ```yaml, etc.)
- Headings: `#` for title, `##`/`###` for sections, no skipping levels
- Front matter: keep existing fields, fill in `tags`, `categories`, `description`, `keywords` based on content
- Dates: `YYYY-MM-DD` or `YYYY-MM-DD HH:mm:ss`
