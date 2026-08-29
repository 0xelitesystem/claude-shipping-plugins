# release-guard

The ways a desktop release ships broken while every job exits zero.

Nothing here is caught by a red build. A publish matrix creates two draft releases
for one tag and both API calls succeed. Two architectures write the same update
manifest and the second one deletes the first behind a single warn line. A staged
rollout picks a cohort that never changes, so the same slice of the fleet absorbs
every regression and the rest are never sampled. A signing certificate's renewal
date walks away from the calendar because the validity cap is no longer a round
number of years. And an updater whose `publisherName` is unset reports every
downloaded installer as verified, because the function that verifies signatures
returns `null` for "no problem" and `null` is exactly what it returns when there is
nothing configured to check against.

Every claim in these skills was read out of source or computed, and the skills say
which file and which function.

## Commands

| Command | Argument | What it does |
| --- | --- | --- |
| `/audit-release` | path to workflow yml | Full sweep: publish matrix, rollout, signature verification, signing calendar |
| `/rollout-plan` | starting percentage | Builds a ramp with stop conditions, and says who the canary actually samples |
| `/cert-timeline` | certificate issue date | Computes real renewal dates, and the release order for a subject change |

## Skills

- **release-matrix-audit**: expands a GitHub Actions matrix, counts how many legs
  can each create a release for one tag, and derives update manifest names from
  electron-builder's own naming functions rather than hardcoding them. Publisher
  aware, so it does not false-positive on race-hardened actions.
- **staged-rollout-audit**: reads the shipped `isStagingMatch()` bucket formula and
  explains what it selects. The cohorts nest, ramping only adds installs, and
  nothing rotates between releases because the version never enters the
  calculation.
- **signing-timeline-audit**: the CA/Browser Forum validity cap and the drift it
  creates, plus the release ordering that lets an already-installed app accept a
  new signing subject. Windows only.
- **updater-signature-check**: whether the Windows updater verifies anything at
  all, the four paths on which verification is skipped and reported as success, and
  the CN-only fallback that accepts a certificate on its common name alone.

## Scope

Windows and Electron. macOS signing is Apple's own PKI and is deliberately out of
scope rather than guessed at. No certificate prices and no CA product names: price
tables rot faster than anything else on a page.

## Privacy

These skills and commands are plain Markdown instructions. They run inside your own
Claude Code session, read only the files you point them at, and send nothing
anywhere. There is no telemetry, no analytics, no network call, and no bundled MCP
server.

## Independence

This is an independent project. It is not affiliated with, endorsed by, or sponsored
by Anthropic, GitHub, Microsoft, or the CA/Browser Forum. Claude Code,
electron-builder, and electron-updater are named here because they are what these
skills analyse.

## License

MIT. See the LICENSE file at the root of this repository.
