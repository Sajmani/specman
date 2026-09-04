# TODO

## Resuming — read this first

Updated 2026-08-27. Work spans six sibling repositories. Nothing is lost if a session ends:
everything below is on disk, though **none of it is committed**.

| Repo | State | Committed? |
| --- | --- | --- |
| `specman` | The method. `process.md` at pin `919563cd`, five amendments landed | `process.md` **modified**, `TODO.md` **untracked** |
| `gemini-cli` | Synced to `919563cd` + banner + one staged `[LOCAL]` amendment | `spec/` **untracked**; `GEMINI.md`, `.prettierignore` modified |
| `birdsync` | Synced to `919563cd` + banner; gained a `specman` manifest entry | `spec/process.md`, `spec/sources.md` **modified** |
| `org-sec` | Example: spec-only repo | Committed, tagged `v1.0` `v1.1` |
| `server-framework` | Example: framework. Stale pin fixed, `acceptance.md` added | **Modified, uncommitted** — tags `v1.10` `v1.11` predate the fix |
| `greeter` | Example: the app. `acceptance.md` added | **Modified, uncommitted** — tags `v1.20` `v1.21` predate it |

**Task 1 is done.** Next action is **task 3** (Goals), which task 4 depends on. Task 2's
remaining work is the OTel layering question and two unresolved findings.

**The canonical pin is `sha256:919563cd…`.** Two project copies record it. A project copy is
never byte-identical to canonical — it carries a banner — so the check is that the diff contains
*the banner and nothing else but `[LOCAL]` rows*:

```bash
diff -u process.md ../gemini-cli/spec/process.md   # banner + 1 staged amendment
diff -u process.md ../birdsync/spec/process.md     # banner only
```

**Three things a fresh session will not otherwise know:**

- `server-framework` and `greeter` now have meta-criteria that read the spec bundle from the
  repository root. They run as part of `go test ./...`, so a spec edit can break the build —
  which is the point.
- `gemini-cli` has **no `node_modules`** — `npm run format` fails on a missing binary. Use
  `npx prettier@3.5.3` (the version the repo pins) to check formatting. `spec/process.md` and
  `spec/sources/` are deliberately excluded from Prettier and must stay that way.
- Go on this machine needs `-ldflags=-linkmode=external` or tests abort with
  `missing LC_UUID` (go1.22.4 vs the macOS 26 linker).

**Blocking item parked in gemini-cli:** `CR-001` in `gemini-cli/spec/decisions.md` has no expiry
or review date, so it is not yet a valid risk acceptance — and `wcag` is a mandatory source, so
that record is the only thing between the project and "work stops". Two dates from the owner
closes it. Full list in `gemini-cli/spec/TODO.md`.

Sessions of record for provenance: `ses_fc5410b4effe1MMYZpTO63UgOz` (2026-08-25/26, the
adoption and the worked example) and `ses_e5fbd2501adffec0u7ckMN5M7V` (2026-08-27, task 1 and
the stale-pin fix). The `grill-me` skill exists now, and task 3 is the one that most needs it.

Working plan for evolving the method. Tasks are ordered by dependency: each one's output is the
next one's input.

## 1. Land the pending amendments from gemini-cli — **DONE 2026-08-27**

Five amendments are in canonical. The four staged in gemini-cli were upstreamed with their
`[LOCAL]` markers dropped, re-dated to the landing date, and each given an explicit
**Exercised** clause in its revision row — that clause is where the honesty about evidence now
lives, because the revision log is the only artifact that travels with `process.md` to every
project. A fifth was written during the landing.

| # | Amendment | Exercised |
| --- | --- | --- |
| 1 | Keep the repository's formatter off the spec bundle | Fully — failure reproduced, then prevented |
| 2 | `Status` field on the source manifest | In two projects; has not yet caught a mistake its author had not already seen |
| 3 | How amendments reach the canonical copy | First full cycle completed by this landing. The *rejection* path is still untested |
| 4 | Decouple spec commits from code; `implements.md` | Fully, by the three-repo example. The staleness bound it requires remains unchecked |
| 5 | A project copy records its provenance: `sources.md` entry + pointer banner | **Not exercised** — reasoned, landed unstaged, earns its evidence at birdsync's next refresh |

