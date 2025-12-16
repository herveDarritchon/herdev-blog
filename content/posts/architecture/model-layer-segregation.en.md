---
categories:
  - "Architecture"
categories_weight: 52
series:
  - "Software Architecture Principles"
tutorials:
  - "Http4k"
slug: "model-layer-segregation"
tags:
  - "Development"
  - "Architecture"
  - "Back-End"
tags_weight: 31
title: "Model Layer Segregation"
date: 2022-07-26T14:48:33+01:00
draft: true
weight: 3
---

In this post we discuss the problem of separating layers and models to simplify code and avoid cross-layer impact of design choices. Proper separation reduces coupling between layers and makes components easier to test and evolve.

A concrete example is serialization of Kotlin sealed classes with Jackson. If you properly separate mapping logic (e.g. using DTOs and mappers) you avoid needing to expose sealed subtypes across layers: the API layer talks only to the parent type or mapped DTOs.

The main recommendation is to keep serialization concerns inside a dedicated mapping boundary rather than leaking subtype details across the whole stack.
