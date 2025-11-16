# Business Requirements and Rules

This document outlines all the business requirements, constraints, and rules that govern the Maze Generator's behavior. Use this document to reproduce or modify the system while maintaining its intended functionality.

## Functional Requirements

### FR-1: Maze Generation

**Requirement**: The system SHALL generate a randomized, fully-connected maze on each execution.

**Rules**:
- FR-1.1: All rooms in the maze MUST be accessible from any starting point
- FR-1.2: Each execution MUST produce a different maze layout
- FR-1.3: The maze MUST be composed of discrete rooms arranged in a grid
- FR-1.4: Rooms MUST connect via open doors in cardinal directions (N, E, S, W)
- FR-1.5: The default maze MUST contain 12 columns × 9 rows of rooms (108 total)

**Rationale**: Ensures playable, varied game levels without isolated areas.

### FR-2: Room Structure

**Requirement**: The system SHALL construct mazes from predefined room pieces.

**Rules**:
- FR-2.1: Each room MUST be 8 tiles wide × 11 tiles tall
- FR-2.2: Rooms MUST support 0-4 open doors
- FR-2.3: Room pieces MUST match the door configuration of their assigned room
- FR-2.4: The system MUST support 15 room piece types:
  - 1 four-way intersection
  - 4 T-junctions (up, down, left, right)
  - 2 straight hallways (horizontal, vertical)
  - 4 corners (upper-left, upper-right, bottom-left, bottom-right)
  - 4 dead ends (up, down, left, right)
- FR-2.5: Each piece type MUST have at least 1 visual variation
- FR-2.6: Variation selection MUST be weighted by assigned probabilities

**Rationale**: Provides structural variety while maintaining consistent maze topology.

### FR-3: Visual Output

**Requirement**: The system SHALL generate a Tiled Map Editor compatible TMX file.

**Rules**:
- FR-3.1: Output MUST be in TMX (Tiled Map Editor XML) format
- FR-3.2: Map dimensions MUST be 100 tiles × 100 tiles
- FR-3.3: Tile dimensions MUST be 16 pixels × 16 pixels
- FR-3.4: Map MUST contain exactly 3 layers:
  - Layer 1: "background" (floors, shadows, decorations)
  - Layer 2: "walls" (structural elements)
  - Layer 3: "props" (decorative objects)
- FR-3.5: Tile data MUST be compressed using zlib DEFLATE
- FR-3.6: Compressed data MUST be encoded as base64
- FR-3.7: Each tile MUST be stored as a 32-bit little-endian unsigned integer
- FR-3.8: Output file MUST be named `giraffe.tmx`
- FR-3.9: Output MUST reference tileset file `stone_cave.tsx`

**Rationale**: Ensures compatibility with Tiled Map Editor and game engines.

### FR-4: Shadow Rendering

**Requirement**: The system SHALL automatically place shadow tiles to enhance visual depth.

**Rules**:
- FR-4.1: Shadows MUST only appear on walkable floor tiles
- FR-4.2: Shadows MUST be placed adjacent to wall tiles
- FR-4.3: Shadow tile ID MUST equal wall tile ID + 20
- FR-4.4: Each floor tile MUST have at most one shadow
- FR-4.5: Pillars (standalone walls) MUST receive shadows on all exposed sides
- FR-4.6: Shadow placement MUST check west, then north, then other directions

**Rationale**: Creates depth perception without manual artist intervention.

### FR-5: Floor Decoration

**Requirement**: The system SHALL add decorative elements to enhance visual variety.

**Rules**:
- FR-5.1: Floor decorations MUST only appear on walkable tiles
- FR-5.2: System MUST support the following decoration types:
  - Sand dunes (tile 33)
  - Floor variations (tiles 30-32)
  - Pebble clusters (tiles 80-89)
  - Rock props (tiles 60-63)
- FR-5.3: Sand dunes MUST have a 15% placement probability
- FR-5.4: Floor variations MUST have a 27% placement probability
- FR-5.5: Rock props MUST be spaced at least 5 tiles apart (grid-based)
- FR-5.6: Rock props MUST maintain minimum 7-tile distance from other props
- FR-5.7: Rock props MUST be placed on the props layer
- FR-5.8: Other decorations MUST be placed on the background layer

