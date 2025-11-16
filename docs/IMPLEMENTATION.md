# Implementation Details and Architecture

This document provides technical details about the codebase structure, architecture, and implementation.

## Project Structure

```
Maze-Generator/
├── index.js              # Main entry point and maze generation logic (777 lines)
├── helpers.js            # Utility functions for data manipulation (66 lines)
├── pieces.js             # Maze piece/tile definitions (86 lines)
├── xml.js                # XML file generation and manipulation (69 lines)
├── temp.xml              # Template XML file for Tiled maps
├── package.json          # NPM dependencies and configuration
├── package-lock.json     # Locked dependency versions
├── .gitignore            # Git ignore rules
├── docs/                 # Documentation (this folder)
└── presets/              # 15 JSON preset files for maze pieces
    ├── corner_bottom_left.json
    ├── corner_bottom_right.json
    ├── corner_upper_left.json
    ├── corner_upper_right.json
    ├── end_down.json
    ├── end_left.json
    ├── end_right.json
    ├── end_up.json
    ├── horizontal_hallway.json
    ├── intersection.json
    ├── t_interaction_down.json
    ├── t_interaction_left.json
    ├── t_interaction_right.json
    ├── t_interaction_up.json
    └── veritcal_hallway.json (note: typo in filename)
```

## Module Architecture

### Module System

The project uses **ES6 modules** (ESM):
```javascript
// package.json
{
  "type": "module"
}
```

All files use `import`/`export` syntax instead of `require()`/`module.exports`.

### Module Dependencies

```
index.js (main)
  ├── imports: helpers.js (FromBuffer, FillMap, InjectMap, getRandomIntInclusive)
  ├── imports: pieces.js (all 15 room piece definitions)
  ├── imports: xml.js (InjectIntoXML)
  ├── imports: fs (filesystem operations)
  └── imports: zlib (compression)

helpers.js
  └── imports: zlib (decompression)

xml.js
  ├── imports: fs (read/write files)
  └── imports: xml2js (XML parsing and building)

pieces.js
  └── exports: room piece constants (no imports)
```

## Core Components

### 1. index.js - Main Application

**Location**: `/home/user/Maze-Generator/index.js`

#### Constants and Configuration

```javascript
// Map dimensions
const width = 100;
const height = 100;

// Room grid size
const COLS = 12;  // Number of room columns
const ROWS = 9;   // Number of room rows

// Tile dimensions (in tiles, not pixels)
const ROOM_WIDTH = 8;
const ROOM_HEIGHT = 11;

// Difficulty settings
const WACKING = false;
const WACKING_ITERATIONS = 35;

// Visual settings
const SAND_DUNES_CHANCE = 0.15;
const FLOOR_SHIT_CHANCE = 0.27;
const DISTANCE_BETWEEN_PROPS = 5;
const DISTANCE_BETWEEN_POOPS = 7;
```

#### Global Data Structures

```javascript
// Three-layer map storage (100x100 tiles each)
let background = [];  // Layer 1: floors, shadows, decorations
let walls = [];       // Layer 2: structural walls
let props = [];       // Layer 3: decorative props

// Maze generation state
let STACK = [];       // DFS stack for room traversal
let ROOMS = [];       // 2D array of Room objects [col][row]
let iteration = 0;    // Safety counter for infinite loop prevention
```

#### Room Class

**Definition**: Lines 259-353

```javascript
class Room {
  constructor(x, y) {
    this._doors = [false, false, false, false];  // N, E, S, W
    this._piece = null;      // Selected room piece variation
    this._deadEnd = false;   // Is this a dead end?
    this._used = false;      // Has this room been visited?
    this._first = false;     // Is this the starting room?
    this._x = x;             // Column index
    this._y = y;             // Row index
  }

  // Getters and setters for all properties
  get doors() { return this._doors; }
  set doors(value) { this._doors = value; }
  // ... (similar for all properties)

  // Key methods
  fixPiece() {
    // Determines which room piece to use based on door configuration
    this._piece = generatePiece(figureOutPiece(this._doors));
  }

  fixDeadEnd() {
    // Checks if room has exactly one door (dead end)
    let count = this._doors.filter(door => door === true).length;
    this._deadEnd = (count === 1);
  }

  openDoor(n) {
    // Opens door in direction n (0=N, 1=E, 2=S, 3=W)
    this._doors[n] = true;
  }
}
```

