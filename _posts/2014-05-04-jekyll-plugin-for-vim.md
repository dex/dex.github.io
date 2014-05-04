---
layout: post
title: "Jekyll plugin for VIM"
published: true
tags: [vim]
categories: [software]
---

There is a VIM plugin for *Jekyll* called [jekyll.vim](https://github.com/csexton/jekyll.vim/).

It is easy to use and only has three commands.

- `JekyllList` lists all posts
- `JekyllPost` creates a post with YAML front-matter prepared
- `JekyllBuild` build whole site

But you have to define a global variable `g:jekyll_path` to your jekyll folder.

Some other useful settings:

	let g:jekyll_post_suffix = "md"
	let g:jekyll_prompt_tags = "true"
	let g:jekyll_prompt_categories = "true"
