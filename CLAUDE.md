# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is **waz-theme**, a custom fork of Ghost's official "Headline" theme (from `TryGhost/Themes`), adapted for a German/Swiss-language local news publication. 
It's a pure Handlebars (`.hbs`) theme for the Ghost CMS — no JS framework, just Ghost template helpers, hand-written CSS, and a small amount of vanilla JS.

## Commands

Package manager is **pnpm** (`pnpm-workspace.yaml`, `pnpm-lock.yaml`).

```bash
pnpm install        # install dependencies
pnpm run dev        # gulp default task: build CSS/JS/locales, then watch + livereload
pnpm test           # runs `gscan .` — Ghost's theme validator/linter (the closest thing to a test suite)
pnpm run zip        # build + package the theme into dist/headline.zip for upload to Ghost
```

`pnpm test` (gscan) is the main correctness check — run it after any template/helper changes to catch invalid Handlebars helpers, deprecated APIs, or structural issues Ghost will reject.

## Build pipeline (`gulpfile.js`)

- **CSS**: `assets/css/screen.css` → PostCSS (`postcss-easy-import`, `autoprefixer`, `cssnano`) → `assets/built/screen.css` (+ sourcemaps).
- **JS**: concatenates `@tryghost/shared-theme-assets` v1 lib files (`lightbox.js`, `dropdown.js`, `pagination.js`) + its `main.js`, then optionally any local `assets/js/lib/*.js`, then local `assets/js/main.js` → uglify → `assets/built/main.min.js`.
- **Locales**: `mergeLocales` (from `@tryghost/theme-translations`) merges `./locales-local` into `./locales`. `locales-local/` doesn't exist yet in this repo — create it if adding custom/override translation strings beyond the upstream Headline catalog.
- `assets/built/*` is generated — never hand-edit it; edit the sources in `assets/css/` and `assets/js/` instead.

## Template architecture (Ghost theme conventions)

- **`default.hbs`** is the base layout — `<head>`, site header/nav, footer, and script/style includes. Other top-level templates start with `{{!< default}}` and render into `{{{body}}}`.
- Ghost's routing → template resolution:
  - `home.hbs` — homepage (page 1), `index.hbs` — post list fallback (page 2+)
  - `post.hbs` — single post, `page.hbs` — single page, `tag.hbs` — tag archive, `author.hbs` — author archive
- **`custom-*.hbs`** files (`custom-full-feature-image.hbs`, `custom-wide-feature-image.hbs`) appear as selectable "Custom template" options for posts/pages in Ghost Admin. They use `{{#contentFor "body_class"}}` to inject extra body classes (`has-full-image`, `has-wide-image`, `is-head-transparent`), which `default.hbs` outputs via `{{{block "body_class"}}}`.
- **`partials/`** — reusable snippets: `loop-grid` / `loop-minimal` (post cards in different densities), `post-meta`, `feature-image`, `related-posts`, `comments`, `topic-grid` / `topic-minimal` (tag-based content sections), `pswp` (PhotoSwipe lightbox markup). `partials/icons/` holds inline SVG icon partials referenced as `{{> "icons/<name>"}}`.

## Theme settings (`package.json` → `config`)

Custom settings exposed under Ghost Admin → Design, accessed in templates via `@custom.*`:
- `navigation_layout` (Logo left/middle/Stacked), `header_style` (Light/Accent/Dark), `title_font` / `body_font` (sans vs serif) — these drive body classes like `is-head-*-logo`, `is-head-brand`, `is-head-dark`, `has-serif-title`, `has-serif-body` in `default.hbs`.
- `white_publication_logo_for_transparent_header`, `email_signup_header`, `email_signup_text`, `footer_text`.
- `enter_tag_slugs_for_primary_sections` / `enter_tag_slugs_for_secondary_sections` — comma-separated tag slugs that control which tags get featured as homepage sections in `home.hbs` (falls back to top tags by post count if unset).
- `image_sizes` (xs/s/m/l/xl/xxl) defines the responsive breakpoints used by `{{img_url ... size="..."}}` srcsets throughout the partials.
- `posts_per_page: 7` controls pagination size.

## Styling

`assets/css/screen.css` imports `@tryghost/shared-theme-assets` v1 base styles plus local `fonts.css`, then layers theme-specific overrides (topic grid/list sections, article layout, hero/cover, subscribe banner, footer, responsive breakpoints). Self-hosted **Inter** (sans) and **Lora** (serif) fonts live in `assets/fonts/`, declared via `--font-sans` / `--font-serif` custom properties that back the `title_font` / `body_font` setting toggle.

## Internationalization (`locales/`)

- `en.json` is the source-of-truth string catalog for `{{t "..."}}` calls.
- Other locale files (`de.json`, `de-CH.json`, `fr.json`, `nl.json`, etc.) hold translations; an empty string means untranslated and falls back to English.
- `de-CH.json` (Swiss German) is actively being filled in.
- `context.json` is a generated reference mapping each string to every theme/file across the TryGhost theme monorepo where it's used — informational only, not consumed at runtime.
