# Objectspace

## Proposal for an Agent-Native Computing Model

**Status:** Founding proposal  
**Objective:** Design and build a general-purpose computing environment in which persistent state is represented as typed, identity-stable semantic objects and relationships rather than pathname-addressed files arranged into hierarchical folders.

**Companion documents:**

- [`docs/doctrine.md`](docs/doctrine.md) — non-negotiable philosophy and foundational invariants
- [`docs/architecture.md`](docs/architecture.md) — layering, dependency rules, and the bootstrap sequence
- [`docs/open-questions.md`](docs/open-questions.md) — unresolved decisions in this proposal, not yet doctrine

---

# 1. Executive Summary

Objectspace is a proposal to replace the traditional filesystem as the primary abstraction through which programs, humans, and software agents interact with persistent computer state.

Traditional computers organize durable information primarily as:

```text
pathname → byte stream
```

For example:

```text
/home/user/projects/foo/src/auth/login.ts
```

The location of an artifact is part of its identity. Relationships between artifacts are usually implicit. Programs reconstruct meaning from filenames, directory structures, configuration files, import paths, source text, package manifests, databases, and conventions layered on top of the filesystem.

Objectspace replaces this model with:

```text
identity → typed object → immutable revision → explicit relationships
```

An authentication module might instead be represented as:

```text
Object: AuthenticationService
Type: SoftwareComponent
Revision: BLAKE3:...
Implements: Authentication
DependsOn:
    SessionService
    UserStore
TestedBy:
    AuthenticationContractTests
OwnedBy:
    CustomerPortal
```

There is no canonical pathname.

There does not need to be a canonical file.

Files and folders may still exist as compatibility projections, import/export formats, or human views, but they cease to be the source of truth.

Objectspace is designed particularly for a future in which **software agents are major operators of computers and major creators of software**.

Instead of requiring an agent to explore thousands of files and reconstruct architecture from textual clues, Objectspace exposes the actual semantic structure of the computer directly:

- what an object is;
- what depends on it;
- what it depends on;
- what capabilities it possesses;
- what changed;
- who or what produced it;
- which tests govern it;
- which application owns it;
- which revisions exist;
- and what the blast radius of a proposed modification would be.

The long-term ambition is larger than creating a better storage system.

Objectspace proposes a different **machine model**:

> Humans and agents manipulate semantic objects directly. Storage location, serialization, dependency resolution, version history, sandboxing, and much of application plumbing become services of the computing environment rather than responsibilities repeatedly reconstructed by individual applications.

Objectspace itself should be engineered almost entirely by software agents.

This is not incidental. **Agentic engineerability is a core architectural constraint.**

The project should therefore optimize aggressively for:

- architectural transparency;
- machine-verifiable invariants;
- deterministic interfaces;
- explicit dependencies;
- small blast radii;
- strong typing;
- immutable history;
- automatic rollback;
- capability-based security;
- simple build and verification commands;
- parallel agent execution;
- and extremely low agent comprehension cost.

The recursive goal is important:

> The first Objectspace must be built by agents operating through a traditional filesystem. Mature Objectspace should make those same agents dramatically better at engineering Objectspace itself and creating entirely new applications.

---

# 2. The Problem

The hierarchical filesystem is one of computing's most durable abstractions.

It is also an extraordinarily weak representation of meaning.

A conventional filesystem natively knows very little:

```text
name
parent directory
byte contents
timestamps
permissions
```

Almost everything else has to be inferred or rebuilt elsewhere.

Consider a modern software repository:

```text
README.md
package.json
src/
lib/
components/
routes/
services/
auth.ts
auth.test.ts
Dockerfile
tsconfig.json
.env.example
```

To understand that system, a coding agent must independently determine:

- which files are authoritative;
- what architectural layers exist;
- which modules implement which capabilities;
- which tests correspond to which behavior;
- which imports represent important dependencies;
- what can safely be changed;
- what configuration governs production;
- how the application is deployed;
- which files are generated;
- which files are obsolete;
- which interfaces are public;
- which assumptions are merely conventions.

Modern tooling repeatedly compensates for this deficiency by constructing richer representations over the filesystem:

```text
Filesystem
   ↓
Git object graph
   ↓
Package dependency graph
   ↓
IDE semantic index
   ↓
Language server
   ↓
Build graph
   ↓
Container image
   ↓
Deployment manifest
   ↓
Observability graph
```

Each subsystem reconstructs meaning that the underlying computer does not intrinsically understand.

Objectspace asks:

> What if the computer's native persistent model contained this structure directly?

---

# 3. Core Thesis

A computer does not inherently require files.

Physical storage ultimately exposes lower-level mechanisms resembling blocks, pages, sectors, memory regions, or object storage primitives.

Filesystems are software interpretations layered over those primitives.

Therefore:

> Files can be removed as a foundational abstraction without removing persistent storage.

Objectspace replaces:

```text
hierarchical path
+
opaque byte stream
```

with:

```text
stable identity
+
type
+
immutable revision
+
structured payload
+
explicit graph relationships
+
capabilities
+
provenance
```

---

# 4. Objectspace Philosophy

## 4.1 Identity over location

