---
title: "4 révélations qui vont transformer votre approche de la null-safety en Java (JSpecify + NullAway)"
date: 2026-01-09T21:00:00+01:00
description: "JSpecify 1.0 unifie enfin les annotations de nullité, et NullAway les fait respecter au build. 4 révélations concrètes, avec exemples TYPE_USE, Maven, et pièges de migration."
slug: "java-null-safety-jspecify-nullaway-4-revelations"
categories: ["jvm", "engineering"]
series: "null-safety"
tags: ["java", "null-safety", "jspecify", "nullaway"]
draft: false
---

## TL;DR

- **JSpecify 1.0.0** met fin à la tour de Babel des `@Nullable` en standardisant **4 annotations stables**.
- **@NullMarked** inverse le défaut : *non-null par défaut*, `@Nullable` devient l’exception visible.
- **IDE et CI convergent** : IntelliJ et NullAway se sont alignés (notamment sur les suppressions).
- **TYPE_USE** rend la nullité “chirurgicale” (tableaux, génériques), mais il faut apprendre la grammaire.

<!--more-->

## Introduction

La `NullPointerException` est tellement structurante qu’on en a fait une légende : Tony Hoare a qualifié l’invention de la référence nulle de « billion-dollar mistake[^spring-null-safety] ».  
Et en Java, on a longtemps traité le problème… en surface : une pluie d’annotations, des conventions par framework, et des outils qui ne lisaient pas tous la même sémantique.

Le résultat, vous l’avez vécu :

- une API annotée “à la Spring”, une lib annotée “JetBrains”, un module qui traîne du JSR-305,  
- IntelliJ qui râle, la CI qui passe (ou l’inverse),  
- et au final… la confiance dans l’outillage s’érode.

Ce qui change depuis **JSpecify 1.0.0** (stable)[^jspecify-release-1-0-0] et l’adoption côté tooling (notamment **NullAway**, qui supporte JSpecify « out of the box »)[^nullaway-jspecify-support] : on sort de la débrouille. La nullité devient **un contrat** lisible *partout* (IDE + build) et **vérifiable**.

Voici 4 révélations concrètes qui vont changer votre manière d’écrire (et de relire) du Java.

---

## Révélation 1 — L’industrie a enfin trouvé un accord (et ça change tout)

### Avant : des annotations partout, une sémantique nulle part

Historiquement, “mettre `@Nullable`” en Java ne voulait pas dire grand-chose sans préciser **quel dialecte** : JSR-305, JetBrains, Eclipse, Spring, Checker Framework… chacun avec des nuances (défaut nullable vs non-null, portée, compatibilité outils). Spring lui-même raconte avoir construit sa null-safety initiale sur une base JSR-305 “dormante mais répandue”, faute de mieux.[^spring-null-safety]

### Maintenant : un standard *consensus-driven* et tool-independent

**JSpecify 1.0.0** annonce explicitement : les 4 annotations (`@Nullable`, `@NonNull`, `@NullMarked`, `@NullUnmarked`) sont **officielles** et ne subiront plus de changements incompatibles.[^jspecify-release-1-0-0]  
Et surtout : la page “About” liste une coalition rare (Google, JetBrains, Oracle, Uber, Broadcom/Spring, Microsoft, Meta, etc.).[^jspecify-about]

Ce n’est pas “une tentative de plus”. C’est le moment où l’écosystème dit : **on arrête de se contredire**.

### Code : avant / après (import + package)

```java
// Avant : lequel choisir ?
import javax.annotation.Nullable;
import org.jetbrains.annotations.Nullable;
import org.springframework.lang.Nullable;

// Après : UN standard
import org.jspecify.annotations.Nullable;
```

Et surtout, vous pouvez poser le contrat au bon endroit : le package.

{{< codesnip path="external/herdev-labs/java/java-null-safety/src/main/java/com/headevlabs/tutorials/revelation1/package-info.java" start="1" end="11" lang="java" >}}

Exemple complet (projet `java-null-safety`) : un `@NullMarked` au niveau package, puis un service dont le retour nullable est *obligatoirement* traité.

{{< codesnip path="external/herdev-labs/java/java-null-safety/src/main/java/com/headevlabs/tutorials/revelation1/UserService.java" start="1" end="21" lang="java" >}}

**Impact concret :** quand votre IDE, votre CI, et vos dépendances “parlent” la même nullité, vous arrêtez de bricoler des conventions locales.

