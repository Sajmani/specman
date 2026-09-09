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

**Task 1 is done.** Next action is **task 7's research half** — locating this method inside the
Requirements Management discipline — because it may already have vocabulary for what task 3 is
about to invent. Then task 3 (Goals), which task 4 depends on. Task 2's remaining work is the
OTel layering question and two unresolved findings.

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

### Goals are what validation validates against

**Added 2026-08-27, and it gives this task a job rather than a genre.** CMMI separates
verification from validation: VER ensures "work products meet their specified requirements", VAL
demonstrates the product "fulfills its intended use when placed in its intended environment".
Every criterion this method writes is verification. Nothing validates.

The reason nothing validates is that validation needs something to check *against*, and the
intended use is not written down anywhere. That is precisely what a Goal is. So:

> **Verification checks a requirement against the code. Validation checks a requirement against
> its Goal.**

This matters because it rescues Goals from being decorative. Rationale prose that nobody can
fail is exactly the "adoption by gesture" this method warns about. If a Goal is what a validation
criterion reads, it has to be stated precisely enough to be checked — which is the same standard
already applied to requirements.

CMMI has two distinct validation activities and both are relevant: **RD SP 3.5 Validate
Requirements** asks whether these are the right requirements, and the **VAL** process area asks
whether the built product serves its intended use. **RD SP 3.1, Establish Operational Concepts
and Scenarios**, is the practice of writing the intended use down in the first place.

#### A live instance, already in the repository

`org-sec/R1` says every request produces exactly one log record, and states its own rationale:

> Rationale: incident reconstruction. A request that left no trace cannot be investigated.

`TestLogsEveryRequest` passes. It asserts exactly one record and that the record contains
`path=`. R1 is verified, completely and honestly.

Now read what the record actually contains — `framework.go`'s `logRequest` emits:

```
method=GET path=/hello u=<redacted>
```

**No timestamp. No status code. No latency.** You cannot reconstruct an incident from that. You
cannot order it against an outage window, tell whether it failed, or tell whether it was slow.
The requirement is satisfied and its stated purpose is not served, and every check in the
repository is green.

That is the whole argument for this task in one example, and it was sitting in the worked example
unnoticed. It also lines up with the owner's original Goals chain: observability serves SLO
assessment through latency and failure rates, and this record contains neither.

#### Goals already half-exist, informally

`org-sec/product.md` carries a `Rationale:` line on three of its five requirements, written
before this task was conceived. That is a proto-Goals field arrived at by instinct. Two
consequences: the notation partly exists and should be built on rather than replaced, and the
coverage is already uneven — two requirements have no rationale at all, which nothing detects.

#### It unifies three open items

All three are the same shape — *requirement satisfied, purpose unserved*:

| Open item | Read as a validation failure |
| --- | --- |
| This task | R1 verified; incident reconstruction impossible |
| Task 2's OTel question | R1 verified; logs in a format nobody can aggregate, so org-wide analysis is impossible |
| Task 2's finding 3, the scope gap | R2 verified within query parameters; the purpose — no PII in logs, ever — assured only by greeter's accidental shape |

If validation criteria against Goals are the answer, one mechanism closes all three, and the OTel
question stops being about layering and becomes about which Goal the format serves.

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

## 7. Rename to `reqman`, and locate the method in Requirements Management

Two halves. The second should probably run **before task 3**, because Requirements Management is
a discipline with fifty years of vocabulary and it may already have a name and a shape for what
task 3 is about to invent from scratch.

### The framing that governs this task

**The goal is utility, not novelty.** Stated by the owner on 2026-08-27, and it corrects the axis
this task was first written on — an earlier draft sorted findings into "aligned" and "different",
and kept score on what could be claimed as new. That is the wrong question.

The purpose is to use LLMs and agents to improve the practice of software engineering, by
streamlining three things: managing requirements that arrive from multiple sources, generating
code that satisfies them, and verifying that code against tests and other acceptance criteria
generated from those same requirements.

Two consequences for how RM gets read:

- **Where RM has a good idea, take it.** Fifty years of practice outweighs a few weeks of
  reasoning, and adopting its vocabulary makes the method teachable to people who already know
  the discipline.
- **Where RM has complexity or toil that exists only because humans had to do the work by hand,
  automate it.** Much of RM's reputation for heaviness comes from clerical labor — maintaining
  traceability matrices, chasing suspect links, re-certifying after a source changes. An agent
  does that work for free. Humans are then spent on what they are actually for: domain
  expertise and judgment.

