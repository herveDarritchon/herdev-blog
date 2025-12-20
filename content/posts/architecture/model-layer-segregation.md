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

Dans ce post, on va discuter du problème de découpage des couches et des modèles, ce qui va permettre de simplifier le
code, en n'ayant pas de soucis d'impact des choix d'une couche à l'autre. Exemple, la serialization d'une sealed class
en Jackson. Si on découpe et on utilise un mapper alors on n'a plus de sous-type de la sealed class mais seulement la
classe mère.