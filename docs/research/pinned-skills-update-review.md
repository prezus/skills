# Pinned skills update review

Date: 2026-08-21

> Implementation status: applied on 2026-08-21. The comparisons below document the
> previous pins and the upstream versions selected for the update.

## Scope and method

This review compares every skill that was listed in [`vendor-manifest.json`](../../vendor-manifest.json) before the 2026-08-21 update with its upstream source. It compares `main`-pinned vendors against current upstream `main`; Herdr is compared against the latest stable release because its skill is release-matched to the installed CLI.

Primary-source baselines:

| Vendor | Pinned | Reviewed target | Upstream comparison |
|---|---|---|---|
| `mattpocock/skills` | `9603c1cc8118d08bc1b3bf34cf714f62178dea3b` | `5b15a47f2d7150f545fbcacbfe381787fc0230dc` (`main`) | [compare](https://github.com/mattpocock/skills/compare/9603c1cc8118d08bc1b3bf34cf714f62178dea3b...5b15a47f2d7150f545fbcacbfe381787fc0230dc) |
| `dmmulroy/anti-slop` | `b5d2288db1f00469a1d5f2e3b0e265e5a5676fd0` | `6d538555cb151d4121ed51a27db81890eacf8ae9` (`main`) | [compare](https://github.com/dmmulroy/anti-slop/compare/b5d2288db1f00469a1d5f2e3b0e265e5a5676fd0...6d538555cb151d4121ed51a27db81890eacf8ae9) |
| `dmmulroy/skills` | `8603380821fee6a77c82639f364ce8fe4f5a92be` | same (`main`) | [commit](https://github.com/dmmulroy/skills/commit/8603380821fee6a77c82639f364ce8fe4f5a92be) |
| `herdrdev/herdr` | `v0.8.0` / `346411fa21afd297f5ed3b3fa56f9e3fbf7654b7` | `v0.8.2` / `9eb521456ac0d19d3ab3d9d7cea3cca10baa8a4c` | [compare](https://github.com/herdrdev/herdr/compare/v0.8.0...v0.8.2) |

The assessment below describes behavior offered by the newer skill files, not every unrelated change in each upstream repository.

## Executive recommendation

1. **Update `install-anti-slop` first.** It has the largest correctness and capability gain: bug fixes, stronger type-resolution logic, six new generic rules, agent-tooling ignores, a faster Oxlint API, and an optional Effect rule set.
2. **Update `herdr` only with the installed Herdr binary.** The `v0.8.2` skill adds important blocked-agent safety. It is deliberately release-matched, so do not update the skill independently of Herdr.
3. **Review and sync the Matt Pocock collection as a set, not piecemeal.** The shipped collection changed shape: one skill was renamed/rebuilt and three new skills were graduated. The highest-value changes are in `ask-matt`, `diagnosing-bugs`, `prototype`, `domain-modeling`, `grilling`, and the replacement for `writing-great-skills`.
4. **No action is needed for `bro` or `coding-standards`.** Their pin is current.
5. **Before syncing Matt's collection, test cross-agent invocation.** Several newer files replace slash-command wording with “call the Skill tool.” That is clearer in harnesses exposing such a tool, but this repository targets multiple agents; verify Pi, Cursor, OpenCode, Claude Code, and Codex behavior before accepting that wording wholesale.

## `mattpocock/skills`: each pinned skill

Current upstream `main` ships 25 skills instead of the pinned set's 22. Of the pinned skills, one is unchanged, several have only editorial/tool-invocation changes, and six have substantial behavioral improvements.

| Pinned skill | Change | What the newer version offers | Recommendation |
|---|---|---|---|
| `ask-matt` | **Major** | Adds a five-way phase-boundary decision tree (`continue`, `/clear`, handoff, subagent, `/compact`), corrects the prototype lifecycle, and routes to the newly shipped `wizard`, `to-questionnaire`, `wait-what`, and `writing-for-agents` skills. [Current source](https://github.com/mattpocock/skills/tree/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/ask-matt) | Update with the full collection; the router otherwise remains stale. |
| `diagnosing-bugs` | **Major** | Adds explicit secret redaction for commands, output, and captured artifacts, including env-var guidance and a stop/ask rule when redaction removes necessary evidence. It also makes cross-skill dispatch explicit. [Relevant commits](https://github.com/mattpocock/skills/compare/9603c1cc8118d08bc1b3bf34cf714f62178dea3b...bda79a3) | Update; this is a concrete security improvement. Note that the old final architecture-postmortem handoff was removed. |
| `grill-with-docs` | Small | Replaces ambiguous “run `/grilling` using `/domain-modeling`” wording with two explicit skill-tool calls. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/grill-with-docs/SKILL.md) | Take only if cross-agent skill invocation is verified. |
| `triage` | Small–medium | Aligns its grilling dependency with the newer round-based grilling flow, stops trying to invoke the user-only setup skill itself, and clarifies multi-skill calls. Most remaining churn is editorial. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/triage/SKILL.md) | Update with the collection. |
| `improve-codebase-architecture` | Small | Uses harness-neutral “spawn a sub-agent” language instead of naming one harness's `Agent` parameters, and makes the `codebase-design` dependency explicit. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/improve-codebase-architecture/SKILL.md) | Useful cross-harness cleanup; verify skill-tool wording. |
| `setup-matt-pocock-skills` | Small | Cleans stale `to-prd`/retired-skill references and frontmatter; core setup behavior is unchanged. [Current source](https://github.com/mattpocock/skills/tree/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/setup-matt-pocock-skills) | Update with the collection, low urgency. |
| `tdd` | Medium | Explicitly reaches for `codebase-design` when the interface or seam itself is undecided, making the test-design dependency clearer. Other changes are editorial. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/tdd/SKILL.md) | Worth updating after invocation compatibility is checked. |
| `to-spec` | Small | Uses “spec” consistently instead of PRD and tells the user to run setup rather than trying to invoke a user-only skill. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/to-spec/SKILL.md) | Update with collection. |
| `to-tickets` | Medium | Removes the instruction to automatically continue into `/implement`, preserving a clean user-controlled phase boundary. It also tells the user to run setup rather than chaining into a user-only skill. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/to-tickets/SKILL.md) | Update; safer and less surprising workflow behavior. |
| `wayfinder` | Small–medium | Makes research, prototype, grilling, and domain-modeling dispatch explicit and stops self-invoking setup. Core decision-map behavior is unchanged. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/wayfinder/SKILL.md) | Update with collection after invocation check. |
| `implement` | **None** | No files under the pinned skill changed. [Current source](https://github.com/mattpocock/skills/tree/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/implement) | No skill-level benefit from updating. |
| `prototype` | **Major** | Replaces the logic prototype's terminal UI with one self-contained, shareable HTML demo aimed at non-developers. It adds free-play controls, guided scenario tabs, domain-language presentation, and preserves a pure liftable logic module inside the page. UI-prototype behavior is otherwise mostly unchanged. [Feature commit](https://github.com/mattpocock/skills/commit/6bcbcb0) | Update if browser-shareable logic demos fit the team's workflow; this deliberately changes the artifact type. |
| `research` | Editorial only | Only punctuation changed; process and guarantees are identical. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/research/SKILL.md) | No practical need to update individually. |
| `domain-modeling` | Medium | Broadens and sharpens automatic triggers: discussing codebase terminology, writing/editing `CONTEXT.md`, and writing/editing ADRs now invoke the skill directly. [Trigger commits](https://github.com/mattpocock/skills/compare/9603c1cc8118d08bc1b3bf34cf714f62178dea3b...54bc6b6) | Update; better discovery and fewer missed domain-modeling moments. |
| `codebase-design` | Small | Replaces harness-specific parallel-agent syntax with harness-neutral wording. The deep-module vocabulary itself is unchanged. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/codebase-design/SKILL.md) | Low-risk cleanup. |
| `code-review` | Small–medium | Uses harness-neutral subagent language, stops self-invoking setup, removes stale PRD terminology, and quotes frontmatter safely. The two-axis algorithm and smell baseline are unchanged. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/code-review/SKILL.md) | Update with collection; mostly portability and correctness of invocation. |
| `resolving-merge-conflicts` | Editorial only | One punctuation rewrite; behavior is identical. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/resolving-merge-conflicts/SKILL.md) | No practical need individually. |
| `grill-me` | Small | Replaces slash-command prose with an explicit call to the `grilling` skill. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/productivity/grill-me/SKILL.md) | Take only after cross-agent invocation testing. |
| `grilling` | **Major** | Changes from one-question-at-a-time interviewing to a round-based design-tree frontier. Each round asks every currently unblocked question, gives recommendations, separates questions with horizontal rules, and delegates fact-finding while continuing independent branches. [Feature history](https://github.com/mattpocock/skills/commits/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/productivity/grilling/SKILL.md) | Update if faster parallel questioning is preferred; keep pinned if strict one-question pacing is intentional. |
| `handoff` | Small | Clarifies that suggested skills should name explicit Skill-tool calls for the next agent. [Current source](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/productivity/handoff/SKILL.md) | Low value unless the target harness supports that invocation model. |
| `teach` | Editorial only | Punctuation and style cleanup across templates; no workflow change. [Current source](https://github.com/mattpocock/skills/tree/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/productivity/teach) | No practical need individually. |
| `writing-great-skills` | **Replaced** | Upstream deleted this skill and replaced it with `writing-for-agents`. The replacement expands scope from skills to all agent-consumed docs (`AGENTS.md`, `CLAUDE.md`, pointed-to docs), adds context pointers, context vs cognitive load, information hierarchy, completion criteria, leading words, pruning, and separate skill mechanics. [Rename/restructure commit](https://github.com/mattpocock/skills/commit/1fc6573) [Current replacement](https://github.com/mattpocock/skills/tree/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/productivity/writing-for-agents) | Replace rather than retain under the old name, unless backward-compatible `/writing-great-skills` invocation is required. |

### Newly shipped Matt skills not in the current pin

A full manifest-derived sync would also add:

- [`wizard`](https://github.com/mattpocock/skills/tree/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/engineering/wizard): generates an interactive Bash wizard for human-only infrastructure, credentials, CI-secret, dashboard, migration, or cutover steps.
- [`to-questionnaire`](https://github.com/mattpocock/skills/tree/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/productivity/to-questionnaire): turns a decision blocked on another person into a questionnaire.
- [`wait-what`](https://github.com/mattpocock/skills/tree/5b15a47f2d7150f545fbcacbfe381787fc0230dc/skills/productivity/wait-what): re-pitches the last response in clearer contextual language when it did not land.

The fourth collection-shape change is the `writing-great-skills` → `writing-for-agents` replacement described above. These changes are reflected in the current [plugin manifest](https://github.com/mattpocock/skills/blob/5b15a47f2d7150f545fbcacbfe381787fc0230dc/.claude-plugin/plugin.json).

## `dmmulroy/anti-slop`: `install-anti-slop`

**Change level: major. Recommended update.**

The newer skill and bundled plugin offer:

- **Five new generic rules:**
  - `no-module-mocking`
  - `no-reflect-apply`
  - `no-reflect-get`
  - `no-unknown-returns`
  - `require-safety-comment-for-type-assertion`
- **Optional Effect support:** an `anti-slop-effect` plugin with `no-service-constructor-imports`, enabled only for a direct `effect` dependency or explicit request.
- **Safer installation defaults:** ignores known agent-tooling directories and the copied plugin itself; for Vite+, it updates both lint and format ignores.
- **Correctness fixes:** preserves conditional object-spread semantics, guards generic-substitution cycles, resolves aliases by lexical meaning, respects lexical type binders, and allows legitimate `typeof` checks in type guards.
- **Better enforcement and diagnostics:** bans unknown returns and unsafe Reflect access, adds actionable diagnostics and violation examples.
- **Performance:** adopts Oxlint's alternative plugin API.

Sources: [full comparison](https://github.com/dmmulroy/anti-slop/compare/b5d2288db1f00469a1d5f2e3b0e265e5a5676fd0...6d538555cb151d4121ed51a27db81890eacf8ae9), [current skill](https://github.com/dmmulroy/anti-slop/blob/6d538555cb151d4121ed51a27db81890eacf8ae9/skills/install-anti-slop/SKILL.md), and the upstream commits for [test/assertion boundaries](https://github.com/dmmulroy/anti-slop/commit/9b80d9a), [agent-tooling ignores](https://github.com/dmmulroy/anti-slop/commit/446268e), and [Effect rules](https://github.com/dmmulroy/anti-slop/commit/6d53855).

This is not a docs-only update: the vendored asset tree changes substantially, so sync the entire `skills/install-anti-slop/` directory wholesale.

## `dmmulroy/skills`: each pinned skill

Upstream `main` still points at the exact pinned commit `8603380821fee6a77c82639f364ce8fe4f5a92be`.

| Skill | Newer offer | Recommendation |
|---|---|---|
| [`bro`](https://github.com/dmmulroy/skills/blob/8603380821fee6a77c82639f364ce8fe4f5a92be/bro/SKILL.md) | None; pin equals upstream `main`. | No action. |
| [`coding-standards`](https://github.com/dmmulroy/skills/blob/8603380821fee6a77c82639f364ce8fe4f5a92be/coding-standards/SKILL.md) | None; pin equals upstream `main`. | No action. |

## `herdrdev/herdr`: `herdr`

**Change level: small diff, important safety behavior. Recommended with Herdr `v0.8.2`.**

The release-matched skill changes only two passages, but they materially improve agent control:

- `agent start` now documents that a blocked startup returns `agent_not_ready` immediately while preserving the agent name for reads and key sends; callers should wait for idle before prompting.
- `agent prompt` now documents `agent_blocked`: it refuses to inject input into an approval/question dialog, instructing the operator to inspect the UI and ask the user before answering.
- Prompt submission wording now matches actual delayed Enter behavior instead of claiming atomic text-plus-Enter submission.

Sources: [`v0.8.0...v0.8.2` skill diff](https://github.com/herdrdev/herdr/compare/v0.8.0...v0.8.2#diff-fafea549), [`v0.8.2` skill](https://github.com/herdrdev/herdr/blob/v0.8.2/skills/herdr/SKILL.md), and [`v0.8.2` changelog](https://github.com/herdrdev/herdr/blob/v0.8.2/CHANGELOG.md#082---2026-08-19).

There are commits on Herdr `master` after `v0.8.2`, but `skills/herdr/SKILL.md` has not changed after that release. Follow the repository's existing release-matching rule: update this skill to `v0.8.2` only if the installed Herdr binary is also `v0.8.2`.

## Proposed update order

1. Confirm installed Herdr version with `herdr --version` before touching its pin.
2. Sync and review `dmmulroy/anti-slop` wholesale.
3. Run a small cross-agent test of nested skill invocation, especially the phrase “call the Skill tool.”
4. If compatible, re-derive and sync the complete current `mattpocock/skills` plugin set, handling the `writing-great-skills` rename as remove + add.
5. Leave `dmmulroy/skills` unchanged.