So the research output is not a comparison table. It is three lists: **adopt**, **automate**,
**keep human**.

This thesis is not in `process.md`. Its "Why" section argues from drift — requirements in
someone's head cannot be checked — and never says what agents are for or what humans are for.
That looks like a missing paragraph in the method itself, and a candidate amendment, but it is
the owner's framing to land rather than something to insert unilaterally.

### The research: what to adopt, what to automate, what stays human

Starting point: <https://en.wikipedia.org/wiki/Requirements_management>. Treat it as a lead, not
an authority — the article carries two "needs more citations" banners. The real sources it points
at are CMMI's split between **Requirements Development (RD)** and **Requirements Management
(REQM)**, IREB, PMI's *Requirements Management — A Practice Guide*, and Gotel & Finkelstein's
1994 "An Analysis of the Requirements Traceability Problem".

RM defines itself as *documenting, analyzing, tracing, prioritizing and agreeing on requirements,
then controlling change and communicating to stakeholders*, across five phases: Investigation,
Feasibility, Design, Construction and Test, Release.

Candidates, not conclusions. Each needs checking against the primary sources before it moves.

**Adopt — RM has this and the method is poorer without it:**

| Candidate | Why |
| --- | --- |
| **Prioritization** | In RM's one-sentence definition and absent here. Not currently a decision, just a hole: nothing says which requirement to satisfy first when effort is finite |
| **Bidirectional traceability, named as such** | Forward (requirement to code to test) exists via criteria; backward exists via `// Verifies:` citations. The pair has a name and a literature; use them |
| **Pre-requirements traceability** | RM's term for tracing a requirement to its origin and rationale. That is what task 3 is inventing. Adopt the name and see what else comes with it |
| **The four verification methods** | analysis, inspection, testing, demonstration. This method reaches for testing almost exclusively. Some requirements — a conduct policy, a license notice — are only ever verifiable by inspection, and saying so is better than recording a permanent gap |
| **RM's vocabulary generally** | *baseline*, *surrogate requirement*, *suspect link*, *RTM*. Free interoperability with everyone who already knows the field |

**Automate — toil that exists because a human had to do it by hand:**

| Toil in RM | What replaces it |
| --- | --- |
| **Maintaining the traceability matrix.** Famously the most-hated artifact in the discipline, hand-built and immediately stale | Derive it. Every citation is already in the code and every criterion names its requirement; `acceptance.md`'s table should be generated and checked, not typed |
| **Chasing suspect links.** RM flags a downstream artifact when its source changes, then a human reads everything flagged | An agent reads the source diff and proposes the downstream change; the human approves it. The flag is the start of the work, not the whole of it |
| **Elicitation as interviews.** RM's Investigation phase assumes meetings, because reading everything was infeasible | `context2spec` reads the code, the issue tracker, the docs and the vendored sources directly. Interviews remain for what is written nowhere |
| **Transcribing external prose into numbered requirements** | Agent-drafted, human-approved. Already how this works; RM treats it as skilled manual labor |
| **Keeping imported copies consistent** — which the literature says "must be carried out oneself" | A hash and a criterion. This is `AC-M1`, and it is free to run |
| **The "Big Freeze"** — organizations stop developing because re-certification costs too much | If every criterion is a test, re-certification is one command. This is the strongest argument the method has and it should be made in exactly these terms |

**Keep human — judgment and domain expertise, which is what people are for:**

| Stays human | Why |
| --- | --- |
| **Both gates.** Whether the requirements are the right ones, and whether the criteria really bite | An agent that approves its own requirements has written a tautology |
| **Conflict resolution.** The `CR-###` decision itself | An agent can surface the conflict, state the options and draft the record. Choosing which source loses is a business call |
| **Tier assignment** | Whether a source is mandatory or advisory is a legal and commercial judgment, not a textual one |
| **Risk acceptance** | Must be signed by someone who can be held to it, and must expire |
| **Goals and value** | Why a requirement exists, and what it is worth. Task 3 |
| **Retrieval an agent is refused** | Already twice today: iNaturalist and IBM both answer automated requests with a challenge or a 403. The rule that a human fetches what an agent cannot is load-bearing |

Also worth settling: CMMI splits RD from REQM. `context2spec` looks like RD and everything after
it looks like REQM. If that mapping holds it is a better spine for the phases than the current
three-part loop, and it comes with existing literature attached.

#### First pass, 2026-08-27: does RM cover versioned external sources?

