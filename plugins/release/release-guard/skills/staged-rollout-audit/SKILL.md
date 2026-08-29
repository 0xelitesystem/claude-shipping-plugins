---
name: staged-rollout-audit
description: Use when planning, reviewing, or debugging a staged rollout or canary release for an Electron app using electron-updater's stagingPercentage. Triggers on "staged rollout", "stagingPercentage", "canary percentage", "ramp the release to 25 percent", "why are the same users always on the beta", "why did nobody new get the update", or when a latest.yml with a stagingPercentage field is pasted.
---

# Staged Rollout Audit

electron-updater's staged rollout bucket is computed from a per-install id and
nothing else. The version being rolled out never enters the calculation. So two
consecutive releases at the same percentage select the IDENTICAL set of installs,
and every install outside that set is never a canary for anything.

## The formula, as shipped

From `AppUpdater.js`, `isStagingMatch()`, in the electron-updater on disk (check
the version in the project's own `node_modules` before quoting anything):

    stagingPercentage = parseInt(rawStagingPercentage, 10) / 100
    const stagingUserId = await this.stagingUserIdPromise.value
    const val = UUID.parse(stagingUserId).readUInt32BE(12)
    const percentage = val / 0xffffffff
    return percentage < stagingPercentage

`stagingUserId` comes from `getOrCreateStagingUserId()`, which reads a UUID from
`.updaterId` in the app's userData directory, and generates one from random bytes
the first time if the file is absent. It is written once and then reused forever.

Read the two inputs to that comparison: the install's own persisted UUID, and the
configured percentage. The version string, the release id, the channel, and the
date are all absent. `percentage` is a constant for the lifetime of an install.

If `stagingPercentage` is null the function returns `true` immediately, and if it
parses to NaN it logs a warning and returns `true`. Both mean everyone updates.

## What follows, structurally

These are invariants of the formula, not measurements. State them flatly.

- **The cohorts nest.** The 10 percent cohort is a strict subset of the 25 percent
  cohort, which is a strict subset of the 50 percent cohort. Same comparison,
  larger threshold.
- **Ramping only adds.** Moving the gate from 10 to 25 adds installs and drops
  exactly zero. That part is a genuine feature, and it is why the design looks
  right at first glance.
- **Consecutive releases reuse one cohort.** Ten releases in a row at 10 percent
  are canaried by one 10 percent slice of the fleet. Overlap between any two
  releases at the same percentage is exactly 1.0.
- **Coverage does not accumulate.** After ten 10 percent releases, the fraction of
  installs that have EVER received a canary is still about 10 percent. It does not
  climb toward the majority of the fleet the way independent draws would.

The consequence people care about: your canary population is a fixed minority who
absorb every regression you ship, while the other 90 percent give you no early
signal at all, ever. If that minority skews (early adopters, one geography, one
hardware profile), the skew is permanent.

## The fix, and what it changes

Salt the hash with a per-release value. Bucket on
`hash(installId + releaseIdentifier)` rather than on `installId` alone, where the
release identifier is the version string or a release id. This is the convention
in the mainstream flagging systems (LaunchDarkly, Unleash, Statsig all bucket on a
key that includes the flag or experiment identity).

What it changes:

- Release-to-release overlap drops from 1.0 to roughly the nominal rate. Two
  independent 10 percent cohorts overlap around 10 percent of the time by chance.
- Coverage accumulates. Across ten releases, a large majority of the fleet will
  have been a canary at least once.
- **The nesting property survives within a single release.** The release
  identifier is fixed for that release, so ramping 10 to 25 to 50 still only adds
  installs and still drops none. You lose nothing you were relying on.

What it costs: you cannot recognise "the canary group" as a stable population
across releases, because there no longer is one. That is the point, not a defect.

## The 0xffffffff edge case, and how to describe it

Dividing by `0xffffffff` instead of `0x100000000` means a maximal bucket yields
exactly 1.0, and `1.0 < 1.0` is false. An install whose UUID bytes are all `ff` at
that offset never receives even a 100 percent staged release.

It is a real off-by-one. It is also unreachable in practice: the probability is
about 2.3e-10 per install, so you need on the order of three billion installs
before there is a coin-flip chance that one user is affected. Say BOTH halves. An
audit that presents this as a live risk loses the reader who does the arithmetic,
and with it every other finding on the page.

## Two more findings from the same file, do not drop them

- **Setting a channel silently arms downgrades.** The `channel` setter ends with
  `this.allowDowngrade = true`, unconditionally. Switch a user to `beta` for any
  reason and you have also authorised the updater to move them backwards. If that
  is not what you want, set `allowDowngrade` explicitly AFTER setting `channel`.
- **You cannot pull a staged release by re-releasing it.** Users who already took
  the bad version are on it. Rolling back requires publishing a version number
  ABOVE the broken one. Re-uploading a fixed build under the same version leaves
  every updated install stuck, because the updater compares versions.

## Process

1. **Find where `stagingPercentage` is set.** It lives in the generated
   `latest.yml` / `latest-mac.yml`, usually written by hand or by a release script
   after electron-builder produces it. Confirm the value actually reached the
   published manifest, not just the local one.
2. **Confirm the electron-updater version** from the project's `node_modules` and
   read `isStagingMatch()` there. Do not quote line numbers you have not opened.
3. **Ask what the ramp plan is.** Percentages, and how many releases they cover.
   The finding only bites across releases, so a single one-off canary at 10 percent
   before a full rollout is a legitimate use with no problem to report.
4. **Report the cohort behaviour** for that plan: who is in it, whether it is the
   same population as last release, and what fraction of the fleet will never have
   seen a canary by the end of the plan.
5. **Offer the remedies in order:**
   - If they own the update feed, salt the bucket by shipping a custom
     `isUserWithinRollout` (the field exists on the updater) or by generating a
     per-release manifest.
   - If they cannot change the client, rotate the population manually: vary the
     percentage between releases so the boundary moves, and accept that this only
     shifts the edge rather than reshuffling the set.
   - If neither is possible, say so plainly and tell them to stop treating the
     canary result as fleet-representative.
6. **Check the two side findings** every time: an explicit `allowDowngrade` after
   any `channel` assignment, and a documented rollback procedure that increments
   the version.

## What NOT to do

- **MUST NOT** hardcode simulated cohort sizes or overlap percentages. Each
  install draws its own random UUID, so a simulated cohort at 10 percent is
  genuinely different every run. Report what the run in front of you produced, or
  report the structural facts with no numbers at all.
- **MUST NOT** present the `0xffffffff` off-by-one as a practical risk without the
  probability alongside it.
- **MUST NOT** claim the fleet "gradually gets covered" by repeated staged
  releases. It does not.
- **MUST NOT** describe the nesting property as the bug. Nesting is correct and
  useful. The bug is that nothing rotates between releases.
- **MUST NOT** invent a configuration option that does not exist in the installed
  version. Check the file.

## Calibration

A single canary release at one percentage, followed by a full rollout, has no
finding worth raising beyond the two side items. The finding is worth the user's
attention when they run staged rollouts as a standing practice, release after
release, and believe they are sampling their fleet.

## Related

Run the shipped formula against freshly generated install ids, unsalted and
salted side by side, and read the diff between the two implementations:
https://0xelitesystem.github.io/staged-rollout-cohort-simulator/
