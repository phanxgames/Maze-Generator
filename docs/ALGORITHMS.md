# Algorithms and Maze Generation

This document provides detailed explanations of the algorithms used in the Maze Generator.

## Maze Generation Algorithm

### Overview

The generator uses a **Randomized Depth-First Search (DFS) with Backtracking** algorithm, also known as the "Recursive Backtracker" algorithm. This is one of the most popular maze generation algorithms due to its simplicity and ability to create mazes with long, winding corridors.

### Algorithm Characteristics

**Advantages**:
- Generates perfect mazes (exactly one path between any two points)
- Creates long, twisting passages
- High "river" factor (tendency to create long corridors)
- Guaranteed to visit all rooms
- Simple to implement

**Disadvantages**:
- Can create very difficult mazes with long dead ends
- Limited branching compared to other algorithms
- Deterministic pattern if not properly randomized

**Maze Properties**:
- **Perfect Maze**: No loops, no isolated areas
- **Tree Structure**: Mathematically forms a spanning tree
- **Single Solution**: Exactly one path between any two rooms
- **Fully Connected**: All rooms are accessible

### Detailed Algorithm Steps

#### Phase 1: Initialization

**Location**: `index.js` lines 364-398

```javascript
// 1. Create empty room grid
ROOMS = new Array(COLS);
for (let c = 0; c < COLS; c++) {
  ROOMS[c] = new Array(ROWS);
  for (let r = 0; r < ROWS; r++) {
    ROOMS[c][r] = new Room(c, r);
  }
}

// 2. Select random starting position
const startX = getRandomIntInclusive(0, COLS - 1);
const startY = getRandomIntInclusive(0, ROWS - 1);

// 3. Initialize stack with starting room
ROOMS[startX][startY].used = true;
ROOMS[startX][startY].first = true;
STACK.push(ROOMS[startX][startY]);
```

**Purpose**:
- Creates a grid of unvisited rooms
- Randomly selects a starting point for variation
- Marks starting room as visited

#### Phase 2: Depth-First Search with Backtracking

**Location**: `index.js` lines 400-480

```javascript
while (STACK.length > 0 && iteration < 5000) {
  iteration++;

  // Get current room from top of stack (don't pop yet)
  const current = STACK[STACK.length - 1];

  // Randomly shuffle directions to explore
  const directions = shuffleDirections();

  // Try each direction
  let foundValidMove = false;
  for (let i = 0; i < directions.length; i++) {
    const dir = directions[i];

    // Calculate adjacent room coordinates
    let nextX = current.x;
    let nextY = current.y;

    switch(dir) {
      case 1: nextY--; break;  // North
      case 2: nextX++; break;  // East
      case 3: nextY++; break;  // South
      case 4: nextX--; break;  // West
    }

    // Check if move is valid
    if (withinBounds(nextX, nextY)) {
      const next = ROOMS[nextX][nextY];

      if (!next.used) {
        // Valid move found!

        // Open doors between current and next room
        current.openDoor(dir - 1);
        next.openDoor(oppositeDoor(dir - 1));

        // Mark next room as visited
        next.used = true;

        // Add to stack
        STACK.push(next);

        foundValidMove = true;
        break;  // Stop trying other directions
      }
    }
  }

  // If no valid moves, backtrack
  if (!foundValidMove) {
    STACK.pop();
  }
}
```

**Key Concepts**:

1. **Stack-Based Traversal**:
   - Stack keeps track of current path
   - Top of stack = current room
   - Deeper in stack = rooms to backtrack to

2. **Random Direction Selection**:
   - Shuffles [1,2,3,4] array each iteration
   - Prevents predictable patterns
   - Creates natural-looking mazes

3. **Door Opening**:
   - Opens door in current room toward next room
   - Opens opposite door in next room toward current room
   - Creates bidirectional connection

4. **Backtracking**:
   - When stuck (no unvisited neighbors), pop stack
   - Return to previous room
   - Try other branches

**Visual Example**:

```
Initial Grid (all unvisited):
[ ][ ][ ][ ]
[ ][ ][ ][ ]
[ ][ ][ ][ ]

Start at (1,1):
[ ][ ][ ][ ]
[ ][X][ ][ ]
[ ][ ][ ][ ]

Move East:
[ ][ ][ ][ ]
[ ][X][X][ ]
[ ][ ][ ][ ]

Move South:
[ ][ ][ ][ ]
[ ][X][X][ ]
[ ][ ][X][ ]

Dead end! Backtrack to (2,1), try South:
[ ][ ][ ][ ]
[ ][X][X][ ]
[ ][ ][X][ ]
      └─[X]

Continue until all cells visited...
```

