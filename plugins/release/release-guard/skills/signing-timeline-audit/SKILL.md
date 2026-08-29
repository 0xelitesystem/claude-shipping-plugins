---
name: signing-timeline-audit
description: Use when planning or reviewing the calendar and the rollout order for a Windows code signing certificate. Triggers on "when does my signing cert expire", "code signing renewal reminder", "460 day certificate", "CSBR validity period", "we are switching signing certificates", "changing our publisher name", or when a user asks how to rotate a certificate without breaking auto-updates.
---

# Signing Timeline Audit

Two things about Windows code signing are on a clock, and both of them are set up
to be missed. The certificate's maximum life is no longer a round number of years,
so an annual reminder is wrong every single year. And a new signing identity only
protects installs made AFTER you ship it, because the updater checks against a
publisher list that is frozen inside the copy of the app already on disk.

This skill is Windows only. Apple Developer ID runs on Apple's own PKI and is
outside the scope of the requirements quoted here.

## Part 1: the validity cap, and the drift it creates

CA/Browser Forum Code Signing Baseline Requirements, section 6.3.2, quoted:

> "For Code Signing Certificates issued before March 1st, 2026, the validity
> period MUST NOT exceed 39 months. For Code Signing Certificates issued on or
> after March 1st, 2026, the validity period MUST NOT exceed 460 days."

460 days is about 15.1 months. It is 95 days more than a year, roughly 26 percent
longer. (For scale, the previous 39 month cap is about 1187 days, near 2.6 times
the new one.)

The consequence is arithmetic, not opinion. Each renewal falls 460 days after the
last, which is 95 days LATER in the calendar than an annual anniversary. So the
month advances by roughly one quarter each cycle, and drifts about two weeks later
within the month each time round. Worked from the earliest possible issue date:

    issued   2026-03-01
    cycle 1  2027-06-04   Jun
    cycle 2  2028-09-06   Sep
    cycle 3  2029-12-10   Dec
    cycle 4  2031-03-15   Mar    back to March, at cycle 4
    cycle 5  2032-06-17   Jun
    cycle 8  2036-03-28   Mar

It returns to the starting MONTH at cycle 4, and it never returns to the starting
DAY. Across two dozen cycles it does not land within three days of the original
date. So:

- **An annual calendar reminder is wrong every year**, by a growing margin.
- **A quarterly mental model is nearly right and still drifts** about a fortnight
  per cycle, which is exactly the kind of error that is fine for three cycles and
  then is not.
- The only correct approach is a date computed from the actual issue date, one
  cycle at a time, re-derived after every renewal.

Renewal is not instant either. Budget for identity revalidation and for hardware
token or cloud HSM provisioning. Set the working deadline before the expiry, not
on it, and put the reminder on the computed date minus your lead time.

## Part 2: the rollout order, which is the part that actually breaks users

When you sign under a new subject (a new organisation name, a new CA, a renewed
certificate with a changed DN), the installers you publish carry the new signer.
Every already-installed copy of your app validates downloaded installers against
the publisher list that was baked into it at ITS build time. That list lives in
`app-update.yml` inside the installed app's resources directory, and the updater
reads it from disk, not from the network.

So an old install has never heard of your new signing subject and will refuse the
update it just downloaded, with `ERR_UPDATER_INVALID_SIGNATURE`. Nothing about the
new certificate being valid helps: the old app is not asking whether the signature
is valid, it is asking whether the subject is on ITS list.

**The order that works:**

1. Ship a release, signed with the CURRENT certificate, whose config lists BOTH
   the current publisher subject and the incoming one. `publisherName` accepts an
   array, and the verifier tries each entry.
2. Wait for that release to propagate. How long depends on your update cadence and
   how often your users open the app. Weeks, not days.
3. Only then switch the signing identity. Installs that took step 1 accept the new
   signer. Installs that did not, will not.
4. Keep the old subject in the list for at least one more cycle after the switch.
   Removing it early re-creates the same break in the opposite direction.

Installs that were never updated during step 1 are stuck on their old version.
This is recoverable, but not from inside the app: the user downloads the new
installer manually and the NSIS installer upgrades in place. That means recovery
needs out-of-band contact, a mailing list or a website notice, so plan the
messaging before you switch, not after.

## Part 3: reputation, stated accurately

Microsoft's position is that changing your certificate AFFECTS the publisher trust
signal. It does not carry over. Do not write that reputation is "reset" or
"destroyed", and do not attach a number of downloads or days to its recovery.

Also worth saying, because it surprises people: file-hash reputation restarts at
zero for EVERY new binary, regardless of which certificate signed it. That is not
a rotation penalty, it is how it always works, and it is why the first hours of
every release look different from the rest.

## Process

1. **Get the issue date**, not the expiry date. If only an expiry is known, ask for
   the issue date, because the cap depends on which side of 2026-03-01 the
   certificate was issued on.
2. **Apply the right cap.** Issued before 2026-03-01: up to 39 months. Issued on or
   after: up to 460 days. The cap is a maximum, so confirm the actual validity
   period on the certificate rather than assuming the CA sold the maximum.
3. **Compute the next 4 to 6 renewal dates** by repeated addition of the actual
   validity period. Show the month for each so the drift is visible.
4. **Subtract the lead time** and give the user a single concrete reminder date,
   with the explicit note that it must be recomputed after each renewal.
5. **Ask whether the subject is changing** at the next renewal. A same-subject
   renewal from the same CA usually keeps the DN stable, in which case Part 2 does
   not apply. A CA change, an organisation rename, or a move to a different legal
   entity all change the DN.
6. **If the subject is changing, walk the four step order in Part 2** and identify
   the specific release that must carry both subjects. Name the version number.
7. **Cross-check against `updater-signature-check`** to confirm the app is actually
   verifying anything at all. If `publisherName` is unset, none of Part 2 applies,
   and the finding to report is the missing verification instead.

## What NOT to do

- **MUST NOT** write that renewal walks EARLIER. It walks later. 460 days is more
  than a year, not less.
- **MUST NOT** claim it takes many cycles to revisit a month. It returns to the
  starting month at cycle 4.
- **MUST NOT** quote a day count for 39 months other than approximately 1187.
- **MUST NOT** include certificate prices, CA product names, or vendor comparisons.
  Price tables rot faster than anything else on the page, and being wrong about
  money costs you the reader's trust in the arithmetic.
- **MUST NOT** say a certificate change "permanently strands" users. It silently
  breaks in-app updates with no in-app recovery. Manual re-download plus the
  in-place installer upgrade works.
- **MUST NOT** say publisher reputation is "reset". It does not carry over.
- **MUST NOT** apply any of this to macOS. Different PKI, different rules.

## Calibration

For a first certificate purchase there is exactly one deliverable: the computed
renewal date, with the drift explained so the user does not put an annual reminder
in the calendar. Part 2 is not relevant yet and should be flagged as something to
read before the SECOND certificate, not walked through now.

## Related

Longer plain-language reference on Windows signing, SmartScreen, and what changes
when the certificate does:
https://github.com/0xelitesystem/windows-code-signing-and-smartscreen-reference
