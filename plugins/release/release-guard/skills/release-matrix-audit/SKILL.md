---
name: release-matrix-audit
description: Use when auditing a GitHub Actions workflow that builds and publishes a desktop app from a matrix, especially an Electron app built with electron-builder. Triggers on "why do I have two draft releases", "duplicate release for one tag", "latest.yml keeps getting overwritten", "review my release workflow", "will this matrix step on itself", or when the user pastes a workflow yml containing a publish step.
---

# Release Matrix Audit

Two matrix legs publishing one tag can each create a release, and can each
overwrite the other's update manifest. Both legs exit 0. Nothing in the run
summary turns red. This skill finds those cases before the tag is cut.

## The two findings, and which one is publisher specific

### Finding A, duplicate draft releases. PUBLISHER SPECIFIC.

electron-builder with `--publish always` is a read-then-create with no lock. In
`gitHubPublisher.ts`, `getOrCreateRelease()` lists `/repos/OWNER/REPO/releases`
and scans the result for an existing draft. The source carries its own
explanation of why it lists instead of asking for the tag:

    // we don't use "Get a release by tag name" because "tag name" means existing
    // git tag, but we draft release and don't create git tag

If no draft is found and `publish` is `always` (or a CI tag is present), it calls
`createRelease()`, which is a bare POST to `/releases`.

State the mechanism precisely. GitHub permits several DRAFT releases carrying the
same `tag_name`, because a draft has no git tag behind it yet. So two legs that
both list before either creates do NOT collide with a 422. They both succeed. You
get two drafts for one tag, and no error anywhere.

Do not write that release creation "fails with an unhandled 422". It does not
fail. The file does define `doesErrorMeanAlreadyExists()`, but that helper is
called from exactly one place, inside the ASSET UPLOAD path. Release creation has
no already-exists handling at all, and does not need any to produce this bug.

### Finding B, colliding update manifests. PUBLISHER INDEPENDENT.

Derive manifest names from the source rather than hardcoding them.
`updateInfoBuilder.ts`:

    function getUpdateInfoFileName(channel, packager, arch) {
      const osSuffix = packager.platform === Platform.WINDOWS ? "" : `-${packager.platform.buildConfigurationKey}`
      return `${channel}${osSuffix}${getArchPrefixForUpdateFile(arch, packager)}.yml`
    }

    function getArchPrefixForUpdateFile(arch, packager) {
      if (arch == null || arch === Arch.x64 || packager.platform !== Platform.LINUX) {
        return ""
      }
      return arch === Arch.armv7l ? "-arm" : `-${Arch[arch]}`
    }

Read those two functions together and the collisions fall out. The arch prefix is
empty for every platform that is not Linux. The os suffix is empty for Windows.
So:

- `latest.yml` is produced by BOTH `win/x64` and `win/arm64`.
- `latest-mac.yml` is produced by EVERY mac arch (x64, arm64, universal).
- Linux is the only platform whose non-x64 legs get a distinct file name.

Lead with the Windows case when both apply. The mac case is the one people have
written about. The Windows one is not, and windows-11-arm runners are generally
available now, so arch-split Windows matrices are new and growing.

On collision, `gitHubPublisher.ts` `overwriteArtifact()` does exactly this:

    log.warn({ file: fileName, reason: "already exists on GitHub" }, "overwrite published file")
    ... GET assets, then DELETE the matching asset by id

Then it re-uploads. One warn line in a log nobody reads. Whichever leg finishes
last owns `latest.yml`, and every installed client on the other architecture is
handed that manifest.

### Publisher awareness is mandatory

Do NOT flag `softprops/action-gh-release` for Finding A. That action checks for a
422 carrying a validation error code of `already_exists`, and recovers by finding
the release from the tag listing. It is race hardened for creation. It does still
delete a colliding asset before re-uploading, so Finding B applies to it.

Without this distinction the audit false-positives on one of the most common
Electron release patterns, and a tool that cries wolf on the normal case gets
uninstalled.

## The second order trap, once two drafts exist

`gh release view` cannot show you the problem. In cli/cli, `FetchRelease` starts
two goroutines, one looking up the published release and one looking up the
draft, and returns whichever answers first:

    first := <-results
    if first.error == nil { cancel(); <-results; return first.release, nil }

