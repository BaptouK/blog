+++
title = "Pourquoi j'ai choisi Zola plutôt que Hugo"
date = 2026-04-05
description = "Comparaison entre Zola et Hugo pour un blog statique."
[taxonomies]
tags = ["zola", "web", "programmation"]
+++

Quand j'ai voulu créer ce blog, j'ai hésité entre Hugo et Zola. Voici pourquoi j'ai choisi Zola.

## Hugo : le poids lourd

Hugo est le générateur statique le plus populaire. Énorme communauté, des tonnes de thèmes, une doc complète. Mais...

- La syntaxe des templates Go est horrible
- La configuration est un labyrinthe
- Trop de fonctionnalités dont je n'ai pas besoin

## Zola : la simplicité

Zola est écrit en Rust (un seul binaire, pas de dépendances). Ce qui m'a convaincu :

1. **Templates Tera** — syntaxe claire, proche de Jinja2
2. **Un seul binaire** — `zola serve` et c'est parti
3. **Sass intégré** — pas besoin de pipeline CSS externe
4. **Hot reload** — modifications en temps réel

## Verdict

Pour un blog personnel, Zola est parfait. Simple, rapide, et le thème tabi est superbe.

```bash
# Installer Zola
cargo install zola

# Créer un site
zola init mon-blog
cd mon-blog
zola serve
```

Pas de regrets.
