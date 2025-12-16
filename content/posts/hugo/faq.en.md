---
categories:
  - "Hugo"
categories_weight: 62
series:
  - "Blogging with Hugo"
tutorials:
  - "Hugo"
slug: "hugo-faq"
tags:
  - "Development"
  - "Hugo"
  - "Tutorials"
tags_weight: 41
title: "Hugo FAQ"
date: 2022-07-31T10:48:33+01:00
draft: true
weight: 3
---

- If you get errors about missing themes, check that the theme submodule is initialized.
- Common error: "found no layout file for \"HTML\" for kind \"home\"" — verify your theme includes the required layout files.
- To check configured submodules run:
  - `git config --file .gitmodules --get-regexp path | awk '{ print $2 }'`
- To fetch the theme submodule run one of:
  - `git submodule update --init --recursive`
  - `git submodule update --recursive --remote`
- To preview drafts locally run Hugo in draft mode:
  - `hugo server --buildDrafts`
  - or `hugo server -D`

These commands help diagnose theme and submodule issues and let you run the site locally with drafts enabled.
