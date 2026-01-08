---
title: "AI & Java : passer des prompts au logiciel (vraiment) – mémoire, outils, JSON et Quarkus"
date: 2026-01-06
description: "Ce que montre une démo Quarkus/LangChain4j : arrêter de « discuter avec un LLM » et commencer à construire une application fiable (mémoire, outils, sorties structurées, garde-fous)."
tags:
  [
    "java",
    "quarkus",
    "agents",
    "rag",
    "architecture",
    "langchain4j",
  ]
series: "ai-agents-and-java"
---

> Inspiré de la conf vJUG CONNECT “AI & Java: From Structured Prompts to Smarter Apps” ([YouTube](https://www.youtube.com/watch?v=8l1obGG_6jQ)).

## TL;DR

Si tu retiens une seule idée : **le “prompt engineering” n’est pas le sujet**. Le sujet, c’est **l’ingénierie du contexte** et **du contrat** entre ton code et le modèle :

- **mémoire** (court/long terme, cohérence, conflits),
- **outils** (fonctions appelables par le LLM),
- **sorties structurées** (JSON/records/enums au lieu de texte libre),
- **observabilité + timeouts + fallbacks** (parce que “ça marche en démo” ne vaut rien en prod).

La conf le dit sans détour : ce n’est pas de la magie, c’est du logiciel.

---

## 1) “Le modèle se souvient”… non. Il a une fenêtre de contexte.

Un passage clé de la conf remet les pendules à l’heure : ce qu’on appelle souvent “mémoire” côté LLM, c’est d’abord la **working memory** (la mémoire de travail) : _les tokens présents dans la fenêtre de contexte_ et les relations qu’ils induisent.

Conséquence directe pour nous, devs : ton job n’est pas de “trouver le bon prompt”. Ton job est de **décider ce qui entre dans la fenêtre**, et **comment**.

C’est là que la conf bascule vers un vocabulaire plus juste : **context engineering**. Et les leviers de contexte, ce n’est pas que du texte :

- des **tool calls** (le modèle peut demander une info au code),
- des **données récupérées** (RAG),
- des **contraintes** (sortie JSON only, schémas implicites, règles),
- de la **mémoire persistée** (au sens application, pas “LLM”).

---

## 2) RAG + tools + agents : la pente glissante (et pourquoi tu y vas quand même)

Le speaker fait un rappel utile sur RAG : dans la vraie vie (exemple évoqué : données sensibles type dossiers patients), tu ne balances pas un dump au modèle. Tu **récupères la bonne quantité d’info**, au bon moment, puis tu demandes au LLM de répondre.

Ensuite vient la suite logique : une fois que tu fournis des **fonctions** (tools) au modèle — “voilà les options disponibles pour répondre” — tu glisses très vite vers un **agent** :

- un état,
- une boucle,
- des appels à outils / APIs,
- et une gestion de mémoire.

Et là, il prévient : tu finis par donner **de plus en plus d’outils** à l’agent. Ça marche… jusqu’au moment où tu dois gérer :

- la cohérence,
- les conflits,
- les erreurs,
- et la performance.

Bref : **on retombe sur des problèmes d’architecture**.

---

## 3) La mémoire “agentique” : deux dimensions qui vont te casser en prod

La conf propose une grille simple (et très “ingé”) : la mémoire d’un agent explose sur **deux axes**.

### Axe 1 — Le temps

Ton agent doit gérer :

- du **court terme** (ce qu’il doit accomplir maintenant),
- du **long terme** (ce qui persiste au fil des sessions),
- et surtout la **réconciliation**.

Exemple donné : aujourd’hui “j’aime la pizza pepperoni”, hier “j’étais plutôt hamburgers”. Si on repose la question “qu’est-ce que j’aime ?”, l’agent doit savoir :

- que ça dépend du temps,
- que ça peut évoluer,
- et comment arbitrer.

Donc : **entity reconciliation** + **conflict resolution**. Pas sexy. Indispensable.

### Axe 2 — L’espace (multi-agent)

Avec plusieurs agents / tâches en parallèle :

- qui écrit quoi ?
- quel est le “scope of truth” de chaque info ?
- comment tu synchronises / merges l’état ?

Et là encore, le speaker insiste : ce n’est pas “AGI”, c’est **du logiciel**. Les mêmes questions qu’en distribué reviennent : concurrence, cohérence, journalisation, arbitrage.

---

## 4) Le piège : itérer avec les mêmes données foireuses (et croire que le LLM “va finir par comprendre”)

Un moment très vrai de la conf : le “horrible loop”.
Tu demandes au modèle de corriger un truc.
Il répond.
Tu redemandes.
Il répond.
Et ça tourne.

Pourquoi ? Parce que tu lui redonnes **les mêmes entrées**, les mêmes ambiguïtés, les mêmes contraintes implicites. Le modèle ne “répare” pas un problème de spécification.

Phrase importante (paraphrase fidèle de l’idée) : **le code d’implémentation mélange le “quoi” et le “comment”**. Donc tu perds l’intention, tu perds la reproductibilité, et tu te retrouves à bricoler.

---

## 5) La réponse : structurer _avant_ de coder (Knowledge.md, usage specs, décisions persistées)

La démo “spec-first” est l’un des trucs les plus actionnables de la conf :

- un agent lit un **Knowledge.md** (le contexte, les règles, la base),
- il détecte des **usage specs** (contrats d’usage),
- il pose des questions (bootstrapping),
- et il écrit une **spec** + un fichier de décisions (type “agents.md”) pour persister les choix.

Le point n’est pas l’outil exact (dans la conf, il est question de “Tessle”, de Claude, et d’un serveur MCP), le point est la méthode :

> **Le prompt n’est pas un message. C’est un artefact versionné.**  
> Et les décisions importantes doivent survivre à la session.

Ça change tout : tu arrêtes de “converser”, tu **construis un pipeline**.

---

## 6) Côté Java : Quarkus + LangChain4j, ou comment intégrer proprement

La partie live-coding est très parlante : démarrer avec un projet Maven, DI, et une “AI service” enregistrée par annotation. L’intérêt : tu passes d’un appel brut à un modèle à un **service typé** que ton code peut appeler.

### Exemple minimal (illustratif)

L’idée montrée : un greeter, puis on le rend “smart”, puis on lui ajoute des règles.

```java
import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.UserMessage;
import io.quarkiverse.langchain4j.RegisterAiService;

@RegisterAiService
public interface Greeter {
    @SystemMessage("You are a helpful greeter. Always answer in French.")
    String greet(@UserMessage String name);
}
```

La conf montre aussi l’usage d’un **Dev UI** pour tester, et surtout l’importance de **logger requêtes/réponses** quand ça part en vrille (et ça part en vrille).

---

## 7) Tools : quand ton LLM doit arrêter d’inventer et appeler ton code

Démo simple et efficace : “donne l’heure courante”.
Oui, ton code _pourrait_ juste appeler `LocalTime.now()`. Mais le point est architectural : tu donnes au LLM une **capacité vérifiable** via un outil.

Et la conf montre un détail qui te fera gagner du temps : si tu oublies l’annotation tool… ça ne marche pas, et tu peux passer 10 minutes à “débugger l’IA” alors que tu as juste oublié une annotation.

```java
import dev.langchain4j.agent.tool.Tool;
import jakarta.enterprise.context.ApplicationScoped;
import java.time.LocalTime;

@ApplicationScoped
public class TimeTools {

    @Tool("Return the current local time")
    public String time() {
        return LocalTime.now().toString();
    }
}
```

À partir de là, tu peux donner au modèle une “toolbox” : et tu reviens au vrai sujet de la conf — **context engineering**.

---

## 8) Sorties structurées : arrêter le texte libre, passer au JSON (records, enums)

C’est le cœur “From Structured Prompts to Smarter Apps” : tant que tu restes en texte libre, tu peux faire une démo. En prod, tu vas pleurer.

La conf illustre le switch : au lieu de `String`, on retourne un **objet** (record), avec une **enum**, et on force un JSON exploitable.

```java
public enum PetType { DOG, CAT, FISH, BIRD }

public record Pet(String name, PetType type) {}
```

Et côté service IA :

```java
@RegisterAiService
public interface PetService {
    @SystemMessage("Return only JSON.")
    Pet suggestPet(@UserMessage String constraints);
}
```

Le speaker insiste sur le fait que tu peux ajouter des **guardrails** (détection “JSON only”, validation, etc.). Et surtout : **timeouts** et **fallbacks**. Parce qu’une app qui attend indéfiniment une réponse “intelligente” est une app morte.

---

## 9) Front + back : l’intégration compte (Quinoa) et le “media type” te rattrape

Un moment de vérité en live : un endpoint renvoie “un truc pas JSON”, ça ne marche pas, et quelqu’un dans la salle rappelle… le **media type**.

C’est bête, mais c’est exactement ça le sujet : quand tu branches un LLM sur une app, tu ne sors pas des lois de la physique. Tu dois :

- maîtriser tes endpoints,
- tes contrats,
- tes formats,
- ton front/back.

Et c’est pour ça qu’un framework d’intégration (ici, Quarkus + extension front type Quinoa) a de la valeur : il réduit les zones grises.

---

## 10) Check-list “architecture” pour arrêter de jouer et commencer à livrer

Si tu veux transformer une démo LLM en système, prends ça comme un minimum vital :

### Contrat

- Sorties **structurées** (JSON/records/enums), pas du texte “à parser”.
- “JSON only” + validation stricte + gestion d’erreur explicite.

### Contexte

- RAG : _retrieve the right amount_, pas “toute la base”.
- Outils : privilégier l’appel à ton code plutôt que l’invention.
- Contexte versionné : Knowledge.md / usage specs / décisions persistées.

### Mémoire

- Séparer : court-terme / long-terme.
- Prévoir : réconciliation, conflits, temporalité.
- En multi-agent : scopes de vérité + stratégie de merge.

### Résilience

- **Timeouts** partout.
- **Fallbacks** (dégradé acceptable > panne totale).
- Observabilité : logs req/rsp, tracing, métriques de latence, taux d’échec.

### Sécurité

- Ne pas brancher un LLM “en direct” sur des systèmes sensibles.
- Gouverner les outils (permissions, filtrage, audit).

---

## Conclusion

La conf vJUG a un mérite rare : elle ramène l’IA à sa place.
Un LLM, c’est puissant… mais **ça ne remplace pas l’architecture**. Ça la rend juste plus visible, plus tôt, et plus brutalement.

Donc si tu veux “des apps plus smart” en Java :

- arrête de “trouver des prompts”,
- commence à **concevoir des contrats**, de la **mémoire**, des **outils**, et des **garde-fous**,
- et traite ton système IA comme n’importe quel système distribué : **observable, résilient, testable**.