An object is identified by what it **is**, not where it happens to be displayed.

Moving something through a UI does not change its identity.

Renaming it does not invalidate references.

Reorganizing a project does not break dependencies.

---

## 4.2 Relationships over containment

Directories encode essentially one relationship:

```text
A is inside B
```

Objectspace should support arbitrary typed relationships:

```text
implements
depends-on
tested-by
owned-by
derived-from
configured-by
deployed-as
documents
supersedes
authorized-by
produces
consumes
```

Containment may still exist when useful, but it is one relationship among many.

---

## 4.3 Views over folders

A human may still prefer something resembling:

```text
My Projects
    CRM
    Travel Platform
```

That should be a **view**, not physical organization.

A view might mean:

```text
objects
where owner = me
and type = Application
order by modified descending
```

The same object can appear simultaneously in:

```text
Customer Portal
Recently Modified
Authentication
Needs Review
Created by Agent 17
Production
```

without copying or symbolic-link semantics.

---

## 4.4 Immutable history over destructive mutation

Objects possess stable conceptual identities, but individual revisions are immutable.

```text
ObjectId
    │
    ├── Revision 41
    ├── Revision 42
    └── Revision 43
```

An update creates a revision.

It does not overwrite history.

---

## 4.5 Explicit authority over ambient authority

Programs should receive only the capabilities they require.

An application does not automatically possess:

```text
filesystem access
internet access
camera access
microphone access
email access
database access
```

Instead it receives explicit authority:

```text
CustomerStore.read
InvoiceStore.write
Camera.capture
Network(api.stripe.com).connect
Email.send(from=sales@example.com)
```

What has not been granted cannot be used.

---

## 4.6 Semantics over serialization

A configuration object is not fundamentally JSON.

A document is not fundamentally DOCX.

A program is not fundamentally `.rs`.

Those may be useful representations.

The canonical entity is the object.

Serialization is a projection.

---

## 4.7 Machine-readable architecture over convention

Important architectural facts must not exist only in documentation.

If:

```text
Graph
```

may depend on:

```text
Storage
```

but Storage may not depend on Graph, the system should enforce that relationship mechanically.

Architecture should increasingly become executable policy.

---

# 5. Agent-Native Design

Objectspace should be explicitly designed around software agents as first-class operators.

This creates requirements different from those of human-first operating systems.

A human often benefits from visual familiarity and spatial organization.

An agent benefits disproportionately from:

- exact identity;
- typed interfaces;
- dependency graphs;
- structured output;
- explicit invariants;
- transactional operations;
- deterministic queries;
- precise blast-radius analysis.

Objectspace should optimize for both, but the underlying model should favor semantic precision.

---

# 6. Agent Comprehension Cost

Objectspace should introduce a project-wide engineering metric:

## Agent Comprehension Cost — ACC

**ACC is the amount of system context an autonomous agent must inspect or infer before safely performing a change.**

For example:

### Conventional repository

Changing authentication may require reading:

```text
71 files
11 configuration files
4 README sections
3 package manifests
28 search results
```

before the agent reasonably understands the consequences.

### Objectspace

The agent asks:

```text
graph.impact(Authentication)
```

and receives:

```text
AuthenticationInterface
PasswordAuthenticator
SessionService
CustomerPortal
AdminPortal
AuthPolicy
AuthenticationContractTests
```

The relevant semantic closure may contain eight objects.

That reduction directly converts:

```text
tokens
+
model reasoning
+
exploration time
```

into:

```text
engineering throughput
```

ACC should therefore become a first-class architectural KPI.

---

# 7. Fundamental Data Model

Every persistent entity should be represented through a small number of universal concepts.

## 7.1 Object

A conceptual entity with stable identity.

Example:

```text
ObjectId: 018F...
Type: SoftwareComponent
Name: SessionValidator
```

`ObjectId` remains stable over the lifetime of the conceptual object.

---

## 7.2 Revision

A particular immutable state of an object.

```text
RevisionId: BLAKE3(...)
ObjectId: 018F...
ParentRevision: ...
SchemaVersion: 3
Payload: ...
```

Changing an object creates a new revision.

---

## 7.3 Type

Objects possess explicit semantic types.

Examples:

```text
Application
SoftwareComponent
Interface
Function
Schema
Test
Policy
Workflow
Image
Document
Dataset
Credential
User
Device
Deployment
```

Object types are schema-governed.

---

## 7.4 Edge

Typed relationships connect objects.

```text
source: AuthService
relation: depends-on
target: SessionStore
```

Edges should themselves be capable of carrying metadata where necessary.

---

## 7.5 Capability

A capability represents authority to perform a particular action against a particular resource or interface.

Capabilities should be unforgeable and deny-by-default.

---

## 7.6 Provenance

Important revisions should record where they came from.

Potential fields include:

```text
created-by
derived-from
agent
model
task
transaction
timestamp
verification-result
source-object
```

Objectspace should be able to answer:

> Why does this exist?

---

# 8. World Revisions

Objectspace should version not only individual objects but the **logical state of the machine**.

Define:

```text
WorldRevision
```

as a consistent mapping from ObjectIds to visible RevisionIds plus relevant graph state.

