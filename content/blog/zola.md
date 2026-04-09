+++
title = "Zola"
date = 2026-04-05
description = "Présentation de Zola, le générateur de site statique que j'utilise pour ce blog."
[taxonomies]
tags = ["zola", "web", "programming"]
+++

[Zola](https://www.getzola.org/) est un générateur de site statique écrit en Rust. Il transforme des fichiers Markdown en un site HTML prêt à déployer — pas de base de données, pas de serveur dynamique, juste des fichiers statiques.

## Pourquoi Zola

- **Un seul binaire** — pas de dépendances, pas de Node.js, pas de npm. On installe, on lance, ça marche.
- **Rapide** — écrit en Rust, le build est quasi instantané même avec beaucoup de pages.
- **Templates Tera** — une syntaxe claire et lisible, proche de Jinja2/Django.
- **Sass intégré** — compilation CSS native, sans pipeline externe.
- **Hot reload** — `zola serve` et chaque modification s'affiche en temps réel dans le navigateur.

## Pourquoi je l'ai choisi

Je voulais un outil simple pour un blog personnel, sans usine à gaz. Zola coche toutes les cases : léger, rapide à prendre en main, et le thème [tabi](https://github.com/welpo/tabi) offre tout ce dont j'ai besoin (dark mode, table des matières, shortcodes, SEO).


Le résultat : un blog statique hébergé sur mon [homelab](@/projects/homelab.md), déployé en quelques secondes.