**Rationale**: Creates visual interest while maintaining performance and aesthetics.

### FR-6: Difficulty Control

**Requirement**: The system SHALL support optional difficulty reduction via "wacking."

**Rules**:
- FR-6.1: Wacking MUST be disabled by default (WACKING = false)
- FR-6.2: When enabled, wacking MUST create 35 additional connections
- FR-6.3: Additional connections MUST be placed randomly
- FR-6.4: System MUST NOT open doors that are already open
- FR-6.5: Wacking MUST execute after initial maze generation
- FR-6.6: Wacking MUST preserve maze connectivity (not break existing paths)

**Rationale**: Provides difficulty tuning for different player skill levels.

## Non-Functional Requirements

### NFR-1: Performance

**Requirements**:
- NFR-1.1: Maze generation MUST complete in less than 1 second on modern hardware
- NFR-1.2: System MUST handle infinite loop scenarios (5000 iteration limit)
- NFR-1.3: Memory usage MUST remain under 100MB during execution
- NFR-1.4: File output MUST complete within 2 seconds

**Rationale**: Ensures responsive generation for rapid iteration during game development.

### NFR-2: Determinism and Randomness

**Requirements**:
- NFR-2.1: System MUST use pseudo-random number generation (Math.random())
- NFR-2.2: Each execution MUST produce different output (no fixed seed)
- NFR-2.3: Direction exploration MUST be randomized (Fisher-Yates shuffle)
- NFR-2.4: Starting position MUST be randomly selected
- NFR-2.5: Room piece variations MUST be selected using weighted random choice

**Rationale**: Balances variability with reproducibility for debugging.

### NFR-3: Data Integrity

**Requirements**:
- NFR-3.1: System MUST validate all array indices before access
- NFR-3.2: System MUST repair disconnected rooms automatically
- NFR-3.3: System MUST NOT generate mazes with isolated areas
- NFR-3.4: Tile IDs MUST be valid integers within tileset range
- NFR-3.5: Compressed data MUST be reversibly decompressible

**Rationale**: Prevents runtime errors and ensures valid output.

### NFR-4: Maintainability

**Requirements**:
- NFR-4.1: System MUST use ES6 module syntax
- NFR-4.2: Code MUST separate concerns (maze logic, helpers, pieces, XML)
- NFR-4.3: Constants MUST be defined at module level
- NFR-4.4: Magic numbers SHOULD be replaced with named constants (aspiration)

**Rationale**: Facilitates code understanding and modification.

## Business Rules

### BR-1: Maze Topology Rules

**Rules**:
- BR-1.1: A maze is **valid** if and only if all rooms are reachable from any starting room
- BR-1.2: A maze is **perfect** if there is exactly one path between any two rooms (no loops)
- BR-1.3: By default, generated mazes MUST be perfect
- BR-1.4: When wacking is enabled, mazes MAY be imperfect
- BR-1.5: Dead ends MUST be preserved even when wacking is enabled

**Verification**:
```javascript
// A room is a dead end if exactly one door is open
const deadEndCount = room.doors.filter(d => d === true).length;
const isDeadEnd = (deadEndCount === 1);
```

### BR-2: Room Piece Selection Rules

**Rules**:
- BR-2.1: Room pieces MUST be selected based on door configuration
- BR-2.2: Door configuration is a 4-element boolean array: [North, East, South, West]
- BR-2.3: There are 15 valid door configurations (excluding all-false)
- BR-2.4: Each configuration maps to exactly one piece type
- BR-2.5: Piece variations are selected using probability-weighted random choice

**Mapping**:
```
Door Pattern [N, E, S, W] → Piece Type
─────────────────────────────────────
[1, 1, 1, 1] → intersection
[1, 1, 1, 0] → t_interaction_down
[1, 1, 0, 1] → t_interaction_up
[1, 1, 0, 0] → corner_bottom_left
[1, 0, 1, 1] → t_interaction_left
[1, 0, 1, 0] → vertical_hallway
[1, 0, 0, 1] → corner_bottom_right
[1, 0, 0, 0] → end_up
[0, 1, 1, 1] → t_interaction_right
[0, 1, 1, 0] → corner_upper_left
[0, 1, 0, 1] → horizontal_hallway
[0, 1, 0, 0] → end_right
[0, 0, 1, 1] → corner_upper_right
[0, 0, 1, 0] → end_down
[0, 0, 0, 1] → end_left

(1 = true, 0 = false)
```