For example:

```text
World 417
```

represents one complete committed logical state.

Agents do not directly mutate World 417.

They derive candidate worlds:

```text
World 417
   ├── Candidate A
   ├── Candidate B
   └── Candidate C
```

After validation:

```text
Candidate B
    ↓
World 418
```

This creates extremely powerful properties:

- transactional system-wide changes;
- atomic application upgrades;
- immediate rollback;
- speculative agent development;
- reproducible execution;
- branching;
- deterministic testing;
- parallel engineering.

---

# 9. Transaction Model

All meaningful mutation should happen transactionally.

An agent should be able to:

```text
transaction.begin
object.modify
object.create
edge.add
edge.remove
program.build
program.test
transaction.diff
transaction.commit
```

Until commit, the current world remains unchanged.

If verification fails:

```text
transaction.abort
```

No partially modified state leaks into the working environment.

---

# 10. Proposed Architecture

The initial architecture should be layered approximately as follows:

```text
┌──────────────────────────────────────────────────┐
│               HUMAN / AGENT SURFACES             │
│                                                  │
│ UI · IDE · CLI · MCP · Native Agent Protocol     │
└────────────────────────┬─────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────┐
│                OBJECTSPACE KERNEL                │
│                                                  │
│ Identity                                         │
│ Schemas                                          │
│ Semantic graph                                   │
│ Query engine                                     │
│ Transactions                                     │
│ World revisions                                  │
│ Capabilities                                     │
│ Provenance                                       │
└──────────────┬──────────────────┬────────────────┘
               │                  │
┌──────────────▼──────────┐ ┌─────▼─────────────────┐
│   COMPONENT RUNTIME     │ │     OBJECT STORE      │
│                         │ │                       │
│ Wasmtime                │ │ Immutable revisions   │
│ WebAssembly Components  │ │ Graph indexes         │
│ WIT interfaces          │ │ Transaction log       │
│ Capability injection    │ │ Content blobs         │
└──────────────┬──────────┘ └─────┬─────────────────┘
               │                  │
               └────────┬─────────┘
                        │
┌───────────────────────▼──────────────────────────┐
│                 LINUX SUBSTRATE                  │
│                                                  │
│ Drivers · scheduler · VM · networking · GPU      │
└──────────────────────────────────────────────────┘
```

The phrase **Objectspace Kernel** initially refers to the semantic core, not a replacement hardware kernel.

---

# 11. Bootstrap Strategy

Objectspace must not attempt to emerge fully formed.

It should bootstrap incrementally.

## Stage 0 — Conventional development

Agents work in a normal repository:

```text
Git
Cargo
Rust files
Linux/macOS
```

The implementation itself is conventional.

---

## Stage 1 — Objectspace-in-a-file

The first Objectspace store lives in something like:

```text
objectspace.img
```

To the host OS this is one ordinary file.

Internally it contains:

```text
objects
revisions
edges
schemas
indexes
transactions
worlds
```

No Objectspace user needs to know that the host file exists.

---

## Stage 2 — Objectspace becomes the application model

Native applications store state as Objectspace objects rather than ordinary application files.

Agents interact through the Objectspace API.

Linux remains underneath.

---

## Stage 3 — WebAssembly application runtime

Applications become Wasm Components executed through Objectspace.

Their authority comes entirely from capabilities supplied by Objectspace.

---

## Stage 4 — Objectspace-native development

The Objectspace source code itself is imported into Objectspace.

Agents stop relying primarily on:

```text
grep
find
cat
git diff
```

and begin relying on:

```text
object.query
graph.dependencies
graph.impact
world.fork
transaction.diff
program.verify
```

---

## Stage 5 — Self-hosting

Objectspace builds newer Objectspace revisions from within Objectspace.

The original conventional repository becomes bootstrap infrastructure.

---

## Stage 6 — Direct block storage

The storage backend may optionally move from a host database file to a raw storage partition.

The traditional filesystem is removed from Objectspace persistence.

---

## Stage 7 — Optional kernel replacement

Only if objectively justified, Linux may eventually be replaced with:

```text
Objectspace microkernel
+
existing compatible driver strategy
```

This should not be an early project objective.

Linux can remain an invisible hardware compatibility substrate indefinitely if that produces better velocity and reliability.

---

# 12. Proposed Implementation Stack

## Core language

**Rust**

Rust should implement the trusted Objectspace core because it provides:

- memory safety;
- high performance;
- strong type guarantees;
- exhaustive enums;
- predictable resource management;
- excellent WebAssembly integration;
- low-level hardware access when eventually needed;
- strong testing and verification tooling.

One implementation language across most of the trusted core also dramatically reduces agent context switching.

---

## Repository

**Cargo workspace monorepo**

Bootstrap development should deliberately favor simplicity.

Initial repository:

```text
objectspace/
    crates/
        identity/
        schema/
        store/
        graph/
        transaction/
        world/
        capability/
        query/
        runtime/
        agent-api/
        compatibility/
    apps/
    tests/
    xtask/
```

The exact boundaries should evolve, but dependency direction must be machine-enforced.

---

## Bootstrap persistence

**redb**, hidden behind an Objectspace-owned storage interface.

