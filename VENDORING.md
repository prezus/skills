# Vendoring & syncing guidelines

How we bring third-party skills into this repo and keep them up to date. Current upstreams
are recorded in `vendor-manifest.json`; the process below generalizes to any source.

## Principles

1. **Vendored = a pinned copy, not a live dependency.** We commit the actual skill files
   so the repo is self-contained, works offline, and works with any agent. We are not a
   plugin subscriber.
2. **Never hand-edit a vendored skill.** Local edits are silently destroyed on the next
   sync. If you need a change, either (a) contribute it upstream and re-sync, or
   (b) *adopt* the skill (see [Adopting a skill](#adopting-a-skill)).
3. **Every sync is deliberate and reviewed.** We pin an exact upstream commit in
   `vendor-manifest.json` and only move it when we've read the diff.
4. **Flatten, don't nest.** Upstream groups skills in `engineering/`, `productivity/`, …
   Agents discover skills only one level deep, so we vendor them flat: `skills/<name>/`.
5. **Preserve provenance and license.** Update `vendor-manifest.json` and keep each
   vendor's license file (MIT requires the notice to travel with the copy).

## What is vendored

For `mattpocock/skills`, the set is defined by the upstream plugin manifest
(`.claude-plugin/plugin.json` → `skills[]`), which is Matt's curated, shipped list —
currently 22 skills (`engineering` + `productivity`). We deliberately **exclude**
`in-progress/`, `deprecated/`, `personal/`, and `misc/` — they're experimental or
Matt-specific.

For `herdrdev/herdr`, we vendor the single release-matched `skills/herdr/SKILL.md` file.
Pin a Herdr release tag and resolved commit, and refresh `skills/LICENSE-herdr` when
updating it. `herdr --skill` may be used when the installed binary matches that release.

`vendor-manifest.json` is the source of truth for exactly which skills we carry and where
each came from upstream.

## Syncing to a newer upstream

> Prerequisite: know the commit you're moving to. Default to upstream `main`'s current HEAD;
> pin a specific SHA/tag if you want reproducibility.

1. **Clone upstream at the target ref** into a scratch/gitignored location (never commit
   the clone):
   ```sh
   git clone https://github.com/mattpocock/skills.git repos/mattpocock-skills
   cd repos/mattpocock-skills && git checkout <ref>   # or leave on main
   ```
2. **Re-derive the shipped set** from the upstream manifest, so we track Matt's curation
   rather than a hardcoded list:
   ```sh
   node -e "require('./.claude-plugin/plugin.json').skills.forEach(s=>console.log(s.replace(/^\.\//,'')))"
   ```
3. **Flatten each shipped skill** into `skills/<name>/`, replacing the existing copy
   wholesale (`rm -rf` then `cp -R`) so upstream deletions inside a skill are reflected:
   ```
   repos/.../skills/engineering/code-review/  ->  skills/code-review/
   ```
4. **Reconcile the set** against the previous manifest:
   - **Added** upstream → appears as a new `skills/<name>/`.
   - **Removed** upstream (dropped from `plugin.json`) → delete our `skills/<name>/`
     **unless we've adopted it** (then leave it and mark it adopted).
   - **Renamed** upstream → treat as remove + add; the slash command changes with the
     directory name, so grep the repo/docs for the old name.
5. **Review the diff.** `git diff --stat` then read `git diff`. Vendored churn is expected;
   what you're checking for is (a) anything that touches an adopted skill, (b) new
   `scripts/` that run commands, (c) new config dependencies (e.g. a skill that now needs
   `docs/agents/…`).
6. **Update provenance.** Bump `pinnedCommit`, `pinnedRef`, `vendoredOn`, and the `skills[]`
   list in `vendor-manifest.json`. Refresh the vendor's license file if upstream's changed.
   Update the vendored list in `README.md` if the set changed.
7. **Commit** with the upstream SHA in the message, e.g.
   `chore(vendor): sync mattpocock/skills @ <short-sha>`.

The sync tooling itself (a `sync-matt-skills.sh` or similar) is intentionally not committed
yet — it's owned separately. Whatever form it takes, it must implement steps 1–6 above and
leave step 5 (human diff review) and step 7 (commit) to a person.

### Alternative: the `npx skills` installer

[`vercel-labs/skills`](https://github.com/vercel-labs/skills) (`npx skills add mattpocock/skills`)
can also pull and update these skills, and it fans out to each agent's own directory with a
`skills-lock.json` for reproducibility. We don't use it as our primary path because our
install model is **one canonical dir + symlinks** (see below), which the installer's
per-agent fan-out would duplicate. Reach for it only if we ever switch to per-agent copies.

## Adopting a skill

When we want to own and modify a vendored skill:

1. Move it out of the vendored set conceptually: remove its entry from the `mattpocock/skills`
   vendor in `vendor-manifest.json` (or move it to an `adopted` list).
2. Add a short note at the top of its `SKILL.md` body recording the upstream origin + the
   commit it forked from, so provenance isn't lost.
3. From then on it's ours — future syncs must skip it (they only touch names still listed
   under a vendor).

## The install model (context for why we flatten)

Skills are deployed by [prezus/dotfiles](https://github.com/prezus/dotfiles) as **one
canonical directory that every agent reads**, via two symlinks:

```
~/.agents/skills  ->  ~/Projects/skills/skills   # Codex, Cursor, OpenCode, Pi (native)
~/.claude/skills  ->  ~/.agents/skills           # Claude Code (only reads ~/.claude/skills)
```

Because agents scan `<dir>/<name>/SKILL.md` exactly one level deep and derive the slash
command from the directory name, the vendored tree **must** be flat with clean names —
hence the flatten step. Full install steps live in `dotfiles/INSTALL.md`.

## Checklist for a sync PR

- [ ] Cloned upstream at a known ref (scratch/gitignored)
- [ ] Re-derived the shipped set from upstream `plugin.json`
- [ ] Flattened + replaced each skill wholesale
- [ ] Reconciled added / removed / renamed skills (respected adopted skills)
- [ ] Read the diff
- [ ] Updated `vendor-manifest.json` (commit, date, skill list) + the vendor license
- [ ] Updated `README.md` vendored list if the set changed
- [ ] Committed with the upstream short-SHA in the message