#### Key Functions

##### 1. GenerateMaze()

**Location**: Lines 364-580
**Purpose**: Main maze generation algorithm using DFS with backtracking

**Algorithm**:
```
1. Initialize ROOMS grid (COLS x ROWS)
2. Select random starting position
3. Mark starting room as used, push to STACK
4. WHILE STACK is not empty AND iteration < 5000:
   a. Get current room from top of STACK
   b. Shuffle direction array [1,2,3,4] randomly
   c. FOR each direction in shuffled array:
      - Calculate adjacent room coordinates
      - IF adjacent room is within bounds AND not used:
        * Open door in current room
        * Open opposite door in adjacent room
        * Mark adjacent room as used
        * Push adjacent room to STACK
        * Continue to next iteration
   d. IF no valid moves found:
      - Pop current room from STACK (backtrack)
5. Fix any disconnected rooms
6. Optionally run "wacking" to create loops
7. Assign room pieces based on door configurations
8. Inject room pieces into map data
```

**Safety Features**:
- Iteration limit: 5000 steps maximum
- Boundary checking on all movements
- Disconnected room repair

##### 2. figureOutPiece(doors)

**Location**: Lines 165-257
**Purpose**: Maps door configuration to appropriate room piece type

**Logic**: Uses a series of conditionals to match door patterns:
```javascript
function figureOutPiece(doors) {
  if (doors[0] && doors[1] && doors[2] && doors[3]) {
    return intersection;  // All 4 doors
  }
  if (!doors[0] && doors[1] && !doors[2] && doors[3]) {
    return horizontal_hallway;  // East-West only
  }
  // ... 13 more cases for different door combinations
}
```

##### 3. GetWalkableTiles()

**Location**: Lines 584-726
**Purpose**: Identifies walkable areas and adds shadow effects

**Process**:
1. Scans all 100×100 tiles
2. Identifies walkable floor tiles (IDs 50-59)
3. For each walkable tile:
   - Checks adjacent tiles for walls
   - Determines if shadow should be placed
   - Selects appropriate shadow tile (111-130)
4. Handles pillar shadows (360-degree shadowing)
5. Updates background layer with shadow tiles

**Shadow Mapping**:
```javascript
// Wall tiles (91-110) map to shadow tiles (111-130)
const shadowOffset = 20;  // 91 + 20 = 111
```

##### 4. AddFloorProps()

**Location**: Lines 730-748
**Purpose**: Distributes decorative elements across walkable floor

**Distribution Logic**:
1. Iterates through map with DISTANCE_BETWEEN_PROPS spacing
2. For each candidate position:
   - Checks if tile is walkable floor
   - Ensures minimum distance from other props
   - Randomly selects rock prop (tiles 60-63)
   - Places on props layer

### 2. helpers.js - Utility Functions

**Location**: `/home/user/Maze-Generator/helpers.js`

#### FromBuffer(buffer, width, height)

**Lines**: 5-21
**Purpose**: Decompresses zlib+base64 encoded tile data

```javascript
export function FromBuffer(buffer, width, height) {
  // 1. Decode base64 to buffer
  const buf = Buffer.from(buffer, 'base64');

  // 2. Decompress zlib data
  const data = zlib.inflateSync(buf);

  // 3. Convert to 2D array of tile IDs
  let result = [];
  for (let i = 0; i < data.length; i += 4) {
    // Read 32-bit little-endian integer
    const tileId = data.readUInt32LE(i);
    result.push(tileId);
  }

  return result;
}
```