### BR-3: Tile Assignment Rules

**Rules**:
- BR-3.1: Tiles 50-59 represent walkable floor
- BR-3.2: Tiles 91-110 represent walls and pillars
- BR-3.3: Tiles 111-130 represent shadows (offset +20 from walls)
- BR-3.4: Tiles 30-33 represent floor variations and sand
- BR-3.5: Tiles 80-89 represent pebble decorations
- BR-3.6: Tiles 60-63 represent large rock props
- BR-3.7: Tile 0 represents empty/transparent
- BR-3.8: When injecting pieces, tile 0 MUST NOT overwrite existing tiles

**Overwrite Rule**:
```javascript
// Only overwrite if source tile is non-zero
if (sourceTile !== 0) {
  targetMap[targetIndex] = sourceTile;
}
```

### BR-4: Coordinate System Rules

**Rules**:
- BR-4.1: Room coordinates use (column, row) indexing, 0-based
- BR-4.2: Tile coordinates use (x, y) indexing, 0-based
- BR-4.3: Conversion: tileX = roomColumn × ROOM_WIDTH
- BR-4.4: Conversion: tileY = roomRow × ROOM_HEIGHT
- BR-4.5: 1D array index = y × width + x
- BR-4.6: Origin (0, 0) is top-left corner
- BR-4.7: X-axis increases rightward (east)
- BR-4.8: Y-axis increases downward (south)

**Direction Encoding**:
```javascript
const DIRECTION = {
  NORTH: 0,  // y - 1
  EAST:  1,  // x + 1
  SOUTH: 2,  // y + 1
  WEST:  3   // x - 1
};
```

### BR-5: Layer Assignment Rules

**Rules**:
- BR-5.1: Background layer MUST contain: floors, shadows, floor variations, pebbles, sand
- BR-5.2: Walls layer MUST contain: structural walls from room pieces
- BR-5.3: Props layer MUST contain: large decorative rocks
- BR-5.4: Layers MUST be rendered in order: background → walls → props
- BR-5.5: Wall layer defines collision boundaries
- BR-5.6: Background layer is always walkable
- BR-5.7: Props layer MAY define collision (game engine decision)

### BR-6: Compression Rules

**Rules**:
- BR-6.1: Each tile MUST be stored as 32-bit unsigned integer
- BR-6.2: Integers MUST use little-endian byte order
- BR-6.3: Tile array MUST be compressed with zlib DEFLATE
- BR-6.4: Compressed data MUST be encoded as base64 ASCII
- BR-6.5: Decompression MUST be lossless (exact original data)

**Encoding Process**:
```
Tile ID → 32-bit LE Integer → Buffer → zlib Compress → base64 Encode → String
   50   →   [50,0,0,0]      → Buffer → compressed   →   "eJw..."  → Output
```

## Configuration Constraints

### CC-1: Map Size Constraints

**Hard Limits**:
- Minimum map size: 10×10 tiles (too small for rooms)
- Maximum map size: 1000×1000 tiles (memory/performance limit)
- Default map size: 100×100 tiles (**REQUIRED**)

**Room Grid Constraints**:
- Minimum grid: 2×2 rooms (too small for interesting mazes)
- Maximum grid: Calculated by (mapWidth / ROOM_WIDTH) × (mapHeight / ROOM_HEIGHT)
- Default grid: 12×9 rooms (**REQUIRED**)

**Derived Constraint**:
```
COLS × ROOM_WIDTH ≤ width
ROWS × ROOM_HEIGHT ≤ height

Default: 12 × 8 = 96 ≤ 100 ✓
         9 × 11 = 99 ≤ 100 ✓
```

### CC-2: Probability Constraints

**Valid Ranges**:
- All probabilities: 0.0 ≤ p ≤ 1.0
- SAND_DUNES_CHANCE: Default 0.15 (15%)
- FLOOR_SHIT_CHANCE: Default 0.27 (27%)

