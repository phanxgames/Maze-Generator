# Functionality and Features

This document describes all the features and capabilities of the Maze Generator.

## Core Functionality

### 1. Procedural Maze Generation

The generator creates unique mazes each time it runs using a randomized depth-first search algorithm.

**Key Characteristics**:
- **Grid-based Layout**: Mazes are composed of interconnected rooms arranged in a grid
- **Guaranteed Connectivity**: All rooms are accessible from any starting point
- **Random Variation**: Each execution produces a different layout
- **Dead-end Detection**: Automatically identifies rooms with only one exit

**Configuration**:
- Default grid: 12 columns × 9 rows (108 rooms total)
- Room size: 8 tiles wide × 11 tiles tall
- Total map: 100×100 tiles (slightly padded for borders)

### 2. Room Piece System

The generator uses 15 predefined room types, each with multiple visual variations:

#### Room Types

| Type | Doors | Description | Variations |
|------|-------|-------------|------------|
| `horizontal_hallway` | East, West | Horizontal corridor | 6 variations |
| `vertical_hallway` | North, South | Vertical corridor | 6 variations |
| `corner_upper_left` | East, South | Upper-left corner turn | 1 variation |
| `corner_upper_right` | West, South | Upper-right corner turn | 1 variation |
| `corner_bottom_left` | North, East | Bottom-left corner turn | 1 variation |
| `corner_bottom_right` | North, West | Bottom-right corner turn | 1 variation |
| `t_interaction_up` | North, East, West | T-junction facing up | 3 variations |
| `t_interaction_down` | South, East, West | T-junction facing down | 3 variations |
| `t_interaction_left` | North, South, West | T-junction facing left | 3 variations |
| `t_interaction_right` | North, South, East | T-junction facing right | 3 variations |
| `intersection` | All 4 directions | Four-way intersection | 3 variations |
| `end_up` | North only | Dead end facing up | 3 variations |
| `end_down` | South only | Dead end facing down | 3 variations |
| `end_left` | West only | Dead end facing left | 3 variations |
| `end_right` | East only | Dead end facing right | 3 variations |

#### Variation Selection

Each room piece has multiple visual variations with weighted probability:
- Variations have assigned probabilities (e.g., 0.4, 0.3, 0.3)
- Random selection uses these weights for natural distribution
- Prevents repetitive appearance even when same room type is used multiple times

### 3. Difficulty Control: "Wacking" Mode

The generator includes an optional difficulty reduction feature called "wacking":

**Purpose**: Create additional connections between rooms to reduce backtracking

**Configuration**:
```javascript
const WACKING = false;  // Set to true to enable
const WACKING_ITERATIONS = 35;  // Number of additional connections to create
```

**Behavior**:
- Randomly selects pairs of adjacent rooms
- Opens additional doors between them
- Creates loops and alternative paths
- Makes mazes easier to navigate
- Reduces the number of dead ends

**Use Cases**:
- Beginner-friendly levels
- Exploration-focused gameplay
- Reducing player frustration
- Creating more open dungeon layouts

### 4. Visual Enhancement System

#### Shadow Rendering

Automatically adds shadows to enhance depth perception:

**Shadow Placement Rules**:
- Shadows appear below and to the right of walls
- Pillars receive 360-degree shadow treatment
- Walkable floor tiles can receive shadows from adjacent walls
- Creates visual depth without manual placement

**Implementation**:
- Scans all tiles in the map
- Identifies walls and pillars (tiles 91-110)
- Places appropriate shadow tiles (tiles 111-130) at calculated positions
- Handles edge cases and boundaries

#### Floor Decoration System

Adds variety to floor tiles with multiple decoration types:

##### Pebble Clusters
- **Single Pebbles**: Small rocks scattered on floor
- **Double Pebbles**: Pairs of small rocks
- **Triple Pebbles**: Groups of three rocks
- **Placement**: Random distribution across walkable areas

##### Sand Dunes
- **Probability**: 15% chance (SAND_DUNES_CHANCE = 0.15)
- **Appearance**: Tile ID 33
- **Purpose**: Break up monotonous floor patterns

