---
description: Audit a palette for WCAG AA by math, every text token against every background token, both themes.
argument-hint: "[path to html or css]"
---

# Contrast

Audit the colour palette in the given file for WCAG AA. If no path is given, find
the stylesheet or the HTML that defines the theme tokens.

Verify BY MATH. A rendered page only shows the colours it happens to be painting,
and semantic tokens do not paint until the tool produces a finding, which is how a
token can sit under 4.5:1 while a screenshot check reports everything fine.

## Workflow

### Step 1: Extract the tokens
Read the CSS custom properties out of the file. Capture every theme: the `:root`
block, the `@media (prefers-color-scheme: dark)` block, and any `[data-theme]`
override. Flag any token defined in one theme and not the other, because it silently
inherits a value nobody chose. Do not retype hex values; read them.

### Step 2: Classify
Split the tokens into text candidates (anything that can become a `color`, including
`--ink`, `--ink-soft`, the accents, and every semantic colour) and background
candidates (anything that can become a `background`). Add the on-accent pair, meaning
the text that sits on a filled accent button, as its own case.

### Step 3: Compute the grid
Use `skill: "contrast-audit"` to compute every text token against every background
token, in every theme, using the WCAG relative-luminance formula. Do not prune the
grid to pairs currently in use. Hold body text to 4.5:1, large text (24px, or
18.66px bold) to 3:1, and UI components and focus rings to 3:1.

### Step 4: Report the table
Print one table per theme: text tokens down the side, background tokens across, the
ratio in each cell, a marker on every cell under threshold. Show the near misses too,
because a 4.6 is the next failure.

### Step 5: Fix and re-run
Propose a darker or lighter value for each failing token and re-run the entire grid,
because moving one token can push a neighbouring pair out of range. Confirm that no
muted or border token is used as a text colour anywhere in the stylesheet. Only once
the math is clean, check composited reality: where translucent rows or overlays sit
behind text, composite the alpha up to an opaque background and re-check those pairs.
