---
description: Run the full pre-publication gate on a repo before it goes public.
argument-hint: "[path to repo]"
---

# Ship Check

Run every pre-publication check against the repo at the given path. If no path is
given, use the current working directory. Report a single verdict at the end:
SHIP or BLOCKED, with the blocking items listed.

Treat this as a gate, not a review. Fix the repo when it fails; never relax a check
to get a verdict.

## Workflow

### Step 1: Establish what actually ships
Read `.gitignore` and list the files that will be in the published tree, dotfiles
included. Note whether the repo has history that will publish alongside the current
tree. Say which repo type this is (browser tool, library, docs or content repo,
plugin), because some checks are NOT-APPLICABLE for some types and that has to be
stated rather than skipped.

### Step 2: Licence, privacy, copyleft, secrets, links
Use `skill: "compliance-gate"` to:
- confirm a `LICENSE` file exists, is the full licence text, and names the right
  holder and year, and that every other place the repo states its licence agrees
- confirm the privacy claim is both STATED and TRUE, by grepping the shipped files
  for network calls, external scripts, analytics, and cookies
- resolve every dependency and vendored file to an SPDX identifier read out of its
  own LICENSE, and confirm nothing copyleft or unresolved is in the tree
- scan every file, and the history if there is one, for secret-shaped strings
- confirm every relative link, anchor, and outbound URL in the README resolves,
  with exact case

### Step 3: Attribution
Search the tree for "ported", "adapted", "derived from", "based on", "inspired by",
and for vendored directories or files that do not match the repo's own style. If
anything came from elsewhere, use `skill: "third-party-notices"` to build or verify
the `## Third-party notices` section, fetching each upstream LICENSE rather than
writing a holder from memory. If nothing was copied, confirm the README says so
plainly where a reader might otherwise wonder.

### Step 4: Contrast
If the repo ships any HTML or CSS, use `skill: "contrast-audit"` to check the palette
by math: every text token against every background token, in both themes. Do not
accept a rendered-page check in place of the table. Report the full grid, not just
the worst case.

### Step 5: Published claims
Use `skill: "honesty-audit"` on the README and any landing or docs page: every
statistic, quote, comparison, and superlative verified against a named source or
cut; affiliate and sponsored links disclosed next to the link; independence stated
wherever a third party is named.

### Step 6: Verdict
Print one line per check: PASS, FAIL, or NOT-APPLICABLE with the reason. Then the
verdict. For every FAIL give the file, the line, and the specific fix, never a
summary like "some issues found". If any check could not be run, report it as
unverified and name it; do not fold it into PASS.