##### Rock Props
- **Type**: Large decorative rocks (tiles 60-63)
- **Distribution**: Placed at regular intervals
- **Spacing**: Controlled by DISTANCE_BETWEEN_PROPS (default: 5 tiles)
- **Minimum Distance**: DISTANCE_BETWEEN_POOPS (default: 7 tiles) from other props
- **Placement**: On props layer, not background

##### Floor Variation
- **Probability**: 27% chance (FLOOR_SHIT_CHANCE = 0.27)
- **Tiles**: Dirt/stone variations (tiles 30-32)
- **Purpose**: Natural-looking floor texture variation

### 5. Three-Layer Architecture

The generator creates separate layers for different visual elements:

#### Layer 1: Background
- **Contents**: Floor tiles, shadows, floor variations, pebbles, sand dunes
- **Purpose**: Base layer that players walk on
- **Tiles Used**:
  - Floor: 50-59, 30-32
  - Shadows: 111-130
  - Pebbles: 80-89
  - Sand: 33

#### Layer 2: Walls
- **Contents**: Structural walls, pillars, barriers
- **Purpose**: Define maze structure and collision boundaries
- **Tiles Used**: 91-110
- **Collision**: These tiles should be marked as non-walkable in game engine

#### Layer 3: Props
- **Contents**: Decorative rocks and large props
- **Purpose**: Visual interest without affecting core maze structure
- **Tiles Used**: 60-63 (large rocks)
- **Note**: May or may not be collidable depending on game design

### 6. Data Compression

Efficient storage of tile data:

**Compression Pipeline**:
1. Store tile IDs as 32-bit little-endian integers
2. Compress with zlib (deflate algorithm)
3. Encode as base64 string
4. Embed in XML structure

**Benefits**:
- Reduces file size significantly
- Standard format for Tiled Map Editor
- Maintains data integrity
- Fast decompression at runtime

### 7. Error Prevention

Built-in safeguards to ensure maze quality:

#### Disconnected Room Repair
After maze generation, the system:
1. Identifies any rooms not connected to the maze
2. Randomly selects an adjacent room that is connected
3. Opens a door between them
4. Repeats until all rooms are accessible

#### Infinite Loop Prevention
- Maximum iteration limit: 5000 steps
- Prevents hanging if algorithm encounters edge case
- Logs warning if limit is reached

#### Boundary Validation
- All room coordinates validated against grid bounds
- Prevents array index errors
- Handles edge rooms correctly

## Output Capabilities

### TMX File Generation

**File Specification**:
- Format: Tiled Map Editor XML (.tmx)
- Version: Tiled 1.x compatible
- Encoding: UTF-8
- Compression: zlib + base64

**File Contents**:
```xml
<map version="1.0" orientation="orthogonal" width="100" height="100" tilewidth="16" tileheight="16">
  <tileset firstgid="1" source="stone_cave.tsx"/>
  <layer name="background" width="100" height="100">
    <!-- Compressed tile data -->
  </layer>
  <layer name="walls" width="100" height="100">
    <!-- Compressed tile data -->
  </layer>
  <layer name="props" width="100" height="100">
    <!-- Compressed tile data -->
  </layer>
</map>
```

**Output Location**: `giraffe.tmx` in the project root directory

## Limitations and Constraints

1. **Fixed Output Filename**: Always generates `giraffe.tmx` (hardcoded)
2. **No CLI Options**: No command-line arguments for customization
3. **Fixed Tileset Reference**: Always references `stone_cave.tsx`
4. **No Configuration File**: All settings must be modified in source code
5. **Single Maze Per Execution**: Generates one maze per run
6. **No Preview Mode**: Must open in Tiled to visualize result

## Extensibility

The system is designed to be extended:

- **Custom Presets**: Add new room types by creating JSON files in `presets/`
- **Custom Tilesets**: Modify `temp.xml` to reference different tilesets
- **Map Size**: Change `width` and `height` constants in `index.js`
- **Grid Size**: Modify `COLS` and `ROWS` constants
- **Decoration Probabilities**: Adjust chance constants for different aesthetics