**Yes — more than expected, and it has names for things this method invented independently.**
Checked against the Requirements traceability and Traceability matrix articles, both read in
full. Terms worth searching on later: *baseline*, *surrogate requirement*, *suspect link*,
*pre-requirements traceability*, *requirements traceability matrix (RTM)*.

Four concepts map almost directly:

| RM concept | The equivalent here |
| --- | --- |
| **Baseline** — an immutable approved snapshot; an RTM correlates "any two *baselined documents*", so that "when an item is changed in one baselined document, it is easy to see what needs to be changed in the other" | A spec tag, and a source pin in `PROVENANCE.md` |
| **Surrogate requirement** — RM tools import an external artifact so it can be traced with the tool's own machinery | Vendoring a source into `sources/<name>/` and transcribing it to `<name>/R#` |
| **Outdated surrogates** — named explicitly as the risk that the imported copy drifts from its origin | Exactly the stale `org-sec` pin fixed on 2026-08-27, and the reason `AC-M1` exists |
| **Suspect link** — when an upstream item changes, downstream links are flagged for re-verification | The framework's coverage check going red when `org-sec` moved to v1.1 |

External sources are squarely in scope for RM: traceability is *prescribed* by DO-178C, ISO
26262 and IEC 61508, which are themselves external standards. And on importing them, the
literature says the burden of keeping version and format consistent "must be carried out
oneself" — which is a fair description of what `sources.md` and `PROVENANCE.md` are for.

So the honest position is **not** that this method invented external-source management. It
reinvented a chunk of it — which, under the utility framing above, is fine: the question is
whether the mechanism works, not who got there first. Four things it does that RM does not, each
worth keeping for a stated reason rather than for being new:

- **Cryptographic integrity, not just a snapshot.** An RM baseline freezes a copy *inside the
  tool's database* and detects that someone edited a requirement object. A recorded SHA-256
  detects that the stored bytes no longer match what was recorded *for any reason* — including
  a formatter silently rewriting the file, which is the failure that produced amendment 1.
- **Quotation checking.** Nothing found so far in RM verifies that a quoted passage still
  appears *verbatim* in the source document. RM traces requirement-to-requirement links, not
  requirement-to-literal-text. `check-quotations.py` has no counterpart yet identified.
- **Precedence among sources.** RM resolves conflicting requirements through a change control
  board and communication. Tiers declare, in advance and in writing, which source outranks which.
- **The tool-free answer to a documented pathology.** RM literature records the "Big Freeze":
  organizations stop developing because re-certification costs too much. This method's bet is
  that a refresh is cheap when the check is a test, so the freeze never sets in. That is a
  direct response to a named failure and should be framed as one.

Not yet verified: the IBM DOORS documentation on suspect links and baselining external standard
modules returned 403 to automated retrieval, so the DOORS specifics above come from secondary
description only. Retrieve them by hand before relying on the detail — the method's own rule
about a human fetching what an agent cannot applies here.

Still open after this pass: whether RM has any notion of a source that is *implemented* rather
than merely complied with — the `org-sec` case from task 2 — and whether ISO/IEC/IEEE 29148
covers imported requirements more directly than the traceability literature does.

#### Second pass, 2026-08-27: CMMI REQM is a closer fit than the overview suggested

Read the CMMI process-area definitions. **REQM has exactly five specific practices, and four of
them already exist here under other names.** This is the best available spine for describing
what the method does, and it should probably replace the ad-hoc phase vocabulary.

| CMMI REQM practice | What it is here |
| --- | --- |
| SP 1.1 Understand Requirements | `context2spec`; the transcription of a vendored source into `<name>/R#` |
| SP 1.2 **Obtain Commitment to Requirements** | **The weak one.** `Owner` records who to ask for an exception; approval authority records who signs. Neither is an act of *commitment* by the people who have to deliver |
| SP 1.3 Manage Requirements Changes | `decisions.md`, `CR-###`, and the source-refresh cycle |
| SP 1.4 Maintain **Bidirectional** Traceability | `acceptance.md`'s table forward, `// Verifies:` citations backward. Arrived at independently, both directions present |
| SP 1.5 **Ensure Alignment Between Project Work and Requirements** | `implements.md`, almost exactly. Worth noting that CMMI made this a named practice and this method reinvented it as an artifact |

Four further findings, each with a consequence:

- **Verification and validation are different things, and only one is done here.** CMMI splits
  them: VER ensures "work products meet their specified requirements"; VAL demonstrates the
  product "fulfills its intended use when placed in its intended environment". Every criterion in
  `acceptance.md` is VER. **Nothing validates that a requirement was the right one to have.**
  Gate 1 is supposed to, but it is framed as review rather than as validation with its own
  criteria, and a gate with no criteria is a conversation. **Followed up in task 3** — validation
  needs a Goal to check against, which is what task 3 is for.
- **Peer review is a first-class practice, not a nicety.** VER SG 2 is *Perform Peer Reviews*,
  with three practices under it including analyzing peer review data. This method mentions
  review constantly and never specifies it.
- **Baselines and integrity are Configuration Management, not requirements work.** CM SP 1.3
  is *Create or Release Baselines* and CM SG 3 is *Establish Integrity*, including *Perform
  Configuration Audits*. So the pin, the hash and `AC-M1` are CM in CMMI's taxonomy. Useful for
  explaining the method to anyone who already thinks in these terms, and mild evidence that
  `sources.md` is doing two jobs that could be named separately.
- **REQM is Maturity Level 2 and RD is Maturity Level 3.** CMMI says you manage requirements
  before you get good at developing them, and files REQM under Project Management while RD is
  Engineering. That ordering is the reverse of how this method grew, and it is an argument for
  the rename: the mature, basic, do-this-first discipline is the one called *management*.

Prioritization has a home too: **RD SP 3.4, "Analyze Requirements to Achieve Balance"**. So the
gap identified in the adopt list is a named practice with literature behind it, not an invention.

Not yet read: ISO/IEC/IEEE 29148, the IREB CPRE syllabus, PMI's practice guide, and Gotel &
Finkelstein 1994. The CMMI material above is from the process-area summaries, not the CMU/SEI
technical report itself (CMU/SEI-2010-TR-033) — read that before quoting any of it as normative.

### The rename: `specman` → `reqman`

Short for *requirements manager*. Adopting the name is a claim of lineage, so do the research
first: it inherits the discipline's expectations, and readers will arrive expecting elicitation,
prioritization and baselines. Two arguments to weigh against it — the bundle is broader than
requirements (`arch.md` and `implements.md` are descriptive, not requirements at all), and
"prioritizing" is in RM's definition while being absent here.

The mechanical scope is smaller than it looks, because **canonical `process.md` never names
itself** — zero occurrences. So the base pin does not change and both project copies stay valid.

| Where | Occurrences | Note |
| --- | --- | --- |
| `README.md`, `SKILL.md`, `TODO.md` | 4 each | `SKILL.md` needs its `name:` and its trigger description reworded |
| `gemini-cli/spec/sources.md` | 5 | The source entry name, its heading, and the Origin URL |
| `birdsync/spec/sources.md` | 2 | Same |
| Both copies' `process.md` | 2 each | Banner only — the canonical URL and the `#specman--…` anchor the banner links to |
| `process.md` (canonical) | **0** | Nothing to change; the pin survives |

Two practical notes. Renaming the GitHub repository leaves a redirect, so
`github.com/Sajmani/specman` keeps resolving and existing provenance citations do not break —
but they should still be updated, since a redirect is not a record. And the skill is registered
by directory path, so renaming the directory de-registers it until it is re-installed.

**The artifact directory becomes `reqs/`.** Decided 2026-08-27, reversing the "leave it" default
recorded earlier the same day. `spec/` names the wrong thing once the method is called reqman —
and it was already the wrong name, since the directory holds sources, decisions, criteria and
conformance records, not a specification.

This is the expensive half of the rename, and unlike the rest of it the cost is real:

- Every project moves `spec/` to `reqs/` — currently gemini-cli and birdsync, plus the three
  task 2 examples.
- Canonical `process.md` says `spec/` throughout, so **this one does change the base pin**, and
  both project copies need re-pinning afterwards. The rest of the rename does not.
- Every criterion that reads a path moves with it: `check-pins.py`, `check-quotations.py`,
  `check-contrast.py`, and `spec_test.go`'s `sourcesDir`/`sourcesManifest` constants.
- `spec/process.md` is named in both `.prettierignore` and gemini-cli's `GEMINI.md`, and in the
  banner link in each copy.
- The three-phase names `context2spec`, `spec2test`, `spec2code` embed "spec" too. Renaming the
  directory without renaming the loop leaves the vocabulary half-migrated — decide both together
  or neither.

Sequence it after the research and after task 4's simplification pass, so the text is only
rewritten once.