Amendment 5 was not planned. birdsync recorded the canonical origin in a banner with **no pin**,
so its provenance worked only until canonical moved — which this landing did. gemini-cli had the
pin but nothing in the file pointed at it. Two projects had invented half the mechanism each.
Canonical now requires both halves and states that a copy is never byte-identical to its pin: the
diff should contain the banner and nothing else but `[LOCAL]` rows.

### What this landing exposed

Going looking for evidence rather than trusting the notes turned up a live defect and one
correction to the plan.

**`server-framework`'s vendored `org-sec` pin was stale in two files.** The v1.1 refresh replaced
the vendored bytes and the manifest's `Version` field and left *both* hash records on the v1.0
value. `TestVocabularyCoverage` passed throughout, because the prose it parsed was the correct
new prose — the pin was the only thing wrong and the pin was the one thing unchecked. Root cause:
the repository had **no `acceptance.md`**, so the meta-criterion `process.md` already requires
(every vendored copy matching its recorded hash) existed nowhere. The method was right; the
example did not follow it.

Fixed, with `acceptance.md` and two meta-criteria added, both watched failing against the real
defect before the fix rather than against a repaired tree. `greeter` gained an `acceptance.md`
too. The same refresh had also left "for org-sec v1.0" in a paragraph of `implements.md` while
the table above it said v1.1 — **nothing checks prose, and a conformance record is mostly
prose**, which is now a recorded gap.

**A sixth amendment is staged in gemini-cli** as a result, and it went through two drafts. The
first said a hash lives in `PROVENANCE.md`, or inline in the manifest when no `PROVENANCE.md`
exists to hold it — the inline case being `apache2`, whose `LICENSE` sits at the repository root
and had no source directory at all. That carve-out is gone. The rule is now:

> **`Integrity` never contains a hash. It names where the hash lives.** The source directory's
> `PROVENANCE.md` where anything is vendored — *including* a file tooling forces to live
> elsewhere, written with a repository-root-anchored path like `//LICENSE`. A tool's own lockfile
> where the tool owns the pin. No row at all where nothing is vendored.

Anchoring at the root rather than writing `../../../LICENSE` states the fact instead of encoding
how deep the spec bundle happens to sit — an accident that changed once already, when the
artifacts were collected under `spec/`.

The second draft also separates a **base pin** from an integrity hash, which the first conflated.
An integrity hash asserts *this file hashes to X*; a base pin records *what upstream was when we
last synced*, for a file the project deliberately edits — a project copy of `process.md` being
the standing case. The file is supposed to differ from it, so a base pin belongs in `Version` and
no `PROVENANCE.md` should ever claim a file matches one. gemini-cli's manifest already had this
right without anyone naming it: its `specman` entry has no `Integrity` row, only a `Version`
carrying the base pin.

That rule covers all nine of gemini-cli's sources with no residue, and it turned the exception
from a fact about file layout into a distinction between two kinds of claim.

### Spun out: separation in space and time

Deferred to its own staged amendment rather than landed, because task 2's worked example
contradicts part of what was drafted. Two axes.

**In time.** Settled and exercised — a tag marks the spec approved to implement, `implements.md`
records conformance against it. What is *not* exercised is the staleness bound the text requires:
both conformance records carry a `Last reconciled` date and nothing compares it to anything.

**In space.** The drafted refinements were:

- **Code always has a local `spec/` directory**, even when the product spec is external — the
  local material being `tech.md`, `arch.md`, `implements.md` and the `code`-level criteria.
- **An external spec is recorded in `spec/sources/`**, pinned and vendored like any other source.

The second is contradicted by the only worked example. `greeter` cites `org-sec/R1`–`R3` by ID
and targets `org-sec v1.1`, but has **no `sources.md`, no vendored copy and no pin** — it reaches
org-sec entirely through `server-framework`. Either a consumer satisfying-by-dependency is exempt
from adopting the source, or greeter is defective. Unresolved, and deliberately not settled by
fiat inside an example repo.

