# skills

Source of truth for our AI Agent Skills — the ones we author ourselves, plus skills
**vendored** from other people's repos so our whole collection is self-contained and
works across every coding agent we use.

Skills follow the [Agent Skills open standard](https://agentskills.io): each skill is a
directory containing a `SKILL.md` (YAML frontmatter + Markdown body), optionally with
`references/`, `scripts/`, and an `agents/openai.yaml` shim for Codex.

## Layout

```
skills/                     # every skill lives here, one level deep (name = slash command)
  <our-skills>/SKILL.md     # skills we author
  code-review/SKILL.md      # vendored from mattpocock/skills — do NOT hand-edit
  install-anti-slop/        # vendored from dmmulroy/anti-slop — includes scripts/assets
  tdd/SKILL.md              # vendored from mattpocock/skills — do NOT hand-edit
  ...
  LICENSE-mattpocock        # upstream MIT license, preserved for attribution
  LICENSE-dmmulroy-anti-slop # upstream MIT license, preserved for attribution
  LICENSE-herdr              # upstream Apache-2.0 license, preserved for attribution
vendor-manifest.json        # provenance: upstream source, pinned commit, per-skill mapping
VENDORING.md                # how to sync / update vendored skills — READ BEFORE re-syncing
NOTICE                      # attribution
LICENSE                     # MIT (our own work)
```

The `skills/` directory is flat on purpose: agents discover skills exactly one level deep
(`skills/<name>/SKILL.md`), and **the directory name becomes the invocation name** (`/code-review`).
Nested category folders would break discovery, so we flatten upstream's `engineering/`,
`productivity/`, … into a single level.

## How these are installed on a machine

Skills are deployed by our dotfiles ([prezus/dotfiles](https://github.com/prezus/dotfiles)),
not from this repo directly. The model is **one canonical directory, symlinked so every
agent reads it**:

```
~/.agents/skills  ->  ~/Projects/skills/skills   # read natively by Codex, Cursor, OpenCode, Pi
~/.claude/skills  ->  ~/.agents/skills           # Claude Code (only reads ~/.claude/skills)
```

See `~/Projects/dotfiles/INSTALL.md` for the exact steps. There is **no single path all
agents read**, so this two-symlink fan-out is the minimum that covers all five.

## Vendored skills

Vendored sources are pinned in `vendor-manifest.json`:

- [mattpocock/skills](https://github.com/mattpocock/skills) (MIT): the 22 skills Matt
  ships in his Claude Code plugin (`engineering` + `productivity`).

**Engineering:** `ask-matt` · `code-review` · `codebase-design` · `diagnosing-bugs` ·
`domain-modeling` · `grill-with-docs` · `implement` · `improve-codebase-architecture` ·
`prototype` · `research` · `resolving-merge-conflicts` · `setup-matt-pocock-skills` ·
`tdd` · `to-spec` · `to-tickets` · `triage` · `wayfinder`

**Productivity:** `grill-me` · `grilling` · `handoff` · `teach` · `writing-great-skills`

- [dmmulroy/anti-slop](https://github.com/dmmulroy/anti-slop) (MIT):
  `install-anti-slop`, including its bundled Oxlint plugin and installer.

- [herdrdev/herdr](https://github.com/herdrdev/herdr) (Apache-2.0): `herdr`, the
  release-matched instructions for controlling Herdr panes, tabs, workspaces, commands,
  and agents.

> **Do not hand-edit vendored skills.** They are re-synced from upstream. To change one,
> either adopt it (move it out of the vendored set and record that in `vendor-manifest.json`)
> or contribute the change upstream. See [VENDORING.md](./VENDORING.md).

Some of these (`triage`, `to-tickets`, `code-review`, `implement`) expect you to have run
`/setup-matt-pocock-skills` first, which writes an issue-tracker + labels config into
`docs/agents/` of the *target* project.

## Credits

Vendored skills retain their upstream licenses — see `skills/LICENSE-mattpocock`,
`skills/LICENSE-dmmulroy-anti-slop`, and `skills/LICENSE-herdr`. Thanks to Matt Pocock
for the [skills](https://github.com/mattpocock/skills) collection, Dillon Mulroy for
[anti-slop](https://github.com/dmmulroy/anti-slop), and the Herdr contributors for
[Herdr](https://github.com/herdrdev/herdr). Vendoring workflow inspired by
[dmmulroy/skills](https://github.com/dmmulroy/skills).