Objectspace should never expose redb semantics to higher layers.

Example:

```rust
trait ObjectStore {
    fn get_revision(...);
    fn put_revision(...);
    fn query_edges(...);
    fn begin_transaction(...);
    fn commit(...);
}
```

Later implementations might include:

```text
RedbBackend
MemoryBackend
RawBlockBackend
DistributedBackend
```

---

## Runtime

**Wasmtime**

Objectspace applications should eventually execute primarily as WebAssembly Components.

Benefits include:

- strong isolation;
- portable execution;
- language independence;
- deterministic interfaces;
- explicit host capabilities;
- resource governance;
- embeddability.

---

## Interface definition

**WIT**

WIT should define stable application/service contracts.

Example:

```wit
interface contacts {
    record contact {
        id: string,
        name: string,
        email: option<string>,
    }

    get: func(id: string) -> option<contact>;
    create: func(name: string) -> contact;
}
```

An application declares what it imports and exports.

Objectspace can then reason explicitly about those contracts.

---

# 13. Application Model

An Objectspace-native application should itself be a graph.

Example:

```text
CRM
 │
 ├── contains → ContactSchema
 ├── contains → CompanySchema
 ├── contains → ContactListView
 ├── contains → CompanyView
 ├── contains → ReminderWorkflow
 │
 ├── requires → GoogleIdentity
 ├── requires → NotificationService
 │
 ├── governed-by → CRMPolicy
 │
 └── verified-by → CRMContractTests
```

An application therefore becomes much more than an executable blob.

Objectspace understands its architecture.

---

# 14. Code Representation

Objectspace should resist the temptation to invent a new programming representation too early.

## Initial representation

Source code can remain textual.

Example:

```text
Object:
    type: RustModule
    name: auth
    content: "pub fn authenticate(...) ..."
```

This is already fundamentally different from a file.

It has stable identity and relationships.

---

## Derived semantic structure

Language tooling can derive:

```text
defines
calls
implements
imports
uses-type
tested-by
```

from the source.

This information can populate the graph.

---

## Long-term possibility

Later, Objectspace may experiment with canonical structured code:

```text
Function
    Parameters
    ReturnType
    Body AST
```

Text becomes a rendered editing representation.

However, this should happen only after evidence demonstrates that structured canonical code materially improves the system.

Objectspace should not require the simultaneous invention of a new programming language.

---

# 15. Agent Interface

The primary machine interface should be typed and semantic.

A compatibility MCP server should be provided early so existing frontier agents can use Objectspace immediately.

Long term, Objectspace may expose its own optimized native agent protocol.

Core operations might include:

```text
object.get
object.query
object.create
object.revise

schema.describe
schema.validate

graph.dependencies
graph.dependents
graph.path
graph.impact

transaction.begin
transaction.diff
transaction.commit
transaction.abort

world.get
world.fork
world.compare
world.rollback

program.build
program.test
program.run

capability.inspect
capability.request

provenance.trace
```

Outputs should be structured objects, not prose or terminal text.

---

# 16. Why Objectspace Is Particularly Powerful for Agents

## Exact dependency knowledge

Agents no longer need to infer dependencies primarily through search.

They ask the graph.

---

## Exact blast-radius calculation

Before a change:

```text
graph.impact(SessionValidator)
```

can return every semantically dependent object.

---

## Cheap speculative execution

An agent can fork a world, perform a change, build it, execute tests, inspect behavior, and discard the candidate world without altering production state.

---

## Better parallelism

Multiple agents can operate against independent candidate worlds.

Semantic merges can eventually reason about objects and contracts rather than merely textual line ranges.

---

## Better debugging

The system can answer:

```text
which revision introduced this behavior?
what task created it?
what agent authored it?
what tests verified it?
which world first contained it?
what changed between those worlds?
```

without reconstructing that information from Git, CI logs, and deployment systems.

---

## Smaller context requirements

Agents can inspect only the relevant semantic closure.

This directly reduces ACC.

---

# 17. Agentic Engineering Doctrine

Objectspace should be engineered under the hard constraint that essentially all production code may be authored by agents.

Humans may:

- define product goals;
- establish architectural doctrine;
- define acceptance criteria;
- approve major architectural decisions;
- evaluate user experience;
- prioritize work.

Agents should perform:

- implementation;
- refactoring;
- test generation;
- documentation;
- migrations;
- debugging;
- routine architecture changes;
- performance optimization;
- code review;
- adversarial review;
- release preparation.

---

# 18. Rules for an Agent-Buildable Codebase

## Rule 1 — Prefer boring technology

Novelty belongs in Objectspace itself.

Dependencies underneath it should be mature, documented, and predictable.

---

## Rule 2 — Minimize abstraction depth

Agents should not have to traverse eight wrappers to discover actual behavior.

---

## Rule 3 — Keep dependency direction obvious

Architecture should resemble:

```text
identity
   ↑
store
   ↑
graph
   ↑
transaction
   ↑
capability
   ↑
runtime
   ↑
applications
```

Cycles and forbidden dependencies should fail CI.

---

## Rule 4 — One obvious way to perform common operations

Avoid competing test runners, build systems, configuration systems, or architecture patterns.