**Variation Probabilities**:
- Sum of all variations for a piece: SHOULD equal 1.0 (not enforced)
- Individual variation probability: 0.0 < p ≤ 1.0
- Selection algorithm: Weighted random choice

### CC-3: Spacing Constraints

**Distance Rules**:
- DISTANCE_BETWEEN_PROPS: Minimum 1, Default 5, Maximum width/2
- DISTANCE_BETWEEN_POOPS: Minimum 1, Default 7, Maximum width/2
- Constraint: DISTANCE_BETWEEN_POOPS > DISTANCE_BETWEEN_PROPS (recommended)

**Rationale**: Prevents prop clustering and maintains visual balance.

### CC-4: Iteration Constraints

**Safety Limits**:
- Maximum maze generation iterations: 5000 (hardcoded)
- Expected iterations: Approximately COLS × ROWS × 2
- WACKING_ITERATIONS: Default 35, Range [0, COLS × ROWS × 4]

**Behavior on Limit**:
- System logs warning if limit reached
- Continues with partial maze (may have disconnected rooms)
- Error correction attempts to fix connectivity

## Validation Rules

### VR-1: Pre-Generation Validation

**Checks**:
1. COLS > 0 AND ROWS > 0
2. COLS × ROOM_WIDTH ≤ width
3. ROWS × ROOM_HEIGHT ≤ height
4. All room piece presets loaded successfully
5. Template XML file exists and is valid

**Action on Failure**: System MUST terminate with error message

### VR-2: Post-Generation Validation

**Checks**:
1. All rooms marked as used (connected)
2. All rooms have at least one open door
3. All rooms have assigned piece variations
4. No array index out of bounds errors

**Action on Failure**: Attempt error correction, warn user

### VR-3: Output Validation

**Checks**:
1. All tile IDs are non-negative integers
2. Compressed data can be decompressed
3. Decompressed data matches original tile count
4. XML structure is well-formed

**Action on Failure**: System MUST NOT write invalid file

## Reproduction Checklist

To reproduce this system from scratch, ensure:

### Essential Requirements

- ✓ Use randomized depth-first search with backtracking
- ✓ Generate 12×9 room grid (108 rooms)
- ✓ Each room is 8×11 tiles
- ✓ Final map is 100×100 tiles
- ✓ Support 15 room piece types with variations
- ✓ Create 3 layers: background, walls, props
- ✓ Automatically place shadows next to walls
- ✓ Add floor decorations (sand, pebbles, variations)
- ✓ Distribute rock props with minimum spacing
- ✓ Compress tile data with zlib + base64
- ✓ Output TMX format compatible with Tiled
- ✓ Ensure all rooms are fully connected
- ✓ Support optional "wacking" for easier mazes

### Configuration Values

- ✓ Map size: 100×100 tiles
- ✓ Tile size: 16×16 pixels
- ✓ Room grid: 12 columns × 9 rows
- ✓ Room size: 8×11 tiles
- ✓ Sand dunes chance: 15%
- ✓ Floor variation chance: 27%
- ✓ Prop spacing: 5 tiles
- ✓ Prop minimum distance: 7 tiles
- ✓ Wacking iterations: 35
- ✓ Max iterations: 5000
- ✓ Output filename: giraffe.tmx
- ✓ Tileset reference: stone_cave.tsx

### Algorithm Requirements

- ✓ Shuffle directions randomly each iteration
- ✓ Use stack for backtracking (not recursion)
- ✓ Fix disconnected rooms after generation
- ✓ Mark dead ends correctly
- ✓ Select room pieces based on door configuration
- ✓ Use weighted random choice for variations
- ✓ Map wall tiles to shadow tiles (+20 offset)
- ✓ Check prop proximity before placement
- ✓ Only overwrite non-zero tiles when injecting

### Data Format Requirements

- ✓ Use 32-bit little-endian integers for tiles
- ✓ Compress with zlib DEFLATE algorithm
- ✓ Encode compressed data as base64
- ✓ Store layers in correct TMX structure
- ✓ Reference external tileset file
- ✓ Use orthogonal map orientation

This checklist ensures all business requirements and rules are implemented correctly.
