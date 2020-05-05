---
layout: post
title:  "Building a VuePress documentation system"
date:   2020-05-04
categories: VuePress documentation
---

In this post, I'll writeup my experiences following along [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-build-a-documentation-system-with-vue-and-vuepress) to build a VuePress based documentation system.

## VuePress

VuePress is another static site generator. It generated pre-rendered static HTML sites, and presents an SPA experience to the reader.

## Steps to Minimal VuePress, Take 1

```sh
mkdir example-vuepress
cd example-vuepress
npm init -y
npm install -V vuepress
```

... and then the tutorial unfortunately lost it. It's unclear to me whether *counter* and *guide* are files or directories, and there is no global overview how the project should look like in the end. I should have taken the disclaimer at the pages top literally.

I'm now switching to the [official guide](https://vuepress.vuejs.org/guide/) - this looks very mature *and* actually works.

## Steps to Minimal VuePress, Take 2

There is a *very* [minimal walkthrough](https://vuepress.vuejs.org/guide/getting-started.html#global-installation) on the VuePress main site, which I'll try next. This went smooth, turns out the minimal directory structure is
    
    .
    ├── docs
    │   └── README.md
    └── package.json

## VuePress Directory Model

The [full directory structure](https://vuepress.vuejs.org/guide/directory-structure.html#default-page-routing) establishes many opportunities for extension. Also the default page routing is handled nicely IMO, with the file -> URL mapping being

| Relative Path       | Page Routing |
| ----                | ---- |
| /README.md          | / |
| /guide/README.md	| /guide/ |
| /config.md          | /config.html |

## .vuepress/config.js

This file is the global configuration for this VuePress site. The [reference documentation](https://vuepress.vuejs.org/config/) contains many more options.

For starters, I tried to set `markdown.lineNumbers` to true. Turns out as always, things are not as simple.

 - First, I thought this is a key, and directly set `"markdown.lineNumbers"` to true - did not work.
 - Then I googled - OK, it does formatting on *code blocks*. Now what exactly is a code block? According to [daringfireball](https://daringfireball.net/projects/markdown/syntax#precode), this is a paragraph indented by at least 4 spaces - did not work.
 - [Further googling](https://v1.vuepress.vuejs.org/guide/markdown.html#line-highlighting-in-code-blocks) revealed that the key actually *is* meant to address an object `markdown` with a key `linenumbers` - still did not work.
 - Finally, switching to `````` style blocks works.

![vuepress-codeblock](/assets/vuepress-codeblock.png)

## Other noteworthy stuff

#### Frontmatter

I wondered what the post header that is required in Jekyll posts means. 

    ---
    layout: post
    title:  "Building a VuePress documentation system"
    date:   2020-05-04
    categories: VuePress documentation
    ---

It turns out this is interpreted by [Frontmatter](https://vuepress.vuejs.org/guide/markdown.html#frontmatter). VuePress uses a project called [gray-matter](https://github.com/jonschlinkert/gray-matter) for parsing front matter.

In the end, these variables represent *metadata* that now can be used inside a post, or from the outside, to e.g. build up indices.