#### Phase 3: Error Correction

**Location**: `index.js` lines 482-508

```javascript
// Find and fix any disconnected rooms
for (let c = 0; c < COLS; c++) {
  for (let r = 0; r < ROWS; r++) {
    const room = ROOMS[c][r];

    if (!room.used) {
      // Found a disconnected room!

      // Get all adjacent rooms that ARE connected
      const candidates = [];
      if (c > 0 && ROOMS[c-1][r].used) candidates.push({room: ROOMS[c-1][r], dir: 1});
      if (c < COLS-1 && ROOMS[c+1][r].used) candidates.push({room: ROOMS[c+1][r], dir: 3});
      if (r > 0 && ROOMS[c][r-1].used) candidates.push({room: ROOMS[c][r-1], dir: 0});
      if (r < ROWS-1 && ROOMS[c][r+1].used) candidates.push({room: ROOMS[c][r+1], dir: 2});

      if (candidates.length > 0) {
        // Pick a random adjacent connected room
        const chosen = candidates[getRandomIntInclusive(0, candidates.length - 1)];

        // Connect them
        room.openDoor(chosen.dir);
        chosen.room.openDoor(oppositeDoor(chosen.dir));
        room.used = true;
      }
    }
  }
}
```

**Purpose**:
- Handles rare edge cases where rooms might be unreachable
- Ensures 100% maze connectivity
- Maintains perfect maze property (adds minimal connections)

#### Phase 4: Optional "Wacking" (Loop Creation)

**Location**: `index.js` lines 510-545

```javascript
if (WACKING) {
  for (let i = 0; i < WACKING_ITERATIONS; i++) {
    // Pick a random room
    const rx = getRandomIntInclusive(0, COLS - 1);
    const ry = getRandomIntInclusive(0, ROWS - 1);
    const room = ROOMS[rx][ry];

    // Pick a random direction
    const dir = getRandomIntInclusive(0, 3);

    // Calculate adjacent room
    let nx = rx, ny = ry;
    switch(dir) {
      case 0: ny--; break;  // North
      case 1: nx++; break;  // East
      case 2: ny++; break;  // South
      case 3: nx--; break;  // West
    }

    // If valid and door not already open
    if (withinBounds(nx, ny) && !room.doors[dir]) {
      // Open the door (create a loop)
      room.openDoor(dir);
      ROOMS[nx][ny].openDoor(oppositeDoor(dir));
    }
  }
}
```

**Purpose**:
- Converts perfect maze to imperfect maze
- Creates loops and alternative paths
- Reduces difficulty by adding shortcuts
- Controlled by WACKING_ITERATIONS (default: 35 connections)

**Effect on Maze Properties**:
- **Before**: Perfect maze, tree structure, unique paths
- **After**: Imperfect maze, graph structure, multiple paths
- **Dead ends**: Reduced but not eliminated
- **Difficulty**: Easier to navigate

#### Phase 5: Room Piece Assignment

**Location**: `index.js` lines 547-580

```javascript
// For each room in the grid
for (let c = 0; c < COLS; c++) {
  for (let r = 0; r < ROWS; r++) {
    const room = ROOMS[c][r];

    // 1. Determine if it's a dead end
    room.fixDeadEnd();

    // 2. Select appropriate room piece based on doors
    room.fixPiece();

    // 3. Decompress the selected piece variation
    const decompressedPiece = FromBuffer(
      room.piece.map.layers[1].data._,  // Wall layer from piece
      ROOM_WIDTH,
      ROOM_HEIGHT
    );

    // 4. Calculate tile position in main map
    const tileX = c * ROOM_WIDTH;
    const tileY = r * ROOM_HEIGHT;

    // 5. Inject piece into main walls layer
    InjectMap(walls, decompressedPiece, tileX, tileY, ROOM_WIDTH, ROOM_HEIGHT);
  }
}
```

**Room Piece Selection Logic**:

The `figureOutPiece()` function maps door configurations to piece types:

```
Door Pattern          → Room Piece Type
─────────────────────────────────────────
[T, T, T, T]         → intersection
[T, T, T, F]         → t_interaction_down
[T, T, F, T]         → t_interaction_up
[T, T, F, F]         → corner_bottom_left
[T, F, T, T]         → t_interaction_left
[T, F, T, F]         → vertical_hallway
[T, F, F, T]         → corner_bottom_right
[T, F, F, F]         → end_up
[F, T, T, T]         → t_interaction_right
[F, T, T, F]         → corner_upper_left
[F, T, F, T]         → horizontal_hallway
[F, T, F, F]         → end_right
[F, F, T, T]         → corner_upper_right
[F, F, T, F]         → end_down
[F, F, F, T]         → end_left

T = true (door open)
F = false (door closed)
Array indices: [North, East, South, West]
```

### Helper Algorithms

#### Direction Shuffling

**Location**: `index.js` lines 430-448

```javascript
function shuffleDirections() {
  const arr = [1, 2, 3, 4];  // N, E, S, W

  // Fisher-Yates shuffle
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];  // Swap
  }

  return arr;
}
```

**Purpose**: Ensures random, unbiased direction exploration

**Algorithm**: Fisher-Yates shuffle (O(n) time, O(1) space)

#### Opposite Door Calculation

**Location**: `index.js` lines 355-362

```javascript
function oppositeDoor(index) {
  switch (index) {
    case 0: return 2;  // North ↔ South
    case 1: return 3;  // East ↔ West
    case 2: return 0;  // South ↔ North
    case 3: return 1;  // West ↔ East
  }
}
```

**Purpose**: Finds the door index from the adjacent room's perspective

**Example**:
- Room A opens East door (index 1)
- Room B (to the east) must open West door (index 3)
- oppositeDoor(1) = 3

## Shadow Placement Algorithm

### Overview

The shadow algorithm enhances visual depth by placing shadow tiles around walls and pillars.

### Algorithm

**Location**: `index.js` lines 584-726

```javascript
for (let y = 0; y < height; y++) {
  for (let x = 0; x < width; x++) {
    const index = y * width + x;
    const tileId = background[index];

    // Check if this is a walkable floor tile
    if (tileId >= 50 && tileId <= 59) {

      // Check tile to the left (west)
      if (x > 0) {
        const leftIndex = y * width + (x - 1);
        const leftTile = walls[leftIndex];

        // If left tile is a wall, place shadow
        if (leftTile >= 91 && leftTile <= 110) {
          const shadowId = leftTile + 20;  // 91->111, 92->112, etc.
          background[index] = shadowId;
          continue;
        }
      }

      // Check tile above (north)
      if (y > 0) {
        const aboveIndex = (y - 1) * width + x;
        const aboveTile = walls[aboveIndex];

        if (aboveTile >= 91 && aboveTile <= 110) {
          const shadowId = aboveTile + 20;
          background[index] = shadowId;
          continue;
        }
      }

      // Similar checks for other directions...
    }
  }
}
```

**Shadow Rules**:
1. Only place shadows on walkable floor tiles
2. Check adjacent tiles for walls/pillars
3. Map wall tile ID to shadow tile ID (+20 offset)
4. Priority: West > North > Other directions
5. One shadow per tile (first match wins)

**Special Case: Pillars**:
- Pillars are standalone wall tiles
- Receive shadows on all sides
- Create realistic 3D depth effect

## Prop Distribution Algorithm

### Overview

Distributes decorative rocks across the floor at regular intervals.

### Algorithm

**Location**: `index.js` lines 730-748

```javascript
for (let y = 0; y < height; y += DISTANCE_BETWEEN_PROPS) {
  for (let x = 0; x < width; x += DISTANCE_BETWEEN_PROPS) {
    const index = y * width + x;
    const tileId = background[index];

    // Check if this is a walkable floor
    if (tileId >= 50 && tileId <= 59) {

      // Check if too close to other props
      let tooClose = false;
      for (let checkY = y - DISTANCE_BETWEEN_POOPS; checkY <= y + DISTANCE_BETWEEN_POOPS; checkY++) {
        for (let checkX = x - DISTANCE_BETWEEN_POOPS; checkX <= x + DISTANCE_BETWEEN_POOPS; checkX++) {
          if (withinBounds(checkX, checkY)) {
            const checkIndex = checkY * width + checkX;
            if (props[checkIndex] >= 60 && props[checkIndex] <= 63) {
              tooClose = true;
              break;
            }
          }
        }
        if (tooClose) break;
      }

      // If clear, place a random rock
      if (!tooClose) {
        const rockId = getRandomIntInclusive(60, 63);
        props[index] = rockId;
      }
    }
  }
}
```