The open question — whether an external *product* spec is the same kind of thing as a vendored
terms document — got a better discriminator than the one originally posed. It is not "implement
versus merely comply." **It is whether you edit it.** Everything in `spec/sources/` stays
byte-identical to upstream; `process.md` is amended in place, which is exactly why gemini-cli
keeps it out of `sources/` while still giving it a manifest entry. org-sec is not edited by its
consumers, so it vendors normally. That reasoning is now in canonical for `process.md`'s own
case and should generalize.

Still unresolved from the staging notes: with several implementations `sources.md` is shared but
a source's tier is per-adoption, and the subject vocabulary must be stack-neutral, which
gemini-cli's is not.

## 2. Requirements satisfied by a dependency

**Design settled 2026-08-26 by interview; not yet validated.** The OO metaphor that motivated
this (interface / abstract class / inheritance) was deliberately **dropped**: what it describes
is a shared source plus published conformance reports, and "inheritance" misleads by implying
the guarantee is automatic when it is conditional and parameterized. The term is **satisfied
by** — the obligation still exists and the consumer remains accountable; someone else met it.

### The example

Two requirements: every server request is logged (observability), and no PII is ever logged
(security). A `greeter` server answers `?u=sameer` with `hello, sameer`; `u` is PII and must be
scrubbed from the log. A `server-framework` provides logging-with-scrubbing, and greeter
satisfies both requirements by using it.

### Settled decisions

| # | Decision |
| --- | --- |
| 1 | Observability and security are an **org-wide source** (`org-sec`) that both the framework and the app adopt. The framework provides a mechanism; it does not own the requirements |
| 2 | Satisfaction is **conditional**, not automatic. The framework covers only what routes through it, so the consumer owes a **non-escape check** |
| 3 | Recorded in the consumer's `implements.md` as a **"Satisfied by"** citation naming the dependency and its version |
| 4 | The non-escape check is **static analysis** — forbid the escape route (direct log calls) so new code fails the build rather than silently violating |
| 5 | The citation **pins the dependency version**; bumping it is a source-pin refresh, and re-verifies |
| 6 | An **unclassified value is scrubbed** — fail closed. You opt into logging by classifying a value as non-PII |
| 7 | `org-sec` defines PII categories as a **prose vocabulary with stable IDs**; the framework **materializes it machine-readably** (generated types). The framework is the transcription point: it converts prose to code once, and N apps compile against that instead of each reinterpreting |
| 8 | The framework's satisfaction claim is a **normative promise in its `product.md`**, not just a report in `implements.md`. Consumers pin and cite it, so dropping it must be a breaking change |
| 9 | A **vocabulary coverage check** in the framework asserts its schema covers every ID in the source's vocabulary — makes the transcription falsifiable |
| 10 | `org-sec` sets a **scrub floor** (redact / hash / truncate); a framework may exceed it |
| 11 | The generated schema is **part of the framework's public interface**, since app code references the types |
| 12 | Satisfaction may be **partial and scoped**: "satisfied by server-framework v1.10 (query params); app-owned (request bodies)". The remainder must be named and carry its own criterion |

### Spec versioning

A **minor** bump is substitutable: it may only **add requirements or tighten existing ones**,
never remove or relax one. A **major** bump may break a guarantee the previous version gave.

Under that discipline conformance claims are **ordered**: satisfying v1.1 implies satisfying
v1.0, by construction rather than convention. A claim is therefore only meaningful with a
version attached — "conforms to org-spec" is not a statement. An implementation pinned to an
older version is not *broken* by a newer one landing; it is pinned to an older target.

Worked chain:

```
org-sec v1.0 ──────────── server-framework v1.10 ──── greeter v1.20
org-sec v1.1 (adds        server-framework v1.11
  device_id, minor)
```

greeter bumps its dependency to server-framework v1.11 and thereby starts satisfying org-sec
v1.1. Its claim is **declared and then checked**: a criterion asserts the org-sec version it
claims matches what the pinned framework actually promises.

