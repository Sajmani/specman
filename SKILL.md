---
name: specman
description: Set up or run spec-driven development in a repository — composing requirements from product, technical, and external sources (terms of service, brand guidelines, regulation, best practice), resolving conflicts between them by precedence, and enforcing the results with acceptance criteria. Use when the user wants to adopt spec-driven development, bootstrap a spec/ directory, write product.md or tech.md, figure out which external rules govern a project, vendor a source's terms into the repo, resolve a conflict between requirements, or asks about specman, process.md, context2spec, spec2test, or spec2code.
---

# specman

Spec-driven development: turn requirements — including the ones somebody else wrote — into
code, through checks that fail when a requirement is violated.

`process.md`, beside this file, is the method. **It is normative and this file is not.** Read
it before doing anything substantial; this file only covers how to apply it, which it does not
say much about.

## What this is for

Most projects are governed by rules the team didn't write: platform terms of service, API
usage policies, brand guidelines, accessibility law, language conventions. Nobody records
them as requirements, so they live in whoever read them last — and you find out you violated
one when a user's account is suspended.

specman is three ideas:

- **Composition** — assemble requirements from every source that applies, with provenance.
- **Resolution** — when they disagree, decide by declared precedence, and record it.
- **Criteria** — every requirement gets a check that fails when it's violated.

## Adopting it in a new project

Follow `process.md`'s [Adopting this in a project](process.md) section. The steps that get
skipped, in order of how often:

1. **Copy `process.md` into the project** as `spec/process.md`.
2. **Point the agent instruction file at it.** Add to `AGENTS.md` (or `CLAUDE.md`) that
   changes follow `spec/process.md` and that `spec/` is normative. Without this the document
   is invisible: agents read the instruction file automatically and read nothing else unless
   told. Skipping this step is the single most common way an adoption fails.
3. **Write the project bindings first** — standing checks above all, since everything after
   is verified by running them. They go in `tech.md` when it is otherwise empty.
4. **Then `sources.md`**: what governs this project that the project didn't write. An hour,
   and where the surprises are.
5. **Product requirements lazily** — the ones a change touches, when it touches them.

Do not create all five documents empty. `acceptance.md` and `decisions.md` appear when there
is a check to record or a conflict to resolve.

## Sizing

Ask how big the user wants this before producing it. A complete adoption for a small tool can
be `process.md`, a bindings table, and three entries in `sources.md`. The apparatus scales
with what a mistake costs, how many outside rules apply, and how many people and agents touch
the code — not with enthusiasm.

Producing five full documents unasked is the characteristic failure of an agent handed this
method.

## Working a change

- Behavior change: spec first, then criteria, then code.
- Bug fix: the spec already forbids it — add the criterion that catches it, then fix. If no
  requirement covers it, that is a spec gap; add the requirement.
- Refactor: no spec change, and the criteria must stay green. If assertions have to change,
  it isn't a refactor.

Stop at the human gates: after the requirements, and after the criteria.

## Rules worth stating up front

These are in `process.md` in full, and are the ones an agent most often gets wrong.

**A criterion you have not watched fail is not a criterion.** Break the code deliberately and
confirm the check goes red. A test written around observed behavior certifies whatever the
code currently does, bugs included — the standard way a retrofit goes wrong.

**Never derive an expected value from a run.** If a check's expected value came from executing
the code, or from a fixture nobody verified against the real system, it passes by construction.
Get it from the requirement, or from the real system, and say which.

**Code generation never resolves a conflict.** If implementation reaches a subject with two
live values, stop and return to the requirements. Deciding it in code makes a product decision
silently and leaves no record.

**Evidence and inference are labeled separately.** "This cleanup tool exists, so the bug must
be real" is a hypothesis. Check it — version control settles most such claims in one command —
or record it as a hypothesis with the check that would settle it.

**Never edit a quotation or a vendored document.** Quoted terms of service are verbatim
citations with recorded hashes. Correcting a publisher's spelling or tidying their punctuation
falsifies the citation and breaks the record.

**Read the issue tracker.** It is named as an input and skipped more than any other, and it is
the place a defect has most likely already been diagnosed by the person who hit it.

## Vendoring an external source

When a project adopts terms of service, guidelines, or a standard:

- Vendor a copy into `spec/sources/<name>/`, with `PROVENANCE.md` (origin URL, SHA-256,
  retrieval date, who retrieved it and how) and `requirements.md` (numbered requirements that
  quote the source verbatim, then say what it means for this project).
- **Strip active content** — `<script>`, `<style>`, `<link>` — from saved pages, and record
  both hashes. A page saved from a browser carries the publisher's scripts, and those carry
  the publisher's secrets; this has put a third party's API key into a public repository.
- If the publisher blocks automated retrieval, ask the human to save the page. Do not work
  around the block, and **do not fill the gap from memory** — an invented rate limit is
  indistinguishable from a real one once it has a requirement ID beside it.

## Reference

- `process.md` — the method: artifacts, IDs, precedence tiers, the three phases, gates,
  drift control, failure modes. Normative.
- A worked example, including twelve resolved conflicts and what the process got wrong:
  <https://github.com/Sajmani/birdsync/tree/main/spec>