---

## Rule 5 — Local reasoning should usually be sufficient

A modification to a subsystem should require understanding the smallest possible surface.

---

## Rule 6 — Encode invariants as code

Documentation is insufficient for critical rules.

---

## Rule 7 — No silent architectural magic

Dependency injection, reflection, code generation, macros, and hidden runtime registration should be used cautiously.

Agent-visible behavior should remain easy to trace.

---

## Rule 8 — Prefer schemas and enums over free-form strings

Invalid states should increasingly become unrepresentable.

---

## Rule 9 — Every dangerous operation is transactional

An agent should be able to fail safely.

---

## Rule 10 — Optimize for deletion

The simpler implementation that satisfies the invariant is preferable.

---

# 19. Development Workflow

A typical agentic change should look like:

```text
Task
  ↓
Impact analysis
  ↓
Candidate world / branch
  ↓
Implementation
  ↓
Local verification
  ↓
Independent test agent
  ↓
Adversarial review
  ↓
Full verification
  ↓
Commit
```

The implementation agent should not be considered sufficient evidence of correctness.

---

# 20. Multi-Agent Roles

Important changes should use independent roles.

```text
Specification Agent
        ↓
Implementation Agent
        ↓
Verification Agent
        ↓
Adversarial Agent
        ↓
Simplification Agent
        ↓
Merge Gate
```

Different model families may be used where useful to reduce correlated failure modes.

---

# 21. Verification Strategy

The verification architecture is central to making agentic development credible.

## Compiler verification

Rust itself acts as a major correctness gate.

---

## Unit testing

Fast deterministic unit tests for local behavior.

---

## Contract testing

Interfaces should carry executable behavioral contracts wherever practical.

---

## Property testing

Objectspace should test invariants rather than only examples.

For example:

```text
For any committed transaction:

every visible ObjectId resolves to exactly one visible revision.

A failed transaction cannot alter the visible WorldRevision.

An object revision is immutable after creation.

A capability cannot gain broader authority through delegation.

World rollback reproduces the previous logical state.
```

---

## Fuzz testing

Priority targets:

```text
serialization
query parsing
transaction recovery
graph operations
Wasm boundaries
capability validation
storage corruption handling
```

---

## Mutation testing

Agent-written tests must be tested themselves.

If deliberate behavioral mutations do not fail the suite, the suite is insufficient.

---

## Model checking

Kani should be considered for critical Rust components involving:

```text
transaction logic
capabilities
unsafe code
serialization
allocator structures
revision visibility
```

---

## Formal verification

Verus or equivalent tools should be selectively used where the consequences justify the cost.

Formal proof should not become a project-wide requirement.

Speed remains important.

---

# 22. One Verification Command

The bootstrap repository should expose one canonical command:

```bash
cargo xtask verify
```

This should orchestrate increasingly expensive gates:

```text
format
compile
architecture validation
lint
unit tests
contract tests
integration tests
property tests
mutation tests
fuzz corpus
model checks
selected proofs
```

Agents should not need to memorize project folklore.

---

# 23. Security Model

Objectspace should be capability-oriented from inception.

Security should not be retrofitted later.

The objective is not merely stronger security.

Capability security also makes agent-generated software substantially easier to trust.

An agent-created weather application receiving only:

```text
Network(weather.example).connect
Location.city.read
UI.render
```

cannot secretly read:

```text
Email
Contacts
Credentials
OtherApplications
Microphone
```

because those resources are simply absent from its world.

---

# 24. Permissions as Semantic Objects

Policies should themselves be represented explicitly.

Example:

```text
Role: Salesperson
    can → Contact.read
    can → Contact.write

Role: SalesManager
    inherits → Salesperson
    can → Contact.delete
```

Objectspace can inspect and reason over these policies before execution.

---

# 25. Application Creation

One of Objectspace's strongest long-term use cases is dramatically faster software generation.

Today, asking an agent to create a CRM may produce:

```text
repository
package manager
framework configuration
directory structure
database ORM
API routes
authentication integration
permission middleware
UI framework
configuration files
Dockerfile
deployment configuration
tests
```

Much of this is repeated machinery.

Objectspace should move common application infrastructure into the platform.

An agent could instead construct:

```text
Application CRM

Schema Contact
Schema Company
Schema Note
Schema Reminder

Relationship Contact.Company

Policy SalesAccess

Capability GoogleIdentity

View ContactList
View ContactDetail
View CompanyDetail

Workflow ReminderNotification
```

The platform already supplies:

```text
identity
persistence
transactions
versioning
permissions
querying
deployment
rollback
capability isolation
audit history
```

Application engineering becomes increasingly focused on **novel domain behavior** rather than plumbing.

---

# 26. Example Agent Workflow

User:

> Create a basic internal CRM with companies, contacts, notes, reminders and Google authentication. Managers can delete contacts but normal salespeople cannot.

Agent:

```text
world.fork

application.create CRM

schema.create Company
schema.create Contact
schema.create Note
schema.create Reminder

graph.link Contact belongs-to Company
graph.link Note belongs-to Contact

policy.create Salesperson
policy.grant Salesperson Contact.read
policy.grant Salesperson Contact.write

policy.create SalesManager
policy.inherit SalesManager Salesperson
policy.grant SalesManager Contact.delete

capability.attach GoogleIdentity

view.create ContactList
view.create ContactDetail
view.create CompanyDetail

workflow.create ReminderNotification

program.generate-tests
program.verify

transaction.commit
```

This could eventually create an entire useful application in one atomic transaction.

---

# 27. Native Human Interface

Objectspace should not force humans to think in database queries.

The human UI can preserve familiar metaphors where useful.

Possible surfaces include:

## Collections

Saved semantic views resembling folders.

---

## Search

Universal structured and natural-language search.

---

## History

Every object has native revision history.

---

## Relationships

Users can inspect:

```text
Used By
Created From
Belongs To
Related To
Produced By
```

---

## Activity

A system-wide history of meaningful semantic changes.

---

## Application spaces

Applications may present customized views over their object graphs.

---

# 28. Compatibility

Objectspace should not declare war on files.

It should subordinate them.

## Import

```text
JPEG → Image Object
PDF → Document Object
JSON → Structured Data Object
source tree → Software Objects
```

---

## Export

```text
Image Object → JPEG
Document Object → PDF
Application → source tree
Dataset → CSV
```

---

## File projection

Legacy software may receive a temporary virtual filesystem.

For example, an object can be projected as:

```text
/tmp/objectspace-projection/config.json
```

The canonical object remains:

```text
Configuration Object #A19F
```

The pathname is ephemeral compatibility infrastructure.

---

# 29. POSIX Compatibility

A later compatibility subsystem may expose:

```text
open
read
write
directory traversal
```

against generated views of Objectspace.

This must remain a boundary adapter.

Objectspace internals must never begin depending on POSIX path semantics.

---

# 30. Initial Non-Goals

Objectspace should explicitly refuse several tempting projects during the bootstrap phase.

Do **not** initially:

- write a hardware kernel;
- write device drivers;
- invent a programming language;
- make ASTs canonical;
- create a distributed database;
- create a novel GUI toolkit;
- replace Git before Objectspace is capable of doing so;
- design a blockchain;
- solve global distributed consensus;
- implement every application category;
- optimize prematurely for billions of objects.

The first task is proving the machine model.

---

# 31. MVP

The first meaningful Objectspace MVP should demonstrate the core thesis.

It should support:

### Objects

```text
create
read
revise
delete/tombstone
```

### Types and schemas

Typed validation.

### Relationships

Typed graph edges.

### Transactions

Atomic candidate changes.

### World revisions

Committed logical snapshots.

### Queries

Find objects by:

```text
type
metadata
relationship
revision
provenance
```

### History

Inspect previous revisions.

### Capabilities

A minimal deny-by-default capability system.

### Agent API

Agents can fully operate Objectspace without touching its backing files.

### Example runtime

At least one WebAssembly application consuming capabilities.

### Compatibility

Basic import/export to conventional files.

---

# 32. MVP Demonstration

A compelling first demonstration would be an agent building a small application entirely through Objectspace.

For example:

> Create a task manager with projects, tasks and deadlines.

The demo should prove:

1. no application filesystem tree exists;
2. the agent creates typed application objects;
3. dependencies are explicit;
4. the app executes through Wasm;
5. the application receives explicit capabilities;
6. a world snapshot exists before modification;
7. the agent modifies the application;
8. Objectspace calculates the impact graph;
9. tests execute;
10. rollback restores the previous world immediately.

That demonstration would communicate the concept better than hundreds of pages of architecture.

---

# 33. Repository Structure During Bootstrap

One possible initial repository:

```text
objectspace/
│
├── Cargo.toml
├── rust-toolchain.toml
│
├── docs/
│   ├── doctrine.md
│   ├── architecture.md
│   ├── invariants.md
│   ├── object-model.md
│   ├── capability-model.md
│   └── adr/
│
├── crates/
│   ├── os-id/
│   ├── os-schema/
│   ├── os-store/
│   ├── os-graph/
│   ├── os-transaction/
│   ├── os-world/
│   ├── os-query/
│   ├── os-capability/
│   ├── os-runtime/
│   ├── os-agent-api/
│   └── os-compat/
│
├── examples/
│   └── task-manager/
│
├── tests/
│   ├── conformance/
│   ├── crash/
│   └── adversarial/
│
└── xtask/
```

Names should remain boring and obvious.

---

# 34. Foundational Documents

Before substantial implementation, the project should create five short authoritative documents.

## doctrine.md

Non-negotiable philosophy.

---

## architecture.md

Layering and dependency rules.

---

## invariants.md

Properties that must always remain true.

---

## object-model.md

Identity, revision, edge, schema and world semantics.

---

## agent-development.md

How autonomous agents are expected to change and verify the system.

These should remain concise enough for an agent to load frequently.

---

# 35. Proposed Foundational Invariants

At minimum:

1. **Object identity is stable.**

2. **Revisions are immutable.**

3. **Every committed world is internally consistent.**

4. **Failed transactions cannot mutate committed state.**

5. **All persistent relationships reference valid identities or explicit tombstones.**

6. **Capabilities are deny-by-default.**

