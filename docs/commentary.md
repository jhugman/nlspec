# NLSpec Commentary

This is a companion to [nlspec.nlspec.md](./nlspec.nlspec.md), which defines the NLSpec format normatively. This document provides the analytical layer: the evidence drawn from three exemplar specs, cross-spec observations, and the structural explanation for why the format produces thorough specifications. The spec tells you *what to do*; this commentary shows you *where we saw it* and *why it works*.

---

## Table of Contents

1. [The Exemplar Specs](#1-the-exemplar-specs)
2. [The Preamble in Practice](#2-the-preamble-in-practice)
3. [Formal Grammar: When and When Not](#3-formal-grammar-when-and-when-not)
4. [Language and Voice in Practice](#4-language-and-voice-in-practice)
5. [Edge Cases in Practice](#5-edge-cases-in-practice)
6. [Inter-Spec Dependencies in Detail](#6-inter-spec-dependencies-in-detail)
7. [Design Rationale as a Pattern](#7-design-rationale-as-a-pattern)
8. [Scope Boundaries as a Pattern](#8-scope-boundaries-as-a-pattern)
9. [Why NLSpecs Are Thorough](#9-why-nlspecs-are-thorough)

---

## 1. The Exemplar Specs

The patterns in the NLSpec spec were extracted from three real specifications. Each covers a different domain and exercises different parts of the format:

| Spec | Domain | Lines | DoD items | Distinguishing features |
|------|--------|-------|-----------|------------------------|
| attractor-spec.md | Pipeline orchestration | 2,312 | 95 | BNF grammar (37 productions), condition expression language, 11 DoD subsections |
| unified-llm-spec.md | Multi-provider LLM client | 2,169 | 70 | Provider mapping tables, validation matrices, (Critical) markers, no grammar |
| coding-agent-loop-spec.md | Coding agent architecture | 1,451 | 59 | TOOL keyword (11 definitions), hard dependency on companion spec, EBNF in appendix only |

The three form a layered stack (see Section 6.4). Together they exercise every pattern in the spec at least once, and no single pattern appears in only one spec except where the domain genuinely demands it.

---

## 2. The Preamble in Practice

*Commentary on spec Sections 3.1-3.5.*

### 2.1 Opening Paragraphs

Each exemplar spec opens with one or two sentences that name the artifact and its purpose. No jargon, no implementation detail:

> "A DOT-based pipeline runner that uses directed graphs (defined in Graphviz DOT syntax) to orchestrate multi-stage AI workflows."
>
> "This document is a consolidated, language-agnostic specification for building a unified client library that provides a single interface across multiple LLM providers."
>
> "This document is a language-agnostic specification for building a coding agent -- an autonomous system that pairs a large language model with developer tools through an agentic loop."

Note the consistency: all three mention "language-agnostic" or avoid naming a language. All three name the thing and what it does in a single breath.

### 2.2 Problem Statements

Each spec names the pain concretely before proposing any solution:

> "Without a structured orchestration layer, developers either write fragile imperative scripts or build ad-hoc state machines that are difficult to visualize, version, or debug."
>
> "Each provider -- OpenAI, Anthropic, Google Gemini, and others -- exposes a different HTTP API with different message formats, tool calling conventions, streaming protocols, error shapes, and authentication mechanisms."

The pattern is consistent: status quo, then pain, then solution. The pain is specific enough that a reader who has felt it will recognize it immediately.

### 2.3 Design Principles

From the attractor-spec, two principles that demonstrate the bold-lead format and the named-constraint style:

```markdown
**Declarative pipelines.** The `.dot` file declares what the workflow
looks like and what each stage should do. The execution engine decides
how and when to run each stage.

**Pluggable handlers.** Each node type is backed by a handler that
implements a common interface. New node types are added by registering
new handlers.
```

Each principle name is a compressed summary of the constraint. The sentence that follows explains the consequence: "declares what... the engine decides how" tells you the boundary between user and system.

### 2.4 Reference Projects

From the attractor-spec:

> "The following open-source projects solve related problems and are worth studying for patterns, trade-offs, and lessons learned. They are not dependencies; implementors may take inspiration from any combination of them."

The framing matters: "not dependencies" prevents a coding agent from adding unwanted imports.

### 2.5 Layering

From the attractor-spec:

> "Attractor defines the orchestration layer: graph definition, traversal, state management, and extensibility. It does NOT require any specific LLM integration."

The "does NOT" is doing work: it tells a coding agent not to wire in an LLM client as a dependency of the core library. Without this, a reasonable agent might assume LLM integration is required because every example pipeline calls an LLM.

---

## 3. Formal Grammar: When and When Not

*Commentary on spec Section 11.*

Of the three exemplar specs, the distribution of formal grammar is:

| Spec | Grammar usage | Count |
|------|--------------|-------|
| attractor-spec | BNF throughout body (DOT subset + condition expressions) | 37 `::=` productions |
| coding-agent-loop-spec | EBNF-like notation in Appendix A only (apply_patch format) | 1 appendix |
| unified-llm-spec | None | 0 |

This distribution is not accidental. The attractor-spec defines a DSL (a DOT subset) and an expression language (conditions), both of which are parseable syntaxes. The coding-agent-loop-spec defines one file format (apply_patch). The unified-llm-spec defines APIs and data structures with no parseable syntax.

The lesson: grammar is a tool for a specific job (specifying parseable syntax), not a marker of spec quality. A spec with no grammar is not incomplete; it simply has no syntax to define.

From the attractor-spec, an example showing how grammar structure encodes operator precedence:

```
ConditionExpr  ::= AndGroup ( '||' AndGroup )*
AndGroup       ::= Clause ( '&&' Clause )*
```

This shows that `&&` binds tighter than `||` structurally, without a separate precedence table.

---

## 4. Language and Voice in Practice

*Commentary on spec Section 13.*

The three voice modes described in the spec (prescriptive present, imperative, declarative) are easier to understand through examples drawn from the exemplar specs.

### 4.1 Prescriptive Present Tense

Behavior stated as fact:

> "After a node completes, the engine selects the next edge."
>
> "The context is a thread-safe key-value store."
>
> "Streaming is a first-class operation, not a flag on a blocking call."

Note the third example: it uses the present tense to make a design *decision* sound like a *fact*. This is intentional. It eliminates hedging.

### 4.2 Imperative for Requirements

> "Every graph must have exactly one start node."
>
> "Handlers MUST be stateless or protect shared mutable state with synchronization."
>
> "Each provider adapter MUST use the provider's native, preferred API."

The shift from lowercase "must" to uppercase "MUST" varies across specs. Both work. What matters is that the requirement is unambiguous.

### 4.3 Declarative for Design Constraints

> "The execution engine does not know about handler internals."
>
> "Variable expansion is simple string replacement, not a templating engine."
>
> "The library does not invent its own model namespace."

Note the pattern: these state what the system does *not* do. Each negative constraint prevents a class of over-engineering. Without "not a templating engine," a coding agent might reasonably build Handlebars-style template support.

### 4.4 Motivational Framing

> "This is the entire value proposition: the complexity of three different APIs is handled once in the adapters so that downstream consumers never have to think about it."
>
> "The default protects against runaway processes without being so short that normal operations fail."

The first example is unusually direct for a spec. It works because it tells the coding agent *what matters most* -- a priority signal, not just a requirement.

### 4.5 The Critical Marker

From the unified-llm-spec:

> "### 2.7 Native API Usage (Critical)"
>
> "### 2.10 Prompt Caching (Critical for Cost)"

Only the unified-llm-spec uses (Critical) markers, and only twice. The attractor-spec and coding-agent-loop-spec do not use them at all. This is a tool for specs where certain sections risk being underestimated, not a required element.

### 4.6 Anti-Jargon Discipline

From the attractor-spec, the same terms are used consistently across 2,312 lines: "Context" always means the key-value store. "Outcome" always means the StageStatus record. "Handler" always means the node execution interface. No synonyms. A coding agent never has to wonder whether "result" and "outcome" mean the same thing.

---

## 5. Edge Cases in Practice

*Commentary on spec Section 14.*

### 5.1 A Real Fallback Chain

From the attractor-spec, the failure routing cascade:

```
1. Fail edge: route to edge labeled "fail" if present
2. Retry target: jump to node.retry_target if set
3. Fallback retry target: jump to node.fallback_retry_target if set
4. Pipeline termination: fail the pipeline
```

Four steps, each with a concrete action, terminating in a definitive outcome. Compare this to a prose description like "the engine handles failures appropriately" -- a coding agent cannot implement "appropriately."

### 5.2 A Real Error Category Table

From the coding-agent-loop-spec:

```
| Error Type          | Example                          | Recovery |
|---------------------|----------------------------------|----------|
| FileNotFound        | read_file on nonexistent path    | Model can search for correct path |
| EditConflict        | old_string not found             | Model can read file and retry |
| ShellTimeout        | Command exceeded timeout_ms      | Model can retry with longer timeout |
```

The Recovery column tells the LLM (which is both implementor and user of the system) how to respond. This is a table that serves two audiences simultaneously.

### 5.3 A Real Default with Rationale

From the unified-llm-spec:

> "Unknown errors default to retryable. This is a deliberate conservative choice: transient network issues and novel provider error codes are more common than permanent failures."

The word "deliberate" signals that this was a considered decision, not an arbitrary default. The rationale ("transient... more common than permanent") prevents an implementor from flipping the default without understanding the trade-off.

---

## 6. Inter-Spec Dependencies in Detail

*Commentary on spec Section 15.2.*

The spec compresses inter-spec dependencies into two tables. This section expands the evidence: the five named mechanisms, their placement, and the quotes that demonstrate each one.

### 6.1 Hard Dependencies: Three Mechanisms

**Mechanism 1: Preamble declaration.** The first paragraph states the dependency before any content:

> "This spec layers on top of the [Unified LLM Client Specification](./unified-llm-spec.md), which handles all LLM communication."

This tells the reader immediately: *stop and read that spec first.*

**Mechanism 2: "Relationship to X" section.** A dedicated subsection lists the exact types imported:

```markdown
### 1.5 Relationship to the Unified LLM SDK

This spec assumes the companion Unified LLM Client Specification
is implemented. The agent loop imports and uses these types directly:

- `Client`, `Request`, `Response` -- for LLM communication
- `Message`, `ContentPart`, `Role` -- for conversation history
- `Tool`, `ToolCall`, `ToolResult` -- for tool definitions and results
- `StreamEvent` -- for streaming responses
- `Usage` -- for token tracking
- `FinishReason` -- for stop condition detection
```

The import list makes the contract explicit and testable: if these types exist with these names, the dependency is satisfied.

**Mechanism 3: Layer selection with "does NOT use" clarification.** When a dependency exposes multiple layers, the consuming spec states which layer it uses and why:

> "The agent loop does NOT use the Unified LLM SDK's `generate()` high-level function (which has its own tool loop). It uses the low-level `Client.complete()` and implements its own loop because it needs to interleave tool execution with output truncation, steering message injection, event emission, timeout enforcement, and loop detection -- concerns that the SDK's generic tool loop does not handle."

This prevents implementors from wiring to the wrong layer. Without it, a reasonable coding agent would reach for the high-level `generate()` function.

### 6.2 Soft Dependencies: Two Mechanisms

**Mechanism 4: Interface boundary declaration.** The spec defines its own interface and states it does not require a specific implementation:

> "Attractor defines the orchestration layer: graph definition, traversal, state management, and extensibility. It does NOT require any specific LLM integration."
>
> "The codergen handler takes a backend that conforms to the `CodergenBackend` interface (Section 4.5)."

The INTERFACE pseudocode block is the contract. Anything that implements it satisfies the dependency.

**Mechanism 5: Companion mention as one option among many.** The spec names companion specs as *possible* implementations alongside alternatives:

> "use the companion [Coding Agent Loop](./coding-agent-loop-spec.md) and [Unified LLM Client](./unified-llm-spec.md) specs, spawn CLI agents (Claude Code, Codex, Gemini CLI) in subprocesses, run agents in tmux panes with a manager attaching to them, call an LLM API directly, or anything else."

This communicates: *these specs were designed together, but you are not locked in.*

### 6.3 Summary of Mechanisms

| Mechanism | Placement | Strength | What it communicates |
|-----------|-----------|----------|---------------------|
| Preamble declaration | First paragraph | Hard | "Read this other spec first" |
| "Relationship to X" section | Section 1.x | Hard | Exact types imported; the contract surface |
| "Does NOT use" clarification | Near dependency declaration | Hard | Which layer is consumed and why |
| Interface boundary | Layering section + INTERFACE block | Soft | "Implement this interface; we don't care how" |
| Companion mention | Layering section | Soft | "These specs were designed together but are not required" |

### 6.4 The Dependency Stack

The three exemplar specs form a layered stack:

```
attractor-spec  ──soft──▶  coding-agent-loop-spec  ──hard──▶  unified-llm-spec
```

The lower layer (unified-llm-spec) has no dependency declarations at all -- it is self-contained. The middle layer (coding-agent-loop-spec) hard-depends on the lower. The top layer (attractor-spec) soft-depends on both, through an interface boundary that any backend can satisfy.

---

## 7. Design Rationale as a Pattern

*Commentary on spec Section 5.*

The spec contains its own design rationale. This section describes the *pattern* for writing design rationale sections in NLSpec documents, as observed in the exemplars.

### 7.1 The Bold-Question Format

Design decisions that justify *why not* are collected in a "Design Decision Rationale" appendix. Each entry is a bold question followed by the answer:

> **Why provider-aligned toolsets instead of a universal tool set?** Models may be trained on specific tool formats. GPT-5.2-codex is trained on apply_patch; forcing it to use old_string/new_string editing produces worse results.
>
> **Why a single Request type instead of per-method parameter lists?** A single Request object is easier to construct, pass around, modify, and serialize than many keyword arguments.

The question format matters: it names the alternative that was rejected. This prevents implementors from proposing the same rejected alternative.

### 7.2 Inline Anti-Pattern Documentation

Rejected approaches also appear inline, near the relevant section:

> "A single method with a `stream: boolean` flag was rejected because the return types are fundamentally different."
>
> "No separate `send_tool_outputs` method. Tool results are sent by including them in the message history."

This prevents implementors from making mistakes the author already considered. The inline placement ensures the anti-pattern is visible when the reader is studying the relevant section, not buried in an appendix.

### 7.3 Temporal Anchors

When a spec includes data that will age (model catalogs, API versions, pricing), it marks it explicitly:

> "**At the time of writing (February 2026),** the top models available through each provider's API are:"

This tells a future reader (or coding agent) that the data may be stale, without invalidating the surrounding patterns.

---

## 8. Scope Boundaries as a Pattern

*Commentary on spec Section 4.*

The spec has its own Out of Scope section. This section describes the *pattern* for writing scope boundary sections, as observed in the exemplars.

### 8.1 The Out-of-Scope Item Structure

Each out-of-scope item includes four elements:

1. **Name** -- the feature or concern being excluded
2. **What it is** -- a one-sentence description
3. **Why it is excluded** -- why it does not belong in this spec
4. **Where the extension point is** -- how it could be added later

From the coding-agent-loop-spec:

> **MCP (Model Context Protocol).** An MCP client can extend the agent with tools from external servers. The tool registry supports registering MCP-discovered tools with namespaced names. This is a natural extension but not a core requirement.
>
> **Sandbox / Security Policies.** The `ExecutionEnvironment` abstraction provides a natural hook -- a `SandboxedLocalExecutionEnvironment` could wrap the default environment.

The extension point is critical. Without it, an out-of-scope item sounds like a permanent exclusion. With it, the item becomes a deferred feature with a known integration path. A coding agent reading this knows exactly where to add the capability if asked to.

### 8.2 Layering as Scope Justification

The Out-of-Scope section uses layering to explain *why* something is excluded: "that's handled by the layer below" or "that belongs in a layer above." This overlaps with inter-spec dependency declarations (see Section 6) but serves a different purpose: dependencies say what is imported; scope boundaries say what is excluded and why.

---

## 9. Why NLSpecs Are Thorough

The feeling of thoroughness in these specs is not a matter of diligence. It is a structural property that emerges from three interlocking mechanisms.

### 9.1 The Intended Audience Is a Coding Agent

The attractor-spec readme defines the term explicitly:

> "**NLSpec** (Natural Language Spec): a human-readable spec intended to be directly usable by coding agents to implement/validate behavior."

And the building instructions are:

> "Supply the following prompt to a modern coding agent: `Implement Attractor as described by...`"

This audience constraint is the forcing function. A human reader can fill in gaps with domain knowledge, ask clarifying questions, and make reasonable assumptions. A coding agent cannot. Every ambiguity becomes a wrong implementation. Every missing edge case becomes a bug. Every undefined default becomes a guess. NLSpecs are thorough because **incompleteness is a bug when the reader is a machine**.

This explains the patterns described in the spec: the pseudocode (machines need unambiguous algorithms), the tables with default columns (machines need every value spelled out), the explicit fallback chains (machines need to know what happens when things go wrong), and the formal grammars where syntax is involved (machines need exact parse rules). Each pattern exists to close a gap that a human could paper over but a coding agent would stumble on.

### 9.2 The Definition of Done Mirrors the Body

The Definition of Done subsections map 1:1 to the spec's body sections. From the attractor-spec:

| DoD subsection | Body section |
|----------------|--------------|
| 11.1 DOT Parsing | 2. DOT DSL Schema |
| 11.2 Validation and Linting | 7. Validation and Linting |
| 11.3 Execution Engine | 3. Pipeline Execution Engine |
| 11.4 Goal Gate Enforcement | 3. (subsection) |
| 11.5 Retry Logic | 3. (subsection) |
| 11.6 Node Handlers | 4. Node Handlers |
| 11.7 State and Context | 5. State and Context |
| 11.8 Human-in-the-Loop | 6. Human-in-the-Loop |
| 11.9 Condition Expressions | 10. Condition Expression Language |
| 11.10 Model Stylesheet | 8. Model Stylesheet |
| 11.11 Transforms | 9. Transforms and Extensibility |

Every body section has a corresponding DoD subsection. Every DoD item references a specific behavior described in the body. This creates a **closed loop**: if a body section exists without DoD coverage, the gap is visible. If a DoD item exists without a body section explaining it, that gap is also visible. The two halves of the spec audit each other.

### 9.3 The Requirement Density Is High

The three specs have 95, 70, and 59 checklist items across 2312, 2169, and 1451 lines respectively -- roughly **one testable requirement per 25 lines of spec**:

| Spec | Lines | DoD items | Lines per item |
|------|-------|-----------|----------------|
| attractor-spec | 2,312 | 95 | ~24 |
| unified-llm-spec | 2,169 | 70 | ~31 |
| coding-agent-loop-spec | 1,451 | 59 | ~25 |

This density means the spec cannot get away with vague hand-waving. Every section must contain enough detail to support its corresponding checklist items. The checklist acts as a compression test: if you can't write a concrete checkbox for it, the body section isn't specific enough.

### 9.4 The Closed Loop in Practice

The three mechanisms reinforce each other:

```
Audience (coding agent) ──forces──▶ Precision in body sections
                                          │
Body sections ──generate──▶ DoD checklist items
                                          │
DoD checklist items ──audit──▶ Body sections for gaps
                                          │
Gaps discovered ──force──▶ More precision in body sections
```

The result is a spec that converges toward completeness not because the author tried to be thorough, but because the structure makes incompleteness visible.
