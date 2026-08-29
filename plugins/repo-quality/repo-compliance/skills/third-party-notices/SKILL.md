---
name: third-party-notices
description: Use when code, prose, data or configuration came from another project, or when a README says ported, adapted, derived, based on, inspired by, or borrowed. Produces a Third-party notices section naming the project, the copyright holder copied verbatim from that project's LICENSE, the licence, the URL, and exactly what was taken. Triggers on "add attribution", "third-party notices", "I copied this from", "we vendored X", or a licence review before publishing.
---

# Third-party Notices

If you ported, adapted, or quoted at length from someone else's project, the README
carries a `## Third-party notices` section. This is a condition of the permissive
licences that let you take the code at all, not a courtesy.

## When it applies

It applies when you copied code, prose, data, configuration, or a test corpus, and
when you rewrote it while keeping its structure. "Adapted" is copying. "Ported to
another language" is copying. A short excerpt is still copying: MIT requires the
notice to travel with copies AND with substantial portions, and the cost of a
one-paragraph notice is nothing next to the alternative.

It does NOT apply when you reimplemented behaviour from a specification, a public API
contract, or your own measurements of what a system does. That is not copying. But say
so explicitly instead of leaving it ambiguous, in the same README, in plain words:

```
Behaviour was measured empirically against the published format. No code from any
other project is reproduced here.
```

An ambiguous origin reads worse than a disclosed one. A reader who cannot tell whether
you copied assumes you did.

## Process

1. **List every source.** Walk the tree for vendored directories, single files with a
   foreign code style, snippets pasted from a gist or an answer site, data files, icon
   sets, fonts, and regexes lifted wholesale. Ask what came from where; do not assume
   a clean-room origin because nobody mentioned one.

2. **Fetch the upstream LICENSE file for each source.** Open the actual file at the
   actual commit or tag you took from. Not the GitHub sidebar label, not the licence
   field in `package.json`, not the README badge, not your memory of it.

3. **Copy the copyright holder line verbatim.** Character for character out of that
   LICENSE. Names have initials, middle names, company suffixes, "and contributors",
   and years that are ranges. Getting it approximately right is getting it wrong,
   because the whole obligation is to reproduce the notice as given.

4. **Record the licence name exactly as the file states it.** MIT, BSD-3-Clause,
   Apache-2.0, ISC. If the file is Apache-2.0, check whether the upstream ships a
   `NOTICE` file too. Apache-2.0 section 4(d) requires that NOTICE content to be
   carried forward, and it is a separate file from LICENSE.

5. **Write down what was taken, specifically.** Not "some helpers". Name the function,
   the file, the algorithm, or the block of prose, and roughly how much. A future
   reader needs to know which parts of the repo are governed by that upstream notice.

6. **Emit the section** into the README, one entry per source:

   ```
   ## Third-party notices

   ### <Project name>
   - Source: <URL to the repository or file>
   - Licence: <exact licence name from the LICENSE file>
   - Copyright: <holder line, copied verbatim from that project's LICENSE>
   - Taken: <the specific function, file, algorithm, or passage, and how much>
   ```

   If the upstream licence requires the full licence text to travel with the code, and
   MIT, BSD and Apache-2.0 all do, include it: either the full text under the entry, or
   a `third-party/` directory holding each upstream LICENSE verbatim, referenced by
   path from the entry.

7. **Re-check the repo's own licence for compatibility.** A permissive upstream can be
   included in a permissive project. A copyleft upstream cannot be vendored into one at
   all -- that is a stop-and-raise, not a notice you can write your way out of.

## What NOT to do

- **MUST NOT** write a copyright holder from memory, from a search result, or from
  another project's attribution list. Fetch the LICENSE.
- **MUST NOT** paraphrase, shorten, reformat, or "clean up" a copyright line. Verbatim
  means verbatim, including the year range and any trailing "and contributors".
- **MUST NOT** substitute your own copyright line for the upstream one. Both appear.
  Replacing theirs is the single most common licensing mistake in a fork.
- **MUST NOT** rely on a licence badge, a repository sidebar label, or a package
  manifest field. Those are metadata someone typed; the LICENSE file is the licence.
- **MUST NOT** claim a clean-room reimplementation unless it genuinely was one. If the
  original source was open in another window while the replacement was written, it is
  adaptation, and it gets a notice.
- **MUST NOT** bury the section in a docs subfolder. It goes in the README that ships
  with the repo.

## Edge cases

- **No LICENSE file upstream at all.** Then the work is under exclusive copyright and
  you have no permission to include it. Remove it, or get written permission and record
  that instead. "It was public on the internet" is not a licence.
- **Answer-site and forum snippets.** These carry their site's content licence, which
  is often a share-alike creative-commons term rather than a code licence. Treat any
  non-trivial snippet as needing attribution, and prefer rewriting the logic from the
  explanation instead of copying the block.
- **The upstream changed licence between versions.** The licence that applies is the
  one in the tree at the commit you took from. Record that commit or tag in the entry.
- **Dual-licensed upstream.** Name which of the two you are relying on, and comply
  with that one.
- **Generated or transpiled output of an upstream tool.** The output is usually yours,
  but any runtime or template the tool embeds in the output is not. Check what got
  embedded.
- **Fonts, icons, and sample data.** These have licences too, frequently stricter than
  the code around them, and they are the ones most often shipped unnoticed.