#### InjectMap(data, object, x, y, width, height)

**Lines**: 31-59
**Purpose**: Injects a room piece into the main map at specified coordinates

```javascript
export function InjectMap(data, object, x, y, width, height) {
  for (let row = 0; row < height; row++) {
    for (let col = 0; col < width; col++) {
      const sourceIndex = row * width + col;
      const targetIndex = (y + row) * 100 + (x + col);

      const tileId = object[sourceIndex];

      // Only overwrite if source tile is not empty (ID 0)
      if (tileId !== 0) {
        data[targetIndex] = tileId;
      }
    }
  }
}
```

### 3. pieces.js - Room Piece Definitions

**Location**: `/home/user/Maze-Generator/pieces.js`

**Purpose**: Exports constants containing room piece variations

**Structure**:
```javascript
export const horizontal_hallway = [
  { map: HORIZONTAL_HALLWAY_JSON.variations[0].map, probability: 0.2 },
  { map: HORIZONTAL_HALLWAY_JSON.variations[1].map, probability: 0.2 },
  // ... more variations
];

export const corner_upper_left = [
  { map: CORNER_UPPER_LEFT_JSON.variations[0].map, probability: 1.0 }
];

// ... 13 more piece types
```

**Data Format**:
- Each piece is an array of variation objects
- Each variation has `map` (tile data) and `probability` (weight)
- Map data is compressed (zlib+base64)

### 4. xml.js - TMX File Generation

**Location**: `/home/user/Maze-Generator/xml.js`

#### InjectIntoXML(background, walls, props)

**Lines**: 4-68
**Purpose**: Generates final TMX file from template and layer data

**Process**:
1. Read `temp.xml` template file
2. Compress each layer's tile data:
   ```javascript
   // Convert tile array to binary buffer
   const buffer = Buffer.alloc(tiles.length * 4);
   tiles.forEach((tile, i) => {
     buffer.writeUInt32LE(tile, i * 4);
   });

   // Compress with zlib
   const compressed = zlib.deflateSync(buffer);

   // Encode as base64
   const encoded = compressed.toString('base64');
   ```
3. Parse template XML using xml2js
4. Inject compressed data into layer elements
5. Build final XML
6. Write to `giraffe.tmx`

## Data Flow

### Complete Execution Flow

```
START
  ↓
1. Import dependencies and room pieces
  ↓
2. Initialize empty map buffers (background, walls, props)
  ↓
3. GenerateMaze()
   ├─ Create ROOMS grid
   ├─ Run DFS algorithm
   ├─ Fix disconnected rooms
   ├─ Optional: Run wacking
   └─ Inject room pieces into walls layer
  ↓
4. GetWalkableTiles()
   ├─ Identify walkable floors
   ├─ Place shadows on background layer
   └─ Add floor variations (sand, pebbles)
  ↓
5. AddFloorProps()
   └─ Place decorative rocks on props layer
  ↓
6. InjectIntoXML(background, walls, props)
   ├─ Compress all three layers
   ├─ Build TMX XML structure
   └─ Write to giraffe.tmx
  ↓
END
```

## Technical Specifications

### Tile Encoding

**Format**: 32-bit little-endian unsigned integers

```
Tile ID: 0x00000032 (50 in decimal)
Binary representation: [0x32, 0x00, 0x00, 0x00]
Stored in buffer as: 32 00 00 00
```

### Compression Specifications

- **Algorithm**: zlib DEFLATE
- **Encoding**: base64
- **Tile size**: 4 bytes per tile (32-bit integer)
- **Uncompressed size**: 100 × 100 × 4 = 40,000 bytes per layer
- **Compressed size**: Varies, typically 70-85% reduction

### Coordinate Systems

**Room Grid Coordinates**:
- Origin: (0, 0) = top-left room
- X-axis: Increases right (0 to COLS-1)
- Y-axis: Increases down (0 to ROWS-1)

