# Project Notes

## Changing the Site Font

Two files need updating:

1. **`_config.yml`** (~line 445) — Update the Google Fonts URL to load the desired font family and weights.
2. **`_sass/_base.scss`** (~line 7) — Set `font-family` on the `body` selector. Must target `body` (not `html`) because Bootstrap's `bootstrap.min.css` sets `font-family` on `body` and loads first. The custom `main.css` (compiled from SCSS) loads after Bootstrap at line 69 of `_includes/head.liquid`, so a `body` rule in `_base.scss` overrides it.
