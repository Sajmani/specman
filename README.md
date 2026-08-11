# specman

Spec-driven development for projects governed by rules they didn't write.

Most software is subject to requirements from several authors at once: the team's own product
and technical decisions, plus platform terms of service, API usage policies, brand guidelines,
accessibility law, language conventions. The imported ones are rarely written down as
requirements, so they live in whoever read them last — and the first sign that one was
violated is often a suspended account or a compliance finding.

specman is a method for handling that, and an [agent skill](SKILL.md) that applies it.

- **Composition** — assemble requirements from every source that applies, each with a
  provenance and a precedence tier.
- **Resolution** — when they disagree, decide by tier where that settles it, escalate to a
  human where it doesn't, and record the outcome with its evidence.
- **Criteria** — every requirement gets a check that fails when the requirement is violated,
  and no check is trusted until it has been watched failing.

## Contents

| File | What it is |
| --- | --- |
| [`process.md`](process.md) | The method. Normative, and project-independent |
| [`SKILL.md`](SKILL.md) | An agent skill that applies it, in the `SKILL.md` convention |

## Using it

**As a human:** copy `process.md` into your project as `spec/process.md` and follow
[Adopting this in a project](process.md#adopting-this-in-a-project). The step people skip is
pointing the project's `AGENTS.md` at it — a copy nothing references changes nothing.

**As an agent skill:** the repository root is the skill directory — `SKILL.md` with `name`
and `description` frontmatter — so any agent that reads that convention can use it.

Clone it into your tool's skills directory:

```
git clone https://github.com/Sajmani/specman <your-skills-dir>/specman
```

In CloudCode you can instead point at a clone anywhere, which is easier to keep updated —
verified working:

```jsonc
// ~/.config/cloudcode/cloudcode.json
{ "skills": { "paths": ["/path/to/specman"] } }
```

Either way, ask for it by intent — "set up spec-driven development here", "what governs this
project?", "resolve this conflict between requirements" — and the skill loads.

## How big should an adoption be?

Smaller than you'd think. A complete adoption for a small tool can be `spec/process.md`, a
bindings table, and three entries in `sources.md`. The apparatus should scale with what a
mistake costs, how many outside rules apply, and how many people and agents touch the code.

Producing five full documents because the method describes five documents is the
characteristic failure, and `process.md` says so.

## A worked example

[birdsync](https://github.com/Sajmani/birdsync/tree/main/spec) is a real project retrofitted
with this method: a Go tool that copies bird observations from eBird into iNaturalist, and is
therefore governed by two services' terms of service, an API usage policy, and a community
guideline whose breach suspends the *user's* account rather than the author's.

It is worth reading for the parts that went wrong as much as the parts that went right.
[`decisions.md`](https://github.com/Sajmani/birdsync/blob/main/spec/decisions.md) records
twelve conflicts, including one where the analysis cited evidence it had not checked and had
to retract it — twice, in the same entry.

## Status

Early, and written by being used. `process.md`'s revision log lists every change to the method
and what prompted it; most entries are mistakes made while applying it to a real project.

## License

MIT. See [LICENSE](LICENSE).
