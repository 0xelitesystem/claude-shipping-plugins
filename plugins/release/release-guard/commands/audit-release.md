---
description: Sweep a desktop release setup for the failures that exit zero, from publish workflow to updater signature verification.
argument-hint: "[path to workflow yml]"
---

# Audit Release

Full sweep of a desktop release pipeline. Every finding this command hunts for is
one that leaves a green check mark: duplicate draft releases, an overwritten
update manifest, a canary cohort that never rotates, a certificate whose renewal
date drifts away from the calendar, and an updater that skips signature
verification entirely.

Argument: a path to a GitHub Actions workflow file. If none is given, look for
`.github/workflows/*.yml` and ask which one publishes before proceeding.

## Workflow

### Step 1: Establish what is being shipped

Read the project before analysing it. You need:

- the workflow file, and every publish step inside it
- the electron-builder config (`package.json` `build` block,
  `electron-builder.yml`, or `electron-builder.json5`)
- the electron-updater version from `node_modules`, if present, because the
  mechanisms differ between versions and you must quote the one installed here

If this is not an Electron project, say so and stop rather than mapping the
findings onto a stack they do not apply to.

### Step 2: Audit the publish matrix

Use `skill: "release-matrix-audit"` to:

- identify each publish step and its publisher, then apply publisher aware rules
- expand the matrix and map each leg to a (platform, arch) pair
- count how many legs can independently create a release for one tag
- derive the update manifest file name for every leg and group by name

Report the clean line if it is clean. Do not manufacture a finding.

### Step 3: Audit the rollout policy

Use `skill: "staged-rollout-audit"` to:

- find whether `stagingPercentage` is used, and at what values
- explain which installs the cohort selects and whether it rotates between
  releases
- check the two side findings: an implicit `allowDowngrade` after any `channel`
  assignment, and whether a rollback procedure that increments the version exists

If staged rollouts are not in use, say so in one line and skip to the next step.

### Step 4: Audit signature verification

Use `skill: "updater-signature-check"` to:

- locate `publisherName` in the config, and in a packaged `app-update.yml` if one
  is available
- determine whether the Windows build is signed at all
- flag a bare common name where a full distinguished name belongs
- report the fail-open paths that apply to this project

### Step 5: Audit the signing calendar

Use `skill: "signing-timeline-audit"` to:

- compute the renewal date from the certificate's actual issue date under the cap
  that applies to it
- state plainly that an annual reminder will be wrong
- if a subject change is planned, produce the release ordering that keeps existing
  installs updatable

Skip this step entirely if the project is unsigned, and say why: there is no
timeline to audit before the first certificate, and the finding from Step 4 is the
one that matters.

### Step 6: Report

One report, ordered by what breaks users soonest, not by which step found it.

```
RELEASE AUDIT   <repo> @ <workflow path>

BLOCKING
  ...

WORTH FIXING BEFORE THE NEXT TAG
  ...

CONTEXT, NOT FINDINGS
  ...
```

Every finding names the file and the mechanism. Every fix is concrete enough to
apply without a follow-up question. If a section is empty, print it empty rather
than filling it.

### Step 7: Stop

Do not apply fixes unless the user asks. This command reports; the user decides
what ships.
