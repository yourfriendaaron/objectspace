# Objectspace Open Questions

**Status:** Open. Not doctrine.

Each question below names a decision that will otherwise be made accidentally by
implementation. The founding documents either assume both sides of the question
or do not raise it.

---

## 1. Edge provenance: derived or asserted?

The entire value proposition rests on the graph being accurate.

No founding document states where edges come from or what keeps them true.

Two origins exist, with different failure modes:

```text
derived    extracted from a source revision by tooling
asserted   authored by a human or an agent
```

Derived edges are recomputable and can be invalidated automatically. But if all
edges are derived from source text by language tooling, then Objectspace's
advantage over a cached language-server index is persistence and query surface,
not a new source of truth. That is a narrower claim than the proposal makes.

Asserted edges carry information no extractor can recover — `owned-by`,
`governed-by`, `supersedes`, intent, ownership, authority. They also drift
silently. An asserted edge that no longer matches reality is documentation rot
holding a database's authority.

Both kinds are needed. The documents use both without distinguishing them:

```text
architecture.md §12   defines, calls, implements, uses-type    → derived
proposal §13          contains, requires, governed-by,
                      verified-by                              → asserted
```

### Proposed resolution

Every `Edge` records its origin.

```text
Edge
    source
    relation
    target
    origin: Derived { from: RevisionId, extractor, extractor-version }
          | Asserted { provenance, verification-strategy }
```

Derived edges are invalidated when the RevisionId they were extracted from
changes. Asserted edges require an explicit, named verification strategy or they
are marked unverified in every query result that returns them.

The system must never present the two as equivalent.

### Until resolved

Do not build any feature that depends on edge accuracy without knowing which
kind of edge backs it.

### Prior art worth reading first

```text
Datomic    immutable facts, database-as-value, time-travel query
Unison     content-addressed code — ships what proposal §14 defers
```

---

## 2. Agent Comprehension Cost is unmeasured

ACC is named the primary engineering KPI in all three documents.

No document defines how to compute it.

Doctrine §8 states:

> If a dependency is forbidden, CI must reject it.
> If an invariant matters, verification must check it.
> Documentation alone is insufficient.

ACC currently fails that test. An unmeasured KPI cannot distinguish two
architectures, detect a regression, or falsify the core thesis.

### Proposed minimum benchmark

```text
fixed task suite      N well-specified changes, stable across runs
control               conventional repository — filesystem, git, grep
treatment             same system represented in Objectspace

measured per task     input tokens consumed
                      objects or files inspected
                      tool calls before first correct edit
                      wall-clock to passing verification
                      task success rate at fixed gates
```

Run it from Phase 1, when the control and treatment are still small enough to
represent both cheaply. The suite must exist before there is anything to be
defensive about.

### This is also the falsification test

If ACC does not drop measurably once the graph is available, the thesis is
weaker than claimed. That should be discovered in month three, not year three.

Context windows are growing and retrieval is improving. Comprehension cost is
the part of the Objectspace argument most exposed to model progress. The
benchmark is how the project finds out whether that erosion is happening.

---

## 3. Projection gravity

Risk §40 names ecosystem incompatibility and mitigates it with "strong
import/export and POSIX projection."

That mitigation has a dynamic the documents do not acknowledge:

```text
better projection
      ↓
less incentive to move anything native
      ↓
projection becomes the operative interface
      ↓
engineering effort concentrates at the boundary permanently
```

Every compiler, linker, package manager, test runner, debugger and CI system
speaks pathnames.

Bootstrap Phases 6-7 require either:

```text
project source back out to disk so cargo can run
    → the file tree remains the operative reality, plus a layer

or reimplement the toolchain boundary
    → a violation of doctrine §11, prefer reuse beneath the innovation
```

WinFS is the relevant precedent. It did not fail because the object model was
wrong.

### Open

- Which is the honest Phase 6 plan?
- Is there a third option — toolchain adapters that read from Objectspace
  directly, contributed upstream rather than reimplemented?
- What is the acceptable steady-state cost of the boundary?

### Proposed constraint

Make projection dependence a measured number rather than a vibe. Define it and
track it from the first day a projection exists:

```text
projection dependence =
    operations in a development session that touch a POSIX projection
    ÷ total operations
```

Target: monotonically decreasing across phases. If it plateaus high, the
compatibility layer has become the product and the roadmap should say so
explicitly rather than continuing to describe projections as temporary.

---

## 4. The two MVPs disagree

The founding documents specify two different first milestones.

```text
architecture.md §20              proposal §31
─────────────────────────        ─────────────────────────
create object                    objects: create/read/revise/tombstone
immutable revision               types and schemas
typed relationship               relationships
begin / commit transaction       transactions
produce WorldRevision            world revisions
query graph                      queries
fork world                       history
revise object                    capabilities (deny-by-default)
compute impact                   agent API
compare worlds                   Wasm runtime + one application
abort safely                     file import/export
rollback exactly

"Nothing else is required                 ← adds schema versioning,
 to prove the core model."                  capability system, Wasmtime,
                                            WIT, MCP, compat layer
```

The second is a multi-person-year program at the verification bar the doctrine
sets (property tests, fuzzing, mutation testing, Kani, selective Verus).

The first is buildable and proves the thesis.

### Recommendation

`architecture.md §20` is the MVP. Proposal §31 is a Phase 4-6 description and
should be relabelled as such so no agent reads it as the first milestone.

Capabilities, Wasm and MCP are all defensible next steps. None of them are
required to demonstrate that a semantic object graph with immutable worlds
lowers the cost of change.

---

## 5. Framing: which half is durable?

Two distinct claims are bundled together in the founding documents.

```text
comprehension    agents spend less context reconstructing structure
coordination     agents can speculate, verify, roll back and merge safely
```

The comprehension claim erodes as models improve.

The coordination claim strengthens as agents become more numerous and more
autonomous. Transactional isolation, exact rollback, capability scoping and
provenance are safety properties, not comprehension aids. Ten agents working
candidate worlds behind verified merge gates is a problem that gets harder with
better models, not easier.

Both claims are worth making. The order matters for what gets built first and
what the first demonstration shows.

### Open

Should the project lead with fail-safe speculative execution for parallel
agents, and treat reduced comprehension cost as the second-order effect?

The MVP in `architecture.md §20` already demonstrates the coordination claim in
full. It demonstrates the comprehension claim only by assertion.
