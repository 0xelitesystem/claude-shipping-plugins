# claude-shipping-plugins

A Claude Code plugin marketplace for the parts of shipping software that fail
quietly. Every finding packaged here is one that leaves a green check mark: a
release workflow that publishes twice, a canary cohort that never rotates, a
certificate renewal date that walks away from your calendar, an updater that skips
signature verification, a repo that goes public missing the one file it needed.

Two plugins:

| Plugin | What it covers |
| --- | --- |
| `release-guard` | Desktop release pipelines. Publish matrices, update manifests, staged rollouts, Windows code signing timelines, updater signature verification. |
| `repo-compliance` | The pre-publish checks a linter will not do for you. Licence and third-party notices, WCAG AA contrast verified by math, honesty in published claims. |

## Install

Add the marketplace, then install the plugin you want:

```
/plugin marketplace add 0xelitesystem/claude-shipping-plugins
/plugin install release-guard@elitesystem-shipping
/plugin install repo-compliance@elitesystem-shipping
```

## A plugin is not a skill

Worth being precise about, because the two words get used interchangeably and they
are different layers.

**A skill** is one markdown file, `SKILL.md`, with YAML frontmatter and
instructions. Its `description` is the trigger: Claude reads it, decides the
current task matches, and loads the body. A skill is a unit of knowledge. It has no
opinion about when you invoke it and nothing to install beyond dropping the folder
in place.

**A plugin** is a package around skills. It adds:

- **slash commands** (`commands/*.md`), which are deliberate entry points. A skill
  fires when Claude judges it relevant; a command fires when you type it. Commands
  orchestrate: `/audit-release` runs four skills in a defined order and merges one
  report out of them.
- **a manifest** (`.claude-plugin/plugin.json`) carrying a name, a version, and an
  author, so the thing can be distributed, updated, and pinned.
- **optional hooks and MCP servers**, when a plugin genuinely needs to run
  something or reach a service. Neither plugin here does, so neither ships a
  `.mcp.json` or a `hooks/` directory.

**A marketplace** is the outermost layer: one `.claude-plugin/marketplace.json`
listing several plugins by relative path, so `/plugin marketplace add` on this repo
offers both at once.

So the stack is: knowledge lives in a skill, the workflow around that knowledge
lives in a command, distribution lives in a plugin, and a marketplace holds several
plugins.

## The layer below

If you want the skills without the packaging, these repos are the layer under this
one:

- [claude-skills-collection](https://github.com/0xelitesystem/claude-skills-collection),
  working skills you can drop into any project, one folder each.
- [claude-skills-templates](https://github.com/0xelitesystem/claude-skills-templates),
  starting points for writing your own.
- [claude-skills-validator](https://github.com/0xelitesystem/claude-skills-validator),
  checks a `SKILL.md` before you ship it.

Start there if you are learning the format. Come here when you want commands and
versioning on top.

## Where release-guard came from

This packages what came out of building 37 auto-updating Electron apps and then
reading the machinery underneath them: the publish workflows, the update
manifests, the staged rollout formula, and the updater's own verification path,
all read out of source rather than recalled. The findings are source reading and
arithmetic, not incident reports.

Being straight about the boundary of that: **none of those 37 apps is code signed
yet.** The signing material in this plugin is written from the position of someone
standing in front of the first certificate purchase, reading the CA/Browser Forum
requirement and the updater's verification code, and working out the calendar and
the rollout order before buying. It is arithmetic and source reading, verified. It
is not a war story about rotating a certificate under load, and it does not pretend
to be.

Where a claim in a skill has a number attached, the number was computed or read
from a file. Where it could not be, the skill states the mechanism without the
number and says so.

## Related tools

Browser versions of two of these audits, single file and no network calls, plus
the longer signing write-up:

- [electron-release-matrix-auditor](https://0xelitesystem.github.io/electron-release-matrix-auditor/),
  paste a workflow and see which legs collide.
- [staged-rollout-cohort-simulator](https://0xelitesystem.github.io/staged-rollout-cohort-simulator/),
  runs the shipped rollout formula against generated install ids, unsalted next to
  salted.
- [windows-code-signing-and-smartscreen-reference](https://github.com/0xelitesystem/windows-code-signing-and-smartscreen-reference),
  the longer plain-language write-up behind the signing skills.

## Privacy

These plugins are markdown. They contain no code, no telemetry, no analytics, and
no network calls. Nothing in this repo phones home, collects usage data, or
transmits anything about your projects anywhere. When Claude Code runs one of these
skills it reads your files locally, the same as any other prompt you write.

There is no MCP server in either plugin and no `.mcp.json`, so nothing here opens a
connection to any service on your behalf.

## Independence

This is an independent project. It is not affiliated with, endorsed by, or
sponsored by Anthropic, GitHub, Microsoft, or the CA/Browser Forum. Claude Code,
electron-builder, electron-updater, and the other products named in these skills
are named because they are what the skills analyse. Naming them implies no
relationship in either direction.

## Third-party notices

No code or content from another project is reproduced here. The plugin and
marketplace directory layout follows the structure documented for Claude Code
plugins; that is a file layout, not copyrightable content, and none of the text in
this repo is taken from it.

Where a skill describes the behaviour of a third-party library, that behaviour was
read from source or measured on a running copy, and it is described in this repo's
own words. Short identifier-level excerpts appear only where naming the function is
the point.

The CA/Browser Forum Code Signing Baseline Requirements are quoted in one place,
attributed inline, as a normative requirement under discussion.

## Contributing

Findings only, with a receipt. A pull request that adds a claim needs the file, the
function, and how you verified it. A claim that cannot be checked at review time
gets cut rather than softened.

## License

MIT. Copyright (c) 2026 0xelitesystem.
