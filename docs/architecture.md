# Objectspace Architecture

## 1. Objective

Objectspace replaces pathname-addressed files as the canonical persistent abstraction with:

```text
ObjectId
+
Type
+
Immutable Revision
+
Typed Relationships
+
Capabilities
+
Provenance
```

The initial implementation runs on Linux.

Linux is a hardware and compatibility substrate, not the user-visible machine model.

## 2. Architecture

```text
┌──────────────────────────────────────────────┐
│            HUMAN / AGENT SURFACES           │
│                                              │
│ UI · CLI · IDE · MCP · Native Agent API      │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│             OBJECTSPACE KERNEL               │
│                                              │
│ Identity                                     │
│ Schema                                       │
│ Graph                                        │
│ Query                                        │
│ Transactions                                 │
│ World revisions                              │
│ Capabilities                                 │
│ Provenance                                   │
└──────────────┬──────────────────┬────────────┘
               │                  │
               ▼                  ▼
┌──────────────────────┐  ┌────────────────────┐
│  COMPONENT RUNTIME   │  │    OBJECT STORE    │
│                      │  │                    │
│ Wasmtime             │  │ Immutable revisions│
│ Wasm Components      │  │ Graph indexes      │
│ WIT                  │  │ Transactions       │
│ Capability injection │  │ Content blobs      │
└────────────┬─────────┘  └─────────┬──────────┘
             │                      │
             └──────────┬───────────┘
                        ▼
┌──────────────────────────────────────────────┐
│              LINUX SUBSTRATE                 │
│                                              │
│ Drivers · VM · scheduler · network · GPU     │
└──────────────────────────────────────────────┘
```

## 3. Implementation Stack

### Core language

Rust.

The trusted core should remain Rust unless a concrete requirement justifies another language.

### Repository

Cargo workspace monorepo.

### Bootstrap storage

`redb`, hidden behind an Objectspace-owned storage interface.

### Application runtime

Wasmtime.

### Component model

WebAssembly Components.

### Interface contracts

WIT.

### Agent access

Typed native API with an MCP compatibility adapter.

### Verification

Primary tools:

```text
rustc
clippy
cargo-nextest
property testing
fuzzing
mutation testing
Kani
selective Verus
```

## 4. Layer Dependency Rules

Dependencies flow downward only.

Conceptually:

```text
identity
   ↑
schema
   ↑
store
   ↑
graph
   ↑
transaction / world
   ↑
capability
   ↑
runtime
   ↑
agent API / applications / compatibility
```

Exact crate boundaries may evolve.

The dependency direction may not.

Forbidden dependencies must fail CI.

Examples:

```text
store → runtime           FORBIDDEN
graph → agent-api         FORBIDDEN
identity → query          FORBIDDEN
core → POSIX projection   FORBIDDEN
```

Compatibility layers may depend on core.

Core may never depend on compatibility.

## 5. Core Entities

### ObjectId

Stable conceptual identity.

```text
ObjectId
```

must not encode:

- pathname;
- physical storage location;
- current revision;
- display hierarchy.

### RevisionId

Immutable identity of one exact object state.

Prefer content-derived identifiers where appropriate.

```text
RevisionId = hash(canonical revision representation)
```

### ObjectRevision

Contains at minimum:

```text
ObjectId
RevisionId
Type
SchemaVersion
Payload
ParentRevision
Provenance
```

### Edge

Typed relationship:

```text
source ObjectId
relation Type
target ObjectId
metadata
```

### WorldRevision

Immutable description of one complete committed logical world.

### Capability

Unforgeable authority permitting a specific operation against a specific object, interface, or external resource.

## 6. Mutation Model

Committed objects are never modified in place.

Mutation is:

```text
existing revision
       ↓
new immutable revision
```

System mutation occurs inside transactions.

```text
transaction.begin
    ↓
create / revise / link / unlink
    ↓
validate
    ↓
build / test / verify
    ↓
commit OR abort
```

No intermediate transaction state becomes globally visible.

## 7. World Model

Agents work against candidate worlds.

```text
World 100
   ├── Candidate A
   ├── Candidate B
   └── Candidate C
```

After verification:

```text
Candidate B → World 101
```

This enables:

- speculative development;
- parallel agents;
- deterministic testing;
- atomic upgrades;
- immediate rollback;
- semantic world comparison.

## 8. Storage Interface

Higher layers must not know which storage engine is used.

Conceptually:

```rust
trait ObjectStore {
    fn get_revision(...);
    fn put_revision(...);
    fn get_world(...);
    fn query_edges(...);
    fn begin_transaction(...);
    fn commit(...);
}
```

