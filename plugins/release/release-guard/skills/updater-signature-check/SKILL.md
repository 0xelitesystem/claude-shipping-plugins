---
name: updater-signature-check
description: Use when checking whether an Electron app's Windows auto-updater actually verifies the installer it downloaded. Triggers on "does electron-updater verify signatures", "publisherName", "ERR_UPDATER_INVALID_SIGNATURE", "is my auto-update secure", "can someone serve a malicious installer to my updater", or when reviewing an electron-builder win config before shipping updates.
---

# Updater Signature Check

On Windows, electron-updater verifies a downloaded installer by shelling out to
PowerShell, reading the Authenticode signature, and comparing the signer
certificate's subject against a `publisherName` value read from `app-update.yml`
inside the installed app. If `publisherName` is null, verification is skipped and
the downloaded installer runs.

That is the finding worth leading with. For an unsigned build the entire
verification path is dead code, and the updater will execute whatever the update
feed hands it. The check does not exist until you sign.

## The mechanism, end to end

Read it from the electron-updater in the project's own `node_modules` and confirm
anything you quote. The behaviour below was read out of an installed copy; version
numbers and line numbers change, the shape has been stable.

**Step 1, `NsisUpdater.verifySignature(tempUpdateFile)`.**

    let publisherName = (await this.configOnDisk.value).publisherName
    if (publisherName == null) { return null }
    ... on ENOENT (no app-update.yml) { return null }
    return await this._verifyUpdateCodeSignature(Array.isArray(publisherName) ? publisherName : [publisherName], tempUpdateFile)

Note the return convention, which is easy to read backwards: `null` means NO
PROBLEM. A non-null string is the failure description. The caller does:

    const signatureVerificationStatus = await this.verifySignature(destinationFile)
    if (signatureVerificationStatus != null) {
      throw newError(`... is not signed by the application owner: ...`, "ERR_UPDATER_INVALID_SIGNATURE")
    }

So both early returns above, null `publisherName` and missing `app-update.yml`,
report success.

`configOnDisk` resolves to `app-update.yml` in the installed app's resources
directory (a `dev-app-update.yml` beside the app path in dev mode). It is a file on
the user's disk, written when THAT build was packaged. This is why the value is
frozen at install time, and why adding a new signing subject cannot help installs
that already exist. See `signing-timeline-audit` for the rollout order that fixes
this.

**Step 2, `windowsExecutableCodeSignatureVerifier.verifySignature()`.**

It spawns PowerShell with the module path cleared and the code page forced, with a
20 second timeout:

    set "PSModulePath=" & chcp 65001 >NUL & powershell.exe
      -NoProfile -NonInteractive -InputFormat None -Command
      "Get-AuthenticodeSignature -LiteralPath '<file>' | ConvertTo-Json -Compress"

Single quotes in the path are doubled before interpolation, which is what stops a
crafted filename from injecting a second PowerShell command.

Then:

- `data.Status === 0` is the valid case. Anything else falls through to failure.
- `data.Path` is normalised and compared to the normalised temp file path. A
  mismatch is treated as an error. If `data.Path` is missing, it logs a warning and
  SKIPS this check rather than failing.
- The signer subject is parsed as a distinguished name. For each configured
  publisher entry: if the entry itself parses as a DN with fields, every field must
  match the subject. Otherwise it falls back to comparing the string against the
  subject's `CN` alone, and logs:
  `Signature validated using only CN <name>. Please add your full Distinguished Name (DN) to publisherNames configuration`

That CN-only fallback is the second finding. A certificate is accepted if its
common name string matches yours, with no check on organisation, country, or
issuer. Configure the full DN.

## The fail-open paths, all of them

Every one of these resolves as "verified" and lets the install proceed:

1. **`publisherName` is null.** Unsigned builds, and signed builds where the config
   never set it.
2. **`app-update.yml` is absent** (ENOENT).
3. **Windows 6.x below 6.3**, an unsupported PowerShell version: logs
   `Ignoring signature validation due to unsupported powershell version` and skips.
4. **The `ConvertTo-Json` probe fails.** When PowerShell errors, the handler first
   tests whether PowerShell can run `ConvertTo-Json test` at all. If that test also
   fails, it warns and skips verification instead of failing.

For contrast, the path that DOES fail closed: PowerShell is present and working
but errors or writes to stderr on the actual file. That rejects.

The design intent behind 3 and 4 is compatibility, not carelessness, but the
practical effect is the same. Anywhere the check cannot run, it is not run and the
update proceeds.

## Process

1. **Locate `publisherName`** in the electron-builder config (`win.publisherName`
   in `package.json`, `electron-builder.yml`, or `electron-builder.json5`). Report
   it as unset if you cannot find it. Do not assume it is derived.
2. **Check what actually shipped.** If a packaged build is available, read
   `app-update.yml` from its resources directory. The config is what you intended;
   `app-update.yml` is what the user's machine will read.
3. **Determine whether the app is signed at all.** Look for a certificate subject,
   a signing hook, a `certificateFile` / `certificateSubjectName` /
   `certificateSha1`, or an azure-trusted-signing / signtool step in CI. If nothing
   signs the Windows build, report the fail-open as the top finding and stop
   pretending there is a verification story.
4. **Check the DN.** If `publisherName` is a bare common name rather than a full
   distinguished name, flag the CN-only fallback and quote the warning the updater
   will emit.
5. **Check for a `verifyUpdateCodeSignature` override.** The updater exposes this
   hook so a native verifier can replace the PowerShell path. If one is set, the
   analysis above does not apply and you must read the replacement instead.
6. **Confirm the update feed is HTTPS** and that the manifest and the installer come
   from the same origin you control. Signature verification is the last line, not
   the first.
7. **Report** in this shape:

```
SIGNING           unsigned (no certificate configured for win)
publisherName     unset
VERIFICATION      SKIPPED. verifySignature returns null when publisherName is null,
                  and null means "no problem" to the caller.
IMPACT            any installer the update feed serves will be executed.
```

## What NOT to do

- **MUST NOT** claim a particular electron-updater version made this fail closed.
  That could not be corroborated. Read the version on disk and describe what it
  does.
- **MUST NOT** cite line numbers you have not opened in the version the project
  actually depends on.
- **MUST NOT** describe the PowerShell interpolation as an injection vulnerability.
  Quotes are escaped and the path is already on disk, so it cannot contain the
  characters Windows forbids in filenames. Describe the escaping as the mitigation
  it is.
- **MUST NOT** tell the user that signing alone secures updates. It closes this
  gap. Feed integrity, HTTPS, and manifest provenance are separate.
- **MUST NOT** recommend disabling verification to get past
  `ERR_UPDATER_INVALID_SIGNATURE`. That error is almost always a publisher subject
  mismatch after a certificate change, and the fix is the rollout order in
  `signing-timeline-audit`.

## Calibration

For an unsigned app there is exactly one finding, and it is severe: no
verification, at all, silently. Everything else on this page is a follow-up for
after the first certificate. For a signed app with a full DN configured and no
override, the honest report is one line saying verification is active, plus the
fail-open list as context, not as findings.

## Related

What changes when the certificate changes, and the order to ship it in:
https://github.com/0xelitesystem/windows-code-signing-and-smartscreen-reference
