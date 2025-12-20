---
title: "NoClassDefFoundError: jakarta/servlet/Filter (root cause + fix)"
date: 2025-12-20T19:10:00+01:00
slug: "kotlin-spring-no-default-package"
draft: false
series: "kotlin-spring-no-default-package"
tags:
  - "spring boot"
  - "kotlin"
description: "Erreur Spring Boot fréquente en Kotlin : warning \"@ComponentScan of the default package\" puis crash \"NoClassDefFoundError: jakarta/servlet/Filter\" (souvent sans starter web). Diagnostic immédiat : ta classe @SpringBootApplication est dans le default package. Fix : mets-la dans un vrai package racine."
---

## TL;DR (répare en 60 secondes)

Si tu vois :

- `** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.`
- puis `NoClassDefFoundError: jakarta/servlet/Filter`

➡️ Verdict : **ta classe `@SpringBootApplication` est dans le _default package_ (pas de `package ...`).**  
➡️ Fix : **déclare un vrai package + place le fichier dans le dossier qui correspond** (et aligne tes tests).

<!--more-->

---

## Diagnostic express (copie-colle → verdict)

### Tu as l’un de ces messages ? (ou plusieurs)

```text
** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.
Failed to introspect Class [org.springframework.boot.web.servlet.support.ErrorPageFilterConfiguration]
Caused by: java.lang.NoClassDefFoundError: jakarta/servlet/Filter
Error processing condition on org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration...
```

✅ **Verdict quasi certain** : **ta classe `@SpringBootApplication` est dans le default package.**

---

## Pourquoi cette erreur est trompeuse (et pourquoi tu vas perdre du temps)

Le dernier message te pousse vers la fausse piste :

```text
NoClassDefFoundError: jakarta/servlet/Filter
```

Réflexe classique : *« OK, j’ajoute `spring-boot-starter-web` »*.

Mauvaise réponse dans la majorité des cas.

* Si tu construis une app **non-web** (CLI/batch), ajouter `starter-web` ne fait que masquer le vrai problème.
* Tu vas embarquer Tomcat/Servlet, changer le comportement de démarrage, et te créer une dette inutile.

Ton problème n’est pas “il manque servlet”.
Ton problème est : **Spring scanne trop large**, parce que ton application est en default package.

---

## Cause racine : le default package fait exploser le périmètre du scan

### Le mécanisme (simple)

`@SpringBootApplication` déclenche notamment un `@ComponentScan`.

* Si ta classe main est dans `com.example.demo`, Spring scanne `com.example.demo` et ses sous-packages.
* Si ta classe main est dans le **default package** (aucun `package ...`) : Spring n’a plus de périmètre clair → scan énorme / imprévisible.

Résultat : Spring peut se mettre à **inspecter des classes dans tes dépendances**, y compris des autoconfig Servlet (ex: `ErrorPageFilterConfiguration`).
Et comme tu n’as pas `jakarta.servlet` sur le classpath (normal sans web), tu prends :

```text
NoClassDefFoundError: jakarta/servlet/Filter
```

👉 `jakarta.servlet.Filter` est un **symptôme secondaire**. La cause est le **package**.

---

## Reproduire le bug (pour comprendre, pas pour souffrir)

### 1) Dépendances (pas de web)

`build.gradle.kts` exemple :

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/build.gradle.kts"
lang="gradle"
>}}

### 2) Classe main dans le default package (l’erreur)

`src/main/kotlin/DemoApplication.kt` **sans** ligne `package ...` :

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/src/main/kotlin/DemoApplication.kt"
lang="kotlin"
>}}

### 3) Résultat attendu

* warning `@ComponentScan of the default package`
* puis crash avec `NoClassDefFoundError: jakarta/servlet/Filter`

---

## La correction propre (évite cette classe d’effets de bord / évite la dérive de scan)

### Étape 1 — Mets un vrai package (obligatoire)

Déplace ton fichier dans un dossier cohérent, par exemple :

`src/main/kotlin/fr/behaska/labs/kotlinspringnodefaultpackage/DemoApplication.kt`

