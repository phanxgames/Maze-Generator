# Maze Generator Documentation

## Overview

The **Maze Generator** is a procedural maze generation tool designed to create randomized dungeon and cave-style mazes for tile-based games. It generates complex maze layouts and exports them as TMX (Tiled Map Editor) XML files, ready for integration into game development workflows.

## Purpose

This tool was created to automate the generation of maze-based game levels with the following objectives:

- **Procedural Level Design**: Generate unique, randomized maze layouts on demand
- **Game Integration**: Output compatible with Tiled Map Editor for seamless game engine integration
- **Visual Variety**: Create visually diverse mazes with multiple room configurations and decorative elements
- **Playability**: Ensure all generated mazes are fully connected and explorable
- **Dungeon Aesthetics**: Produce cave/dungeon-themed environments suitable for roguelike and adventure games

## Key Features

- **Randomized Maze Generation**: Uses depth-first search algorithm with backtracking
- **Configurable Layout**: Default 12x9 room grid, each room 8x11 tiles
- **15 Room Types**: Corners, hallways, intersections, dead ends, and T-junctions
- **Visual Variations**: Multiple visual variations for each room type
- **Decorative Elements**: Automatic placement of shadows, pebbles, sand dunes, and rock props
- **Three-Layer System**: Separate layers for background, walls, and props
- **Compressed Output**: Efficient zlib compression with base64 encoding
- **Difficulty Control**: Optional "wacking" mode to create loops and reduce difficulty

## Output Format

The generator produces `.tmx` files compatible with [Tiled Map Editor](https://www.mapeditor.org/) containing:

- **Map Size**: 100x100 tiles (1600x1600 pixels)
- **Tile Size**: 16x16 pixels
- **Tileset**: References `stone_cave.tsx` for cave/dungeon themed graphics
- **Three Layers**:
  1. **Background**: Floor tiles with shadows
  2. **Walls**: Structural wall and pillar tiles
  3. **Props**: Decorative elements (rocks, pebbles, sand)

## Target Use Cases

- Roguelike game development
- Dungeon crawler level generation
- Cave exploration games
- Procedural content generation for 2D tile-based games
- Game prototyping and level design testing

## Documentation Structure

This documentation is organized into the following sections:

- **[SETUP.md](./SETUP.md)**: Installation and usage instructions
- **[FUNCTIONALITY.md](./FUNCTIONALITY.md)**: Detailed feature descriptions
- **[IMPLEMENTATION.md](./IMPLEMENTATION.md)**: Technical architecture and code structure
- **[ALGORITHMS.md](./ALGORITHMS.md)**: Maze generation algorithm details
- **[BUSINESS_REQUIREMENTS.md](./BUSINESS_REQUIREMENTS.md)**: Rules, constraints, and specifications

## Quick Start

```bash
# Install dependencies
npm install

# Generate a maze
node index.js
```

This will create a file named `giraffe.tmx` in the current directory.

## Project Status

This is a functional prototype suitable for game development. The codebase is production-ready but could benefit from:

- Configuration options (map size, room count, output filename)
- Command-line interface
- Error handling improvements
- Documentation for custom presets
- Unit tests