If strengthening breaks a dependent — say abuse detection relied on `device_id` being logged —
that is **not** a major bump. It is a cross-source conflict between security and observability,
which tiers and a `CR-###` already handle. Only a weakened guarantee is a major bump.

### Built and validated 2026-08-26

Three repos, all green, all checks watched failing before being trusted.

| Repo | Role | Tags |
| --- | --- | --- |
| `org-sec` | Spec only, no code | `v1.0`, `v1.1` |
| `server-framework` | Framework; materializes the vocabulary | `v1.10`, `v1.11` |
| `greeter` | The app, `?u=sameer` | `v1.20`, `v1.21` |

The full chain ran: org-sec `v1.1` added `pii.category.device_id`, the framework's coverage check
went red on the pin refresh, the framework shipped `v1.11`, and greeter picked the guarantee up
by bumping its dependency.

Three checks exist and each was watched failing:

| Check | Broken by | Caught |
| --- | --- | --- |
| `TestVocabularyCoverage` | Removing a category from the framework's schema | Yes |
| `TestNoEscapeFromFramework` | Adding `log.Printf(name)` to greeter | Yes |
| `TestClaimMatchesDependency` | Claiming `v1.2` against a framework promising `v1.1` | Yes |

### What building it contradicted

**1. Decision 15 was wrong: the transitive claim check must be an ordering, not an equality.**
It was written as `claimed == promised`. When the framework moved to org-sec `v1.1` while greeter
still claimed `v1.0`, the check failed — but greeter's claim was *true*, because satisfying `v1.1`
implies satisfying `v1.0`. Under-claiming is honest and must pass; only over-claiming is false.
Corrected to `claimed <= promised` within a major version, and incomparable across majors, since
a major bump may remove a guarantee. The ordering property we derived during the interview turned
out to be load-bearing for the check, not just a nice observation.

**2. The check ignores the pin it exists to validate.** It reads the framework's `product.md`
from a filesystem path, and greeter's `go.mod` has a `replace` directive pointing at that
directory — so it compares against the framework's working tree, not the pinned version. A real
dependency would need the spec read at the pinned version from the module cache. Recorded as a
known hole in `greeter/spec/implements.md` rather than papered over.

**3. Scope is where the real risk lives, and no check covers it.** The framework scrubs query
parameters only. greeter is unexposed solely because it happens to read no request bodies — a
property of its current shape, not a guarantee. The moment it accepts a POST body it owes
`org-sec/R2` directly, and nothing would notice. The non-escape check catches *leaving* the
framework; nothing catches *never routing input through it in the first place*.

**4. A toolchain workaround became a real project binding.** `go test` fails on this machine with
`missing LC_UUID` (go1.22.4 vs the macOS 26 linker); the standing check needs
`-ldflags=-linkmode=external`. Unglamorous, but it is exactly what bindings are for — a standing
check that does not run on the maintainer's machine is not a standing check.

### Next on this example: which layer names the observability format

The Goals in task 3 only pay off if logs from every server in the organization can be
**aggregated**, and that needs a single wire format — concretely, OpenTelemetry. `org-sec` today
says requests must be logged; it does not say in what shape, so two conforming servers can emit
formats that never join up. The question is which layer requires OTel.

The tension is that this is the product/tech split under load. Naming OTel is a **technical**
choice, and task 1 says `tech.md` is per-implementation — but cross-org aggregation is exactly
the case where a per-implementation choice destroys the requirement. A shared outcome needs a
shared mechanism, and something has to name it normatively.

Options to weigh, not yet decided:

