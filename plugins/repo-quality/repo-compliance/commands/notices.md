---
description: Build or verify the README Third-party notices section for code you borrowed.
argument-hint: "[project you borrowed from]"
---

# Notices

Produce the `## Third-party notices` entry for the named project, or verify the
entry that already exists. If no project is named, find the sources yourself.

## Workflow

### Step 1: Identify the sources
If the user named a project, start there. Either way, sweep the repo for anything
that came from elsewhere: vendored directories, single files whose style differs
from the rest, pasted snippets, regexes, icon sets, fonts, sample data, and test
corpora. Ask what came from where rather than assuming a clean-room origin.

### Step 2: Fetch each upstream LICENSE
Open the actual LICENSE file in the upstream repository at the commit or tag the
code was taken from. Not the sidebar label, not the package manifest field, not the
README badge, not memory. If there is no LICENSE file, stop: the work is under
exclusive copyright and cannot be included.

Before fetching, list the hosts and ask the user to confirm; if they decline, report
those entries as unverified and do not write a holder from memory.

### Step 3: Write the entries
Use `skill: "third-party-notices"` to:
- copy the copyright holder line VERBATIM, character for character
- record the exact licence name, and check for a separate `NOTICE` file if the
  upstream is Apache-2.0
- state specifically what was taken: the function, file, algorithm, or passage, and
  roughly how much
- emit one entry per source into the README under `## Third-party notices`, with the
  full upstream licence text carried along either inline or in `third-party/`

### Step 4: Handle the not-copying case
If the work reimplements behaviour from a specification or from your own
measurements, that is not copying and needs no notice, but the README must say so
explicitly rather than leave the origin ambiguous. Add a plain sentence to that
effect.

### Step 5: Check compatibility
Confirm each upstream licence is compatible with the repo's own. A permissive
upstream is fine. A copyleft upstream is a stop-and-raise, not something a notice
can resolve. Report the result and any entry you could not fully verify.
