+++
categories = ['Development']
categories_weight = 44
series = ['How to blog with Hugo']
tutorials = ['Hugo']
slug = 'which-platform-to-blog'
tags = ['Development', 'Mentoring', 'Hugo']
tags_weight = 22
title = 'Which platform to blog'
date = 2021-11-30T11:48:33+01:00
publishdate= 2021-12-06
weight = 13
+++

Quickly after deciding to ***Blog***, I needed to choose my platform. I have found quite a lot of blogging platforms free on Internet ([blogger.com](https://www.blogger.com/), [medium.com](https://medium.com/), ...). But I didn't want to be locked down to one platform and as a developper I wanted to have the same process as coding. So I shorty felt the need to have something simple, something very similar to my day to day job, it's why I chose to do my content in markdown and to deploy it as I push code to a git repo.

## Why Hugo

For my blog, I didn't want something fancy, heavy, low to load and I want it free and with no content added by the platform which could divert the reader from my content. And as I said, I wanted to write my content in Markdown so I knew I definitely need some kind of tool to generate a static blog site from my Markdown files.

After searching a bit, I found some tools ike ([Jeckyll](https://jekyllrb.com/), [Hugo](https://gohugo.io/), ...). I could have chose eaither of these two blog engine but I chose **Hugo** because it is written in [Go](https://go.dev/) and not in [Ruby](https://www.ruby-lang.org/en/) :) If I have to learn something I prefer to learn some Go than some Ruby ;)

Now, I have to choose how to deploy it !

## Why Github Pages

I have some content (e.g. some md files) and I want to deploy it as I deploy my apps in the cloud so I have to choose a Git repository. The two main tools are [Github](https://github.com/) and [Gitlab](https://gitlab.com/).

I have to be honest with you if I had followed my ruled in force I would have chosen Gitlab because it is the repository I used in my day to day work so I have quite a good knowledge of Gitlab pipeline et Hugo is very well integrated with Gitlab. So my advice is :
> Use Gitlab with Hugo :)

But it is always good to learn so I have chosen Github because I don't know Github Actions and I want to learn something new when I can.

## Conclusion

I have a good Workflow to do and deploy my Blog. I have a bunch of markdown files with versionning using Git and when I push my new content (using `git commit`), the new version of my Blog is automatically deployed to my [Github Pages](https://hervedarritchon.github.io).

This is really what I wanted, something simple, something close to how I work and using Hugo Themes I have something with a somehow good looking. 