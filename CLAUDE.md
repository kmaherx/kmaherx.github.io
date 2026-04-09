# Project Notes

## Changing the Site Font

Two files need updating:

1. **`_config.yml`** (~line 445) — Update the Google Fonts URL to load the desired font family and weights.
2. **`_sass/_base.scss`** (~line 7) — Set `font-family` on the `body` selector. Must target `body` (not `html`) because Bootstrap's `bootstrap.min.css` sets `font-family` on `body` and loads first. The custom `main.css` (compiled from SCSS) loads after Bootstrap at line 69 of `_includes/head.liquid`, so a `body` rule in `_base.scss` overrides it.

## Dark/Light Theme Text Color Transitions

Text color (`--global-text-color`) must be set on `body` only, **not** on individual elements (`p`, `h1`, `div`, etc.). CSS cannot animate custom property (`var()`) changes — the value snaps instantly. When individual elements have `color: var(--global-text-color)` set directly, they snap to the new color on theme toggle instead of transitioning smoothly.

The fix (applied in `_sass/_base.scss`): set `color: var(--global-text-color)` on `body` and let all child elements inherit. The `html.transition *` rule in `_base.scss` (line ~973) applies `transition: all 750ms !important` during toggles, which works on inherited color but not on directly-set CSS variable values.

**Do not** re-add explicit `color: var(--global-text-color)` to `p`, `h1`, `h2`, `h3`, `h4`, `h5`, `h6`, `em`, `div`, `li`, `span`, or `strong` selectors.

## Libertinus Serif Bold Weight

Libertinus Serif's `font-weight: 700` (the default for `bold` / `<strong>`) is visually indistinguishable from regular (400) at body text sizes. To make bold text visible, `strong` is set to `font-weight: 900` in `_sass/_base.scss` under both `.post` and `.caption`. This applies to both `**markdown bold**` in post body text and in figure captions (which render markdown via `markdownify` in `figure.liquid`).
