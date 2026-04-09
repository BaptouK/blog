+++
title = "Zola"
date = 2026-04-05
description = "An overview of Zola, the static site generator I use for this blog."
[taxonomies]
tags = ["zola", "web", "programming"]
+++

[Zola](https://www.getzola.org/) is a static site generator written in Rust. It turns Markdown files into a ready-to-deploy HTML site — no database, no dynamic server, just static files.

## Why Zola

- **Single binary** — no dependencies, no Node.js, no npm. Install, run, done.
- **Fast** — written in Rust, builds are near-instant even with many pages.
- **Tera templates** — clean and readable syntax, close to Jinja2/Django.
- **Built-in Sass** — native CSS compilation, no external pipeline needed.
- **Hot reload** — `zola serve` and every change shows up in real time in the browser.

## Why I chose it

I wanted a simple tool for a personal blog, no bloat. Zola checks all the boxes: lightweight, easy to pick up, and the [tabi](https://github.com/welpo/tabi) theme provides everything I need (dark mode, table of contents, shortcodes, SEO).

The result: a static blog hosted on my [homelab](@/projects/homelab.md), deployed in seconds.
