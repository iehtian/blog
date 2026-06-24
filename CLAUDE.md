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

All formatting rules are in [FORMAT_GUIDE.md](FORMAT_GUIDE.md). Follow it when editing or creating posts. Key sections:
- 标点与语言、标题层级、代码块规范 → §1–§10（基础格式）
- 技术介绍类文章结构 → §11.1（五种类型自适应选择）
- API/命令参考的参数描述规范 → §11.1 类型 1（参数表格 + 返回值 + 整合示例）
