# design-doc OpenSpec schema

A custom [OpenSpec](https://github.com/Fission-AI/OpenSpec) schema that reads like a design doc:

| Artifact | Contents | Answers |
|---|---|---|
| `proposal.md` | Title, Intro, Context, Goals, Approach, Trade-offs, External impact, Capabilities | What and why |
| `specs/**/spec.md` (delta specs) | Requirement changes and scenarios (format unchanged from stock) | Required behaviour |
| `design.md` | Technical context, Decisions (with rejected alternatives), Risks, Migration and rollback, Open questions | How |
| `tasks.md` | Groups with a check on every step, optional final verification and Wrap-up | Doing and testing |

Tested with OpenSpec 1.14.1.

## Why not the stock `spec-driven` schema

The stock schema splits the reasoning across files. Its proposal has a 1–2 sentence Why, then What Changes, Capabilities and Impact. The context and the goals live in design.md, after the reader has already seen the list of changes. In practice that means:

- Agents create a lot of verbose, duplicate content across multiple sections.
- Context gets written twice: squeezed into Why, then again in design.md's Context.
- Goals end up in design.md, mixed with implementation constraints, so it's hard to judge whether the change is aimed at the right thing before reading how it's built.

This schema uses the order of a conventional design doc (as in Google design docs, Kubernetes KEPs or Rust RFCs): context, goals, approach, then trade-offs, all in the proposal. A reviewer can agree the problem and the goals before reading any design. An agent gets outcome-level goals to check its work against.

The rest are refinements that the stock schema could mostly get through `config.yaml` rules: facts stated once, results kept out of planning files, and requirements limited to what users care about.

## Differences in detail

Compared with the stock `spec-driven` schema:

**proposal.md**
- Each statement should have a natural location to be stated once, and later sections refer back to it.
- Stock's 1–2 sentence Why becomes a title, a short intro and a Context section.
- Goals are outcomes and move here from design.md. Mechanisms belong in the design.
- A new Approach section gives the shape of the solution before the details.
- Stock design.md's Non-Goals become Trade-offs and limits, after Approach.
- What Changes is gone. Stock's Impact becomes an optional External impact section, limited to who or what outside the change is affected, including breaking changes.
- Capabilities is a list of capability names only.

**Delta specs**
- The agent is told not to over-specify: only behaviour users care about and would notice, with exact numbers only when users rely on them.
- The template shows ADDED, MODIFIED, REMOVED and RENAMED sections; stock shows only ADDED.

**design.md**
- Context becomes an optional Technical context, limited to facts that aren't in the proposal.
- Each decision has its own heading, with its rejected alternatives under it.
- Risks covers only what might go wrong; accepted downsides are in the proposal.
- Migration Plan becomes an optional Migration and rollback section: high-level strategy only, with the steps left to tasks.md.

**tasks.md**
- Like stock, every step has a check and each group lands its own tests. Groups can also end with a check of the running behaviour.
- The agent is prompted to consider a final verification group: delta spec scenarios not covered by automated tests, an end-to-end check, and regressions.
- Tests are tagged `(manual)` or `(deferred)`. When applying the change, the agent asks for manual tests as one batch, records results in one line, and marks skipped tests explicitly.

**Wording**
- "Spec file" and "delta spec" are used throughout, never "spec" alone (see Terms).

See `examples/pool-pump-schedule-retry/` for a full change written with it.

## Terms

| Word | Means |
|---|---|
| **change** | A unit of work: a folder under `openspec/changes/` holding proposal, design, delta specs and tasks |
| **capability** | A named, lasting area of system behaviour, e.g. `user-auth` |
| **spec file** | The file stating one capability's requirements: `openspec/specs/<capability>/spec.md` |
| **delta spec** | A change's edits to a spec file: `openspec/changes/<change>/specs/<capability>/spec.md`. Archiving merges it into the spec file |

Always say "spec file" or "delta spec", never "spec" alone.

## Install in a project

Add this repo as a git submodule at `openspec/schemas/design-doc`:

```sh
git submodule add <repo-url> openspec/schemas/design-doc
```

Clones then need `git clone --recurse-submodules`, or `git submodule update --init`. To pick up template changes, run `git submodule update --remote openspec/schemas/design-doc` and commit.

Select the schema in `openspec/config.yaml`. The `context` glossary keeps the agent's vocabulary consistent:

```yaml
schema: design-doc

context: |
  OpenSpec terms: a "change" is a unit of work (a folder under
  openspec/changes/). A "capability" is a named, lasting area of behavior.
  A "spec file" states one capability's requirements
  (openspec/specs/<capability>/spec.md), and a "delta spec" is a change's
  edits to a spec file. Say "spec file" or "delta spec", never "spec" alone.
  "Create a spec" usually means "create a change".
```

Existing changes keep the schema named in their own `.openspec.yaml`, so archived changes are unaffected.

Check it with `openspec schema validate design-doc`.

## Drawbacks

- This is a fork. The delta-spec instructions are copied from OpenSpec 1.14.1 and don't pick up upstream fixes until someone merges them in by hand.
- The git submodule adds complexity.
- proposal.md has no `## Why` or `## What Changes`, which OpenSpec 1.14 still looks for in two places:
  - `openspec archive` prints a non-blocking warning, then archives normally.
  - `openspec change show <id> --json` fails.

  `openspec validate` doesn't check these headings, and nothing else depends on them.
