---
categories:
  - "Development"
categories_weight: 52
series:
  - "Kotlin tutorials"
tutorials:
  - "Kotlin"
slug: "sealed-class-subtype-serialization"
tags:
  - "Development"
  - "Kotlin"
  - "Jackson"
tags_weight: 31
title: "Sealed Class Subtype Serialization"
date: 2022-07-27T14:48:33+01:00
draft: true
weight: 2
---

Today I worked on a side project and encountered an issue with Jackson serialization of Kotlin sealed class subtypes.

The problem is common: sealed classes and polymorphic serialization can require configuration to preserve subtype information. A typical solution is to control serialization using explicit type identifiers or custom serializers.

A practical approach:

- Define DTOs or interfaces for the public API surface instead of exposing internal sealed subtypes.
- Use Jackson annotations or a custom module to register subtype handling (e.g. @JsonTypeInfo + @JsonSubTypes) when necessary.
- Prefer mapping between domain models and transport DTOs to avoid coupling serialization details across layers.

See: https://stackoverflow.com/questions/68938577/kotlin-sealed-class-subtyping-in-jackson for an example and further discussion.
