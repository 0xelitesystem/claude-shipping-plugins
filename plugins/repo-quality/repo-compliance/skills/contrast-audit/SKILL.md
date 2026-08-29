---
name: contrast-audit
description: Use when checking colour contrast, auditing a palette for WCAG AA, picking or changing theme tokens, or reviewing a design system's light and dark modes. Verifies every text token against every background token by math rather than by looking at a rendered page. Triggers on "check contrast", "is this accessible", "WCAG AA", "audit the palette", "these colours look washed out", or any change to a CSS custom property that holds a colour.
---

# Contrast Audit

Verify a palette BY MATH. Every text token against every background token, in both
themes, whether or not any page currently paints that combination.

This is the whole skill. The rest is why the obvious method does not work.

## Why a rendered page is not enough

The natural check is: open the page, sample the pixels, measure the text against its
background. It is fast, it uses the real composited colours, and it is wrong in a
specific and repeatable way.

Semantic tokens -- `--ok`, `--warn`, `--err`, `--info` -- do not paint until something
produces a finding. A freshly loaded tool with an empty input has no warning row and
no error row on screen, so there is nothing for the sampler to measure. The check
reports the worst ratio it FOUND, which is drawn only from the chrome and the body
copy, and passes comfortably.

That is not a hypothetical. In this portfolio the light-theme `--warn` token
`#a05a00` measured 4.44 against the portfolio's own light background and the `--err`
token `#b8341a` measured 4.11, both under the 4.5 minimum, and both cleared a
render-only check because neither colour is painted on an idle page. They shipped to
137 repos on 2026-08-20 and were caught on 2026-08-27, only once the palette was
audited as a table of numbers instead of a screenshot. The approved replacements are
`#8a4d00` for warn and `#a82d14` for err.

A render-only check can only fail a colour it happened to paint. A math check fails
every colour that COULD be painted.

## Process

1. **Extract the tokens, do not retype them.** Read the CSS custom properties out of
   the file. Get both themes: the `:root` block, the
   `@media (prefers-color-scheme: dark)` block, and any `[data-theme]` override. A
   token defined in only one theme is itself a finding, because it inherits the other
   theme's value and nobody chose that.

2. **Classify each token as text, background, or neither.** Text tokens are the ones
   that can end up as a `color` value: `--ink`, `--ink-soft`, `--accent`, `--accent-2`,
   and every semantic colour. Background tokens are the ones that can end up as
   `background`: `--bg`, `--bg-deep`, `--surface`, `--surface-2`, `--code-bg`. Include
   the on-accent pair (text sitting on a filled accent button) as its own case.

3. **Compute the full cross product.** Every text token against every background
   token, in each theme. Do not prune the grid to the pairs you have seen used. The
   point of the audit is the pair that is not used yet.

   Relative luminance, per the WCAG definition:

   ```
   channel c in 0..255:  s = c / 255
                         L = s / 12.92          if s <= 0.03928
                         L = ((s + 0.055) / 1.055) ** 2.4   otherwise
   luminance = 0.2126 * L(r) + 0.7152 * L(g) + 0.0722 * L(b)
   ratio     = (lighter + 0.05) / (darker + 0.05)
   ```

   Thresholds. AA body text: 4.5:1. AA large text, meaning 24px or larger, or
   18.66px or larger when bold: 3:1. Non-text UI components and focus indicators:
   3:1. When a token is used at more than one size, hold it to the strictest one.

4. **Print the grid, not a verdict.** Output a table per theme: text tokens down the
   side, background tokens across, the ratio in each cell, a marker on anything under
   threshold. A reviewer needs to see the 4.6 sitting next to the 4.4, because the
   4.6 is the next one to break.

5. **Fix by darkening or lightening the token, then re-run the whole grid.** Changing
   one token can push another pair out of range if they share a surface. A fix is not
   done until the full table is clean again.

6. **Check the composited reality only after the math passes.** If the page uses
   translucent rows, highlight bands, or overlays, the effective background is not the
   token value. Composite the alpha up the ancestor chain to an opaque colour and
   re-check those specific pairs. Skipping this produces false PASSES on translucent
   rows and false FAILURES when a sampler reads a half-transparent pixel.

## Rules that are not negotiable

- **MUST NOT** use a muted or border token as a text colour. `--muted` exists to draw
  1px lines against a surface. It is chosen to be barely visible, which is exactly what
  makes it fail AA in both themes at every size body copy uses. The secondary prose
  token is `--ink-soft`. If a design wants dimmer text than `--ink-soft`, the answer is
  a new token that passes, not the border colour.
- **MUST NOT** report a palette as passing on the basis of a screenshot, a browser
  devtools contrast hint, or an automated page scan. All three measure what is painted.
- **MUST NOT** check only the theme you are currently viewing. Dark mode passing tells
  you nothing about light mode; the failures above were light-theme only.
- **MUST NOT** round a 4.4 up or describe it as "close enough". The threshold is a
  threshold.
- **MUST NOT** copy a hex value from memory or from another project's docs. Read it out
  of the stylesheet that ships.

## Edge cases

- **Gradients and images behind text.** There is no single background value. Measure
  against the lightest AND the darkest point of the gradient, and require both to pass.
- **Disabled controls.** WCAG exempts disabled elements from the contrast minimum, but
  a user still has to read them. Report them separately rather than silently skipping.
- **Text on a filled accent button.** The pair is the on-accent token against the
  accent token, not against the page background. It is the pair most often missed
  because neither half is in the text-token list.
- **Placeholder text in inputs.** It is body-size text on the input surface, so it is
  held to 4.5:1. Placeholder colours are frequently set to a muted token by habit.
- **Syntax highlighting.** Every highlight colour is a text token against `--code-bg`.
  A theme with a dozen highlight colours has a dozen more rows in the grid.
- **The design owner pushes back on a darker colour.** Show the number and the
  threshold, offer the nearest passing hue rather than a flat refusal, and let them
  choose between passing colours.
