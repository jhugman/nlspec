# NLSpec: Natural Language Specification Format

This document specifies the NLSpec format -- human-readable specifications intended to be directly usable by coding agents to implement and validate software behavior. It defines the document structure, pseudocode language, writing conventions, and validation mechanisms that make an NLSpec effective.

The intended audience is a coding agent or human author tasked with producing an NLSpec for a new project.

---

## Table of Contents

1. [Overview and Goals](#1-overview-and-goals)
2. [Document Structure](#2-document-structure)
3. [The Preamble](#3-the-preamble)
4. [The "Out of Scope" Section](#4-the-out-of-scope-section)
5. [The "Design Decision Rationale" Section](#5-the-design-decision-rationale-section)
6. [The "Definition of Done" Section](#6-the-definition-of-done-section)
7. [The "Appendix" Section](#7-the-appendix-section)
8. [The Pseudocode Language](#8-the-pseudocode-language)
9. [Type Definitions](#9-type-definitions)
10. [Tables](#10-tables)
11. [Formal Grammar](#11-formal-grammar)
12. [Examples, Diagrams, and Formal Notations](#12-examples-diagrams-and-formal-notations)
13. [Language and Voice](#13-language-and-voice)
14. [Edge Cases and Error Handling](#14-edge-cases-and-error-handling)
15. [Cross-Referencing and Dependencies](#15-cross-referencing-and-dependencies)
16. [Out of Scope](#16-out-of-scope)
17. [Design Decision Rationale](#17-design-decision-rationale)
18. [Definition of Done](#18-definition-of-done)

---

## 1. Overview and Goals

### 1.1 What NLSpec Is

An NLSpec (Natural Language Specification) is a human-readable specification intended to be directly usable by coding agents to implement and validate software behavior. It uses natural language as the primary medium but introduces formal structures -- pseudocode, type definitions, tables, grammars -- at the points where natural language would be ambiguous.

An NLSpec is not a tutorial, a design document, or an API reference. It is a buildable specification: a coding agent supplied with an NLSpec and no other context should be able to produce a conforming implementation.

### 1.2 Problem Statement

Specifications written for human audiences rely on the reader's ability to fill gaps: resolve ambiguities from context, ask clarifying questions, and make reasonable assumptions about defaults and edge cases. When the reader is a coding agent, every gap becomes a wrong implementation, every ambiguity becomes a guess, and every missing default becomes a bug.

Traditional specification formats (RFCs, design docs, API references) are optimized for human readers who will interpret them. NLSpec is optimized for machine readers who will implement them.

### 1.3 Design Principles

**Machine-readable precision.** Every statement in the spec body is precise enough that a coding agent can implement it without asking clarifying questions. Where natural language is ambiguous, formal structures replace it.

**Human-readable narrative.** The document reads as a coherent narrative, not a schema dump. Prose motivates and connects; formal structures define.

**Multiple representations.** No single notation covers all concerns. Prose says why, pseudocode says how, tables say what, and checklists say when you're done. Lean into established notations -- BNF, UML, state diagrams, regular expressions -- wherever they communicate more clearly than prose.

**Closed-loop completeness.** The Definition of Done mirrors the body section-by-section. Gaps in either direction are visible. The two halves of the spec audit each other.

**Minimum sufficient formalism.** Use formal notation where the domain demands it. Do not use it where prose is unambiguous.

**Language agnosticism.** The pseudocode language is translatable to any programming language. The spec does not assume an implementation language.

### 1.4 Reference Documents

The following NLSpec documents serve as exemplars of the format described in this specification. They are not dependencies; authors may study any combination of them for patterns and trade-offs.

| Document | Domain | Notable patterns |
|----------|--------|-----------------|
| [attractor-spec.md](exemplars/attractor-spec.md) | Pipeline orchestration engine | Extensive BNF grammar, condition expression language, 95 DoD items |
| [unified-llm-spec.md](exemplars/unified-llm-spec.md) | Multi-provider LLM client | Provider mapping tables, validation matrices, (Critical) markers |
| [coding-agent-loop-spec.md](exemplars/coding-agent-loop-spec.md) | Coding agent architecture | TOOL definitions, hard dependency on companion spec, integration test pseudocode |

### 1.5 Scope

This spec defines the NLSpec document format: its structure, pseudocode language, writing conventions, table types, and validation mechanisms. It is domain-agnostic -- any software system can be specified using NLSpec.

This spec does NOT define:

- How to decompose a system into specs (that is an architectural decision)
- How to gather requirements (that precedes spec writing)
- Tooling for NLSpec authoring or validation (that is an implementation concern)

---

## 2. Document Structure

### 2.1 The Four-Phase Arc

Every NLSpec follows a four-phase narrative arc:

```
ENUM DocumentPhase:
    WHY     -- Motivate existence, establish constraints
    WHAT    -- Define vocabulary and structure
    HOW     -- Specify behavior with pseudocode
    DONE    -- Make completeness testable
```

| Phase | Typical Sections | Purpose |
|-------|-----------------|---------|
| **Why** | Overview, Problem Statement, Design Principles | Motivate existence, establish constraints |
| **What** | Architecture, Data Model, Syntax/Schema | Define the vocabulary and structure |
| **How** | Execution, Algorithms, Handlers, Tools | Specify behavior with pseudocode |
| **Done** | Definition of Done | Make completeness testable |

This mirrors how a reader learns: understand the problem, learn the vocabulary, study the behavior, then verify understanding. The phases are a narrative progression, not rigid containers -- a section may blend phases where clarity demands it.

### 2.2 Section Numbering

Sections use decimal notation (`2.1`, `3.3`, `10.2`) consistently throughout. Top-level sections are numbered with integers. Subsections are numbered with the parent section number followed by a dot and the subsection number.

Section numbering enables precise cross-references ("see Section 9.4") and makes the document navigable both as a linear read and as a reference.

### 2.3 Table of Contents

Every NLSpec opens with a linked table of contents placed after the opening paragraph and before the first `---` horizontal rule. Each entry is a markdown link to the corresponding heading anchor:

```markdown
## Table of Contents

1. [Overview and Goals](#1-overview-and-goals)
2. [Architecture](#2-architecture)
3. [Data Model](#3-data-model)
...
```

### 2.4 Horizontal Rules as Phase Boundaries

`---` (horizontal rules) separate top-level sections, creating visual breathing room. They signal phase transitions more strongly than a heading alone.

### 2.5 Appendices

Appendices appear after the Definition of Done, holding reference material that would interrupt the narrative flow. See Section 7 for guidance on appendix content and structure.

### 2.6 Parallel Subsection Structure

When a spec documents multiple implementations of the same interface (providers, adapters, platform ports), each implementation gets its own subsection with identical internal structure. If the OpenAI adapter subsection has "Key differences," "Tool list," and "System prompt guidance" slots, the Anthropic and Gemini subsections have the same slots in the same order.

This structural parallelism enforces completeness: a missing slot is visible because the surrounding subsections have it.

---

## 3. The Preamble

The preamble comprises the opening sections of the spec (typically Section 1 and its subsections). It establishes context before any technical content.

### 3.1 Opening Paragraph

The first paragraph of the spec states what the thing is and who it is for:

1. **What it is:** Name the artifact and its function.
2. **Who it is for:** Name the audience or the problem it solves.

No jargon that has not been defined yet. No implementation details. The reader should understand the spec's purpose after reading only this paragraph.

### 3.2 Problem Statement

Immediately after the opening paragraph, a Problem Statement section establishes why this thing needs to exist. It follows a three-part structure:

1. **Name the status quo.** Describe what exists today.
2. **Name the pain.** State what is wrong or missing.
3. **State the solution.** One sentence describing what this spec provides.

### 3.3 Design Principles

Design principles are stated as bold-lead paragraphs: the principle name in bold, followed by a one-to-two sentence explanation of the constraint and its consequence.

```markdown
**Principle name.** One to two sentences explaining the constraint
and what follows from it.
```

Each principle is a named constraint, not an aspiration. It should be possible to check whether an implementation violates a principle.

### 3.4 Reference Projects

When relevant open-source projects solve related problems, list them in a reference projects subsection. Each entry includes:

| Field | Content |
|-------|---------|
| Name | Project name with URL |
| Language | Implementation language |
| Relevance | One sentence naming the specific patterns worth studying |

Preface the list with a statement clarifying that these are exemplars, not dependencies:

> "They are not dependencies; implementors may take inspiration from any combination of them."

### 3.5 Layering and Scope

The preamble states what the spec covers and what it does not cover. Follow the pattern:

1. **State responsibility:** "This spec defines X: [list of concerns]."
2. **State non-responsibility:** "This spec does NOT define Y."
3. **Name the boundary:** If the boundary is another spec or system, name it.

---

## 4. The "Out of Scope" Section

Every NLSpec includes an Out of Scope section that explicitly names excluded features and concerns. Each out-of-scope item follows a four-part structure:

1. **Name** -- the feature or concern being excluded, in bold
2. **What it is** -- a one-sentence description
3. **Why it is excluded** -- why it does not belong in this spec
4. **Where the extension point is** -- how it could be added later

```markdown
**MCP (Model Context Protocol).** An MCP client can extend the agent
with tools from external servers. The tool registry supports registering
MCP-discovered tools with namespaced names. This is a natural extension
but not a core requirement.
```

The extension point is critical. Without it, an out-of-scope item sounds like a permanent exclusion. With it, the item becomes a deferred feature with a known integration path. A coding agent reading this knows exactly where to add the capability if asked to.

The Out of Scope section often uses layering to justify exclusion: "that is handled by the layer below" or "that belongs in a layer above." This overlaps with the preamble's scope statement (see Section 3.5) but serves a different purpose: the preamble states what is included; the Out of Scope section names what was considered and deliberately excluded.

---

## 5. The "Design Decision Rationale" Section

An NLSpec collects design justifications in a dedicated section or appendix. Each entry uses the bold-question format: a bold question naming the rejected alternative, followed by the answer:

```markdown
**Why a single Request type instead of per-method parameter lists?**
A single Request object is easier to construct, pass around, modify,
and serialize than many keyword arguments.
```

The question format matters: it names the alternative that was rejected. This prevents implementors from proposing the same rejected alternative.

### 5.1 Inline Anti-Pattern Documentation

Rejected approaches also appear inline, near the relevant section:

> "A single method with a `stream: boolean` flag was rejected because the return types are fundamentally different."

This prevents implementors from making mistakes the author already considered. The inline placement ensures the anti-pattern is visible when the reader is studying the relevant section.

### 5.2 Temporal Anchors

When a spec includes data that will age (model catalogs, API versions, pricing), it marks the data explicitly:

> "**At the time of writing (February 2026),** the top models available through each provider's API are:"

This tells a future reader (or coding agent) that the data may be stale, without invalidating the surrounding patterns.

---

## 6. The "Definition of Done" Section

The Definition of Done is the final major section of every NLSpec. It transforms the spec into a testable checklist.

### 6.1 Checkbox Format

Each requirement is a single markdown checkbox item:

```markdown
- [ ] Parser accepts the supported DOT subset
- [x] Graph-level attributes are extracted correctly
```

`[x]` means implemented and verified. `[ ]` means not yet implemented. Checkbox state makes the spec a living project tracker.

### 6.2 Section Mirroring

Definition of Done subsections mirror the body sections. Each body section that defines testable behavior has a corresponding DoD subsection. Each DoD item traces to a specific behavior described in its corresponding body section.

This creates a closed loop:

- If a body section has no DoD subsection, its requirements are not testable.
- If a DoD item has no corresponding body section, its requirement is ungrounded.

Both gaps are defects in the spec.

### 6.3 Requirement Density

If a section has many lines but few DoD items, the section is likely too vague. If a section has many DoD items but few lines, the body needs more detail.

### 6.4 Integration Smoke Tests

The final DoD item is a pseudocode integration test that exercises the system end-to-end. The test should be complete enough that a coding agent can translate it directly into a real test:

```
result = system_under_test(minimal_valid_input)
ASSERT result meets expected_output_criteria
```

### 6.5 Validation Matrices

For multi-provider or multi-implementation systems, include a validation matrix in the DoD (see Section 10.4).

### 6.6 HTML Comments for Implementation Details

DoD items may include HTML comments noting how a requirement is satisfied in the implementation:

```markdown
- [x] Tool calls are dispatched through the ToolRegistry
  <!-- pi-mono handles dispatch internally -->
- [x] Streaming responses emit events as chunks arrive
  <!-- uses AsyncIterator over SSE connection -->
```

This links the normative requirement to the implementation without altering the spec text. The comment explains *how*, not *whether*, the requirement is met.

---

## 7. The "Appendix" Section

Appendices hold reference material that is essential for implementation but would interrupt the narrative flow of the body. They appear after the Definition of Done section, lettered sequentially (Appendix A, Appendix B, ...).

### 7.1 What Belongs in an Appendix

Material belongs in an appendix when it meets two criteria:

1. **Essential for implementation** -- a coding agent needs it to build a conforming system
2. **Disruptive to narrative** -- placing it inline would break the reader's flow through the four-phase arc

Common appendix content types:

| Appendix type | Content | Example |
|---------------|---------|---------|
| Attribute catalog | Complete listing of all configurable attributes | Graph, node, and edge attribute tables |
| Format reference | Grammar, schema, or wire format for a subsidiary format | Patch format BNF and operations |
| Error catalog | Error categories with recovery strategies | Error types, examples, and recovery actions |
| Usage examples | Extended examples too long for inline placement | Multi-turn conversation transcripts, API usage patterns |
| Design rationale | Collected design decisions | Bold-question entries (may also appear as a body section) |

### 7.2 Referencing Appendices

The body text references appendices by name:

> "For the complete attribute reference, see Appendix A."

Every appendix must be referenced from at least one body section. An unreferenced appendix is dead weight.

### 7.3 Appendix vs. Body Section

If material is needed to understand the narrative, it belongs in the body. If it is needed to implement but not to understand, it belongs in an appendix. Design rationale may appear as either a body section or an appendix -- the exemplars use both placements.

---

## 8. The Pseudocode Language

NLSpec uses a language-agnostic pseudocode for algorithms and interfaces. The goal is readability and translatability, not adherence to a fixed grammar. Authors may use any pseudocode constructs that are clear in context -- the conventions below establish visual consistency across NLSpec documents.

### 8.1 Conventions

**Keywords** are UPPER CASE. Common keywords include `FUNCTION`, `IF`, `ELSE`, `WHILE`, `FOR`, `FOR EACH`, `RETURN`, `BREAK`, `CONTINUE`, `TRY`, `CATCH`, `RAISE`, `RECORD`, `ENUM`, `INTERFACE`, `AND`, `OR`, `NOT`, `NEW`. The keyword set is not closed -- specs may use additional keywords (`AWAIT`, `YIELD`, `LOOP`, or domain-specific keywords per Section 9.4) as the domain requires.

**Names** follow case conventions:

| Element | Convention | Example |
|---------|-----------|---------|
| Variables and functions | `snake_case` | `current_node`, `execute_with_retry` |
| Types and records | `PascalCase` | `Message`, `StageStatus` |
| Enum values | `UPPER_CASE` | `SUCCESS`, `FAIL` |

**Comments** use `--` (double dash), visually distinct from code and from any major language's comment syntax. Comments explain intent and constraints, not what the code literally does.

**Syntax** uses colon-terminated headers with 4-space indented bodies. Assignment is `=`, equality is `==`. Type annotations use `: Type` suffix (e.g., `name : String | None`, `-> Outcome`).

### 8.2 Labeled Steps

Complex algorithms use numbered steps as inline comments to create a dual representation: the prose above describes the steps in English; the pseudocode implements them. The two reinforce each other.

```
FUNCTION select_edge(node, outcome, context, graph):
    edges = graph.outgoing_edges(node.id)

    -- Step 1: Condition-matching edges
    ...

    -- Step 2: Preferred label match
    ...

    -- Step 3: Highest weight
    ...
```

### 8.3 Layered Composition Notation

When a complex value is assembled from multiple ordered sources, document it as numbered additive layers:

```
final_system_prompt =
    1. Provider-specific base instructions     (from ProviderProfile)
  + 2. Environment context                     (platform, git, working dir)
  + 3. Tool descriptions                       (from the active tool set)
  + 4. Project-specific instructions           (from config files)
  + 5. User instructions override              (appended last, highest priority)
```

Each layer includes its source in parentheses. The numbering establishes assembly order and, where relevant, precedence (later layers override earlier ones).

### 8.4 Behavior Summaries

When a function's pseudocode is complex, follow it with a **Behavior:** bullet list that summarizes the operation's contract in prose:

```
FUNCTION complete(request) -> Response:
    adapter = resolve_adapter(request.provider)
    raw = adapter.send(request)
    RETURN translate(raw)

-- Behavior:
--   - Routes the request to the appropriate provider adapter
--   - Blocks until the provider returns a complete response
--   - Returns a normalized Response regardless of provider
```

The pseudocode defines the implementation; the behavior summary states the contract in human terms. The two reinforce each other.

---

## 9. Type Definitions

NLSpec uses three core type-definition keywords. Specs may also introduce domain-specific definition keywords where their subject matter demands it.

### 9.1 RECORD -- Data Structures

A `RECORD` defines a data structure with named, typed fields. Each field includes a name, type, and inline comment:

```
RECORD Message:
    role          : Role                  -- who produced this message
    content       : List<ContentPart>     -- the message body
    name          : String | None         -- for tool messages
    tool_call_id  : String | None         -- links a tool-result to its call
```

Fields are vertically aligned for scanning. The comment column explains the field's purpose, not just its type.

#### Field Grouping

Large RECORDs use `--` comment lines to group related fields:

```
RECORD StreamEvent:
    type              : StreamEventType | String

    -- text events
    delta             : String | None
    text_id           : String | None

    -- tool call events
    tool_call         : ToolCall | None
    tool_call_id      : String | None
```

#### Constraint Blocks

When fields have constraints beyond their type (validation rules, allowed ranges, format requirements), document them as bold-lead paragraphs immediately after the RECORD:

```
RECORD Tool:
    name        : String
    description : String
    parameters  : Dict
```

**Tool name constraints:** Names must be valid identifiers: alphanumeric characters and underscores, starting with a letter. Maximum 64 characters.

**Parameter schema:** Parameters must be defined as a JSON Schema object with `"type": "object"` at the root.

#### Methods on Records

A RECORD may include FUNCTION members when the data structure has operations that are intrinsic to it:

```
RECORD Context:
    values : Map<String, Any>

    FUNCTION set(key, value):
        values[key] = value

    FUNCTION get(key, default=NONE) -> Any:
        RETURN values.get(key, default)

    FUNCTION snapshot() -> Map<String, Any>:
        RETURN copy of values
```

Use methods when the operation is intrinsic to the data. Use standalone FUNCTION definitions when the operation could work on multiple types.

#### Inline Defaults

Fields may include default values using `= value` syntax:

```
RECORD SessionConfig:
    max_turns               : Integer = 0       -- 0 = unlimited
    default_timeout_ms      : Integer = 10000   -- 10 seconds
```

Inline defaults are appropriate when defaults are simple and benefit from co-location with the field definition. For configuration surfaces with many keys, attribute tables (see Section 10.1) remain preferable because they group defaults into a scannable column.

#### Convenience Constructors and Derived Properties

A RECORD may define factory methods and computed accessors:

```
RECORD Message:
    role    : Role
    content : List<ContentPart>

    -- Convenience constructors
    FUNCTION Message.system(text) -> Message
    FUNCTION Message.user(text) -> Message
    FUNCTION Message.tool_result(call_id, content) -> Message

    -- Derived properties
    PROPERTY text -> String               -- concatenates all text content parts
    PROPERTY tool_calls -> List<ToolCall>  -- extracts tool call parts
```

### 9.2 ENUM -- Closed Value Sets

An `ENUM` defines a closed set of named values. Each value includes an inline comment:

```
ENUM Role:
    SYSTEM       -- High-level instructions shaping model behavior
    USER         -- Human input
    ASSISTANT    -- Model output
    TOOL         -- Tool execution results
```

When an ENUM's values trigger distinct system behavior, follow it immediately with a classification table (see Section 10.3) that defines the behavioral consequence of each value:

```
ENUM StageStatus:
    SUCCESS          -- Stage completed
    PARTIAL_SUCCESS  -- Completed with caveats
    FAIL             -- Stage failed

| Status             | Meaning                                               |
|--------------------|-------------------------------------------------------|
| `SUCCESS`          | Proceed to next edge. Reset retry counter.            |
| `PARTIAL_SUCCESS`  | Treated as success for routing. Notes describe caveats. |
| `FAIL`             | Route to failure path or terminate pipeline.          |
```

The ENUM defines the values; the table defines what the system does in response. Placing them adjacent ensures both are updated together.

### 9.3 INTERFACE -- Behavioral Contracts

An `INTERFACE` defines a behavioral contract -- a set of functions that implementations must provide:

```
INTERFACE Handler:
    FUNCTION execute(node, context, graph) -> Outcome
```

Interfaces define the contract. Subsequent sections specialize them, showing how each implementation conforms. This establishes an *interface-then-specializations* pattern: the interface appears once, and each handler/adapter section demonstrates conformance.

#### Side-Effect and Mutability Contracts

When an interface has constraints on what implementations must NOT do, state them explicitly:

> "Transforms must not modify the input graph. They receive a graph and return a new graph."

Side-effect constraints are as important as functional contracts. They prevent implementors from taking shortcuts that break composability.

#### Capability Metadata

An INTERFACE may include capability flags -- boolean or typed fields that describe what an implementation supports, not what it does:

```
INTERFACE ProviderAdapter:
    FUNCTION complete(request) -> Response
    FUNCTION stream(request) -> AsyncIterator<StreamEvent>

    -- Capability metadata
    supports_streaming           : Boolean
    supports_parallel_tool_calls : Boolean
    context_window_size          : Integer
```

Capability flags inform callers which code paths are available without requiring trial and error.

#### Optional Methods

When an interface has methods that not all implementations need to provide, mark them explicitly:

```
INTERFACE ProviderAdapter:
    -- Required
    FUNCTION complete(request) -> Response
    FUNCTION stream(request) -> AsyncIterator<StreamEvent>

    -- Optional
    FUNCTION close()          -- release resources
    FUNCTION initialize()     -- one-time setup
```

State which methods are required and which are optional. Optional methods include a comment explaining when an implementation would provide them.

### 9.4 Domain-Specific Keywords

Specs may introduce new definition keywords where the domain demands them. Domain-specific keywords follow the same formatting conventions as core keywords:

1. UPPER CASE keyword
2. Colon-terminated header
3. Indented body with typed fields and `--` comments

The spec that introduces a keyword must define it explicitly before first use. Other specs need not adopt it.

Example: a coding agent spec introduces `TOOL` for agent-facing tool interfaces:

```
TOOL spawn_agent:
    description: "Spawn a subagent to handle a scoped task."
    parameters:
        task        : String (required)     -- natural language description
        max_turns   : Integer (optional)    -- turn limit (default: 50)
    returns: Agent ID and initial status
```

Other domains might introduce their own -- `EVENT`, `MIGRATION`, `RULE`, `ENDPOINT` -- as long as they follow the same conventions and are used consistently within their spec.

### 9.5 Namespace Conventions for Shared State

When a spec defines shared mutable state (a context object, a registry, a configuration store), partition keys by prefix to prevent collisions between subsystems:

```
| Prefix        | Owner                   | Purpose                              |
|---------------|-------------------------|--------------------------------------|
| `context.*`   | Application logic       | Semantic state shared between stages |
| `graph.*`     | Engine                  | Graph attributes mirrored at init    |
| `internal.*`  | Engine                  | Bookkeeping (retry counters, timing) |
| `parallel.*`  | Parallel handler        | Branch results and counts            |
```

The prefix table is normative: each subsystem reads and writes only its own namespace.

---

## 10. Tables

Tables in an NLSpec are normative, not decorative. Each table row is a testable requirement. Use whatever table structure communicates the data clearly -- the patterns below are common across exemplars, not an exhaustive taxonomy.

### 10.1 Attribute Tables

Attribute tables define configuration points with defaults. The Default column is critical -- it documents fallback behavior without requiring prose. Every configurable value in the spec must have an explicit default.

```
| Key           | Type    | Default  | Description                          |
|---------------|---------|----------|--------------------------------------|
| `max_retries` | Integer | `0`      | Additional retry attempts            |
| `timeout`     | Duration| `"30s"`  | Maximum execution time per attempt   |
| `prompt`      | String  | `""`     | LLM prompt (supports variable expansion) |
```

### 10.2 Mapping Tables

Mapping tables translate between two or more systems (providers, protocols, formats):

```
| Provider  | Provider Value | Unified Value |
|-----------|----------------|---------------|
| OpenAI    | stop           | stop          |
| Anthropic | end_turn       | stop          |
| Gemini    | STOP           | stop          |
```

Use mapping tables when a spec bridges multiple external systems and identical behavior must be demonstrated across all of them.

### 10.3 Classification Tables

Classification tables assign behavioral implications to categories (error types, status codes, enum values). The Meaning column states what the system *does*, not just what the name means:

```
| Status             | Meaning                                               |
|--------------------|-------------------------------------------------------|
| `SUCCESS`          | Proceed to next edge. Reset retry counter.            |
| `PARTIAL_SUCCESS`  | Completed with caveats. Treated as success for routing. |
| `FAIL`             | Route to failure path.                                |
```

### 10.4 Validation Matrices

Validation matrices ensure parity across multiple implementations. Each cell is a checkbox:

```
| Test Case                | Impl A | Impl B | Impl C |
|--------------------------|--------|--------|--------|
| Basic operation          | [ ]    | [ ]    | [ ]    |
| Error recovery           | [ ]    | [ ]    | [ ]    |
| Edge case handling       | [ ]    | [ ]    | [ ]    |
```

### 10.5 Other Common Table Patterns

Several additional table patterns recur across exemplars:

- **Preset/Policy tables** define named configurations (e.g., retry policies with delay, factor, max attempts)
- **Mode tables** show how mutually exclusive modes affect multiple behavioral dimensions simultaneously
- **Constraint matrices** document valid combinations between two dimensions of a system (e.g., which content types are valid in which message roles)
- **Comparison matrices** document how multiple implementations differ across many dimensions

The principle is the same for all: each row is a testable assertion. If a table row cannot be tested, the table is decorative and should be removed or made precise.

---

## 11. Formal Grammar

NLSpec draws on established specification languages -- BNF, EBNF, and similar grammar notations -- where the domain demands them.

### 11.1 When to Use Formal Grammar

A spec that defines a parseable syntax (a DSL, a file format, an expression language, a wire protocol) should use a formal grammar to define it. A spec that defines only APIs, data structures, and algorithms has no need for one.

Not every NLSpec will have a grammar section. The decision depends on the domain, not on the format.

### 11.2 Syntax-Semantics Separation

The grammar defines *what you can write* (syntax). Semantics (*what it means*) are specified separately in prose and pseudocode. This separation lets readers learn syntax without being overwhelmed by behavior.

### 11.3 Notation Conventions

The exact notation (`::=` for BNF, `=` for EBNF) does not matter. What matters is consistency within a single spec. The grammar must be complete enough to implement a parser: every non-terminal must have a production, and terminal symbols must be unambiguous.

Operator precedence can be documented through grammar structure rather than a separate table:

```
Expr       ::= Term ( '+' Term )*
Term       ::= Factor ( '*' Factor )*
```

This shows that `*` binds tighter than `+` structurally.

---

## 12. Examples, Diagrams, and Formal Notations

### 12.1 Use Established Notations

NLSpec embraces established specification languages and diagram notations wherever they communicate more clearly than prose or pseudocode. Do not reinvent a notation when an established one exists:

| Notation | Use for |
|----------|---------|
| BNF / EBNF | Parseable syntax, DSLs, wire formats (see Section 11) |
| State diagrams | Lifecycle phases, session states, mode transitions |
| Sequence diagrams | Multi-party message flow, request/response patterns |
| ASCII box diagrams | System layers, component boundaries, architecture overview |
| Directory trees | Filesystem artifacts, output layout |
| Regular expressions | Pattern matching, string validation rules |
| Truth tables | Combinatorial logic, valid input combinations |

The principle: use whatever established notation communicates the concept to the reader. If the domain has a standard representation, use it.

### 12.2 Example Progression

Examples progress from minimal to complex:

1. **Minimal viable example** -- the simplest valid instance
2. **Feature example** -- a single feature in isolation
3. **Composition example** -- features combining
4. **Integration smoke test** -- end-to-end pseudocode test

### 12.3 Format and Placement

Examples use fenced code blocks with language hints where applicable (e.g., `` ```dot ``, `` ```python ``, `` ```bash ``). Pseudocode examples use plain fenced blocks without a hint.

Short examples are embedded immediately after the concept they illustrate. Longer examples (full conversations, complex workflows) are placed in appendices with a cross-reference from the body.

### 12.4 Examples Are Illustrative, Not Normative

Pseudocode and tables are normative. Examples show how normative definitions play out in practice. If an example contradicts the pseudocode or a table, the pseudocode or table governs.

---

## 13. Language and Voice

### 13.1 Prescriptive Present Tense

Behavior is stated as fact, in present tense:

> "The engine selects the next edge."
>
> "The context is a thread-safe key-value store."

Do not use future tense ("will select"), conditional ("would select"), or subjunctive ("should select") for core behavior.

### 13.2 Imperative for Requirements

Hard requirements use imperative voice or RFC-style keywords:

> "Every graph must have exactly one start node."
>
> "Handlers MUST be stateless."

### 13.3 Declarative for Design Constraints

Architectural constraints are stated as factual assertions, often in the negative:

> "The execution engine does not know about handler internals."
>
> "Variable expansion is simple string replacement, not a templating engine."

Negative constraints ("does not", "is not") bound the scope of the system.

### 13.4 Motivational Framing

Requirements are accompanied by inline justification when the "why" is not obvious:

> "The default protects against runaway processes without being so short that normal operations fail."

The justification follows the requirement, not the other way around. State what, then explain why.

### 13.5 The Critical Marker

Section titles may be tagged with **(Critical)** to signal concepts that implementors must not skip or underestimate:

```markdown
### 2.7 Native API Usage (Critical)
```

Use sparingly. If more than two or three sections are marked Critical, the marker loses its signal value.

### 13.6 Anti-Jargon Discipline

Introduce terms only when needed and define them immediately on first use. Once a concept has a name, use that name consistently throughout. Do not use synonyms for defined terms.

---

## 14. Edge Cases and Error Handling

### 14.1 Fallback Chains

Complex decisions with multiple fallback paths are documented as numbered cascades:

```
1. Preferred path: do X if condition holds
2. Secondary path: do Y if X is unavailable
3. Terminal path: fail with specific error
```

The cascade makes explicit what happens at each step when earlier steps fail. Every chain must terminate -- the last step cannot fall through.

### 14.2 Boundary Conditions in Pseudocode

Edge cases appear directly in the algorithm pseudocode, not in separate prose:

```
IF attempt < max_attempts:
    retry(node)
ELSE:
    IF node.allow_partial:
        RETURN Outcome(status=PARTIAL_SUCCESS)
    RETURN Outcome(status=FAIL, reason="max retries exceeded")
```

This ensures edge cases are not separated from the code paths they affect.

### 14.3 Error Categories

When the spec defines multiple error types, classify them in a table with three columns:

| Column | Required | Content |
|--------|----------|---------|
| Error Type | Yes | The error category name |
| Example | Yes | A concrete instance |
| Recovery | Yes | What the system does in response |

The Recovery column is required. An error category without a recovery strategy is incomplete.

### 14.4 Default Behavior Documentation

Defaults are stated with their rationale:

> "Unknown errors default to retryable. This is a deliberate conservative choice: transient failures are more common than permanent ones."

This prevents implementors from choosing a different default without understanding the trade-off.

### 14.5 Precedence Hierarchies

When multiple sources can define the same value, document the resolution order as a numbered precedence list (highest to lowest):

```
**Fidelity resolution precedence:**

1. Edge attribute (on the incoming edge)
2. Target node attribute
3. Graph-level default attribute
4. System default: `compact`
```

Precedence hierarchies are distinct from fallback chains (Section 14.1). Fallback chains describe what to try when something fails. Precedence hierarchies describe which source wins when multiple sources define the same value. Every hierarchy must terminate with a default.

### 14.6 Procedural Sequences

When a system must execute a series of steps in a fixed order where every step runs (not just the first that succeeds), document them as a numbered procedural sequence:

```
**Graceful shutdown sequence:**

1. Cancel any in-flight LLM stream
2. Send SIGTERM to all running processes
3. Wait 2 seconds
4. Send SIGKILL to any remaining processes
5. Flush pending events
6. Emit SESSION_END event
7. Transition session to CLOSED
```

Procedural sequences are distinct from fallback chains: every step executes. They are distinct from algorithms (which belong in pseudocode): they describe operations at the system boundary where the steps are imperative commands, not computational logic.

### 14.7 Escape Hatches

When a spec defines an abstraction that hides implementation differences, it may also need to expose a deliberate bypass for cases the abstraction cannot cover:

```
RECORD Request:
    model            : String
    messages         : List<Message>
    provider_options : Map<String, Any>  -- provider-specific passthrough
```

Document escape hatches with an explicit non-portability warning:

> "Code that uses `provider_options` is explicitly not portable. The library documents this tradeoff: when a provider offers a unique feature that does not map to the unified model, provide a pass-through rather than pretending the feature does not exist."

The escape hatch is a deliberate design decision, not a gap in the abstraction. Mark it as such.

---

## 15. Cross-Referencing and Dependencies

### 15.1 Internal References

Internal references use the pattern "(see Section X.Y)" for inline references within the same spec:

> "Edges whose condition evaluates to `true` are eligible (see Section 14)."

Forward references preview upcoming material; backward references recall established concepts.

### 15.2 Inter-Spec Dependencies

When a system is specified across multiple NLSpec documents, each spec declares its relationship to the others. There are two dependency strengths.

**Hard dependencies** mean the spec cannot be implemented without the dependency. They are declared using three mechanisms in combination:

| Mechanism | Placement | What it communicates |
|-----------|-----------|---------------------|
| Preamble declaration | First paragraph | "Read this other spec first" |
| "Relationship to X" section | Section 1.x | Exact types imported; the contract surface |
| "Does NOT use" clarification | Near dependency declaration | Which layer is consumed and why |

A hard dependency lists the exact types imported from the dependency spec. This makes the contract explicit and testable: if those types exist with those names, the dependency is satisfied.

When the dependency exposes multiple layers (e.g., a low-level API and a high-level convenience API), the consuming spec states which layer it uses and why. This prevents implementors from wiring to the wrong layer.

**Soft dependencies** mean the spec can use the other but does not require it:

| Mechanism | Placement | What it communicates |
|-----------|-----------|---------------------|
| Interface boundary | Layering section + INTERFACE block | "Implement this interface; we don't care how" |
| Companion mention | Layering section | "These specs were designed together but are not required" |

**Dependency direction.** Specs lower in a stack are self-contained. Specs higher in a stack declare their dependencies. Dependency arrows always point downward.

### 15.3 External References

External specs, tools, and standards are referenced with full URLs:

> "For reference on DOT syntax, see the Graphviz DOT language specification: https://graphviz.org/doc/info/lang.html"

---

## 16. Out of Scope

**NLSpec authoring tooling.** Linters, validators, or IDE extensions for NLSpec documents. This spec defines the format; tooling is an implementation concern. The integration smoke test in Section 18 provides a starting point for a validator.

**Requirement gathering.** How to determine what a spec should contain. This precedes spec writing and is outside the format specification.

**System decomposition.** How to decide which parts of a system get their own NLSpec vs. being sections within a larger spec. This is an architectural decision that depends on the system.

**Version control conventions.** How to manage NLSpec documents in git, how to handle spec diffs, or how to review spec changes. These are workflow concerns.

---

## 17. Design Decision Rationale

**Why pseudocode instead of a real programming language?** A real language biases the implementation toward that language's idioms, libraries, and type system. Pseudocode is translatable to any language. The spec's job is to be precise, not executable.

**Why `--` for comments instead of `//` or `#`?** `--` is visually distinct from every major language's comment syntax, which prevents readers from assuming the pseudocode *is* a real language. It also avoids colliding with markdown formatting (`#` is a heading).

**Why UPPER CASE keywords?** Keywords must be visually distinguishable from identifiers at a glance. UPPER CASE keywords stand out against `snake_case` names without requiring syntax highlighting.

**Why require a Default column in attribute tables?** Omitting defaults forces the implementor to guess. A coding agent with no default will either pick an arbitrary value or raise an error. The Default column eliminates this class of ambiguity entirely.

**Why mirror DoD sections to body sections instead of organizing by feature?** The mirroring structure creates a mechanical check: does every body section have coverage? Does every DoD item trace to a body section? Organizing by "feature" allows subjective grouping that obscures gaps.

**Why separate syntax (grammar) from semantics (pseudocode)?** Grammars define what is parseable. Pseudocode defines what it means. Mixing them forces the reader to learn two things at once. Separation lets a coding agent implement the parser first, then wire up the semantics -- a natural two-pass approach.

**Why not fully specify the pseudocode language?** Pseudocode's power is that it is obvious in context. A coding agent reading `AWAIT_ALL(futures)` or `items[0..n]` does not need a spec entry to understand the intent. Specifying every keyword and operator would turn the pseudocode into a programming language, which contradicts the language-agnosticism principle.

---

## 18. Definition of Done

### 18.1 Document Structure

- [ ] Document follows the four-phase arc (Why -> What -> How -> Done)
- [ ] All sections use decimal numbering (`X.Y`)
- [ ] Linked table of contents is present after the opening paragraph
- [ ] Horizontal rules separate top-level sections
- [ ] Multiple implementations use parallel subsection structure (if applicable)

### 18.2 Preamble

- [ ] Opening paragraph states what the thing is and who it is for
- [ ] Problem statement follows the status-quo -> pain -> solution structure
- [ ] Design principles are bold-lead paragraphs with named constraints
- [ ] Layering statement declares what the spec covers and does not cover

### 18.3 "Out of Scope" Section

- [ ] Each out-of-scope item includes name, description, exclusion reason, and extension point
- [ ] Layering is used to justify exclusions where applicable

### 18.4 "Design Decision Rationale" Section

- [ ] Design decisions use bold-question format naming the rejected alternative
- [ ] Rejected approaches appear inline near the relevant section (if applicable)
- [ ] Time-sensitive data is marked with temporal anchors (if applicable)

### 18.5 "Definition of Done" Section

- [ ] DoD subsections mirror body sections
- [ ] Each DoD item traces to a specific body section
- [ ] Final item is an integration smoke test (where applicable)
- [ ] Validation matrices are included for multi-implementation specs

### 18.6 "Appendix" Section

- [ ] Appendices (if any) hold reference material, not narrative content
- [ ] Each appendix is referenced from at least one body section
- [ ] Appendices are lettered sequentially (Appendix A, B, C)

### 18.7 Pseudocode Language

- [ ] Keywords are UPPER CASE
- [ ] Names follow case conventions (snake_case, PascalCase, UPPER_CASE)
- [ ] Comments use `--` (double dash)
- [ ] Complex algorithms use labeled steps as inline comments (if applicable)
- [ ] Layered composition uses numbered additive notation (if applicable)
- [ ] Complex functions include behavior summaries (if applicable)

### 18.8 Type Definitions

- [ ] RECORD definitions include name, type, and inline comment for each field
- [ ] Large RECORDs use comment lines to group related fields (if applicable)
- [ ] Field constraints are documented as bold-lead paragraphs after the RECORD (if applicable)
- [ ] ENUM definitions include inline comment for each value
- [ ] ENUM definitions that drive behavior are paired with classification tables (if applicable)
- [ ] INTERFACE definitions specify behavioral contracts
- [ ] INTERFACE side-effect and mutability constraints are stated explicitly (if applicable)
- [ ] Domain-specific keywords (if any) follow core formatting conventions and are defined before first use

### 18.9 Tables

- [ ] Tables are normative (each row is a testable requirement)
- [ ] Attribute tables include a Default column
- [ ] Classification tables include behavioral implications, not just names
- [ ] Error category tables include a Recovery column

### 18.10 Formal Grammar

- [ ] Formal grammar is present only where the domain demands it (parseable syntax)
- [ ] Grammar defines syntax; semantics are specified separately
- [ ] All non-terminals have productions; terminals are unambiguous
- [ ] Notation is consistent within the spec

### 18.11 Examples, Diagrams, and Formal Notations

- [ ] Established notations are used where they communicate more clearly than prose
- [ ] Examples progress from minimal to complex
- [ ] Examples use fenced code blocks with language hints where applicable
- [ ] Short examples are inline; long examples are in appendices
- [ ] Examples are illustrative, not normative

### 18.12 Language and Voice

- [ ] Core behavior uses prescriptive present tense
- [ ] Requirements use imperative voice
- [ ] Design constraints use declarative (often negative) assertions
- [ ] Terms are defined on first use and used consistently throughout
- [ ] (Critical) markers are used sparingly (no more than 2-3 per spec)

### 18.13 Edge Cases and Error Handling

- [ ] Fallback chains are numbered cascades that terminate
- [ ] Edge cases appear in pseudocode, not in separate prose
- [ ] Error categories include recovery strategies
- [ ] Defaults are stated with rationale
- [ ] Precedence hierarchies are numbered lists with a terminal default (if applicable)
- [ ] Procedural sequences document fixed-order operations (if applicable)
- [ ] Escape hatches are documented with explicit non-portability warnings (if applicable)

### 18.14 Cross-Referencing and Dependencies

- [ ] Internal references use "(see Section X.Y)" format
- [ ] Hard dependencies are declared in the preamble with types imported
- [ ] Soft dependencies use INTERFACE boundaries
- [ ] External references include full URLs

### 18.15 Integration Smoke Test

- [ ] A coding agent supplied with this spec and a project description produces a conforming NLSpec document

Minimal acceptance test:

```
FUNCTION validate_nlspec(document) -> ValidationResult:
    errors = []

    -- Phase 1: Structure
    IF document.table_of_contents is missing:
        errors.append("Missing table of contents")
    IF document.opening_paragraph is missing:
        errors.append("Missing opening paragraph")
    IF document.problem_statement is missing:
        errors.append("Missing problem statement")
    IF document.definition_of_done is missing:
        errors.append("Missing Definition of Done section")
    IF document.out_of_scope is missing:
        errors.append("Missing Out of Scope section")

    -- Phase 2: Closed loop
    FOR EACH section IN document.body_sections:
        IF section defines testable behavior:
            IF NOT document.definition_of_done.has_mirror(section):
                errors.append("Body section has no DoD mirror: " + section.title)

    FOR EACH item IN document.definition_of_done.items:
        IF NOT item.traces_to_body_section():
            errors.append("DoD item has no body section: " + item.text)

    -- Phase 3: Pseudocode conventions (if pseudocode present)
    FOR EACH block IN document.pseudocode_blocks:
        FOR EACH keyword IN block.keywords:
            IF keyword != keyword.upper():
                errors.append("Keyword not UPPER CASE: " + keyword)

    -- Phase 4: Table conventions
    FOR EACH table IN document.attribute_tables:
        IF "Default" NOT IN table.columns:
            errors.append("Attribute table missing Default column")

    RETURN ValidationResult(
        valid = errors.length == 0,
        errors = errors
    )
```
