# Objectspace Doctrine

Objectspace is a persistent semantic computing environment in which durable state is represented as typed, identity-stable objects and explicit relationships rather than pathname-addressed files arranged in directory trees.

Files and folders may exist as compatibility projections, import/export formats, or human-facing views. They are never canonical state.

Objectspace is designed for a future in which software agents are primary operators of computers and major creators of software. The system must therefore optimize for machine comprehension, deterministic verification, explicit semantics, transactional modification, and extremely small blast radii.

## Core Principles

### 1. Identity over location

Objects have stable identity independent of:

- names;
- views;
- presentation;
- storage location;
- organization.

Renaming or reorganizing an object must not break references to it.

### 2. Relationships over containment

System structure is represented through explicit typed relationships such as:

```text
depends-on
implements
tested-by
owned-by
derived-from
configured-by
produces
consumes
governed-by
```

Containment is only one possible relationship.

### 3. Objects over files

The canonical entity is a typed object.

Text, JSON, source code, images, PDFs, and other formats are representations of objects, not necessarily their fundamental identity.

### 4. Immutable revisions over destructive mutation

Conceptual objects have stable `ObjectId`s.

Every state of an object has an immutable `RevisionId`.

Changes create new revisions.

Committed history is never silently overwritten.

### 5. Worlds over mutable global state

A `WorldRevision` represents one internally consistent logical state of Objectspace.

Changes occur in candidate transactions or derived worlds.

Only verified changes become committed worlds.

Failed work must not partially alter committed state.

### 6. Explicit capabilities over ambient authority

Applications and agents receive only explicitly granted capabilities.

Authority is deny-by-default.

An application that has not been granted access to a resource must be unable to access it.

### 7. Semantics over serialization

Objectspace should reason about what an object is, not merely how its bytes happen to be encoded.

Serialization is an interchange or compatibility boundary.

### 8. Machine-enforced architecture over convention

Critical architectural rules must be executable.

If a dependency is forbidden, CI must reject it.

If an invariant matters, verification must check it.

Documentation alone is insufficient.

### 9. Local reasoning over global archaeology

A well-designed change should require understanding the smallest possible semantic closure.

The system should continuously reduce **Agent Comprehension Cost**:

> the amount of context an autonomous agent must inspect or infer before safely making a change.

### 10. Speed through strong structure

Safety and velocity are not opposing goals.

Clear architecture, reusable primitives, explicit contracts, immutable history, strong typing, and deterministic verification should enable faster autonomous iteration.

### 11. Prefer reuse beneath the innovation

Novelty belongs in the Objectspace machine model.

Prefer mature infrastructure for:

- hardware support;
- operating-system substrate;
- storage;
- compilation;
- WebAssembly;
- cryptography;
- networking;
- testing.

Do not reinvent infrastructure merely to make Objectspace appear more novel.

### 12. Keep the trusted core small

The semantic kernel should remain minimal.

Complexity should live outside the trusted core whenever possible.

### 13. Prefer obvious implementations

For agentically engineered code:

- explicit is better than magical;
- shallow is better than deeply abstract;
- typed is better than stringly typed;
- one obvious path is better than several equivalent patterns;
- small modules are better than sprawling frameworks;
- deletion is better than unnecessary generalization.

### 14. Compatibility flows outward

Objectspace may project objects into:

```text
files
directories
POSIX paths
JSON
source trees
legacy APIs
```

Compatibility layers may depend on Objectspace.

Objectspace core must never depend on compatibility projections.

### 15. Agents must be able to fail safely

Dangerous changes must be:

- transactional;
- isolated;
- inspectable;
- verifiable;
- abortable;
- reversible.

Speculative agent work should be cheap.

### 16. Verification must be independent

The agent that implements a change is not sufficient evidence that the change is correct.

Important work should be checked through some combination of:

- compiler enforcement;
- independent tests;
- property testing;
- fuzzing;
- mutation testing;
- adversarial review;
- model checking;
- selective formal verification.

### 17. Objectspace should progressively improve its own engineerability

The bootstrap implementation will begin inside a conventional filesystem.

That is temporary scaffolding.

The long-term objective is for Objectspace itself to be developed through Objectspace-native primitives such as:

```text
object.query
graph.impact
world.fork
transaction.diff
program.verify
```

As more of the system becomes native, autonomous engineering should become faster and safer.

## Foundational Invariants

The following must always hold:

1. Object identity is stable.
2. Revisions are immutable.
3. Every committed world is internally consistent.
4. Failed transactions cannot mutate committed state.
5. Persistent relationships resolve to valid identities or explicit tombstones.
6. Capabilities are deny-by-default.
7. Authority cannot silently expand.
8. Every committed change has provenance.
9. Compatibility projections are never canonical state.
10. Core dependency rules are machine-enforced.
11. Canonical identity never depends on a pathname.
12. Any committed world can be rolled back without application-specific reconstruction.

## Primary Engineering KPI

### Agent Comprehension Cost

Every architectural decision should be evaluated partly by:

> How much context must an autonomous agent reconstruct before it can make a correct change?

Objectspace should prefer architectures that expose the relevant semantic closure directly.

The system succeeds when agents spend progressively less effort understanding machinery and progressively more effort solving the actual problem.

## Final Rule

Objectspace is not a better filesystem.

It is a different computing model.

**Unix made everything look like a file.**

**Objectspace should make everything exist as what it actually is.**