---

## Révélation 2 — La règle d’or est inversée : le non-nul devient la norme

Le point qui fait basculer Java du “défensif permanent” vers quelque chose de moderne, c’est **@NullMarked**.

La doc JSpecify résume bien la réalité : dans un scope `@NullMarked`, les types non annotés sont traités comme `@NonNull` (donc « non-null par défaut »), ce qui évite de devoir écrire `@NonNull` partout.[^jspecify-user-guide] C’est exactement le rôle de `@NullMarked` — typiquement posé au niveau package via un `package-info.java`.[^jspecify-nullmarked-javadoc]

### Avant : annoter 80% du code (bruit)

```java
public void process(
  @NonNull String a,
  @NonNull String b,
  @NonNull String c
) {}
```

### Après : annoter le paramètre de la méthode (promoCode)

{{< codesnip path="external/herdev-labs/java/java-null-safety/src/main/java/com/headevlabs/tutorials/revelation2/ShoppingCart.java" start="11" end="26" lang="java" >}}

Exemple concret (projet `java-null-safety`) : `@NullMarked` au niveau classe, un champ `@Nullable`, un paramètre nullable, et un retour nullable.

{{< codesnip path="external/herdev-labs/java/java-null-safety/src/main/java/com/headevlabs/tutorials/revelation2/ShoppingCart.java" start="43" end="75" lang="java" >}}

### Le vrai bénéfice : vous changez ce que “voit” le lecteur

- Quand **tout** est annoté, plus rien ne ressort.
- Quand `@Nullable` est rare, il devient un **signal fort** : “ici, il faut gérer un chemin absent”.

Soyons directs : si vous adoptez JSpecify mais que vous n’utilisez jamais `@NullMarked`, vous ratez une grande partie de la valeur (et vous retombez dans le bruit).

---

## Révélation 3 — Vos outils se parlent enfin (vraiment)

Le problème le plus toxique, ce n’est pas la nullité : c’est **l’incohérence**.

### Le scénario classique

- l’IDE signale un risque,
- NullAway en CI ne voit pas la même chose (ou inverse),
- et vous finissez par ignorer les warnings.

La bonne nouvelle : JetBrains documente noir sur blanc une coordination entre IntelliJ IDEA et NullAway, notamment pour rendre les **suppressions portables**.[^jetbrains-nullability-nullaway]

> IntelliJ reconnaît des suppressions NullAway (ex. `NullAway.Init`) et NullAway accepte des IDs de suppression “style IntelliJ” pour compatibilité.[^jetbrains-nullability-nullaway]

NullAway documente aussi côté compile flags des mécanismes comme **SuppressionNameAliases** (utile quand votre codebase a déjà des suppressions « non-NullAway », ex. l’inspection IntelliJ `DataFlowIssue`).[^nullaway-config]

### Démo : le même bug doit apparaître au même endroit

{{< codesnip path="external/herdev-labs/java/java-null-safety/src/main/java/com/headevlabs/tutorials/revelation3/DataProcessor.java" start="18" end="43" lang="java" >}}

Même principe (projet `java-null-safety`) : un `@Nullable` oublié se voit au même endroit, et une suppression reste portable.

{{< codesnip path="external/herdev-labs/java/java-null-safety/src/main/java/com/headevlabs/tutorials/revelation3/DataProcessor.java" start="45" end="56" lang="java" >}}

### Suppressions : à utiliser comme un scalpel, pas comme un balai

Cas typique documenté : frameworks à cycle de vie (ex. Spring) où NullAway peut signaler “field not initialized” alors que le conteneur garantit l’initialisation. JetBrains cite explicitement la suppression recommandée `NullAway.Init`.[^jetbrains-nullability-nullaway]

```java
@SuppressWarnings("NullAway.Init")
private Repository repo;
```

Le point important : **si vous devez suppress**, faites-le :

- localement (champ/méthode),
- avec un commentaire “pourquoi”,
- et idéalement une tâche de réduction de dette.

---

## Révélation 4 — TYPE_USE : la null-safety devient précise (et donc plus exigeante)

C’est la partie la plus technique, et c’est là que beaucoup se trompent au début.

### TYPE_USE, c’est quoi ?

Les annotations JSpecify s’appliquent sur **l’usage du type** (TYPE_USE), pas juste sur “le champ” ou “la méthode”. NullAway explique aussi qu’à partir de certaines versions, il faut placer les annotations au **bon endroit** (tableaux, types qualifiés), sinon vous aurez des surprises.[^nullaway-jspecify-support]

