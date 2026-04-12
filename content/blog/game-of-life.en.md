+++
title = "Game of Life"
date = 2026-04-11
description = "Implementing Conway's Game of Life in C++ with the SFML library."
[taxonomies]
tags = ["cpp", "sfml", "simulation"]
[extra]
toc = true
+++

Conway's Game of Life is one of those projects every developer encounters sooner or later. Invented in 1970 by British mathematician John Horton Conway, it isn't really a game — there are no players, no objectives, no winners. It's a **cellular automaton**: a system that evolves on its own, governed by a few rules of disarming simplicity.

Here is the link to my project’s repo : [https://github.com/BaptouK/game_of_life](https://github.com/BaptouK/game_of_life)


## How it works

The world is a (theoretically infinite) grid of cells, each either alive or dead. At every generation, each cell's state is recalculated based on its eight immediate neighbours, following four rules:

1. A live cell with **fewer than 2 live neighbours** dies (underpopulation).
2. A live cell with **2 or 3 live neighbours** survives.
3. A live cell with **more than 3 live neighbours** dies (overpopulation).
4. A dead cell with **exactly 3 live neighbours** comes alive (reproduction).

That's it. Four rules — and yet they're enough to produce structures of unexpected richness: oscillators, spaceships that travel across the grid, guns that fire gliders indefinitely...

## The project: C++ and SFML        

I chose to implement the Game of Life in **C++** with the **SFML** library (Simple and Fast Multimedia Library) for a couple of reasons.

First, I wanted to sharpen my C++ fundamentals — memory management, data structures, organising code into classes. The Game of Life is an ideal playground: the problem is well-defined, the logic is clear, and the challenge lies in the implementation rather than in complex business rules.

Second, I'd been wanting to try SFML for a while. It's a multimedia library built for C++, giving access to a graphical window, event handling, and 2D rendering — all through a clean, approachable API. A good way to get into graphics programming without diving straight into raw OpenGL.

## What I learned

Implementing this automaton raised a few concrete questions:

- **Double buffering**: the grid must be updated in a single pass. If you modify cells as you go, already-updated cells corrupt the neighbours calculated afterwards. The solution: two alternating grids — one holding the current state, one for the next generation.

- **Efficient rendering with SFML**: drawing each cell individually with `sf::RectangleShape` gets expensive fast on a large grid. Using a `sf::VertexArray` to batch all quads into a single draw call makes a noticeable difference.

- **Real-time interaction**: pause, speed control, drawing with the mouse — handling SFML events alongside the simulation requires a well-structured game loop.

## Result

A window opens, a grid appears, and cells are born and die with each tick. You can draw with the mouse, pause, speed up or slow down. Nothing spectacular visually — but seeing a **glider** cross the screen for the first time, knowing you wrote the four rules that make it move, is genuinely satisfying.

The Game of Life endures as a classic for good reason: it shows how complexity can emerge from simplicity. And as a project, it delivers a complete loop — rendering, logic, interaction — exactly what you need to build real skills.
