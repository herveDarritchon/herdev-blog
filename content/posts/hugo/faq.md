+++
categories = ['Hugo']
categories_weight = 62
series = ['Blogging with Hugo']
tutorials = ['Hugo']
slug = 'hugo-faq'
tags = ['Development', 'Hugo', 'Tutorials']
tags_weight = 41
title = 'Hugo FAQ'
date = 2022-07-31T10:48:33+01:00
draft = true
weight = 3
+++

- pensez à récupérer les thèmes avec l'erreur :
- found no layout file for "HTML" for kind "home
- lancer la commande pour connaître si il y a bien un submodule:
  - git config --file .gitmodules --get-regexp path | awk '{ print $2 }'
- lancer l'une des commandes pour récupérer le theme en submodule:
  - git submodule update --init --recursive
  - git submodule update --recursive --remote
- lancer hugo serve en mode draft:
  - hugo server --buildDrafts
  - hugo server -D