### Tableaux : l’erreur la plus fréquente (et l’inversion à connaître)

Règle mnémotechnique : **ce qui est juste après `@Nullable` peut être null**.

```java
// Le tableau PEUT être null, ses éléments (String) sont non-null
String @Nullable [] maybeNullArray;

// Le tableau est non-null, ses éléments PEUVENT être null
@Nullable String[] arrayWithNullableElements;
```

Cette logique est explicitée dans la doc JSpecify (syntaxe TYPE_USE)[^jspecify-type-use-syntax] et reprise par NullAway (placement requis).[^nullaway-jspecify-support]

👉 Si vous aviez appris l’inverse, tant mieux : vous venez d’éviter un bug de contrat.

### Génériques : puissance maximale, pièges maximaux

{{< codesnip path="external/herdev-labs/java/java-null-safety/src/main/java/com/headevlabs/tutorials/revelation4/TypeUsePitfalls.java" start="16" end="60" lang="java" >}}

Extraits (projet `java-null-safety`) : placement sur tableaux, puis sur génériques imbriqués (et rappel sur l’invariance).

{{< codesnip path="external/herdev-labs/java/java-null-safety/src/main/java/com/headevlabs/tutorials/revelation4/TypeUsePitfalls.java" start="67" end="82" lang="java" >}}

**Piège à ne pas raconter :** `List<String>` n’est pas un sous-type de `List<@Nullable String>` en Java.
Même si `String` est “plus strict” que `@Nullable String`, **les génériques sont invariants**. Si vous voulez exprimer “liste de strings (potentiellement null)”, vous devez le dire explicitement (`List<@Nullable String>`) ou passer par des wildcards selon le besoin.

### Quand sortir l’artillerie TYPE_USE ?

- **Oui** : API publiques, collections, caches, DTOs partagés, code Kotlin↔Java.
- **Non** : petites méthodes internes où `Optional`/contrôle de flux suffit et où la verbosité tuerait la lisibilité.

---

## Mise en pratique — le défi 1 semaine (sans tout migrer)

Objectif : *preuve par le code*, pas un chantier.

### 1) Ajouter JSpecify (annotations stables)

JSpecify 1.0.0 est publié sur Maven Central.[^jspecify-release-1-0-0]

{{< codesnip path="external/herdev-labs/java/java-null-safety/pom.xml" start="11" end="35" lang="xml" >}}


Extrait `pom.xml` (projet `java-null-safety`) :

### 2) Null-mark un seul package

```java
// src/main/java/com/acme/orders/package-info.java
@org.jspecify.annotations.NullMarked
package com.acme.orders;
```

### 3) Activer NullAway via Error Prone (Maven)

NullAway fournit un exemple Maven complet (Error Prone + NullAway sur le processor path + flags).[^nullaway-config]

{{< codesnip path="external/herdev-labs/java/java-null-safety/pom.xml" start="51" end="85" lang="xml" >}}

Variante (projet `java-null-safety`) : l’activation se fait via un profil Maven, ce qui permet de l’allumer/éteindre pendant une migration.


> Notes “vraies” :
>
> - adaptez `source/target` et surtout **les versions** à votre contexte (JDK, BOM, contraintes d’entreprise).
> - si vous êtes déjà utilisateur NullAway, la doc indique que vous pouvez “swap in” les annotations JSpecify sans générer de nouvelles erreurs *en mode standard* (hors cas de placement TYPE_USE).[^nullaway-jspecify-support]

### 4) Lancer et observer les premiers vrais bugs

```bash
mvn clean compile
```

Si vous utilisez un profil (comme dans l’exemple ci-dessus), lancez plutôt `mvn clean compile -Pnullaway`.

Vous cherchez deux catégories :

- **contrats violés** (retours `@Nullable` déréférencés),
- **contrats flous** (API pas encore marquées, packages non inclus, libs tierces).

---

## Tableau comparatif : Avant / Après

| Aspect                | Avant JSpecify                               | Après JSpecify                                                         |
| --------------------- | -------------------------------------------- | ---------------------------------------------------------------------- |
| Annotations           | Multiples, sémantiques divergentes           | Standard stable (4 annotations)[^jspecify-release-1-0-0]               |
| Défaut                | “nullness unspecified” + conventions locales | `@NullMarked` = non-null par défaut[^jspecify-user-guide]              |
| IDE vs CI             | Divergences fréquentes                       | Alignement IDE/CI (suppressions)[^jetbrains-nullability-nullaway]      |
| Tableaux / génériques | souvent grossier                             | TYPE_USE précis (si maîtrisé)[^nullaway-jspecify-support]              |

