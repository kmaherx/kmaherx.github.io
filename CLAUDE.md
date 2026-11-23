# Project Notes

## Changing the Site Font

The body font is **Crimson Pro**, self-hosted (not Google Fonts):

1. **`_sass/_fonts.scss`** — `@font-face` rules for the `"Crimson Pro Site"` family, generated from `$crimson-site-map`. Weights are shifted at the `@font-face` level (requested 100–500 → face 300, 600 → 500, 700–900 → 600), so no other CSS needs weight changes. Same in light and dark mode. Page/post titles (`.post-title`) and post section headings (`.post h2`–`h6`) request 600 (→ 500) in `_sass/_base.scss`, a step lighter than inline bold (→ 600). `"Crimson Pro Site Name"` is a single-face family (400) used only for the name on the home page (`_pages/about.md`).
2. **`assets/fonts/crimson-pro/`** — the WOFF2 files (latin + latin-ext subsets, from `@fontsource/crimson-pro`) and `OFL.txt`. Changing a face weight in the maps requires adding the matching file.
3. **`_sass/_base.scss`** (~line 7) — Set `font-family` on the `body` selector. Must target `body` (not `html`) because Bootstrap's `bootstrap.min.css` sets `font-family` on `body` and loads first. The custom `main.css` (compiled from SCSS) loads after Bootstrap at line 69 of `_includes/head.liquid`, so a `body` rule in `_base.scss` overrides it.

To switch to a Google-hosted font instead, add it to the Google Fonts URL in `_config.yml` (~line 444, currently only Material Icons) and set it in step 3.

After font/CSS changes, hard-refresh (Cmd+Shift+R): the `bust_css_cache` hash on `main.css` doesn't change with content, so browsers may serve the old CSS.

## Dark/Light Theme Text Color Transitions

Text color (`--global-text-color`) must be set on `body` only, **not** on individual elements (`p`, `h1`, `div`, etc.). CSS cannot animate custom property (`var()`) changes — the value snaps instantly. When individual elements have `color: var(--global-text-color)` set directly, they snap to the new color on theme toggle instead of transitioning smoothly.

The fix (applied in `_sass/_base.scss`): set `color: var(--global-text-color)` on `body` and let all child elements inherit. The `html.transition *` rule in `_base.scss` (line ~973) applies `transition: all 750ms !important` during toggles, which works on inherited color but not on directly-set CSS variable values.

**Do not** re-add explicit `color: var(--global-text-color)` to `p`, `h1`, `h2`, `h3`, `h4`, `h5`, `h6`, `em`, `div`, `li`, `span`, or `strong` selectors.

## Bold Weight (legacy from Libertinus Serif)

`strong` is set to `font-weight: 900` in `_sass/_base.scss` under both `.post` and `.caption`. This was a workaround for the previous font, Libertinus Serif, whose 700 looked like regular at body sizes. With Crimson Pro, 700–900 all map to the same face (600), so the 900 is harmless and can stay.

## Inline LaTeX with Subscripts

Kramdown (the markdown processor) interprets `_` as emphasis before MathJax processes the math. This means inline math like `$\text{KL}_{\text{final}}$` gets mangled — Kramdown sees the `_` between `}` and `{` as an emphasis delimiter.

**Use `$$` (display math) for any expression containing subscripts (`_{}`).** Simple inline math without subscripts (e.g., `$\theta$`, `$x$`, `$L > 1$`) works fine with single `$`.
