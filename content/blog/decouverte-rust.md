+++
title = "Découverte de Rust"
date = 2026-03-15
description = "Mes premières impressions en apprenant Rust."
[taxonomies]
tags = ["rust", "programmation", "apprentissage"]
+++

Rust est un langage fascinant. Le borrow checker est déroutant au début, mais une fois qu'on comprend le concept d'ownership, tout devient logique.

## Ce que j'ai aimé

- La gestion mémoire sans garbage collector
- Le système de types puissant
- Les messages d'erreur du compilateur, incroyablement clairs

## Premier programme

```rust
fn main() {
    let message = String::from("Bonjour le monde !");
    println!("{}", message);
}
```

La courbe d'apprentissage est raide mais ça vaut le coup. Prochaine étape : les lifetimes.