---

## Conclusion — plus qu’une rustine : une fondation

JSpecify ne “supprime” pas `null` du Java. Il fait mieux : il rend la nullité **explicite**, **standardisée**, et **vérifiable**.[^jspecify-user-guide] Et avec NullAway, ce contrat n’est pas un commentaire décoratif : il casse le build quand vous mentez.[^spring-null-safety]

Et si vous travaillez avec Spring, l’écosystème pousse déjà dans cette direction : Spring Framework 7 a basculé sur JSpecify et vise une couverture plus complète (y compris tableaux/génériques), avec enforcement au build.[^spring-null-safety]

### Le call-to-action (raisonnable)

Cette semaine : **un package**, `@NullMarked`, NullAway en CI.
Vous aurez un retour immédiat, et vous saurez si votre codebase a un problème de contrats… ou juste un problème d’habitudes.

---

## Pièges et limites (à connaître avant de vous emballer)

- **TYPE_USE** : placement correct obligatoire (tableaux, types qualifiés), sinon erreurs ou contrats inversés.[^nullaway-jspecify-support]
- **Framework lifecycle** : certains patterns “framework-managed” nécessitent suppression ciblée (ex. init Spring), sinon faux positifs.[^jetbrains-nullability-nullaway]
- **Dépendances tierces** : sans annotations, vous aurez des zones “unannotated” (NullAway a des mécanismes de modèles de libs, mais ce n’est pas gratuit).[^nullaway-config]

---

## Alternatives (quand les choisir)

- **`Optional<T>`** : utile pour exprimer l’absence *dans certains retours*, mais pas adapté partout (params, overhead, signatures existantes). Spring rappelle aussi ces limites, et mentionne Valhalla comme piste future côté coût.[^spring-null-safety]
- **Checker Framework** : excellent mais souvent plus lourd en migration/discipline d’équipe (à évaluer selon votre tolérance aux faux positifs et votre budget outillage).[^stackoverflow-nullable-java]
- **Kotlin** : null-safety native côté type system ; si vous êtes en polyglotte JVM, JSpecify met explicitement en avant les bénéfices d’interop Kotlin (notamment côté analyse statique).[^jspecify-github-releases]

[^spring-null-safety]: Spring Blog — Null Safety in Spring applications with JSpecify and NullAway: <https://spring.io/blog/2025/03/10/null-safety-in-spring-apps-with-jspecify-and-null-away>.
[^jspecify-release-1-0-0]: JSpecify — Release 1.0.0: <https://jspecify.dev/blog/release-1.0.0/>.
[^jspecify-about]: JSpecify — About Us: <https://jspecify.dev/about/>.
[^jspecify-user-guide]: JSpecify — Nullness User Guide: <https://jspecify.dev/docs/user-guide/>.
[^jspecify-type-use-syntax]: JSpecify — User Guide, “Type-use annotation syntax”: <https://jspecify.dev/docs/user-guide/#type-use-annotation-syntax>.
[^jspecify-nullmarked-javadoc]: JSpecify Javadoc — Annotation Interface `NullMarked`: <https://jspecify.dev/docs/api/org/jspecify/annotations/NullMarked.html>.
[^nullaway-jspecify-support]: uber/NullAway Wiki — JSpecify Support: <https://github.com/uber/NullAway/wiki/JSpecify-Support>.
[^nullaway-config]: uber/NullAway Wiki — Configuration: <https://github.com/uber/NullAway/wiki/Configuration>.
[^jetbrains-nullability-nullaway]: JetBrains Blog — One Could Simply Add Nullability Check Support… Without Even Noticing It: <https://blog.jetbrains.com/idea/2025/11/one-could-simply-add-nullability-check-support-without-even-noticing-it/>.
[^stackoverflow-nullable-java]: Stack Overflow — What @Nullable to use in Java (as of 2023/JDK21)?: <https://stackoverflow.com/questions/76630457/what-nullable-to-use-in-java-as-of-2023-jdk21>.
[^jspecify-github-releases]: GitHub — jspecify/jspecify release v1.0.0: <https://github.com/jspecify/jspecify/releases/tag/v1.0.0>.
