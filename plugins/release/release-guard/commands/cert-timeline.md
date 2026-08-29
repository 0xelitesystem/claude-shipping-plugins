---
description: Compute Windows code signing renewal dates from an issue date and produce the release order for a subject change.
argument-hint: "[issue date]"
---

# Cert Timeline

Work out when a Windows code signing certificate actually needs renewing, and what
has to ship before the signing identity changes. Under the current validity cap the
renewal date walks through the calendar, so the answer is a computed date rather
than an anniversary.

Argument: the certificate's issue date, for example `2026-03-01`. Ask for it if it
is missing. An expiry date alone is not enough, because the applicable cap depends
on which side of 2026-03-01 the certificate was issued on.

## Workflow

### Step 1: Collect the inputs

- issue date
- actual validity period on the certificate, if known. The cap is a maximum and
  CAs do not always sell the maximum, so ask rather than assume.
- whether the subject (organisation name, distinguished name, CA) will change at
  the next renewal
- the app's update cadence, and roughly how often users open it. This determines
  how long a two-subject release needs to propagate.

### Step 2: Compute the schedule

Use `skill: "signing-timeline-audit"` to:

- select the correct cap for that issue date and quote the requirement
- compute the next four to six renewal dates by repeated addition
- show the month alongside each date so the drift is visible
- subtract the user's lead time and produce one concrete reminder date

Output:

```
issued        2026-03-01
validity      460 days (CSBR 6.3.2 cap for certificates issued on or after 2026-03-01)

  cycle 1     2027-06-04   Jun
  cycle 2     2028-09-06   Sep
  cycle 3     2029-12-10   Dec
  cycle 4     2031-03-15   Mar

REMINDER      <cycle 1 date minus lead time>
              Recompute after every renewal. An annual reminder is wrong every year.
```

### Step 3: Handle a subject change, if there is one

Only if the distinguished name will change. Use `skill: "signing-timeline-audit"`
for the ordering and `skill: "updater-signature-check"` to confirm what the app
verifies today. Produce:

- the specific version number that must ship listing BOTH the old and the new
  publisher subject, signed with the current certificate
- the propagation window before the signing switch, derived from the cadence given
  in Step 1
- the version at which the old subject can finally be dropped, which is at least
  one cycle after the switch
- the out-of-band message for installs that miss the window, since their recovery
  is a manual download and an in-place installer upgrade, not an in-app update

### Step 4: Sanity check before the first purchase

If this is the first certificate, the deliverable is Step 2 only. Say that the
subject-change ordering is something to read before the SECOND certificate, and do
not walk the user through it now.

Also confirm from `skill: "updater-signature-check"` whether the app currently
verifies anything. An unsigned app with no `publisherName` skips verification
entirely, and that is the finding that outranks the calendar.

## What this command will not do

- No certificate prices, CA product names, or vendor recommendations.
- No macOS. Apple Developer ID is a different PKI with different rules.
- No claim about how long publisher reputation takes to recover. It does not carry
  over across a certificate change, and that is all that can honestly be said.

## Related

https://github.com/0xelitesystem/windows-code-signing-and-smartscreen-reference