7. **Authority cannot silently expand across component boundaries.**

8. **Every committed change has provenance.**

9. **Compatibility projections are never canonical state.**

10. **Core architecture dependency rules are mechanically enforced.**

11. **An object cannot depend on a pathname for identity.**

12. **System rollback never requires reconstruction from application-specific logic.**

These invariants should be tested continuously.

---

# 36. Performance Philosophy

Semantic richness cannot justify unacceptable latency.

Objectspace should aggressively index common graph relationships and object attributes.

Likely techniques include:

```text
content addressing
MVCC
copy-on-write structures
relationship indexes
type indexes
incremental query execution
dependency-closure caching
lazy materialization
incremental builds
```

Because revisions are immutable, caching opportunities should be unusually strong.

If:

```text
RevisionId
```

has not changed, any deterministic analysis based solely on it may be safely reused.

This should benefit both application execution and agent reasoning.

---

# 37. Garbage Collection

Immutable revisions create storage growth.

Objectspace therefore needs explicit retention semantics.

Possible categories:

```text
current
pinned
historical
derived-cache
ephemeral
candidate-world
unreachable
```

Garbage collection should distinguish:

- semantic history worth retaining;
- reproducibility requirements;
- temporary derivations;
- abandoned candidate worlds;
- cached outputs.

Deletion should initially be conservative.

Storage is cheaper than losing provenance.

---

# 38. Distributed Objectspace

Distribution should not be part of the first architecture.

However, stable identity and immutable revisions naturally create future opportunities.

Objects may eventually be:

```text
replicated
synchronized
shared
cached
migrated
```

without changing their conceptual identity.

Content-addressed revisions should make cross-machine verification easier.

This can be explored after single-machine correctness is mature.

---

# 39. Potential Use Cases

## Agentic software engineering

The primary initial use case.

Agents operate over a semantic architecture rather than repositories of loosely related text files.

---

## Rapid application generation

Agents assemble native application graphs using reusable platform capabilities.

---

## Personal computing

Documents, images, conversations, calendar events and projects become queryable semantic objects rather than scattered files.

---

## Enterprise systems

Permissions, provenance, documents, applications and workflows share the same underlying model.

---

## Scientific computing

Datasets can explicitly encode:

```text
derived-from
generated-by
model-version
parameters
experiment
```

making reproducibility native.

---

## Creative work

A rendered asset can retain direct relationships to:

```text
source image
prompt
model
edit history
license
project
```

---

## System administration

Deployments and configurations can become transactionally versioned semantic objects.

---

# 40. Major Risks

## Complexity

Objectspace could become a giant database pretending to be an operating system.

**Mitigation:** keep the semantic kernel extremely small.

---

## Schema rigidity

Strongly typed objects can become cumbersome as domains evolve.

**Mitigation:** first-class schema versioning and migration.

---

## Poor human ergonomics

A graph may be excellent for agents and unpleasant for humans.

**Mitigation:** human-facing views can remain spatial and familiar without making those views canonical.

---

## Ecosystem incompatibility

Existing software expects files.

**Mitigation:** strong import/export and POSIX projection.

---

## Performance

Generic semantic systems can become slow.

**Mitigation:** immutable revision caching, aggressive indexing, explicit profiling and simple hot paths.

---

## Agent-generated systemic bugs

Agents can generate coherent but incorrect implementations.

**Mitigation:** independent verification, adversarial agents, property testing, mutation testing and machine-enforced invariants.

---

## Premature reinvention

The project could waste years rebuilding kernels, compilers and databases.

**Mitigation:** aggressively reuse Linux, Rust, Wasmtime, WIT and existing storage technology.

---

# 41. Project Success Metrics

Objectspace should measure more than raw benchmarks.

## Agent Comprehension Cost

How many objects/tokens must an agent inspect before safely making common changes?

Target: continuously decrease.

---

## Change Blast Radius

How much unrelated system state must be touched for a feature?

Target: minimal.

---

## Agent Success Rate

Percentage of well-specified engineering tasks completed autonomously with all verification gates passing.

---

## Mean Objects Inspected per Change

A practical proxy for ACC.

---

## Rollback Reliability

Percentage of committed worlds that can be restored exactly.

Target:

```text
100%
```

---

## Reproducibility

Can an old WorldRevision reproduce the same application state and executable inputs?

---

## Capability Auditability

Can every meaningful application authority be explained through explicit capabilities?

---

## Application Construction Cost

For standard application classes, measure:

```text
semantic objects created
agent tokens consumed
wall-clock operations
custom code required
```

The long-term objective is for common applications to require dramatically less bespoke code.

---

# 42. Proposed Development Phases

## Phase 0 — Doctrine

Produce:

```text
doctrine
architecture
invariants
object model
agent methodology
```

No major implementation until these agree.

---

## Phase 1 — Core Object Store

Implement:

```text
ObjectId
RevisionId
schemas
immutable revisions
basic edges
transactions
```

---

## Phase 2 — World Model

Implement:

```text
WorldRevision
fork
commit
rollback
diff
```

---

## Phase 3 — Graph and Query

Implement structured graph traversal and impact analysis.

---