Expected backends:

```text
MemoryBackend
RedbBackend
NativeBackend
```

`RedbBackend` is bootstrap infrastructure.

No redb-specific concept may leak into the semantic kernel.

## 9. Query Model

Objectspace must support structured queries over:

```text
identity
type
metadata
revision
relationships
provenance
capabilities
world membership
```

Core graph operations:

```text
dependencies
dependents
neighbors
path
closure
impact
```

Impact analysis is a first-class operation.

## 10. Capability Model

There is no ambient filesystem, network, device, or service authority for native applications.

Capabilities are explicit.

Example:

```text
Application CRM

capabilities:
    Contact.read
    Contact.write
    GoogleIdentity.authenticate
    Network(api.example.com).connect
```

Absence of a capability means absence of authority.

Capability boundaries must be independently testable.

## 11. Runtime Model

Native applications execute primarily as WebAssembly Components.

An application is not an executable file.

It is an application graph containing objects such as:

```text
Application
Component
Interface
Schema
Policy
Workflow
View
Test
CapabilityBinding
```

The runtime materializes executable components from this graph.

## 12. Source Code

During bootstrap, source code remains textual.

Example:

```text
Object
    Type: RustModule
    Payload: source text
```

The object is canonical.

A `.rs` file is not.

Derived semantic relationships may include:

```text
defines
calls
implements
uses-type
tested-by
depends-on
```

Canonical AST storage is explicitly deferred.

## 13. Agent API

The agent interface must expose semantics directly.

Minimum operations:

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

provenance.trace
```

Results must be structured.

Do not make agents parse terminal prose where a typed result is possible.

## 14. Compatibility

Files are boundary formats.

Supported directions:

```text
file → Objectspace import
Objectspace object → file export
Objectspace view → temporary virtual filesystem
```

Legacy programs may receive POSIX projections.

Those paths are ephemeral representations.

Canonical state remains inside Objectspace.

## 15. Bootstrap Sequence

### Phase 0

Conventional Git + Rust development.

### Phase 1

Objectspace stored inside a normal host file.

### Phase 2

Native object, revision, graph, transaction, and world semantics.

### Phase 3

Typed agent API.

### Phase 4

Wasm Component runtime and capabilities.

### Phase 5

First Objectspace-native application.

### Phase 6

Objectspace source represented inside Objectspace.

### Phase 7

Objectspace developed primarily through Objectspace-native tools.

### Phase 8

Self-hosting.

### Phase 9

Optional direct block-device backend.

### Phase 10

Evaluate kernel replacement only if measured benefits justify it.

## 16. Initial Repository

```text
objectspace/
├── Cargo.toml
├── rust-toolchain.toml
├── docs/
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
├── examples/
├── tests/
└── xtask/
```

Do not create additional architectural layers without demonstrated need.

## 17. Verification

One canonical command:

```bash
cargo xtask verify
```

It should orchestrate:

```text
format
compile
architecture rules
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

Agents should need to know one verification entrypoint.

## 18. Architecture Rules for Agentic Engineering

The codebase must favor:

```text
explicit control flow
small modules
strong types
few abstractions
obvious ownership
shallow call chains
deterministic behavior
local reasoning
machine-readable contracts
```

Avoid unless clearly justified:

```text
reflection
global registries
implicit dependency injection
macro-heavy hidden behavior
stringly typed protocols
multiple competing patterns
ambient mutable state
```

## 19. Primary Architecture Metric

Every subsystem should be evaluated using **Agent Comprehension Cost**.

For any common change, ask:

```text
How many objects must an agent inspect?
How many architectural rules must it infer?
How much unrelated state enters context?
Can Objectspace expose the exact dependency closure instead?
```

Architectures that reduce these values should be preferred even when both alternatives are otherwise correct.

## 20. MVP Exit Criteria

The first architecture milestone is complete when Objectspace can:

```text
create object
create immutable revision
create typed relationship
begin transaction
commit transaction
produce WorldRevision
query graph
fork world
revise object
compute impact
compare worlds
abort safely
rollback exactly
```

and all of this is available through the agent API without requiring the caller to manipulate backing files.

Nothing else is required to prove the core model.

## 21. Architectural North Star

The long-term system should allow an agent to reason in terms of:

```text
Application
Schema
Component
Interface
Policy
Workflow
Capability
Test
Deployment
```

rather than:

```text
directory
filename
relative path
configuration file
manifest
shell command
```

The architecture succeeds when the machine exposes the actual structure of software directly enough that agents no longer spend most of their intelligence reconstructing it.