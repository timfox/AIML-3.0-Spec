# AIML 3.0 Specification README

This document is a short companion to the AIML 3.0 draft specification in [SPECIFICATION.md](/SPECIFICATION.md).

## What this is

AIML 3.0 is a draft specification for a modernized Artificial Intelligence Markup Language. It keeps the classic AIML strengths of explicit authored rules, pattern matching, templates, recursion, predicates, and deterministic control flow, while expanding the format to support newer runtime needs.

This draft treats XML, JSON, and Universal Scene Description (USD) as equivalent serializations of the same canonical AIML infoset.

## What is in the spec

The current draft covers:

- classic AIML category, pattern, `that`, `topic`, template, predicate, map, and recursion semantics
- compatibility surfaces from AIML 2.0 and 2.1, including history references, learning, external services, rich media, OOB, and legacy wildcard behavior
- structured directives such as item transfer, invite acceptance, scheduling, meetup state, and conversation ending
- world-state guards and effects such as `requires`, `unless`, `cooldown`, `set_flag`, and `set_slot`
- multi-party conversation support for rooms, groups, presence, participants, and bounded history
- optional LLM mediation under host validation and anti-hallucination rules
- narrative graph support, manifests, partial packages, proofing outputs, and special resources
- interoperability guidance for ECS, event-driven systems, and planner-driven hosts including GOAP-style planning

## What this is for

This spec is intended for:

- interpreter implementers
- validator and authoring-tool builders
- teams designing reusable dialogue and interaction content
- simulation, narrative, and world-runtime systems that want a rule-based interaction layer

It is written as a reusable standard, not as documentation for any one application, game, or engine.

## Draft status

This is a working draft, not a final standard. It is suitable for implementation, experimentation, and interoperability testing, but some extension vocabulary and packaging details may continue to evolve.

## Suggested reading order

If you are new to the spec, start with:

1. the Abstract and Introduction
2. Conformance
3. Syntax and Document Infoset
4. Processing Model
5. Pattern Language and Template Language
6. the optional modules in Section 9
7. the worked examples and profile matrix

## File

- Main spec: [SPECIFICATION.md](/SPECIFICATION.md)