## Phase 4 — Agent API

Expose the complete system through typed machine interfaces and MCP.

Agents should now be capable of manipulating Objectspace directly.

---

## Phase 5 — Capability Runtime

Add Wasmtime, WIT and deny-by-default application capabilities.

---

## Phase 6 — First Native Application

Build the task-manager reference application entirely as Objectspace objects.

---

## Phase 7 — Native Development Objects

Represent Objectspace source modules and build relationships inside Objectspace.

---

## Phase 8 — Self-Hosting Development

Agents begin developing Objectspace primarily through Objectspace.

---

## Phase 9 — Application Platform

Introduce reusable primitives:

```text
identity
data
views
workflows
notifications
search
permissions
network services
```

At this point rapid agent-generated applications become a major objective.

---

## Phase 10 — Storage Independence

Evaluate native raw-block persistence.

Only proceed if there is a measurable benefit.

---

# 43. The Recursive Advantage

Objectspace has an unusual development curve.

At first:

```text
Agent
  ↓
filesystem
  ↓
repository
  ↓
Objectspace
```

The agent suffers from all the limitations Objectspace is intended to remove.

Later:

```text
Agent
  ↓
Objectspace
  ↓
semantic system graph
```

Objectspace itself becomes easier to modify.

The agent gains:

```text
exact impact analysis
semantic history
world branching
structured dependencies
transactional modification
automated provenance
capability-aware testing
```

This makes future Objectspace development faster.

Those improvements can themselves improve the agent interface.

The loop becomes:

```text
Agents improve Objectspace
        ↓
Objectspace improves agent effectiveness
        ↓
Agents improve Objectspace faster
        ↓
Objectspace becomes more agent-native
        ↓
...
```

This recursive relationship should be deliberately cultivated.

---

# 44. Why This Could Matter

Modern software engineering spends enormous effort translating between abstractions that the computer itself does not understand.

Humans and agents repeatedly reconstruct:

```text
what this is
where it belongs
what it depends on
who can access it
what changed
how to restore it
how to run it
how to test it
```

Objectspace proposes making these properties intrinsic.

If successful, it could shift programming from:

> Generate thousands of loosely related textual artifacts and rely on conventions to make them form an application.

toward:

> Construct a valid semantic application graph using durable system primitives.

That would be particularly consequential for software agents.

The limiting factor for an agent would increasingly cease to be:

> Can it understand this repository?

and become:

> Can it understand the actual problem being solved?

That is a much better place to spend intelligence.

---

# 45. Founding Doctrine

The project should operate under the following doctrine:

> **Objectspace is a persistent semantic computing environment in which identity, relationships, version history, capabilities and provenance are fundamental system properties. Files and folders are compatibility projections, not canonical state.**

> **Objectspace is designed for a world in which software agents are primary operators and creators of software. Architecture must therefore optimize for machine comprehension, explicit semantics, transactional modification, deterministic verification and minimal blast radius.**

> **The trusted core must remain small. Existing mature infrastructure should be reused wherever it does not compromise the Objectspace model. Novelty belongs in the semantic machine model, not in unnecessary reinvention beneath it.**

> **The system must become progressively easier for agents to engineer as more of Objectspace itself moves into Objectspace.**

> **Speed and safety are not opposing goals. Strong invariants, transparent architecture and reusable platform capabilities should make correct software dramatically faster to create.**

---

# 46. Immediate Project Start

The first implementation work should not begin with the runtime.

It should begin by freezing the smallest possible architecture contract.

Create:

```text
docs/doctrine.md
docs/invariants.md
docs/object-model.md
docs/architecture.md
docs/agent-development.md
```

Then initialize the Rust workspace.

The first executable milestone should prove only this:

```text
Create Object
      ↓
Create Revision
      ↓
Create Relationship
      ↓
Commit Transaction
      ↓
Produce WorldRevision
      ↓
Query Graph
      ↓
Fork World
      ↓
Modify Object
      ↓
Compare Worlds
      ↓
Rollback Exactly
```

Nothing graphical is required.

No native kernel is required.

No programming language is required.

No ambitious application framework is required.

Once those primitives are correct, everything else can grow upward from them.

---

# 47. Final Vision

A mature Objectspace computer might receive:

> Build an internal inventory application for our warehouse. Track items, suppliers, purchase orders and locations. Managers can adjust inventory manually. Ordinary workers can scan items but cannot change historical transactions. Import the existing spreadsheet and deploy it to the warehouse tablets.

An agent could:

```text
inspect existing business objects
create application
create schemas
create relationships
create policies
attach scanner capability
attach identity capability
import spreadsheet
create views
create workflows
generate verification
simulate application
inspect capability graph
commit world
deploy application
```

without creating a conventional source repository at all.

The result is not a directory containing an application.

The result is an application that **exists natively as part of the machine's semantic world**.

The user can inspect it.

The agent can reason about it.

The system can version it.

The runtime can execute it.

The security model can constrain it.

Another agent can modify it without first reverse-engineering a pile of files.

And the entire change can be atomically rejected, committed or rolled back.

That is Objectspace.

**Unix made everything look like a file.**

**Objectspace should make everything exist as what it actually is.**