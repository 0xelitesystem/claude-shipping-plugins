---
name: compliance-gate
description: Use before publishing a repo, flipping one from private to public, or when the user asks whether something is ready to ship. Runs the pre-publication gate -- licence present and correct, the privacy claim stated and actually true, no copyleft vendored, no secret-shaped strings, cross-links that resolve. Triggers on "is this ready to ship", "make this repo public", "run the compliance check", "pre-publish audit", or a release checklist that includes licensing or privacy.
---

# Compliance Gate

Run this before the repo goes public, not after. Every item below is a blocker, not a
nit, and the section says why. A blocker means: do not publish until it is fixed. If
you find yourself wanting to weaken a check to get past it, fix the repo instead.

The gate is a floor. Passing it means nothing is obviously wrong; it does not mean the
repo is good.

## Process

1. **Enumerate what actually ships.** List every file that will be in the published
   tree, including dotfiles and anything not covered by `.gitignore`. Read the
   `.gitignore` first and say which of its rules are load-bearing (`.env`, `*.pem`,
   `keys/`, `dist/`, build output). A repo cannot be audited from a file listing that
   the ignore rules were never applied to.

2. **LICENSE: present, correct, and the one you think it is.**
   - A `LICENSE` file exists at the repo root. Not `LICENSE.md` referenced from the
     README, not "MIT" asserted in a badge with no file behind it.
   - The text is the full licence body, not a one-line summary.
   - The copyright line names the actual holder and the correct year.
   - Every place the repo names its licence -- README, `package.json`, plugin
     manifest, docs site -- says the same thing as the file.

   **Why it blocks:** with no licence file, the default is exclusive copyright. Nobody
   downstream may legally use the code, which is the opposite of what publishing it
   was for. A mismatch between the badge and the file is worse than either alone,
   because someone will rely on the badge.

3. **The privacy claim: stated, and true.** If the README says the tool runs locally,
   uploads nothing, sets no cookies, or has no analytics, then verify it rather than
   trust it. In the shipped files, grep for:
   - `fetch(`, `XMLHttpRequest`, `navigator.sendBeacon`, `new WebSocket`
   - `<script src=`, `<link href=` pointing at any host
   - `cdn.`, `googleapis`, `unpkg`, `jsdelivr`, `gtag`, `analytics`
   - `document.cookie`, `localStorage` used for anything that leaves the machine

   Then check the inverse: if the tool genuinely makes no network request, the README
   must SAY so in a `## Privacy` section. Both halves are blockers.

   **Why it blocks:** an unverified privacy claim is a false statement to users about
   what happens to their data. The absence of the section is the same failure in the
   other direction -- a tool that is private and never says so gets no benefit from
   being private, and the next contributor has nothing telling them not to add
   telemetry.

4. **Copyleft: none vendored.** Resolve every dependency, vendored directory, pasted
   snippet, and file that arrived from a gist to an SPDX identifier read out of its
   own LICENSE file, then reject anything in the copyleft or reciprocal family. Treat
   an unresolved identifier as a rejection, not as a pass with a note. The permissive
   set this repo accepts is MIT, BSD-2-Clause, BSD-3-Clause, ISC, Apache-2.0, and
   Unlicense; anything outside that list stops the publish until a human rules on it.

   **Why it blocks:** copyleft terms attach to what they are combined with. Vendoring
   one into a permissively licensed repo means the repo's own stated licence is a
   false statement, and in a paid product the exposure is worse than editorial. If a
   capability seems to require one, stop and raise it rather than shipping it quietly.

5. **Secret-shaped strings.** Scan every file, including config, fixtures, test data,
   and docs, for:
   - `-----BEGIN` ... `PRIVATE KEY`
   - `ghp_`, `github_pat_`, `sk-`, `AKIA` followed by a long token body
   - `.env` files that were never ignored
   - hard-coded passwords, connection strings, bearer tokens in example commands

   If the repo has history, the same string in a deleted file is still published.
   Say so plainly.

   **Why it blocks:** this is the cheapest check in the list and the one whose failure
   costs the most. A published key is compromised at publication, not at first use,
   and rotating it is the only fix -- deleting the commit is not.

6. **Cross-links resolve.** Every relative link in the README and docs points at a
   file that exists at that path, with that exact case. Every anchor points at a
   heading that exists. Every absolute URL to the project's own site, demo page, or
   sibling repo is one that is actually live. Check case explicitly: a link that
   works on a case-insensitive local filesystem breaks on the case-sensitive host
   that serves the published page.

   **Why it blocks:** a README is the first and often only page a reader sees. Dead
   links in it are the visible evidence that nothing else in the repo was checked
   either, and a broken demo link makes the project look abandoned on day one.

7. **Report.** One line per check: PASS, FAIL, or NOT-APPLICABLE with the reason.
   For each FAIL give the file, the line, and the specific fix. Do not summarise a
   set of failures as "some issues found".

## What NOT to do

- **MUST NOT** report PASS on a check you did not actually run. "Looks fine" is not a
  result. If a file was unreadable or a URL could not be reached, report it as
  unverified and say which one.
- **MUST NOT** weaken, skip, or add an exception to a check in order to get a repo
  published. The gate failing is the gate working.
- **MUST NOT** write a copyright holder, a licence year, or a project name from
  memory. Read it out of the file.
- **MUST NOT** treat a passing gate as an endorsement of the repo's quality,
  correctness, or usefulness. It only means these specific failures are absent.
- **MUST NOT** delete a repo or rewrite history to hide a cosmetic miss. Fix it in
  place and push. History rewriting is for a leaked secret, and even then the secret
  is still rotated.

## Edge cases

- **The repo is a fork or a template.** The upstream LICENSE stays, and the fork's
  own copyright line is added rather than substituted. Removing the original holder
  is the single most common licensing mistake in a fork.
- **Docs-only or content-only repo.** There is no `index.html` to scan for network
  calls, and the privacy section may genuinely not apply. Say NOT-APPLICABLE with
  the reason; do not invent a privacy section for a repo that collects nothing
  because it runs nothing.
- **A dependency's licence is unclear.** Unclear is a FAIL, not a pass with a note.
  Find the LICENSE file in the dependency or drop the dependency.
- **The secret looks like test data.** Example keys in documentation should be
  obviously fake (`sk-EXAMPLE`, `AKIAEXAMPLE`). If a string is plausible enough that
  a scanner flags it, it is plausible enough to be rotated. Replace it with an
  obvious placeholder.
- **The repo is going public after being private.** Everything above applies to the
  full history, not the current tree, because the history publishes too.