Et ajoute un package en haut du fichier :

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/src/main/kotlin/fr/behaska/labs/kotlinspringnodefaultpackage/DemoApplicationWithDefaultPackage.kt"
lang="kotlin"
>}}

### Étape 2 — Mets tout le reste sous ce package (ou en sous-packages)

* `fr.behaska.labs.kotlinspringnodefaultpackage.controller`
* `fr.behaska.labs.kotlinspringnodefaultpackage.service`
* etc.

### Étape 3 — Aligne tes tests (sinon tu te prends l’autre classique)

Si tes tests ne sont pas dans le même arbre de packages, tu peux obtenir :

```text
Unable to find a @SpringBootConfiguration by searching packages upwards from the test
```
Exemple:

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/src/test/kotlin/DemoApplicationTest.kt"
lang="kotlin"
>}}

Fix : place les tests dans `fr.behaska.labs.kotlinspringnodefaultpackage` (ou sous-package) :

`src/test/kotlin/fr/behaska/labs/kotlinspringnodefaultpackage/...`

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/src/test/kotlin/fr/behaska/labs/kotlinspringnodefaultpackage/DemoApplicationTest.kt"
lang="kotlin"
>}}

---

## “Package directive does not match the file location” : normal (et tu dois écouter l’IDE)

Kotlin (le langage) autorise un `package` qui ne correspond pas au dossier disque.

Mais ton IDE (IntelliJ par exemple) te met ce warning parce que :

* tu vas te piéger en navigation/refactor,
* tu vas relire le dossier et croire que “c’est dans le bon package” (alors que non),
* Spring, lui, se base sur le **package déclaré**, pas sur ton arborescence.

Pour un projet (et a fortiori un exemple de blog) : **package = dossier**. Point.

---

## Checklist de debug (rapide, pragmatique)

### Checklist “je veux comprendre en 30 secondes”

{{< paperchecklist >}}

* [ ] Est-ce que mon fichier `DemoApplication.kt` commence par `package ...` ?
* [ ] Est-ce que le chemin correspond au package ?
  Exemple : `fr.behaska.labs.demo` ↔ `src/main/kotlin/fr/behaska/labs/demo/`
* [ ] Est-ce que je vois le warning `@ComponentScan of the default package` ?
  Si oui : **arrête tout, corrige le package**.
* [ ] Est-ce que mes tests sont dans le même package racine (ou en dessous) ?

{{< /paperchecklist >}}

### Checklist “je veux verrouiller que c’est une app non-web”

Si ton app est CLI/batch, tu peux verrouiller l’intention :

`src/main/resources/application.properties` :

```properties
spring.main.web-application-type=none
```

Ça ne remplace pas le fix du package, mais ça évite les surprises si quelqu’un ajoute une dépendance web plus tard.

---

## FAQ (les questions qui reviennent)

### “Pourquoi Spring essaie de charger des trucs Servlet alors que je n’ai pas web ?”

Parce que ton scan est hors contrôle : il inspecte des classes d’auto-configuration qu’il n’aurait jamais dû toucher dans une app correctement packagée.

### “Est-ce que je peux régler ça en ajoutant `spring-boot-starter-web` ?”

Tu peux… mais tu masques le problème et tu changes ton appli. Tu viens d’ajouter un serveur web juste pour faire taire un symptôme.

### “Et si je ne veux vraiment pas déplacer mes packages ?”

Tu peux forcer le périmètre :

```kotlin
@SpringBootApplication(scanBasePackages = ["fr.behaska.labs.kotlinspringnodefaultpackage"])
class DemoApplication
```

Mais ça reste une mauvaise idée : tu gardes un projet qui se comporte différemment des conventions Spring, et tu vas le payer en friction.

---

## Sources (officielles / reconnues)

Spring Boot Reference — Structuring your code / default package
https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html

Stack Overflow — réponse liée à ce warning (maintainer Spring Boot)
https://stackoverflow.com/questions/28211049/spring-boot-gs-componentscan-and-classnotfoundexception-for-connectionfactory