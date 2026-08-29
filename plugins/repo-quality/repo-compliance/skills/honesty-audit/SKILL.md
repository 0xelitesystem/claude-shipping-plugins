---
name: honesty-audit
description: Use before publishing a README, landing page, changelog, release note, or any marketing copy, and when reviewing copy that contains numbers, quotes, comparisons, or outbound links. Checks that every statistic, testimonial and endorsement is real and verifiable, that affiliate and sponsored links are disclosed next to the link, and that independence is stated wherever a third party is named. Triggers on "review this copy", "is this claim OK", "check the landing page", "add a comparison table", or any draft containing user counts, benchmarks, or quoted praise.
---

# Honesty Audit

Every claim in published copy is either verifiable at the moment of writing or it comes
out. This is exposure, not style. Fabricated endorsements and invented statistics are a
consumer-protection problem in most jurisdictions, and undisclosed paid placement is the
same problem wearing a different hat.

The rule that does the work: **if a claim cannot be verified, cut it.** Do not soften it
into vague language. Vague language is the same claim with the accountability removed.

## Process

1. **Extract every checkable assertion.** Read the copy and pull out, one per line:
   - numbers: user counts, download counts, star counts, percentages, timings,
     benchmark results, "N repos", "used by N teams"
   - quotes and named praise: testimonials, endorsements, logos, "trusted by"
   - comparisons: "faster than X", "unlike Y", "the only tool that"
   - superlatives and absolutes: "best", "first", "never", "always", "zero"
   - implied relationships: "works with X" where X is a company, "for X users",
     "X-compatible"
   - outbound links: any link that pays, or was paid for

2. **Verify each one against a source you can name.** For a number, the source is a
   query you ran, a file you read, or a dashboard you looked at, and you write down
   which. For a quote, the source is a real person who actually said it and agreed to
   its use. "It seems roughly right" is not verification.

3. **Apply the cut rule.** If verification failed or was not possible:
   - CUT the claim. Delete the sentence.
   - Do NOT rewrite "used by 4,000 developers" as "used by thousands of developers" or
     "widely used". The vaguer version asserts the same unverified thing and is harder
     to correct later.
   - Where the underlying point is real but the number is not, state the mechanism
     without the number. "Each request is checked against the palette table" is
     verifiable. "Catches 97% of contrast bugs" is not.

4. **Check disclosure placement on every paying link.** Affiliate links, sponsored
   placements, and paid reviews are disclosed in the page, in the copy, adjacent to
   the link -- before or immediately after it, in the same visual block. A disclosure
   in the footer, in a separate policy page, or behind a tooltip does not count,
   because the reader clicks before reaching it.

5. **State independence wherever a third party is named.** If the tool analyses,
   inspects, converts, or compares a named company's product, the copy says plainly
   that the project is independent and not affiliated with, endorsed by, or sponsored
   by that company. Naming a product as the thing you support is fine and often
   necessary. Letting a reader conclude the company backs you is not, and layout can
   imply it even when no sentence does -- check the logo placement and the headline,
   not only the prose.

6. **Check the tense and the scope of every capability claim.** "Supports X" must be
   true of the shipped version today, not the branch. "Local-first" must be true of
   every code path, not the default one. A roadmap item written in the present tense
   is a false statement with a plan attached.

7. **Report per claim.** VERIFIED with the source, CUT with the reason, or REWRITTEN
   with the before and after. Give the file and line for each.

## What NOT to do

- **MUST NOT** invent, estimate, extrapolate, or round up a statistic, a user count,
  a star count, or a benchmark result. An estimate presented without its uncertainty
  is a fabrication.
- **MUST NOT** write a testimonial, a review, or a quote that no real person said,
  even as placeholder text, even marked as an example. Placeholder testimonials ship.
- **MUST NOT** name a real person or organisation as a user, customer, or adopter
  without their permission.
- **MUST NOT** use a company's logo, trademark, or brand colour in a way that implies
  partnership, certification, or endorsement.
- **MUST NOT** soften an unverifiable claim instead of cutting it. "Many", "most",
  "leading", "industry-standard" and "proven" are the usual disguises.
- **MUST NOT** move a required disclosure away from the link to protect the design.
- **MUST NOT** quietly edit a claim that turned out to be wrong after publication.
  Correct it visibly.

## Edge cases

- **The number is real but will go stale.** Star counts, user counts and version
  numbers drift. Either date the claim in the copy ("as of <date>") or drop it in
  favour of the mechanism. An undated stale number is an inaccurate one.
- **A benchmark you ran yourself.** Publish the method with the result: hardware,
  input size, version, and how many runs. A benchmark without its method is not
  verifiable by the reader, which puts it back in the cut pile.
- **Comparison with a competitor.** State the version compared and the date. A
  comparison against a version they have since fixed is a false claim as of today.
- **"Zero" claims.** "Zero dependencies", "no telemetry", "nothing leaves your
  machine" are excellent claims and are absolute, so one exception makes them false.
  Verify by reading the shipped files, not the intent.
- **The user asks for something the copy cannot support.** Say once that the claim is
  not verifiable and why, then offer the strongest version that IS supportable. Do not
  argue past that, and do not write the unsupported version anyway.
- **Screenshots and demo data.** A screenshot showing fabricated results is a
  fabricated statistic in image form. Label demo data as demo data inside the image.
