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

Today, I was working on my side project and I encountered an issue with [Jackson](/tags/jackson/) serialization of [Kotlin](/tags/kotlin) Sealed Class subtypes.

The case is quite simple and I think it deserves a blog post to explain it and how to fix it :) But ready carefully to the end because there is a bonus solution ;)

You have a sealed class for example
Source : https://stackoverflow.com/questions/68938577/kotlin-sealed-class-subtyping-in-jackson

Comment sérialiser une sealed class Kotlin avec Jackson.
Faire un lien sur le post d'architecture qui explique pourquoi si on découpe les couches, on n'a plus ce souci, ce qui est mieux, plus simple :)
