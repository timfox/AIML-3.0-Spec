# Artificial Intelligence Markup Language (AIML) Version 3.0 Specification

**Latest version:** 'https://github.com/timfox/AIML-3.0-Spec'
**Previous version:** AIML 2.1 community materials (informative references only)
**Editors:** *Tim Fox*

---

## Status of This Document

This file is a **working draft specification** intended for implementation, review, and interoperability testing. It includes an abstract, status section, numbered clauses, RFC 2119 / RFC 8174 keyword usage, and appendices.

**Audience:** This specification is intended for developers, engineers, and technical stakeholders building or maintaining AIML interpreters, validators, authoring environments, systems for content interchange, and cached agentic solutions. It is also relevant to game developers, integration architects, and platform designers who seek to enable interoperable conversational AI—including within gamified environments.

### AIML 3.0 Integration Profile Overview

This draft defines a concrete AIML 3.0 integration profile featuring a serialization-neutral infoset and equivalent encodings in XML, JSON, and USD. It supports interaction directives, world-state effects, multi-party messaging, LLM mediation, performance envelopes, narrative graph packaging, proofing outputs, and interoperability hooks for entity/component systems, event-driven hosts, and planner-driven hosts.

**Implementation status:** The document is suitable for experimental implementations, profile validation, and draft content interchange. Implementers SHOULD treat extension vocabulary, proofing output formats, and host-integration modules as draft surfaces subject to refinement through implementation feedback.

**Open issues:** Future drafts MAY publish companion schemas, example corpora, and stricter conformance fixtures for extension modules and packaging workflows.

---

## Abstract

AIML (Artificial Intelligence Markup Language) is a **declarative dialog rule language** for building conversational agents. AIML 3.0 defines a **serialization-neutral document infoset** of **categories**: each category pairs an **input pattern** with a **template** that produces the agent’s **surface form** (text and optional structured side channels). That infoset MAY be serialized as **XML**, **JSON**, or **Universal Scene Description (USD)**; these serializations are equal citizens of the specification and MUST preserve equivalent processing semantics when they represent the same infoset. AIML 3.0, as specified here, **unifies** the historical AIML 1.x/2.x capability surface under explicit **processing semantics**, **conformance classes**, **normalization profiles**, **interaction directives**, **world-state effects**, **multi-party conversation semantics**, **entity/component/system interoperability**, **event-driven integration**, **planner and goal mediation**, **LLM mediation rules**, and **security constraints**, while reserving **extension modules** for multimodal output, remote services, and client directives.

---

## Table of Contents