**Tile Map Coordinates**:
- Origin: (0, 0) = top-left tile
- X-axis: Increases right (0 to 99)
- Y-axis: Increases down (0 to 99)

**Room-to-Tile Conversion**:
```javascript
tileX = roomX * ROOM_WIDTH;
tileY = roomY * ROOM_HEIGHT;
```

### Door Direction Encoding

```javascript
const DIRECTIONS = {
  NORTH: 0,
  EAST: 1,
  SOUTH: 2,
  WEST: 3
};

// Opposite directions
oppositeDoor(0) => 2  // North <-> South
oppositeDoor(1) => 3  // East <-> West
oppositeDoor(2) => 0  // South <-> North
oppositeDoor(3) => 1  // West <-> East
```

## Dependencies

### NPM Packages

**xml2js** (v0.4.23):
- Purpose: XML parsing and building
- Usage: Reading temp.xml and generating final TMX
- API:
  ```javascript
  import xml2js from 'xml2js';

  // Parse XML string to JavaScript object
  xml2js.parseString(xmlString, (err, result) => {
    // result is a JavaScript object
  });

  // Build XML from JavaScript object
  const builder = new xml2js.Builder();
  const xml = builder.buildObject(jsObject);
  ```

### Node.js Built-in Modules

**fs** (filesystem):
- `fs.readFileSync()`: Load template and preset files
- `fs.writeFileSync()`: Save generated TMX file

**zlib** (compression):
- `zlib.inflateSync()`: Decompress preset data
- `zlib.deflateSync()`: Compress final map data

**Buffer** (binary data):
- `Buffer.from()`: Create buffers from various sources
- `Buffer.alloc()`: Allocate new buffers
- `buffer.readUInt32LE()`: Read 32-bit integers
- `buffer.writeUInt32LE()`: Write 32-bit integers

## Performance Characteristics

### Time Complexity

- **Maze Generation**: O(COLS × ROWS) - visits each room once
- **Shadow Placement**: O(width × height) - scans all tiles
- **Prop Distribution**: O((width/spacing) × (height/spacing))
- **Overall**: O(n) where n = total tiles (10,000)

### Space Complexity

- **ROOMS Array**: O(COLS × ROWS) = ~108 Room objects
- **Map Buffers**: O(3 × width × height) = 30,000 integers
- **Overall**: O(n) linear with map size

### Typical Execution Time

On modern hardware:
- Maze generation: < 10ms
- Shadow and prop placement: < 50ms
- Compression and file writing: < 100ms
- **Total**: < 200ms

## Known Issues and Limitations

1. **Deprecated Buffer Constructor**: Uses `new Buffer.from()` instead of `Buffer.from()`
2. **Hardcoded Output Filename**: Always writes to `giraffe.tmx`
3. **No Error Handling**: Limited validation and error messages
4. **Global State**: Uses module-level variables instead of encapsulation
5. **Magic Numbers**: Many hardcoded values without named constants
6. **No Tests**: No unit or integration tests
7. **Typo in Preset**: `veritcal_hallway.json` should be `vertical_hallway.json`

## Extensibility Points

### Adding New Room Pieces

1. Create JSON file in `presets/` with Tiled map data
2. Import in `pieces.js`
3. Export as constant with variations array
4. Add case in `figureOutPiece()` in `index.js`

### Customizing Map Size

Modify constants in `index.js`:
```javascript
const width = 150;   // New map width
const height = 150;  // New map height
const COLS = 15;     // More room columns
const ROWS = 12;     // More room rows
```

### Adding New Decoration Types

1. Define new tile ID constants
2. Add placement logic in `GetWalkableTiles()` or `AddFloorProps()`
3. Set probability constants
4. Update layer assignment

### Supporting Different Tilesets

Modify `temp.xml`:
```xml
<tileset firstgid="1" source="your_custom_tileset.tsx"/>
```

Ensure tile IDs in code match your tileset indices.