That is non-deterministic, not merely ambiguous. And the draft lookup asks GraphQL
for one release by tag, so the second draft is invisible to `gh` entirely. Tell
the user to check the web UI or the REST listing, never the CLI.

## Process

1. **Read the workflow.** Locate every step that publishes: electron-builder
   invocations carrying `--publish` or `-p`, `softprops/action-gh-release`,
   `ncipollo/release-action`, `gh release create`, or a `publish` block in the
   electron-builder config that a build step will honour.

2. **Identify the publisher for each publish step.** Report Finding A only for
   electron-builder's own publisher. Name the publisher you matched and say why.

3. **Expand the matrix.** Handle the cartesian product, `include:` additions,
   `exclude:` removals, and interpolation of matrix values into `runs-on` and into
   the run command. If the matrix comes from a previous job's output (a `fromJSON`
   over a job output, a dynamically generated list), BAIL EXPLICITLY: say the
   matrix is dynamic, name the expression, and stop. Do not guess at its contents.

4. **Map each leg to (platform, arch).** Prefer an explicit `--x64`, `--arm64`, or
   `--universal` flag on the build command. Fall back to a `runs-on` lookup only
   when no arch flag is present, and stamp that lookup with the date you last
   verified it, because runner images change. Ask the user rather than assume when
   a self-hosted label appears.

5. **Count possible drafts.** For an electron-builder publish, that is the number
   of legs running the publish step for the same tag, unless they are serialised:
   a `needs:` chain, `max-parallel: 1`, or a separate single publish job that all
   the build legs feed.

6. **Compute manifest names** with the derivation above, one per (channel,
   platform, arch). Group by file name. Any group holding more than one leg is a
   collision.

7. **Report** in this shape:

```
PUBLISH STEPS
  build    electron-builder --publish always    publisher: electron-builder
  release  softprops/action-gh-release          publisher: softprops [race hardened]

MATRIX
  windows-latest   win/x64
  macos-latest     mac/arm64

DRAFT RACE
  2 legs can each create a draft for one tag. GitHub accepts both.

MANIFEST COLLISIONS
  latest.yml       win/x64      ok
  latest-mac.yml   mac/arm64    ok
  no shared manifest.
```

   When a workflow is clean, say so in one line and stop:
   `2 legs, 2 platforms, no shared manifest - clean.`

## The fixes, in the order you should offer them

1. **Build on the matrix, publish from one job.** Legs upload artifacts, one
   dependent job downloads them all and publishes once. This removes both findings
   together and is the only fix that scales past two platforms.
2. **Set `--publish never` on all but one leg** and let the designated leg
   publish. Cheap, but the manifest collision survives if two legs of the same
   platform still produce the same file name.
3. **Merge the manifests before upload** when an arch split on one platform is
   unavoidable. electron-builder will not do this for you.
4. **A `concurrency:` group on the workflow** narrows the window between two runs
   of the same tag. It does nothing about two legs inside one run. Offer it as a
   supplement, never as the fix.

## What NOT to do

- **MUST NOT** flag `softprops/action-gh-release` for the draft race. It handles
  422 `already_exists` and recovers from the tag listing.
- **MUST NOT** describe the draft race as a 422, or as a failure of any kind. It
  succeeds twice. That is the whole problem.
- **MUST NOT** hardcode the manifest file names. Derive them from the two
  functions above so the audit stays correct when a platform is added.
- **MUST NOT** invent issue numbers, comment counts, or dates. If you reference
  the long-standing upstream report, describe it honestly: it was closed by the
  reporter two days after filing and is still being referenced years later. Do not
  present it as an open issue, and do not attach a count you have not checked
  against the live API.
- **MUST NOT** manufacture a finding on a clean workflow. Print the clean line.
- **MUST NOT** guess at a dynamic matrix. Bail and name the expression.

## Calibration

A two leg workflow (`windows-latest`, `macos-latest`, one leg per OS) running
electron-builder with `--publish always` has exactly one finding: two possible
drafts, zero manifest collisions. That is the honest and common result. If you
are reporting four findings on a two leg workflow, you are padding.

## Related

The browser version of this audit, paste a workflow and read the same report:
https://0xelitesystem.github.io/electron-release-matrix-auditor/
