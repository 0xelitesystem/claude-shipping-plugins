# repo-compliance

The checks that must pass before a repo goes public, and the ones a linter will not
do for you.

A linter catches syntax. It does not catch a missing LICENSE, a privacy claim that
the code contradicts, a copyright holder written from memory, a colour token that
sits at 4.44 because nobody ever painted it, or a statistic that was never true.
Those are the ones that reach users.

## Commands

| Command | Argument | What it does |
| --- | --- | --- |
| `/ship-check` | path to repo | Runs the whole gate and returns SHIP or BLOCKED |
| `/notices` | project you borrowed from | Builds or verifies the Third-party notices section |
| `/contrast` | path to html or css | Audits the palette for WCAG AA by math, both themes |

## Skills

- **compliance-gate**: licence present and correct, privacy claim stated and
  actually true, no copyleft vendored, no secret-shaped strings, cross-links that
  resolve. Each item explains why it is a blocker rather than a nit.
- **third-party-notices**: what to write when you port, adapt, or quote at length,
  and why the copyright holder is fetched from the upstream LICENSE rather than
  recalled.
- **contrast-audit**: WCAG AA verified as a table of numbers, every text token
  against every background token, in both themes. Explains why checking a rendered
  page misses the semantic colours.
- **honesty-audit**: no fabricated statistics, testimonials, or endorsements;
  disclosure next to the link; independence stated where a third party is named; and
  the rule that an unverifiable claim gets cut, not softened.

## Privacy

These skills and commands are plain Markdown instructions. They run inside your own
Claude Code session, read only the files you point them at, and send nothing
anywhere. There is no telemetry, no analytics, no network call, and no bundled MCP
server.

## Independence

This is an independent project. It is not affiliated with, endorsed by, or sponsored
by Anthropic. Claude and Claude Code are named here only as the product these
plugins are built for.

## License

MIT. See the LICENSE file at the root of this repository.