| # | Where OTel is named | Notes |
| --- | --- | --- |
| A | `org-sec/product.md` names OTel directly; the framework implements it | Simplest. Puts a stack choice in a product spec, and swapping formats becomes a spec change |
| B | `org-sec` requires "the org-wide format", named in a separate org-level binding or registry | Keeps the spec outcome-shaped; the concrete choice versions independently. Adds an indirection to check |
| C | `org-sec` stays outcome-only ("logs must be aggregatable org-wide"); the framework's `product.md` promises OTel | Consistent with decision 8 — the framework's guarantee is the normative promise. But nothing then stops a second framework choosing differently and still claiming conformance |
| D | A separate `org-obs` source, distinct from `org-sec` | Different owner (platform vs. security), different tier, different Goals. Task 3 suggests these were only ever bundled by accident |
| E | Vendor the OTel specification itself into `spec/sources/`, tiered | What the method already does for external rules. Composes with any of A–D rather than replacing them |

Whichever wins has to answer the criterion question: what check fails when a server logs in a
format nobody can aggregate? A vocabulary-coverage-style check against the OTel schema is the
obvious candidate, and it would sit in the same place the PII one does.

### Still open

- The scope gap in finding 3 — what check would catch a service accepting PII the framework never
  sees? Now recorded as a gap in `greeter/spec/acceptance.md`, still unsolved.
- Whether an external product spec (`org-sec`) really belongs in `spec/sources/` alongside
  vendored rules documents. **The "it worked without friction" evidence was wrong** — the first
  refresh silently broke the pin in two files and went unnoticed for a week, because the example
  had no `acceptance.md` to hold the meta-criterion. Fixed 2026-08-27; see task 1. The question
  itself is still open, but it now has a discriminator: what belongs in `spec/sources/` is what
  is *not edited in place*.
- The staleness bound on a conformance record. `process.md` requires one; neither `implements.md`
  has anything a check could read. Either give `Last reconciled` a maximum age and check it, or
  drop the requirement in task 4 as a concept nothing uses.
- Prose goes unchecked. Every criterion in both example repos reads structure or code, and the
  v1.0/v1.1 rot lived in a paragraph. Whether this is fixable or just a known limit is worth a
  paragraph in the method.

## 3. Goals: why a requirement exists

**New, not yet designed.** A requirement states *what* must be true. Nothing in the method
records *why*, and the omission is doing real damage in the task 2 example: `org-sec` says every
request must be logged and never says what that is for. Without the why, a requirement cannot be
weighed against a conflicting one, cannot be scoped correctly, and cannot be retired when its
reason expires.

The proposal is to capture **Goals** explicitly and have each requirement cite the Goals it
serves. Goals form a chain — a requirement serves a capability, which serves an outcome — and one
requirement typically serves several.

Worked from the example: logging requests is not one goal but four chains.

| Capability the requirement enables | Goal it ultimately serves |
| --- | --- |
| Understanding technical performance — latency, failure rates | Assessing whether the server meets its **SLOs** |
| Debugging technical failures | Updating the server so it **meets its SLOs** |
| Product and business insight — adoption, retention, churn, resource consumption | **Revenue** |
| Producing training data for AI models | Further improving technical and product performance |

What this unlocks, and what has to be worked out:

- **Conflict resolution gets a basis.** `decisions.md` currently resolves a conflict by tier and
  owner. Two requirements from equal-tier sources can only be traded off against each other if
  you know what each is *for* — the PII-scrubbing versus abuse-detection clash in task 2's
  versioning section is precisely this, and it was routed to tier precedence and a `CR-###`,
  which decides who outranks whom without ever asking what either requirement is protecting.
- **Scope becomes derivable.** Finding 3 in task 2 — the framework scrubs query parameters and
  nothing notices a request body — is a scope question, and the Goal is what says how wide the
  scope must be.
- **Retirement becomes possible.** A requirement whose Goals have all lapsed is dead weight, and
  today nothing marks it.
- **Open: which artifact holds them.** A `goals.md`, a section of `product.md`, or a field on
  each requirement. Goals are shared across codebases like `product.md`, which argues against
  putting them in a per-codebase file.
- **Open: whether Goals are normative.** They are the reason a requirement exists, but you do not
  implement a Goal directly and no criterion can check one. Probably a fourth category alongside
  normative, descriptive and historical — or the thing that makes `product.md`'s requirements
  reviewable rather than a separate artifact at all.