**Distribution Rules**:
1. Grid-based placement (every 5 tiles by default)
2. Only on walkable floor tiles
3. Minimum distance between props (7 tiles)
4. Random rock variation (tiles 60-63)
5. Placed on props layer (not background)

**Purpose**:
- Creates visual interest
- Breaks up monotonous floor patterns
- Provides landmark references for navigation
- Maintains consistent spacing for clean appearance

## Compression Algorithm

### Tile Data Compression

**Location**: `xml.js` lines 15-40

```javascript
function compressTileData(tiles) {
  // 1. Allocate buffer (4 bytes per tile)
  const buffer = Buffer.alloc(tiles.length * 4);

  // 2. Write each tile as 32-bit little-endian integer
  tiles.forEach((tile, i) => {
    buffer.writeUInt32LE(tile, i * 4);
  });

  // 3. Compress with zlib DEFLATE
  const compressed = zlib.deflateSync(buffer);

  // 4. Encode as base64 string
  const encoded = compressed.toString('base64');

  return encoded;
}
```

**Decompression** (reverse process in `helpers.js`):

```javascript
function decompressTileData(base64String) {
  // 1. Decode base64 to buffer
  const buffer = Buffer.from(base64String, 'base64');

  // 2. Decompress with zlib INFLATE
  const decompressed = zlib.inflateSync(buffer);

  // 3. Read 32-bit integers
  const tiles = [];
  for (let i = 0; i < decompressed.length; i += 4) {
    tiles.push(decompressed.readUInt32LE(i));
  }

  return tiles;
}
```

**Compression Ratio**: Typically achieves 70-85% size reduction

## Random Number Generation

### getRandomIntInclusive(min, max)

**Location**: `helpers.js` lines 61-65

```javascript
export function getRandomIntInclusive(min, max) {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(Math.random() * (max - min + 1) + min);
}
```

**Properties**:
- Inclusive range [min, max]
- Uniform distribution
- Based on Math.random() (pseudo-random)

**Used For**:
- Starting position selection
- Direction shuffling
- Room piece variation selection
- Prop type selection
- Wacking target selection

## Complexity Analysis

### Time Complexity

| Operation | Complexity | Reasoning |
|-----------|------------|-----------|
| Maze Generation | O(C × R) | Visits each room once |
| Direction Shuffle | O(1) | Fixed 4-element array |
| Error Correction | O(C × R) | Single pass over all rooms |
| Wacking | O(W) | W = WACKING_ITERATIONS |
| Piece Assignment | O(C × R) | Single pass over all rooms |
| Shadow Placement | O(w × h) | Scans all tiles once |
| Prop Distribution | O((w/s) × (h/s) × d²) | s=spacing, d=distance check |
| Compression | O(w × h) | Linear in tile count |
| **Total** | **O(w × h)** | Dominated by tile operations |

Where:
- C = COLS (room columns)
- R = ROWS (room rows)
- w = width (tile width)
- h = height (tile height)
- W = WACKING_ITERATIONS
- s = DISTANCE_BETWEEN_PROPS
- d = DISTANCE_BETWEEN_POOPS

### Space Complexity

| Data Structure | Size | Notes |
|----------------|------|-------|
| ROOMS | O(C × R) | ~108 Room objects |
| STACK | O(C × R) worst case | Max depth = total rooms |
| background | O(w × h) | 10,000 integers |
| walls | O(w × h) | 10,000 integers |
| props | O(w × h) | 10,000 integers |
| **Total** | **O(w × h)** | Linear in map size |

## Algorithm Optimizations

### Current Optimizations

1. **Early Termination**: Stops direction search on first valid move
2. **Boundary Checking**: Validates coordinates before array access
3. **Single-Pass Operations**: Most algorithms scan data only once
4. **In-Place Modifications**: Updates arrays directly, no copying

### Potential Improvements

1. **Union-Find**: Could detect disconnected components faster
2. **Bitmasking**: Store door states as bit flags (4 bools → 1 byte)
3. **Spatial Hashing**: Faster prop proximity checks
4. **Streaming Compression**: Compress data as generated, not all at once
5. **Worker Threads**: Parallelize shadow/prop placement for large maps
