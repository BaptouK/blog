+++
title = "Why I chose Zola over Hugo"
date = 2026-04-05
description = "Comparing Zola and Hugo for a static blog."
[taxonomies]
tags = ["zola", "web", "programming"]
+++

When I wanted to create this blog, I hesitated between Hugo and Zola. Here's why I chose Zola.

## Hugo: the heavyweight

Hugo is the most popular static site generator. Huge community, tons of themes, complete docs. But...

- Go template syntax is awful
- Configuration is a maze
- Too many features I don't need

## Zola: simplicity

Zola is written in Rust (single binary, no dependencies). What convinced me:

1. **Tera templates** — clean syntax, close to Jinja2
2. **Single binary** — `zola serve` and you're off
3. **Built-in Sass** — no external CSS pipeline needed
4. **Hot reload** — real-time changes

## Verdict

For a personal blog, Zola is perfect. Simple, fast, and the tabi theme is gorgeous.

```bash
# Install Zola
cargo install zola

# Create a site
zola init my-blog
cd my-blog
zola serve
```

No regrets.
