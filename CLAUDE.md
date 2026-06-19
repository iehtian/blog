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

## Python API Documentation

This rule applies **both when answering questions in conversation and when writing post files**. Whenever the user asks about a Python class or method (e.g., from scikit-learn, pandas, numpy), describe it with the following structure:

1. **Parameters** — list all parameters with:
   - Name and type
   - Whether it's required or optional
   - Default value (if any)
   - Acceptable values / optional values (e.g., `"auto"`, `"sqrt"`, `"log2"`, `int`, `float`, `None`)
   - Brief explanation of what it does
2. **Return value** — explain:
   - Return type (e.g., `ndarray`, `DataFrame`, `self`)
   - Shape or structure of the returned object
   - Key attributes or methods available on the returned object (e.g., `.classes_`, `.feature_importances_`)
3. **Example** — provide a minimal, runnable code snippet demonstrating the most common use case. Cover the "happy path" first; add edge cases only when they are common pitfalls.

Use Chinese descriptions for parameters and return values (matching the blog's language), but keep code identifiers in English. Prefer table format for parameter lists when there are many parameters.
