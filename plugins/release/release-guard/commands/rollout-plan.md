---
description: Plan a staged rollout for an Electron app and show which installs the cohort will actually select.
argument-hint: "[percentage]"
---

# Rollout Plan

Turn a percentage into a plan, and say out loud what that percentage will and will
not sample. The default assumption behind staged rollouts, that a 10 percent
canary gives you a representative early read on the fleet, is not true of
electron-updater's implementation, so the plan has to be built with that in the
open.

Argument: the percentage to start at, for example `10`. If none is given, ask what
the intended ramp is before planning anything.

## Workflow

### Step 1: Read the implementation this project actually has

Do not plan against a remembered formula. Find electron-updater in
`node_modules`, note its version, and read `isStagingMatch()` in `AppUpdater.js`.
If the project ships a custom `isUserWithinRollout`, read that instead, and say so,
because the rest of this command's assumptions may not hold.

Also find where `stagingPercentage` is written into the published manifest.
Confirm the value reaches the published `latest.yml`, not just a local copy.

### Step 2: Explain the cohort

Use `skill: "staged-rollout-audit"` to:

- state which inputs the bucket is computed from, and which are absent
- confirm whether the cohort rotates between releases at the requested percentage
- report the nesting behaviour, so the user knows a ramp only adds installs
- flag the `0xffffffff` off-by-one with its probability attached, or omit it

### Step 3: Build the ramp

Produce a concrete schedule. For each stage, give the percentage, the minimum
soak time before the next stage, and the specific signal that would stop the ramp.
A stage with no stop condition is not a stage, it is a delay.

```
STAGE   PCT   SOAK      STOP IF
  1      10   48h       crash-free sessions below <baseline>
  2      25   48h       ...
  3      50   24h       ...
  4     100   -         -
```

Percentages and soak times are the user's call. Push back once if a stage has no
measurable stop condition, then record what they chose.

### Step 4: Say who is being sampled

This is the step that makes the command worth running. State plainly:

- the same installs will be stage 1 for this release as were stage 1 for the last
  release at the same percentage
- the fraction of the fleet that will never be a canary under this plan
- if the canary population skews in any way the user knows about (early adopters,
  one platform, internal machines), that skew is permanent, not per release

If they want rotation, describe salting the bucket with the release identifier and
what that changes, including that ramping within a single release still only adds
installs.

### Step 5: Write the abort procedure

Every rollout plan needs the rollback written before the rollout starts. Cover:

- rolling the percentage back to 0 stops NEW installs from taking the update and
  does nothing for installs that already took it
- recovering those installs requires publishing a version number ABOVE the broken
  one; re-releasing the same version leaves them stuck
- if a channel is being switched as part of this, `allowDowngrade` is set to true
  as a side effect and must be set back explicitly if that is not intended

### Step 6: Hand back the plan

The schedule table, the sampling caveat, and the abort procedure, in that order.
Do not edit the manifest or the config as part of this command.

## Related

Watch the cohort selection run against generated install ids, unsalted next to
salted: https://0xelitesystem.github.io/staged-rollout-cohort-simulator/
