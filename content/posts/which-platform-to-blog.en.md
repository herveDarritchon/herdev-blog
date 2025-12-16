---
categories:
  - "Development"
categories_weight: 44
series:
  - "How to blog with Hugo"
tutorials:
  - "Hugo"
slug: "which-platform-to-blog"
tags:
  - "Development"
  - "Mentoring"
  - "Hugo"
tags_weight: 22
title: "Which platform to blog"
date: 2021-11-30T11:48:33+01:00
publishdate: 2021-12-06
weight: 13
---

Shortly after deciding to blog, I needed to choose a platform. There are many free blogging platforms (e.g. blogger.com, medium.com), but I didn’t want to be locked into a hosted service. As a developer, I wanted the same workflow as my code: Markdown content versioned in Git and automated deployment.

## Why Hugo

I wanted something simple, fast to load, free, and without platform-injected content that could distract readers. I also wanted to write in Markdown and generate a static site from those files.

After researching, I evaluated tools like Jekyll and Hugo. I chose Hugo because it is written in Go and fits my preferences.

## Why GitHub Pages

My content is a set of Markdown files that I deploy like any other app. The most common Git hosts are GitHub and GitLab. I considered GitLab because I use it at work and Hugo integrates well with GitLab pipelines. However, I chose GitHub to learn GitHub Actions.

## Conclusion

The workflow I use: keep content in Markdown, version it with Git, and trigger an automated build-and-deploy pipeline on push. This gives a simple, reliable process to publish a static blog.
