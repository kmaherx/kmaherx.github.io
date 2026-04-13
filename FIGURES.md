# Figures Style Guide

## Color Palette (extracted from existing figures)

| Color | Hex | RGB | Usage |
|-------|-----|-----|-------|
| Red (primary) | #BF1E2D | (191, 30, 45) | Soft prompt tokens, highlighted elements |
| Light red/pink | #F6D6D6 | (246, 214, 214) | Highlighted response backgrounds |
| Grey (secondary) | #C1C2C4 | (193, 194, 196) | Baseline/default elements |
| Light grey | #E7E8E9 | (231, 232, 233) | Baseline response backgrounds |
| Black | #000000 | (0, 0, 0) | Text, axes |
| White | #FFFFFF | (255, 255, 255) | Background |

## Typography

- Font: Libertinus Serif (matching the site)
- Axis labels: 11pt
- Tick labels: 10pt
- Titles/annotations: 12pt

## Style Rules

- Rounded edges on boxes and bars (corner radius ~3-4px)
- Minimal gridlines (light grey, y-axis only)
- No chart borders/frames
- White background
- Simple, uncluttered layouts

## Results Figures

One figure per placement condition, each containing:
- (Top: placement schematic, added in Illustrator by hand)
- Bar charts or metrics panels below

### Data Sources

All from `experiments/exp0_toy/results/concise/L4/`:

**results.json:**
- `conditions.{prepend,postpend,single_frame,multi_frame}.fraction_explained` — FE
- `conditions.{...}.kl_curve` — 500-step training curves
- `conditions.{...}.manifold.layer_{9,17,22,29}.mean_relative_error` — RelErr per layer
- `conditions.{...}.sae.layer_{9,17,22,29}.jaccard` — Jaccard per layer
- `conditions.{...}.sae.layer_{9,17,22,29}.n_shared` — shared feature count
- `baselines.layer_{...}.random_tokens.mean_relative_error` — random token baseline
- `baselines.layer_{...}.p_true.mean_relative_error` — ground truth baseline

**results_self_verb.json:**
- `conditions.{...}.best_recovery` — best self-verb recovery
- `conditions.{...}.best_candidate` — best verbalization text
- `conditions.{...}.all_candidates` — all candidates with recovery scores

### Key metrics per condition (concise, L=4, layer 17)

| Condition | FE | RelErr | Jaccard | Shared | Best self-verb recovery | Best candidate |
|-----------|-----|--------|---------|--------|------------------------|----------------|
| prepend | 95.5% | 0.266 | 0.007 | 4 | 86.8% (direct) | "You asked me to provide a concise answer." |
| postpend | 92.5% | 0.028 | 0.054 | 33 | 18.2% (direct) | gibberish |
| single_frame | 94.7% | 0.027 | 0.049 | 30 | 86.5% (direct) | "It instructs me to be concise..." |
| multi_frame | 98.9% | 0.025 | 0.061 | 36 | 100% (multi_frame) | "Be concise." |
| p_true | — | 0.004 | — | — | — | — |
| random_tokens | — | 0.003 | — | — | — | — |

Note: prepend and postpend "best recovery" includes manual baselines ("Be concise." = 100% for both). The SP-generated recovery values are: prepend 86.8%, postpend 18.2%, single_frame 86.5%, multi_frame 100%.