- **Open: how far up the chain to go.** "Revenue" is where the example stops, and it stops there
  arbitrarily. Without a rule for termination, every Goal chain climbs to "the company survives"
  and stops being useful.

Do this **before** task 4, since it introduces a concept and the simplification pass judges
whether concepts earn their keep.

## 4. Simplification pass

`process.md` has grown by roughly 95 lines without anything being removed. Look for:

- Redundancy between the new decoupling section and the existing drift-control and
  risk-acceptance sections, which now say overlapping things about gaps.
- The revision log, now 13 entries, several of them narrating incidents rather than stating
  changes.
- Whether `implements.md`, `arch.md` and the descriptive/normative split still read coherently
  together after the additions.
- Concepts introduced but never used in any project: candidates for deletion, not documentation.

Do this **after** tasks 2 and 3, so the vocabulary those settle can simplify the text rather than
adding another layer to it.

## 5. Take the revised method back to birdsync

birdsync has the most mature bundle — `product.md`, `tech.md`, `acceptance.md`, `decisions.md`
with twelve resolved conflicts, `arch.md`, and vendored sources. It is the best test of whether
a change to the method survives contact with a project that already followed the old one.

Its `process.md` was re-synced on 2026-08-27 and is now canonical plus the banner, with a
`specman` entry and base pin added to `sources.md`. This refresh is what will exercise amendment
5, the only one that landed without evidence.

Two things to pick up when returning:

- **`sources.md` contradicts itself.** The header says "**Status: draft, pending Gate 1.** No
  source has been vendored yet", while the Vendoring status section says "All three by-reference
  sources are now vendored and pinned." Pre-existing, not introduced by the re-sync. Left alone
  because it turns on birdsync's Gate 1 state, which is the owner's call.
- **birdsync's manifest predates the `Status` field**, which landed in amendment 2. Adding it is
  a good test of whether that field finds anything the other fields hid — it has not yet caught a
  mistake anywhere, and birdsync is the project with enough sources to give it a chance.

## 6. Return to gemini-cli

Resume the parked work in `gemini-cli/spec/TODO.md`, on the revised method. The blocking item
there is unrelated to any of this: CR-001 needs an expiry and a review date, without which a
mandatory source has no valid risk acceptance behind it.

### The themes question: complying without breaking users

`CR-001` accepts the risk that gemini-cli's themes fall short of WCAG minimum contrast. The
compliant end state is to **restrict the available themes to those that meet the contrast
minimum** — but doing that in one release forcibly reformats the terminal of every user who
chose a low-contrast theme deliberately. Compliance that ships as a surprise regression is how a
mandatory source earns a reputation for being the enemy.

So this is a **managed migration owned by the product team**, not a spec edit, and it is worth
working through because it is the general case: *strengthening* compliance — moving to satisfy a
stronger spec with additional requirements — while an existing population depends on the weaker
behavior. Task 2 established that strengthening is a minor bump for the *spec*; nothing yet says
what the *implementation* owes its existing users when it catches up.

The opportunity worth investigating first: **can gemini-cli detect the user's system-level
accessibility preferences?** If the OS already exposes "I have requested high contrast," then the
migration stops being a global break and becomes targeted — users who asked for high contrast at
the system level are switched (or offered a switch) to the high-contrast theme set, and everyone
else is left undisturbed. That would satisfy the requirement for the population it exists to
protect without imposing it on anyone who did not ask.

To work out:

- Whether such a signal is reliably readable from a terminal application on macOS, Windows and
  Linux, and what it is called on each. Unverified — this is the first thing to check, and the
  whole approach depends on it.
- Whether the correct behavior on detection is **switch automatically** or **offer to switch**.
  Automatic respects a preference the user already expressed to their OS; offering avoids
  surprising them in a second place.
- What is owed to the undetected population, since a terminal user with low vision and no system
  setting is exactly who the requirement is for. Detection narrows the blast radius; it does not
  discharge the requirement, and the residue needs its own record.
- How this lands in the bundle: a dated migration plan that lets `CR-001` expire against
  something real, rather than a risk acceptance renewed indefinitely.