1. [Introduction](#1-introduction)  
2. [Terminology](#2-terminology)  
3. [Conformance](#3-conformance)  
4. [Syntax and Document Infoset](#4-syntax-and-document-infoset)  
5. [Processing Model](#5-processing-model)  
6. [Pattern Language](#6-pattern-language)  
7. [Categories, Topics, and Context](#7-categories-topics-and-context)  
8. [Template Language — Core](#8-template-language--core)  
9. [Template Language — Modules](#9-template-language--modules)  
10. [Predicates, Maps, and Bot Properties](#10-predicates-maps-and-bot-properties)  
11. [Recursion and Control Flow](#11-recursion-and-control-flow)  
12. [Internationalization](#12-internationalization)  
13. [Accessibility Considerations](#13-accessibility-considerations)  
14. [Security and Privacy](#14-security-and-privacy)  
15. [Error Handling and Diagnostics](#15-error-handling-and-diagnostics)  
16. [Interchange, Packaging, and Versioning](#16-interchange-packaging-and-versioning)  
17. [Validation](#17-validation)  
18. [Conformance Testing](#18-conformance-testing)  
- [Appendix A: Element Index](#appendix-a-element-index)  
- [Appendix B: Informative References](#appendix-b-informative-references)  
- [Appendix C: Migration Notes from AIML 2.x](#appendix-c-migration-notes-from-aiml-2x)
- [Appendix D: Profile Matrix](#appendix-d-profile-matrix)
- [Appendix E: Serialization Mapping Example](#appendix-e-serialization-mapping-example)
- [Appendix F: Worked Examples](#appendix-f-worked-examples)
- [Appendix G: Multi-User Dungeon Verb Families](#appendix-g-multi-user-dungeon-verb-families)
- [Appendix H: Choice and Link Semantics](#appendix-h-choice-and-link-semantics)

---

## 1. Introduction

### 1.1 Purpose

AIML is designed for **author-controlled** conversational behavior: writers specify **explicit** stimulus–response structure, with optional **probabilistic** variation and **stateful** branching. AIML is not a general-purpose programming language; it is a **specialized markup** for dialog rules with a **well-defined evaluation model**.

### 1.2 Design goals

1. **Authorability:** rules are readable as data, suitable for version control and review.  
2. **Determinism (configurable):** core tiers SHOULD be implementable without network access.  
3. **Interoperability:** independent engines MUST agree on **core matching and template composition** when operating under the same **normalization profile** and **feature tier**.  
4. **Safety:** potentially dangerous capabilities (learning, remote calls) MUST be **opt-in** and **sandboxed**.

### 1.3 Non-goals

- Full natural-language understanding beyond pattern matching and declared vocabularies.  
- Guaranteed human-like coherence absent author-provided coverage.  
- Standardization of any particular **large language model** interface (such integrations belong in **extension modules** or host-specific profiles).

### 1.4 Document conventions

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in normative sections are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174).

### 1.5 Relationship to Multi-User Dungeon systems

AIML 3.0 is not limited to one-to-one chatter. With the optional modules in §9, it can also act as a **world-facing interaction layer** for rooms, groups, inventory transfer, invitations, scheduling, scene hints, and other MUD-like operations, while keeping final authority in the host simulation.

---

## 2. Terminology

| Term | Definition |
|------|------------|
| **Agent** | The software component that evaluates AIML and emits outputs. |
| **Utterance** | A sequence of **tokens** presented as user input to the matching stage. |
| **Token** | An atomic string in the **token sequence** produced by the active **normalization profile** (often a word; MAY be punctuation-attached per profile). |
| **Category** | A rule consisting of a **pattern**, optional **that-pattern**, optional **topic**, and a **template**. |
| **Pattern** | A string or structured pattern expression matched against an utterance (and possibly other context). |
| **Template** | A tree of **template content** evaluated to produce **primary output** and optional **side effects**. |
| **Graph** | The compiled internal representation of all categories used for matching (historically “Graphmaster”; implementations MAY use other structures with equivalent semantics). |
| **Star capture** | Substrings of the utterance bound to wildcard regions in the matched pattern, numbered for retrieval. |
| **That** | The agent’s **immediately prior surface output** (or a normalized derivative), used for contextual matching. |
| **Topic** | A string label representing an explicit dialog segment; affects category visibility. |
| **Predicate** | A named key/value store scoped per **user session** unless otherwise specified. |
| **Bot property** | A named key/value store scoped to the **agent persona** (shared across sessions unless implementation defines otherwise). |
| **Normalization profile** | A named set of deterministic transforms applied to raw user input and optionally to **that** strings before matching. |
| **Document infoset** | The canonical AIML data model independent of any concrete serialization syntax. |
| **Serialization** | A concrete encoding of the AIML infoset, such as XML, JSON, or USD. |
| **Directive** | A machine-readable side effect emitted alongside or inside a template, such as item transfer, invitation acceptance, scheduling, or conversation termination. |
| **Capability** | A host-granted permission that allows AIML or an AIML-adjacent module to request access to a protected operation or data surface. |
| **Authority** | The host-side subsystem that validates and commits requested state changes, conversation actions, or scene changes. |
| **Transaction** | The per-turn bundle of visible reply text, directives, effects, and host-validated state commits. |
| **Conversation target** | The addressed conversation scope for a turn, such as a direct thread, room, group, or channel. |
| **LLM mediation** | A processing stage in which a large language model may propose text or structured payloads under host validation. |
| **Manifest** | Top-level metadata describing an AIML package, world, or story, such as unique id, title, start node, creator, and profile declarations. |
| **Narrative node** | A named authored unit of content with stable identity, body content, metadata, and optional outgoing links. |
| **Choice link** | An explicit navigational or action edge from one narrative node to another, optionally with label, conditions, and side effects. |
| **Package** | A bundle of AIML documents, narrative nodes, resources, metadata, and optional archives exchanged together. |
| **Proofing profile** | A non-playable analysis or export mode used for linting, review, graph inspection, or transformation rather than live execution. |
| **Entity** | A stable runtime identity in a simulation or scene. |
| **Component** | Typed state associated with an entity. |
| **System** | A host runtime that evaluates or mutates entities and components according to engine rules. |
| **Event** | A typed record that something happened, optionally carrying payload data. |
| **Goal** | A desired world or actor state used by a planner. |
| **Plan** | A sequence or set of actions proposed by a planner to satisfy a goal. |
| **Extension module** | A named optional feature set (tags, attributes, processing steps) with its own conformance token. |

---

## 3. Conformance

### 3.1 Conformance classes

An implementation **MAY** claim one or more of the following conformance classes:

| Class | Requirements |
|-------|----------------|
| **AIML3-Interpreter-Core** | Implements Sections **4–8**, **10–11** for the **Core** feature tier (see §3.2). |
| **AIML3-Interpreter-Extended** | Implements **AIML3-Interpreter-Core** plus modules declared supported from Section **9**. |
| **AIML3-Validator** | Verifies document syntax and static constraints per Section **17**; need not execute templates. |
| **AIML3-AuthoringTool** | Produces syntactically valid AIML 3.0 documents; SHOULD emit declared `xmlns` and `version`. |

Implementations MAY additionally advertise one or more serialization support tokens: **`AIML3-Serialization-XML`**, **`AIML3-Serialization-JSON`**, and **`AIML3-Serialization-USD`**. If an implementation supports more than one serialization, it MUST map them to equivalent infosets per §4.

### 3.2 Feature tiers

**Core tier** includes: the canonical AIML infoset; support for at least one serialization from §4; categories; pattern literals and normative wildcards (§6); templates with literal text, `<random>`, `<condition>`, `<srai>`, `<think>`, `<star/>`, `<thatstar/>`, `<topicstar/>`, `<get>`, `<set>`, `<bot>`, and topic/that hooks per §7–8.

**Extended tier** adds optional modules from §9 (e.g., rich media, out-of-band directives, interaction directives, world-state effects, multi-party messaging, presentation envelopes, LLM mediation, narrative graphs, proofing resources, ECS integration, event-driven integration, planner integration, external services, learning, and host utility tags) **only** if explicitly advertised by the implementation.

### 3.3 Undefined behavior

Where this specification labels behavior **implementation-defined**, conforming interpreters MUST document their choice. Where behavior is **undefined**, interpreters MUST NOT crash a host process due to **syntactically valid** AIML alone; they SHOULD surface a diagnostic and degrade gracefully.

### 3.4 Conformance profiles

Implementations MAY additionally claim one or more **profiles** that compose conformance classes and modules into reusable capability bundles:

| Profile | Required capabilities |
|---------|------------------------|
| **Core Dialog** | `AIML3-Interpreter-Core` plus at least one supported serialization. |
| **Dialog + Directives** | Core Dialog + §9.3 Interaction Directives Module. |
| **Dialog + Messaging** | Dialog + Directives + §9.5 Multi-party Conversation Module. |
| **Dialog + LLM** | Core Dialog + §9.7 LLM Mediation Module; §9.3 and §9.6 are RECOMMENDED. |
| **Full World/Scene** | Dialog + Directives + §9.4 World State and Effects Module + §9.5 Multi-party Conversation Module + §9.6 Presentation Envelope Module + §9.10 Narrative Graph Module + support for at least one rich serialization, RECOMMENDED: XML, JSON, and USD. |
| **Simulation / Planner** | Core Dialog + any of §9.13, §9.14, or §9.15; §9.4 is RECOMMENDED. |
| **Proofing / Analysis** | `AIML3-Validator` plus §9.12 Proofing and Analysis Module; §9.10 Narrative Graph Module is RECOMMENDED. |

---

## 4. Syntax and Document Infoset

### 4.1 Canonical AIML infoset

This specification defines AIML semantics over a **canonical infoset**, not over XML alone. A conforming loader MUST map each supported serialization to an equivalent internal representation before matching and template evaluation.

### 4.2 Serialization families

**XML**, **JSON**, and **USD** are equal citizens of AIML 3.0. A host profile MUST declare which serializations it accepts. If the same authored rules are distributed in more than one serialization, they MUST denote the same categories, patterns, templates, metadata, and directives.

### 4.3 XML serialization

AIML serialized as XML MUST be a **well-formed XML 1.0** document. Unless a host profile states otherwise, the root element MUST be `aiml`.

Implementations MUST support **no-namespace** AIML elements for Core interoperability. Implementations MAY additionally support a **namespace URI** for AIML 3.0 elements; if used, the document MUST declare it on the root and descendants as required by Namespaces in XML 1.0.

The root `aiml` element MAY include:

| Attribute | Semantics |
|-----------|-----------|
| `version` | Document format version string (RECOMMENDED). |
| `xml:lang` | Authoring language of literal content (RECOMMENDED). |

### 4.4 JSON serialization

AIML serialized as JSON MUST be valid JSON text. A conforming JSON serialization MUST represent the same category/template infoset as XML, including `pattern`, `template`, optional `that`, optional topic scoping, and declared metadata.

The following JSON roots are RECOMMENDED:

- An object containing `categories`.  
- An object containing `aiml`, where `aiml.categories` holds the category list.  
- A top-level array of category objects.

JSON template values MAY be encoded as strings, arrays, or structured objects, provided that their evaluation is equivalent to the canonical template tree defined in §8–9.

### 4.5 USD serialization

AIML serialized as **Universal Scene Description (USD)** MUST map the AIML infoset into stable scene-graph or layer objects. A conforming USD serialization MUST preserve category identity, pattern text, optional `that`, topic membership, template structure, and directive metadata without requiring XML as an intermediate source of truth.

Implementations MAY accept any host-permitted USD package form (for example `usda`, `usdc`, or `usdz`). A host profile supporting USD MUST document:

- How categories are identified as prims, paths, or equivalent layer records.  
- Where `pattern`, `that`, `topic`, `template`, and metadata are stored.  
- How composition or layering affects category order and precedence.

### 4.6 Structural invariants

1. Each `category` in the infoset MUST contain exactly one `pattern`-equivalent matcher and exactly one `template`.  
2. `that` MAY appear zero or one times inside a `category`.  
3. Topic scoping MAY be represented by wrapper structure, attributes, relationships, or host metadata, but the resulting infoset MUST be equivalent to categories grouped by topic.  
4. A loader supporting the historical `intent` alias MUST map it losslessly to the same canonical matcher slot as `pattern`. A category MUST NOT contain both `pattern` and `intent` unless the host defines and documents an exact equivalence rule.  
5. A loader MUST reject any XML, JSON, or USD source that cannot be mapped to these invariants without guesswork.

### 4.7 Story or world manifest

A package MAY include a top-level **manifest** describing authored content independent of any one serialization. If a manifest is present, the following fields are RECOMMENDED:

| Field | Meaning |
|-------|---------|
| `id` | Stable unique package, world, or story identifier. |
| `title` | Human-readable package title. |
| `startNode` | Stable identifier of the initial narrative node if §9.10 is used. |
| `profiles` | Declared conformance profiles or host profiles expected by the package. |
| `creator` | Authoring tool or pipeline identifier. |
| `creatorVersion` | Version of the authoring tool or pipeline. |
| `version` | Package or format version string. |
| `tags` | Optional package-level tags. |
| `runtimeProfile` | Optional declared runtime or renderer profile. |
| `proofingProfile` | Optional declared proofing or analysis profile. |

If `startNode` is declared, it MUST reference a valid narrative node when the Narrative Graph Module is present.

### 4.8 Editor metadata

Packages MAY include non-runtime **editor metadata** such as node position, node size, grouping, annotations, color hints, or fold state.

- Editor metadata MUST NOT change runtime behavior unless a host profile explicitly says otherwise.  
- Serializations SHOULD preserve unknown editor metadata during round-trip if possible.  
- Validators MAY ignore malformed editor metadata in runtime-only profiles, but SHOULD surface diagnostics in proofing profiles.

### 4.9 Narrative graph serialization

If §9.10 is supported, a host profile MUST define how narrative nodes and links are encoded in each supported serialization.

The following canonical field names are RECOMMENDED for interchange:

| Field | Meaning |
|-------|---------|
| `id` | Stable node identifier. |
| `name` | Human-readable node title. |
| `tags` | Node tag list. |
| `metadata` | Arbitrary node metadata. |
| `body` | Node body content. |
| `links` | Explicit outgoing links. |

For explicit links, the following field names are RECOMMENDED:

| Field | Meaning |
|-------|---------|
| `label` | Choice or link label. |
| `target` | Destination node id. |
| `conditions` | Visibility or eligibility conditions. |
| `effects` | Side effects applied if selected. |
| `directives` | Optional structured directives associated with the link. |
| `metadata` | Arbitrary link metadata. |

JSON serializations SHOULD use these field names directly. XML and USD serializations MAY use host-specific element or attribute forms, but MUST map losslessly to the same canonical fields.

### 4.10 Special-resource serialization

If §9.11 is supported, a host profile MUST define how reserved resources are represented in each supported serialization.

The following canonical resource names are RECOMMENDED for interchange:

- `global_script`
- `global_stylesheet`
- `init_data`
- `proofing_data`

A resource record SHOULD include at minimum:

| Field | Meaning |
|-------|---------|
| `name` | Reserved or host-defined resource name. |
| `contentType` | Media or payload type. |
| `content` | Resource payload. |
| `metadata` | Optional resource metadata. |

### 4.11 Manifest serialization

If a manifest is present, JSON serializations SHOULD represent it as a top-level `manifest` object or an equivalent top-level field set. XML and USD serializations MAY encode manifest data as root attributes, child elements, layer metadata, or stage metadata, provided the mapping to the canonical manifest fields in §4.7 is documented and lossless.

---

## 5. Processing Model

### 5.1 Pipeline

On each user turn, an **AIML3-Interpreter** MUST execute conceptually:

1. **Deserialize or compile** supported source material (XML, JSON, USD, or equivalent cached form) into the canonical AIML infoset.  
2. **Acquire raw input** string from the host.  
3. **Normalize** per active **normalization profile** → **utterance token sequence** `U`.  
4. **Select active topic** value `T` and **that** value `H` from dialog state.  
5. **Match** an eligible category per §6–7 producing **bindings** (stars, captures). Eligibility MAY include topic, `that`, preconditions, and cooldown checks.  
6. **Evaluate template** per §8–9 producing **primary string** `S` and **side effects** (predicate updates, topic updates, directives, world-state effects, optional module payloads).  
7. **Validate** all requested protected actions against host authority, capability policy, and schema constraints.  
8. **Commit state** atomically at end of turn unless host requires streaming partial commits (implementation-defined; MUST be declared).

### 5.2 Normalization profiles

A normalization profile MUST define at minimum:

- Unicode normalization form (e.g., NFC) applied or explicitly **none**.  
- Case folding policy (binary, locale-independent, or locale-aware).  
- Tokenization rules: which characters are separators; whether punctuation attaches to tokens.  
- Whitespace collapsing across the full raw string vs within tokens.

**Core default profile:** NFC + Unicode case fold (simple) + split on ASCII whitespace + strip leading/trailing whitespace.

### 5.3 Match precedence

When multiple categories match, interpreters MUST apply deterministic precedence:

1. **Specificity:** patterns with fewer wildcards and more literals win over more general patterns.  
2. **Topic specificity:** matching non-default topic wins over default-topic categories if both match.  
3. **That conditioning:** categories with a matching `that` win over categories without `that` when both match `U`.  
4. **Document order tie-break:** earlier-defined category wins if still tied (RECOMMENDED for author predictability).

> **Note (informative):** Implementations historically differed here; AIML 3.0 Core **requires** documenting the metric for “fewer wildcards” (e.g., weight literals +1, `*` weight 0, `_` weight 1).

### 5.4 Template evaluation ordering

Template children MUST be evaluated in **document order** unless an element’s definition explicitly defers evaluation (e.g., guarded branches).

### 5.5 Recursion budget

Interpreters MUST enforce a **maximum `<srai>` depth** `D_max` ≥ 1. If exceeded, evaluation MUST abort the current template path with a **recoverable error** (§15) and SHOULD return a host-defined fallback string.

### 5.6 Cycle detection

Interpreters SHOULD detect **direct cycles** in `<srai>` (repeated identical target input with identical state) and treat them as recoverable errors.

### 5.7 Capability and authority model

AIML content, extension modules, and LLM mediation outputs MAY **request** actions, but the host alone MAY **authorize and commit** them. Conforming hosts MUST document:

- Which capabilities are exposed to AIML evaluation.  
- Which capabilities are read-only versus state-mutating.  
- Which requested actions are advisory and which are commit-bearing.  
- The authority boundary for inventory, permissions, schedules, membership, location, and scene state.

An implementation MUST NOT treat authored content or model output as sufficient authority to mutate protected state without host validation.

### 5.8 Transaction semantics

Each evaluated turn SHOULD be treated as a logical **transaction** consisting of visible reply text plus zero or more directives, effects, and validated payloads.

- Transactions MUST be validated before commit.  
- Transactions MUST apply in a deterministic order documented by the host profile.  
- If a transaction contains both visible text and one or more invalid side effects, the host SHOULD preserve the visible text whenever it can do so safely.  
- Hosts SHOULD provide idempotency guarantees for retried or replayed turns where duplicate commits would be harmful.  
- If a host supports partial commit or streaming, it MUST document which parts of the transaction can still be rejected after text emission.

### 5.9 Runtime architecture interoperability

AIML 3.0 is compatible with multiple host runtime styles, including entity/component/system (ECS) simulation, event-driven architectures, and goal-oriented planners.

- AIML MAY consume **derived runtime context** from entities, components, systems, event logs, or planners, but MUST do so through host-defined, documented views.  
- AIML MUST NOT assume direct writable access to ECS storage, event buses, or planner internals unless a host module explicitly grants such access.  
- Hosts SHOULD expose planner, ECS, and event context as stable, stringifiable facts suitable for matching and template evaluation, rather than leaking engine-internal object graphs.

### 5.10 Session history and learned state

Conforming interpreters SHOULD maintain bounded per-session conversational history sufficient to evaluate classic AIML history references and context-sensitive matching.

- `input` history refers to prior normalized input sentences.  
- `request` history refers to prior user turns, each of which MAY contain one or more sentences.  
- `response` history refers to prior agent turns, each of which MAY contain one or more output sentences.  
- `that` refers to the last sentence of the most recent response unless an indexed form is used.  
- If learning is enabled, per-session learned categories and shared learned categories MUST be stored in distinct scopes.

Hosts MUST document:

- history window size limits  
- sentence-splitting rules used for indexed history access  
- whether learned categories participate in ordinary match precedence or a separate precedence tier

---

## 6. Pattern Language

### 6.1 Literals

A literal substring in a pattern MUST match the identical token sequence segment under the normalization profile.

### 6.2 Wildcards (Core)

Implementations MUST support at least:

| Symbol | Arity | Core semantics |
|--------|-------|----------------|
| `*` | 0 or more tokens | Matches zero or more tokens; binds a **star** region. |
| `_` | exactly 1 token | Matches any single token; binds a **star** region. |

Implementations MAY support additional wildcards (e.g., `#`, `^`) as **Extended** features; if supported, their semantics MUST appear in implementation documentation and SHOULD align with historical AIML 2.x descriptions where practical.

### 6.3 Captures

For each wildcard occurrence left-to-right, interpreters MUST assign **capture indices** for `<star index="n"/>` retrieval. If an index references an unbound capture, the empty string MUST be substituted.

### 6.4 Sets and maps in patterns

If the **Sets/Maps** module is supported, patterns MAY reference finite vocabularies (e.g., `SET:cityName` style tokens per host grammar). Such tokens MUST expand to a matcher equivalent to alternation over literals **without** introducing undefined backtracking across implementations beyond longest-match rules declared by the module.

### 6.5 Historical wildcard compatibility

Implementations claiming AIML 2.x compatibility SHOULD document whether they support the historical wildcard family and precedence order:

- `$word` priority literals  
- `#` zero-or-more wildcard  
- `_` one-or-more wildcard  
- exact word match  
- set match  
- `^` zero-or-more wildcard  
- `*` one-or-more wildcard

If an implementation supports `#`, `^`, or `$word`, it MUST document:

- whether zero-length captures are permitted  
- what value an empty capture expands to  
- the exact precedence order used during matching

---

## 7. Categories, Topics, and Context

### 7.1 Categories

A `category` defines a rule. The `pattern` is matched against `U` when the category is **active**.

Implementations MAY support `intent` as a historical alias or authoring synonym for `pattern`. If supported, `intent` MUST map to the same canonical matcher slot and MUST NOT introduce a second independent match channel.

### 7.2 Topics

Each `topic` has a `name` attribute. A category inside `topic name="X"` is active iff the session topic equals `X`, or if `name="*" ` denotes a **catch-all** topic bucket per host definition (RECOMMENDED: `*` means “any topic”).

### 7.3 That

Optional `that` in a category MUST be matched against the stored **that string** `H` using the **same pattern grammar** as user patterns unless a profile defines a reduced grammar.

### 7.4 Updating topic and that

After successful template evaluation producing primary string `S`:

- **That** MUST be updated to the implementation’s **canonical that form** of `S` (often `S` itself after whitespace normalization).  
- **Topic** updates occur only via explicit template elements or host actions (implementation-defined element names SHOULD be standardized in §9 if present).

### 7.5 Category eligibility metadata

If the **World State and Effects** module (§9.4) is supported, categories MAY declare eligibility metadata in addition to `pattern`, `that`, and `topic`.

The following metadata keys are standardized:

| Key | Meaning |
|-----|---------|
| `requires` | All listed host-visible attributes MUST match for the category to be eligible. |
| `unless` | If all listed host-visible attributes match, the category MUST be ineligible. |
| `cooldown` | Names a host-defined cooldown bucket that MAY temporarily suppress the category after use. |

For `requires` and `unless`, an attribute value MAY be a single string or a finite list of strings. A list value means “match any of these values.”

Hosts MAY expose serialization-specific aliases for these keys, but SHOULD preserve the canonical names in interchange. For example, a JSON authoring system MAY accept `cooldownKey` as an alias for `cooldown`.

Cooldown semantics are host-defined, but a host supporting cooldowns MUST document:

- Cooldown scope, for example per session, per actor, per room, or global.  
- Cooldown lifetime or expiry policy.  
- Whether cooldown is checked before or after tie-breaking among otherwise eligible categories.

---

## 8. Template Language — Core

The semantics in this section are defined over the **canonical template tree**. XML element syntax is used as the canonical notation, but equivalent JSON nodes or USD structures MUST evaluate identically when they encode the same template tree.

### 8.1 Literal text

PCDATA in templates MUST be copied to the primary output, except where `<think>` suppresses visibility (§8.8).

### 8.2 Star elements

| Element | Semantics |
|---------|-----------|
| `<star/>` | Expands to capture 1 of current match unless `index` given. |
| `<star index="n"/>` | Expands to capture *n* (1-based RECOMMENDED). |
| `<thatstar/>`, `<topicstar/>` | Analogous for `that` pattern and topic pattern captures if applicable. |

### 8.3 Random choice

```xml
<random>
  <li>…template…</li>
  <li>…template…</li>
</random>
```

Interpreters MUST choose **exactly one** `li` child with uniform probability among `li` elements unless `weight` attributes are present (Extended).

### 8.4 Conditionals

```xml
<condition name="predicateName">
  <li value="v">…</li>
  <li>…default…</li>
</condition>
```

Interpreters MUST select the first `li` whose `value` equals the predicate’s string value; if none match, the `li` without `value` MUST be chosen if present; otherwise expand to empty string.

Implementations supporting historical AIML condition forms SHOULD additionally document:

- whether `value="*"` tests whether a predicate or local variable is bound  
- whether `<condition>` supports mixed `name`/`var` addressing forms  
- whether `<loop/>` is supported for re-evaluating a condition until a non-looping branch is selected

### 8.5 Predicate and bot access

- `<get name="k"/>` expands to the session predicate `k` or empty if unset.  
- `<set name="k">…</set>` evaluates children, stores result as `k`, and expands to empty unless otherwise specified by host profile.  
- `<bot name="k"/>` expands to bot property `k`.

### 8.6 Recursion

`<srai>…text or template…</srai>` MUST evaluate children to a string `Q`, then **re-invoke** matching on utterance `Q` as if user had said `Q`, carrying forward side effects per §5.1.

### 8.7 Maps

If `<map name="m">key</map>` is implemented in Core for a host, it MUST expand `key` through map `m` or empty if undefined.

### 8.8 Think (silent evaluation)

`<think>…</think>` MUST evaluate children for side effects but MUST NOT copy literal text inside to primary output. Nested `<think>` MUST be allowed.

### 8.9 Case and person transforms

Elements such as `<uppercase>`, `<lowercase>`, `<formal>`, `<sentence>`, `<person>` MAY be supported; if supported, they MUST define Unicode-aware behavior or explicitly document ASCII-only behavior.

### 8.10 History access elements

Implementations MAY support the classic history-reference elements:

| Element | Semantics |
|---------|-----------|
| `<input index="n"/>` | Returns the *n*th previous normalized input sentence. |
| `<request index="n"/>` | Returns the *n*th previous user request or turn. |
| `<response index="n"/>` | Returns the *n*th previous agent response. |
| `<that index="m,n"/>` | Returns the *n*th sentence of the *m*th previous response, counted from the end unless the host documents another convention. |

If these elements are supported:

- Indexing MUST be deterministic and documented.  
- Out-of-range references MUST expand to the empty string or a documented neutral default.  
- Sentence splitting and normalization rules used for history indexing MUST be documented by the host profile.

### 8.11 Local variables

Implementations MAY support category-local variables using `var` forms of classic tags, such as `<set var="x">`, `<get var="x"/>`, and `<condition var="x">`.

If local variables are supported:

- Their scope MUST be limited to the current template evaluation unless the host documents a broader lexical model.  
- Local variables MUST NOT implicitly persist across independent turns.  
- A recursive `<srai>` call MUST NOT inherit caller-local variables unless the host explicitly documents that behavior.

### 8.12 Utility and introspection elements

Implementations MAY support utility elements historically common in AIML deployments, including:

| Element | Semantics |
|---------|-----------|
| `<id/>` | Returns the current session or client identifier. |
| `<program/>` | Returns the interpreter or runtime identifier string. |
| `<vocabulary/>` | Returns a host-defined vocabulary summary, often the count of recognizable words or set members. |
| `<size/>` | Returns a host-defined category-count or graph-size summary. |

These elements MUST be documented as implementation-defined surfaces. Hosts SHOULD avoid exposing sensitive internal identifiers through them.

### 8.13 Text normalization and morphology helpers

Implementations MAY support additional classic text-transform helpers, including `<normalize>`, `<denormalize>`, `<explode>`, `<person2>`, and `<gender>`.

If supported:

- `<normalize>` MUST apply the active or a documented normalization profile.  
- `<denormalize>` MUST be documented as approximate unless the host can guarantee a true inverse.  
- `<explode>` MUST document whether it splits by Unicode code point, grapheme cluster, or host-defined character unit.  
- `<person2>` and `<gender>` MUST document the substitution tables or locale-dependent strategy they apply.

---

## 9. Template Language — Modules

Modules MUST be advertised by implementations. Unless enabled, documents using module tags MUST be rejected by validators or ignored by interpreters according to host policy (MUST choose one policy and document it).

### 9.1 Rich media module

The Rich Media Module standardizes channel-friendly presentational structures that go beyond plain text while remaining serialization-neutral.

Hosts MAY support rich media elements equivalent to:

- buttons with text/postback or text/URL forms  
- replies or quick replies  
- hyperlinks  
- images  
- video  
- cards  
- carousels  
- delays  
- message splits  
- bulleted or ordered lists

If this module is supported:

- The host MUST document the canonical infoset for text/postback pairs, media payloads, and card/carousel composition.  
- Unsupported rich media payloads SHOULD degrade to readable plain text where practical.  
- Interpreters MUST NOT fetch network resources without explicit host permission.  
- Rich media emission MUST remain distinct from authoritative directives or world-state commits.

### 9.2 Out-of-band (OOB) module

This module standardizes secondary machine-readable payloads emitted alongside visible text, historically associated with `<oob>` processing.

If this module is supported:

- Interpreters MUST separate **primary string** from **OOB records** in the host API.  
- Hosts SHOULD treat OOB processing as a second phase after template evaluation, so visible text generation and device or client action handling remain separable.  
- OOB payloads MUST be validated by the host before execution.  
- OOB payloads MUST NOT silently bypass the capability and authority model defined in §5.7.

### 9.3 Interaction directives module

Implementations MAY support host-side **interaction directives** that are evaluated as structured side effects rather than visible prose. A directive MAY be authored as an XML element, a JSON object field, a USD-authored payload, or a host-defined inline marker syntax, provided the resulting infoset is equivalent.

The following directive families are standardized by this profile:

| Directive | Semantics |
|-----------|-----------|
| `give` | Transfer a validated item, entitlement, or inventory object to the user/session. |
| `invite_accept` | Accept a previously offered invitation into a private or controlled interaction context. |
| `date_plan` | Commit a scheduled future rendezvous using a validated location, offset, and slot payload. |
| `meetup` | Commit a lightweight immediate or same-window meetup state using validated location, strength, and phase fields. |
| `conversation_end` | End the current conversation session until the host re-opens or resets it. |

Where a host profile enables inline text markers as an interchange form, the RECOMMENDED canonical spellings are `[GIVE:item_id]`, `[INVITE_ACCEPT]`, `[DATE_PLAN:location_id:day_offset:slot]`, `[MEETUP:location_id:strength:phase]`, and `[CONVERSATION_END]`.

If this module is supported:

- Hosts MUST validate all item identifiers, location identifiers, scheduling slots, and similar directive payloads against host authority before execution.  
- Directives MUST be stripped from end-user-visible output when authored in an inline marker form.  
- Failed validation MUST NOT cause the visible reply to be lost unless the host profile explicitly makes the directive mandatory.  
- If multiple directives of the same family are emitted in one turn, the host MUST apply a deterministic policy and document it.

### 9.4 World State and Effects module

Implementations MAY support structured **world-state guards and effects** on categories. This module gives AIML a MUD-like capability surface while preserving host authority.

#### 9.4.1 Preconditions

If this module is supported, categories MAY declare:

- `requires`  
- `unless`  
- `cooldown`

These fields define rule eligibility as specified in §7.5.

#### 9.4.2 Canonical effects

The following **canonical effect names** are standardized for interchange:

| Effect | Semantics |
|--------|-----------|
| `set_flag` | Set a named boolean-like world or session flag. |
| `clear_flag` | Clear a named boolean-like world or session flag. |
| `set_slot` | Set a named slot in a host-defined scoped record to a string value. |
| `clear_slot` | Clear a named slot in a host-defined scoped record. |

Hosts MAY expose implementation-specific aliases for these effects, but SHOULD preserve the canonical names in interchange formats. For example, a host MAY internally map `set_slot` / `clear_slot` to more specialized runtime effects such as `set_service_flow_slot` / `clear_service_flow_slot`.

#### 9.4.3 Reserved future effects

The following effect names are RESERVED for future standardization:

- `take`  
- `pay`  
- `move`  
- `unlock`  
- `join_group`  
- `leave_group`

An implementation that does not explicitly advertise support for one of these effect families MUST treat it as unsupported.

#### 9.4.4 Execution rules

If this module is supported:

- Effects MUST be validated against host authority before commit.  
- Unsupported or invalid effects MUST NOT mutate state implicitly.  
- Hosts SHOULD apply all accepted effects atomically within the turn transaction.  
- Hosts MUST document effect scope, for example session, actor, room, scene, or global world state.  
- Hosts MUST document whether effects execute before, after, or alongside interaction directives.

### 9.5 Multi-party conversation module

Implementations MAY support **multi-party conversation targets** in addition to direct user-to-agent turns. Supported targets MAY include direct threads, rooms, groups, channels, or other membership-gated scopes.

This module standardizes the following capability families:

| Capability | Semantics |
|------------|-----------|
| `conversation.readRecent` | Read a bounded, policy-filtered recent history window for the current conversation target. |
| `conversation.getParticipants` | Return safe participant identifiers for the current target. |
| `conversation.getMembershipState` | Report whether the querying actor and player/session are valid members of the target. |
| `presence.getVisiblePresence` | Return policy-filtered presence state for visible participants in the current target. |
| `conversation.sendMessage` | Emit a validated AIML-authored line into the current multi-party target. |

If this module is supported:

- Hosts MUST re-check membership and policy for every capability invocation.  
- Implementations MUST support bounded history windows rather than unbounded transcript export.  
- Capability failure MUST NOT leak the existence of hidden conversations through unauthorized enumeration.  
- Hosts MAY expose target context, participant summaries, or recent-history snippets to AIML matching or template evaluation.  
- Multi-party targets SHOULD support audience-aware reply shaping distinct from direct-message style.

### 9.6 Presentation envelope module

Implementations MAY support a validated **presentation envelope** emitted alongside the primary reply. Such envelopes MAY include allowlisted fields for emotion, expression, delivery, pose, gaze, timing, voice style, or equivalent presentation hints. Hosts MUST treat these values as advisory, MUST validate them against documented allowlists, and MAY filter them by output channel.

### 9.7 LLM mediation module

Implementations MAY support **LLM mediation** as an optional generation stage. This module allows a large language model to propose text or structured payloads under host validation; it does not replace host authority or the deterministic semantics of AIML evaluation.

LLM mediation MAY be used in at least the following modes:

- **Fallback mode:** no eligible AIML category matched.  
- **Assistive mode:** AIML explicitly delegates part of a response shape or payload.  
- **Post-processing mode:** a host refines tone, formatting, or presentation without changing authoritative world facts.

If a host supports structured LLM replies, the following fields are RECOMMENDED:

| Field | Meaning |
|-------|---------|
| `reply` or `text` | Visible primary reply string. |
| `giveItemId` | Optional item identifier for a validated `give` directive. |
| `inviteAccept` | Optional boolean for `invite_accept`. |
| `conversationEnd` | Optional boolean for `conversation_end`. |
| `datePlan` | Optional object for `date_plan`, containing `locationId`, `dayOffset`, and `slot`. |
| `meetup` | Optional object for `meetup`, containing validated location, strength, and phase fields if the host supports it. |
| `performance` | Optional presentation envelope payload. |

If this module is supported:

- Invalid or unparseable structured output MUST fall back to plain text handling or host-defined recovery.  
- Model output MUST NOT be treated as authoritative for inventory, possession, location, membership, permission, schedule state, or scene state without host validation.  
- A model MUST NOT imply that an item has been transferred, a plan is scheduled, a room is entered, a group is joined, or a capability is granted unless the corresponding validated directive or effect is accepted by the host.  
- Hosts SHOULD pass only bounded, policy-filtered context into the model.  
- Hosts MUST document when model output can supersede authored AIML, and when it cannot.

### 9.8 External services module

This module standardizes host-mediated access to remote services and selected runtime utilities, historically related to `<sraix>`, `<date>`, `<interval>`, and `<system>`.

If this module is supported, hosts MAY expose:

- a remote call element equivalent to `<sraix>`  
- date and time formatting utilities  
- interval or date-difference utilities  
- tightly sandboxed host-command execution

Remote-call implementations MUST honor:

- timeouts  
- size limits  
- host capability tokens  
- allowlisted destinations or service registries  
- bounded fallback behavior

For `<sraix>`-like behavior, implementations SHOULD document supported request fields such as service name, host, bot identifier, API key handle, hint, response limit, and default fallback text. Hosts SHOULD provide a deterministic failure path, historically similar to `SRAIXFAILED`, that does not recurse indefinitely.

`<system>`-like behavior MUST be disabled by default and MUST run only inside an explicitly documented sandbox if enabled at all.

### 9.9 Learning module

This module standardizes dynamic category acquisition historically associated with `<learn>`, `<learnf>`, and `<eval>`.

Dynamic learning MUST default to **disabled**.

If this module is supported:

- `<learn>` MUST add categories only to a per-session or equivalently scoped learned store.  
- `<learnf>` MUST add categories only to a shared or bot-level learned store.  
- `<eval>` used inside learning payloads MUST evaluate under the same safety limits as ordinary template execution.  
- Learned categories MUST be validated before activation.  
- Hosts MUST support signing, sandboxed storage, or an equivalent trust boundary per host policy (§14).  
- Hosts MUST document duplicate-category handling between authored and learned categories.

### 9.10 Narrative graph module

Implementations MAY support an explicit **narrative graph** in addition to category-based dialog matching. This module standardizes named authored nodes for hub-and-spoke scenes, quest steps, room text, branching beats, and explicit choices.

If this module is supported, each **narrative node** SHOULD provide:

| Field | Meaning |
|-------|---------|
| `id` | Stable machine identifier. |
| `name` | Human-readable node title. |
| `tags` | Optional tag list. |
| `metadata` | Arbitrary host-defined metadata. |
| `body` | Authored content body. |
| `links` | Optional explicit outgoing links or targets. |

Narrative nodes MAY coexist with AIML categories. A host MAY use categories to enter, leave, or react within nodes, or MAY use nodes as higher-level containers that select which categories are active.

#### 9.10.1 Choice links

If explicit links are supported, each link SHOULD provide:

| Field | Meaning |
|-------|---------|
| `label` | Human-readable choice label. |
| `target` | Stable identifier of the destination node, if navigational. |
| `conditions` | Optional host-defined conditions for visibility or eligibility. |
| `effects` | Optional side effects applied if the link is taken. |

Choice links MAY represent navigation, actions, or both. A host profile MUST document whether taking a link changes node focus, world state, conversation target, or all three.

#### 9.10.2 Start node

If a manifest declares `startNode`, the host SHOULD begin graph-aware execution from that node unless a more specific resume rule applies.

### 9.11 Special resources module

Implementations MAY support reserved top-level **special resources** that are not themselves categories or narrative nodes.

The following reserved resource names are RECOMMENDED:

| Resource | Meaning |
|----------|---------|
| `global_script` | Shared script or runtime helper source. |
| `global_stylesheet` | Shared styling or theme resource. |
| `init_data` | Initialization data or default authored state. |
| `proofing_data` | Proofing, debug, lint, or review-only resource payloads. |

Hosts MAY ignore unsupported resources, but SHOULD preserve them during authoring round-trip when possible.

### 9.12 Proofing and analysis module

Implementations MAY support a non-playable **proofing and analysis** mode for inspecting authored content without running the full live simulation.

If this module is supported, hosts SHOULD support one or more of the following:

- linting  
- graph validation  
- dead-end detection  
- unreachable-node detection  
- overlap or shadowing checks  
- export to readable review formats

Proofing outputs MAY be HTML, JSON, plain text, or any host-defined review format. A proofing profile MUST NOT require the full live renderer or simulation loop.

#### 9.12.1 Minimum proofing output contract

If a host claims proofing support, it SHOULD be able to produce at least one machine-readable or review-readable output containing:

| Field | Meaning |
|-------|---------|
| `packageId` | Package or manifest id if known. |
| `profile` | Proofing profile name. |
| `diagnostics` | List of lint or validation findings. |
| `unreachableNodes` | Optional list of unreachable narrative node ids. |
| `deadEnds` | Optional list of nodes or categories with no valid continuation. |
| `shadowedRules` | Optional list of categories or links hidden by stronger rules. |
| `summary` | Human-readable proofing summary. |

Each diagnostic record SHOULD include at minimum a severity, message, and stable reference to the relevant category, node, link, or resource when such a reference is available.

### 9.13 Entity/component/system integration module

Implementations MAY integrate AIML with an **entity/component/system** runtime.

If this module is supported:

- AIML MAY read host-provided entity identifiers, component-derived facts, scene paths, or system summaries.  
- AIML MUST NOT mutate components directly except through validated directives, effects, or host capabilities.  
- Hosts SHOULD expose ECS data to AIML as stable strings, records, or allowlisted attributes rather than raw engine pointers or opaque memory references.  
- A host MAY map narrative nodes, conversation targets, or scene resources onto entities or scene-graph paths, provided the mapping is documented.

#### 9.13.1 Entity identity and scene addressing

Hosts supporting this module SHOULD provide stable identifiers for runtime subjects. RECOMMENDED identity forms include:

- entity ids  
- scene or prim paths  
- layer ids  
- relationship targets

If USD serialization is also supported, hosts SHOULD document how narrative nodes, conversation targets, or authored resources map onto prim paths, layer payloads, or related scene-addressing structures.

#### 9.13.2 Component projection

Hosts SHOULD project component state into AIML-readable facts rather than raw storage. Examples include:

- coarse location or placement labels  
- inventory or entitlement summaries  
- mood or need bands  
- scene-state hints  
- relationship or route summaries

Projected component facts SHOULD be stable across turns and safe for authoring tools to inspect.

### 9.14 Event-driven integration module

Implementations MAY integrate AIML with a typed **event-driven** architecture.

If this module is supported:

- Hosts MAY expose recent or scoped events as readable context for matching or template evaluation.  
- AIML transactions MAY emit typed events as advisory or commit-bearing records if the host validates them.  
- Event records SHOULD carry at least a type name and MAY additionally carry payload, source, timestamp, order key, and correlation metadata.  
- Event emission MUST obey the transaction semantics in §5.8.  
- Hosts SHOULD document ordering, deduplication, replay, and idempotency behavior for emitted events.

#### 9.14.1 Event source ordering

Hosts supporting this module SHOULD document the ordered evaluation of event sources when multiple candidate streams exist, such as:

- authored local content  
- remote live content  
- proposal or planning pipelines  
- calendar hooks  
- contact or messaging hooks  
- unknown or extension sources

If multiple candidate events compete for the same turn, the host MUST document how source precedence and tie-breaking are resolved.

#### 9.14.2 Event result shape

When hosts expose picked or emitted events to AIML or tooling, a shared result shape is RECOMMENDED:

| Field | Meaning |
|-------|---------|
| `type` | Event type name. |
| `source` | Event provenance label. |
| `payload` | Typed or schematized data payload. |
| `selected` | Whether the event was chosen for this turn. |
| `skippedReasons` | Optional diagnostics for why competing events were not chosen. |
| `correlationId` | Optional id for tracing related work across systems. |

### 9.15 Planning and goal-oriented action module

Implementations MAY integrate AIML with **goal-oriented planning** systems, including GOAP-style planners.

If this module is supported:

- Hosts MAY expose planner context such as current goal, placement reason, planner-selected action labels, or world-state summaries to AIML.  
- AIML MAY emit advisory planning hints, but MUST NOT override planner authority unless the host explicitly grants that power.  
- Planner facts exposed to AIML SHOULD be stable, human-readable summaries rather than engine-internal planner graphs.  
- Model output or authored text MUST NOT claim that a planner has chosen, satisfied, or committed a goal unless that fact is supplied by host grounding or validated commit state.

#### 9.15.1 Planner trace projection

Hosts supporting this module SHOULD expose planner or routing traces in a documented, stringifiable form. RECOMMENDED fields include:

| Field | Meaning |
|-------|---------|
| `placementSource` | Why the actor is where they are, for example planner, pin, invite, guest, or authored override. |
| `placementReason` | Short human-readable grounding line. |
| `activeGoalId` | Current or most recent goal identifier. |
| `goalName` | Optional human-readable goal name. |
| `actionId` | Optional id of the first or selected action in the active plan. |

These trace fields MAY be exposed to AIML matching, templates, LLM grounding, proofing tools, or debug exports.

#### 9.15.2 Advisory planning intents

AIML MAY emit advisory hints such as preferred venue class, meeting firmness, task lane, or social intent, but the host MUST document whether these hints:

- only influence prompt/context assembly  
- bias planner heuristics  
- enqueue a proposal for later validation  
- or have no runtime effect outside proofing

---

## 10. Predicates, Maps, and Bot Properties

### 10.1 Scopes

| Store | Default scope |
|-------|----------------|
| Predicates (`get`/`set`) | Per user session |
| Bot (`bot`) | Per agent persona |

Hosts MAY define promotion/demotion rules; if they do, they MUST document persistence boundaries.

### 10.2 Typing

All values are **strings**. Interpreters MUST stringify non-string host injections.

### 10.3 Maps

Maps are **functional** string→string relations. Unless otherwise specified, longest-key wins is **implementation-defined** and MUST be documented.

---

## 11. Recursion and Control Flow

### 11.1 `<srai>` composition

`<srai>` is the primary means of **decomposition**: map many surface forms to one canonical pattern.

### 11.2 `<sr>`

If supported, `<sr>` MUST be defined as shorthand equivalent to a documented `<srai>` expansion.

### 11.3 Evaluation stacks

Interpreters MUST maintain an evaluation stack sufficient to implement `<srai>` and nested templates; stack overflow MUST be a recoverable error.

---

## 12. Internationalization

### 12.1 Unicode

AIML documents MUST be representable in UTF-8. Interpreters MUST accept UTF-8 inputs.

### 12.2 Matching across scripts

Normalization profiles SHOULD be capable of **script-agnostic** tokenization or explicitly label script-specific behavior.

### 12.3 Localized templates

`xml:lang` on nodes SHOULD inform selection of localized literals where hosts maintain parallel documents.

---

## 13. Accessibility Considerations

- Primary output SHOULD remain human-readable when OOB channels carry UI instructions.  
- Media modules SHOULD support alternative text attributes where applicable.  
- Agents SHOULD avoid sole reliance on color/emphasis markers in text without textual content.

---

## 14. Security and Privacy

### 14.1 Threat model

Interpreters MUST treat **user input** as untrusted data that influences **matching only** after normalization; template authors are trusted **only** to the extent enforced by the host.

### 14.2 Learning and code execution

Learning modules MUST NOT execute host code from AIML content. Any **turing-complete** extension MUST be isolated in a host-managed sandbox.

### 14.3 Remote services

External calls MUST be **allowlisted** by URL pattern or host registry.

### 14.4 Privacy

Predicates MUST NOT leak across sessions unless explicitly specified by the host **and** disclosed to end users.

### 14.5 Directive authority

Interaction directives and presentation envelopes MUST remain under host authority. Implementations MUST NOT allow authored AIML alone to mint arbitrary item identifiers, force unauthorized private invitations, create impossible schedule commitments, or inject unrestricted presentation/scene payloads without host validation.

### 14.6 Capability boundaries and anti-enumeration

Capability-gated modules MUST prevent unauthorized discovery of protected threads, rooms, groups, memberships, inventories, or scene objects.

- A denied read MUST NOT reveal hidden targets through distinguishable enumeration side channels unless the host profile explicitly documents that behavior.  
- A denied write MUST NOT be treated as a partial success.  
- Hosts SHOULD minimize the visibility of participant lists, presence state, and recent history to the smallest scope needed for the active conversation target.

### 14.7 LLM grounding and anti-hallucination

If §9.7 is supported, hosts MUST treat model output as untrusted until validated. In particular:

- A model MUST NOT establish world truth merely by asserting it.  
- A model MUST NOT imply possession, location, membership, permission, or schedule commitment unless that fact is supplied by host grounding or accepted through a validated directive/effect.  
- When a model proposes unsupported or invalid structured fields, hosts MUST ignore or sanitize them rather than inventing semantics on the fly.  
- Hosts SHOULD preserve a clear separation between grounded facts, inferred narrative text, and untrusted model proposals.

### 14.8 ECS, event, and planner authority

If §9.13, §9.14, or §9.15 are supported:

- AIML MUST NOT spoof entity ownership, component state, event provenance, or planner decisions.  
- Hosts MUST validate any entity-targeted, event-emitting, or planner-advisory payloads before they affect runtime state.  
- Replayed events or planner updates MUST NOT create duplicate protected commits unless the host explicitly documents replay semantics.

---

## 15. Error Handling and Diagnostics

| Condition | Requirement |
|-----------|-------------|
| Malformed XML | Fatal to parser; MUST NOT silently fix. |
| Malformed JSON | Fatal to loader; MUST NOT silently coerce non-equivalent structures. |
| Malformed USD | Fatal to loader unless the host profile explicitly supports partial recovery. |
| Unknown element in Core | Validator SHOULD flag; interpreter MAY ignore element but MUST NOT ignore unknown **children of `template`** unless documented. |
| Invalid directive payload | Recoverable; directive ignored or rejected per host policy, visible text preserved if possible. |
| Invalid effect payload | Recoverable; ignored or rejected per host policy, with no implicit mutation. |
| Capability denied | Recoverable; host returns a bounded denial result without leaking hidden state. |
| Invalid structured LLM reply | Recoverable; host falls back to plain text handling or documented recovery. |
| Depth limit / cycle | Recoverable; host notified. |
| Invalid capture index | Substitute empty string; MAY log. |

---

## 16. Interchange, Packaging, and Versioning

### 16.1 Multi-file graphs

Hosts MAY concatenate multiple documents or compose multiple layers; order MUST be specified (include order, array order, or layer strength defines tie-breaking per §5.3).

### 16.2 Serialization equivalence

If the same AIML pack is emitted as XML, JSON, and/or USD, the pack author SHOULD assign stable identifiers so equivalence can be tested across serializations. Round-tripping through one serialization MUST NOT change category meaning, match precedence, or directive semantics.

### 16.3 Version strings

Documents SHOULD declare a version string using semantic versioning of this spec profile, e.g., `3.0.0`. XML SHOULD use `aiml/@version`; JSON SHOULD use a top-level `version` or `format` field; USD SHOULD use documented layer or stage metadata.

### 16.4 Renderer, runtime, and proofing separation

Hosts SHOULD distinguish clearly among:

- **authored content/data**  
- **runtime or renderer profile**  
- **proofing or analysis profile**

A package MAY contain resources used only by one of these layers. A proofing profile SHOULD be able to inspect authored data without requiring the live runtime, renderer, or planner loop.

### 16.5 Partial packages and archives

Hosts MAY support **partial package** encoding for import, export, or patch workflows. A partial package MAY contain:

- a subset of categories  
- a subset of narrative nodes  
- optional special resources  
- an optional manifest fragment

If partial packages are supported:

- Stable ids MUST be preserved.  
- Imports MUST define how collisions are handled.  
- Manifest fragments MUST NOT silently override incompatible full-manifest identifiers without host policy.  
- Partial exports SHOULD record enough metadata to be merged, diffed, or reviewed safely.  
- If ECS, event, or planner modules are present, partial exports SHOULD preserve stable scene paths, entity ids, event type/source labels, and planner-trace field names when relevant.

### 16.6 Package merge and patch behavior

When hosts support layered or partial content updates, they MUST document:

- merge precedence  
- id collision policy  
- replacement versus patch semantics  
- whether missing nodes or categories imply deletion or omission

### 16.7 Intermediate and compiled interchange forms

Hosts MAY define intermediate or compiled interchange forms for faster loading, indexing, proofing, or diff workflows, including row-oriented formats historically similar to AIML intermediate files.

If such a form is supported:

- It MUST preserve the canonical infoset defined in §4 without changing category meaning.  
- It SHOULD preserve stable identifiers, source references, and enough template structure to reconstruct the authored form.  
- It MAY flatten category-level fields such as pattern, that, topic, template, source filename, activation metadata, or proofing metadata into tabular or record-oriented encodings.  
- It MUST document escaping, delimiter, and round-trip behavior if it uses CSV-like or line-oriented serialization.

---

## 17. Validation

AIML3-Validators MUST detect:

- Violations of §4 structural invariants.  
- Invalid XML characters.  
- Invalid JSON structure for declared AIML serialization.  
- Invalid USD mapping for declared AIML serialization.  
- Duplicate `id` values if `id` attributes are used by the host schema.  
- References to unknown sets/maps if statically known.  
- Invalid or out-of-range standardized directive payloads if statically known.  
- Invalid `requires`, `unless`, or `cooldown` metadata if statically known.  
- Invalid or unsupported canonical effect names or payloads if statically known.  
- Invalid structured LLM schema fields if the profile claims §9.7 support.  
- Invalid historical `intent`/`pattern` alias use if both are present incompatibly.  
- Invalid history-index forms, local-variable forms, or utility-tag argument forms if the profile claims those classic features.  
- Invalid rich-media text/postback structures, card or carousel composition, or OOB payload wrappers if the relevant modules are claimed.  
- Invalid external-service field combinations, unsupported host-utility arguments, or malformed learn/eval payloads if the relevant modules are claimed.  
- Duplicate narrative node ids if §9.10 is claimed.  
- Invalid manifest `startNode` references if a manifest is present.  
- Broken explicit choice-link targets if statically known.  
- Reserved special-resource name collisions or malformed resource records if §9.11 is claimed.  
- Invalid ECS projection metadata, scene-address mappings, or entity identity collisions if §9.13 is claimed and statically knowable.  
- Invalid event result-shape fields, unsupported event source labels, or malformed correlation metadata if §9.14 is claimed and statically knowable.  
- Invalid planner trace fields or contradictory planner-authority declarations if §9.15 is claimed and statically knowable.

---

## 18. Conformance Testing

A conformance test suite MUST include, at minimum:

1. Literal match and mismatch cases.  
2. `*` and `_` binding and `<star/>` indexing.  
3. `<random>` uniformity smoke test (stochastic with large sample).  
4. `<srai>` depth limit behavior.  
5. `<that>` and `<topic>` precedence cases mirroring §5.3.  
6. Predicate get/set and `<condition>` selection.  
7. Cross-serialization equivalence for at least one pack represented in XML and JSON, and in USD where claimed.  
8. Interaction directive validation and stripping behavior for `give`, `invite_accept`, `date_plan`, `meetup`, and `conversation_end` where claimed.  
9. Category eligibility metadata for `requires`, `unless`, and `cooldown` where claimed.  
10. Effect execution and rejection behavior for `set_flag`, `clear_flag`, `set_slot`, and `clear_slot` where claimed.  
11. History access elements, local-variable scope, and recursive-scope isolation where classic compatibility features are claimed.  
12. Historical wildcard precedence and zero-length capture behavior where `#`, `^`, or priority literals are claimed.  
13. `intent` alias loading and equivalence to `pattern` where claimed.  
14. External-service fallback, timeout handling, and sandbox policy for host-utility tags where claimed.  
15. Learning-store separation for `<learn>` versus `<learnf>` where claimed.  
16. Rich media degradation and OOB separation behavior where claimed.  
17. Multi-party capability checks and anti-enumeration behavior where claimed.  
18. LLM structured-reply validation, fallback, and anti-hallucination rules where claimed.  
19. Narrative-node loading, stable-id preservation, and explicit choice-link traversal where claimed.  
20. Manifest start-node resolution and partial-package merge behavior where claimed.  
21. Proofing-mode outputs for dead ends, unreachable nodes, and shadowed content where claimed.  
22. ECS identity projection, component-view stability, and scene-address mapping where §9.13 is claimed.  
23. Event-source ordering, result-shape validation, deduplication, and replay behavior where §9.14 is claimed.  
24. Planner trace projection, advisory hint handling, and planner-authority boundaries where §9.15 is claimed.

Interpreters claiming **AIML3-Interpreter-Core** MUST pass all Core tests for the declared normalization profile.

---

## Appendix A: Element Index

This appendix is **informative**.

### A.1 Core XML vocabulary

| Element | Section |
|---------|---------|
| `aiml` | §4 |
| `topic` | §7 |
| `category` | §7 |
| `pattern` | §6–7 |
| `that` | §7 |
| `template` | §8 |
| `random`, `li` | §8.3 |
| `condition` | §8.4 |
| `get`, `set`, `bot` | §8.5 |
| `map` | §8.7 |
| `srai` | §8.6, §11 |
| `sr` | §11.2 |
| `think` | §8.8 |
| `uppercase`, `lowercase`, `formal`, `sentence`, `person` | §8.9 |
| `star`, `thatstar`, `topicstar` | §8.2 |
| `input`, `request`, `response`, indexed `that` | §8.10 |
| `id`, `program`, `vocabulary`, `size` | §8.12 |
| `normalize`, `denormalize`, `explode`, `person2`, `gender` | §8.13 |

### A.2 Canonical extension vocabulary

The following names are standardized by this draft at the infoset level. They MAY appear as XML extension elements, JSON object keys, USD metadata fields, or other documented serialization forms.

| Name | Section |
|------|---------|
| `requires`, `unless`, `cooldown` | §7.5, §9.4 |
| `intent` | §4.6, §7.1 |
| `give`, `invite_accept`, `date_plan`, `meetup`, `conversation_end` | §9.3 |
| `set_flag`, `clear_flag`, `set_slot`, `clear_slot` | §9.4 |
| `button`, `reply`, `link`, `image`, `video`, `card`, `carousel`, `delay`, `split`, `list`, `olist` | §9.1 |
| `oob`-style payload containers | §9.2 |
| `sraix`, `date`, `interval`, `system` | §9.8 |
| `learn`, `learnf`, `eval` | §9.9 |
| `manifest` fields (`id`, `title`, `startNode`, `profiles`, `creator`, `creatorVersion`, `version`, `tags`, `runtimeProfile`, `proofingProfile`) | §4.7, §4.11 |
| narrative node fields (`id`, `name`, `tags`, `metadata`, `body`, `links`) | §4.9, §9.10 |
| link fields (`label`, `target`, `conditions`, `effects`, `directives`, `metadata`) | §4.9, §9.10, Appendix H |
| special resources (`global_script`, `global_stylesheet`, `init_data`, `proofing_data`) | §4.10, §9.11 |

---

## Appendix B: Informative References

- **RFC 2119:** Key words for use in RFCs  
- **JSON:** ECMA-404 / RFC 8259  
- **XML 1.0:** W3C XML specification  
- **Namespaces in XML 1.0**  
- **OpenUSD / Universal Scene Description**  
- Historical AIML 2.x community specifications and schemas (for comparative reading only)

---

## Appendix C: Migration Notes from AIML 2.x

Authors migrating from AIML 2.x SHOULD:

1. Pin a **normalization profile** and re-verify patterns that depended on punctuation quirks.  
2. Replace engine-specific wildcard semantics with explicit literals where feasible.  
3. Move remote calls and OOB behaviors behind declared **modules** (§9).  
4. Prefer authoring against the canonical infoset so XML, JSON, and USD serializations remain equivalent.  
5. Externalize world-state guards and effects instead of hiding them in ad hoc host code.  
6. Introduce capability-gated multi-party targets explicitly rather than relying on out-of-band conventions.  
7. Treat LLM output as a proposal layer, not as a new authority tier.  
8. Prefer standardized module claims for rich media, OOB payloads, learning, external services, and host utility tags rather than relying on vendor-only extensions.  
9. Add tests for `<srai>` loops that were previously “accidentally safe” due to undocumented engine limits.

---

## Appendix D: Profile Matrix

This appendix is **informative**.

| Profile | Intended use | Typical modules |
|---------|--------------|-----------------|
| **Core Dialog** | Deterministic authored conversation with no protected world mutation. | Core only. |
| **Dialog + Directives** | Dialog plus validated item transfer, invites, scheduling, or session-ending actions. | §9.3, optionally §9.6. |
| **Dialog + Messaging** | Direct threads, rooms, or group conversations with membership-aware context. | §9.3, §9.5, optionally §9.6. |
| **Dialog + LLM** | Authored AIML plus bounded model generation or fallback under host validation. | §9.7, RECOMMENDED: §9.3 and §9.6. |
| **Classic AIML Compatibility** | Historical AIML 2.0/2.1 authoring surface carried into 3.0 without XML-only assumptions. | §6.5, §8.10–§8.13, §9.1, §9.2, §9.8, §9.9. |
| **Full World/Scene** | A general interaction layer for multi-user or scene-rich simulations. | §9.3, §9.4, §9.5, §9.6, §9.10, optionally §9.7 and USD serialization. |
| **Simulation / Planner** | AIML grounded in ECS state, event streams, or planner context. | §9.13, §9.14, §9.15, RECOMMENDED: §9.4 and §9.5. |
| **Proofing / Analysis** | Non-playable review, lint, and export workflow. | §9.12, RECOMMENDED: §9.10 and §9.11. |

---

## Appendix E: Serialization Mapping Example

This appendix is **informative**.

The following examples describe one logical category:

- Pattern: `COFFEE`
- Visible reply: `One coffee coming up.`
- Preconditions: actor is on shift; high-risk boundary state blocks the rule
- Cooldown: `vendor:coffee`
- Directive: give the user `coffee`
- Effect: set flag `ordered_coffee`

### E.1 XML example

```xml
<aiml version="3.0.0" xmlns:a3="urn:aiml:3.0:ext">
  <category id="vendor_coffee" cooldown="vendor:coffee">
    <pattern>COFFEE</pattern>
    <template>One coffee coming up. [GIVE:coffee]</template>
    <a3:requires npcOnShift="yes"/>
    <a3:unless boundaryRiskBand="high"/>
    <a3:effects>
      <a3:set-flag name="ordered_coffee"/>
    </a3:effects>
  </category>
</aiml>
```

### E.2 JSON example

```json
{
  "version": "3.0.0",
  "categories": [
    {
      "id": "vendor_coffee",
      "pattern": "COFFEE",
      "template": {
        "say": "One coffee coming up.",
        "give": "coffee"
      },
      "requires": { "npcOnShift": "yes" },
      "unless": { "boundaryRiskBand": "high" },
      "cooldown": "vendor:coffee",
      "effects": [
        { "type": "set_flag", "flag": "ordered_coffee" }
      ]
    }
  ]
}
```

### E.3 USD example

```usda
#usda 1.0
def Scope "AIML" {
    def AimlCategory "vendor_coffee" (
        customData = {
            string cooldown = "vendor:coffee"
        }
    ) {
        string pattern = "COFFEE"
        string template = "One coffee coming up."
        custom string[] directives = ["give:coffee"]
        custom string[] requires = ["npcOnShift=yes"]
        custom string[] unless = ["boundaryRiskBand=high"]
        custom string[] effects = ["set_flag:ordered_coffee"]
    }
}
```

The three encodings above are equivalent only if the host maps them to the same canonical infoset and runtime behavior.

---

## Appendix F: Worked Examples

This appendix is **informative**.

### F.1 Item grant

```json
{
  "pattern": "WATER",
  "template": { "say": "Stay hydrated.", "give": "water" }
}
```

### F.2 Invite accept or decline

```xml
<category>
  <pattern>COME OVER</pattern>
  <template>
    <condition name="canAcceptInvite">
      <li value="yes">Sure. [INVITE_ACCEPT]</li>
      <li>Not yet. Let's meet somewhere public first.</li>
    </condition>
  </template>
</category>
```

### F.3 Schedule plan

```json
{
  "pattern": "DINNER TOMORROW",
  "template": {
    "say": "Tomorrow evening works for me.",
    "date_plan": { "locationId": "bistro", "dayOffset": 1, "slot": "evening" }
  }
}
```

### F.4 Group-thread reply

```json
{
  "pattern": "WHO IS COMING",
  "template": "Check the group headcount and tell them I’m in.",
  "requires": { "conversationTargetKind": ["group", "room"] }
}
```

### F.5 Flag-setting service flow

```json
{
  "pattern": "BOOK IT",
  "template": "Done. I’ve marked that as confirmed.",
  "requires": { "serviceFlowNode": "confirm" },
  "effects": [
    { "type": "set_slot", "slot": "appointment_status", "value": "confirmed" },
    { "type": "set_flag", "flag": "booking_confirmed" }
  ]
}
```

### F.6 Story manifest fragment

```json
{
  "manifest": {
    "id": "harbor_weekend_pack",
    "title": "Harbor Weekend",
    "startNode": "intro_arrival",
    "profiles": ["Core Dialog", "Full World/Scene"],
    "runtimeProfile": "scene_runtime_v1",
    "proofingProfile": "graph_review_v1",
    "creator": "AIML Studio",
    "creatorVersion": "2.4.0",
    "version": "3.0.0",
    "tags": ["weekend", "harbor", "seasonal"]
  }
}
```

### F.7 Narrative node with explicit links

```json
{
  "id": "intro_arrival",
  "name": "Arrival",
  "tags": ["intro", "hub"],
  "metadata": {
    "mood": "warm",
    "locationId": "harbor_square",
    "entityPath": "/World/Venues/HarborSquare"
  },
  "body": "The square is crowded, bright, and louder than the message thread made it sound.",
  "links": [
    {
      "label": "Head to the cafe",
      "target": "cafe_entry",
      "conditions": { "venueOpen": "yes" }
    },
    {
      "label": "Ask who is already here",
      "effects": [{ "type": "set_flag", "flag": "asked_headcount" }]
    }
  ]
}
```

### F.8 Partial package fragment

```json
{
  "manifest": {
    "id": "harbor_weekend_pack",
    "version": "3.0.1"
  },
  "categories": [
    { "id": "vendor_coffee", "pattern": "COFFEE", "template": { "say": "One coffee coming up.", "give": "coffee" } }
  ],
  "nodes": [
    { "id": "cafe_entry", "name": "Cafe Entry", "body": "The line moves faster than expected." }
  ]
}
```

### F.8a Special resource fragment

```json
{
  "name": "proofing_data",
  "contentType": "application/json",
  "content": {
    "reviewNotes": ["Cafe branch needs one harsher refusal variant."],
    "owner": "narrative-team"
  },
  "metadata": {
    "editorOnly": true
  }
}
```

### F.9 Event result projection

```json
{
  "type": "incoming_contact",
  "source": "calendar_hook",
  "payload": { "npcId": "marina", "locationId": "harbor_square" },
  "selected": true,
  "skippedReasons": ["remote_live_lower_priority"],
  "correlationId": "evt-2026-04-13-001"
}
```

### F.10 Planner trace projection

```json
{
  "placementSource": "goap_travel",
  "placementReason": "Travel toward evening social goal.",
  "activeGoalId": "meetup",
  "goalName": "Meet up",
  "actionId": "go_to_harbor_square"
}
```

### F.11 ECS-facing node metadata

```json
{
  "id": "room_observation",
  "name": "Room Observation",
  "metadata": {
    "entityId": "npc_marina",
    "scenePath": "/World/Venues/HarborSquare/Npcs/marina",
    "layerId": "player-checkpoint"
  },
  "body": "She is already by the railing, looking like she got here before the weather made up its mind."
}
```

---

## Appendix G: Multi-User Dungeon Verb Families

This appendix is **informative**.

These verbs are not all Core AIML features, but they are useful standardization targets for hosts building world-facing AIML profiles.

| Verb family | Typical meaning | Likely AIML 3 mapping |
|-------------|-----------------|-----------------------|
| `say` | Public speech in the current room or channel. | Primary output; optionally §9.5 multi-party send. |
| `tell` | Directed private message. | Direct conversation target. |
| `whisper` | Low-visibility directed speech. | Host-defined conversation scope or privacy flag. |
| `emote` | Visible social action without spoken quotation. | Primary output plus optional presentation envelope. |
| `look` | Inspect room, actor, or object. | Host capability or read-only world query. |
| `give` | Transfer item or entitlement. | §9.3 `give` directive. |
| `invite` | Offer or accept entry into a private or shared space. | §9.3 `invite_accept` or host-defined invite verbs. |
| `join` | Enter group, room, party, or channel. | Reserved future effect `join_group` or host action. |
| `leave` | Exit group, room, party, or channel. | Reserved future effect `leave_group` or host action. |
| `follow` | Track or accompany another actor. | Host-defined movement or party semantics. |
| `trade` | Exchange items, currency, or promises. | Composite directives and validated world effects. |
| `use` | Invoke an item, object, or affordance. | Host capability plus validated effect transaction. |

---

## Appendix H: Choice and Link Semantics

This appendix is **informative**.

Hosts implementing explicit choice links SHOULD distinguish clearly among these cases:

| Link kind | Expected behavior |
|-----------|-------------------|
| **navigational** | Changes the active narrative node. |
| **action-only** | Applies effects or directives without changing node focus. |
| **hybrid** | Changes node focus and applies effects or directives. |

Recommended link fields:

| Field | Meaning |
|-------|---------|
| `label` | Player-facing or reviewer-facing text for the choice. |
| `target` | Destination node id when navigational. |
| `conditions` | Visibility or eligibility conditions. |
| `effects` | Applied world-state effects if the link is taken. |
| `directives` | Optional structured directives associated with taking the link. |
| `metadata` | Editor or host-specific annotations. |

Recommended proofing checks:

- link with missing `target` when declared navigational  
- target points to unknown node  
- link is permanently hidden by contradictory conditions  
- node has no visible outgoing links and no terminal marker  
- multiple links share the same label but diverge invisibly in behavior

---

**End of draft.**
