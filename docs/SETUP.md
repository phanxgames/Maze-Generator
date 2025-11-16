# Setup and Usage Guide

This guide provides step-by-step instructions for installing, configuring, and using the Maze Generator.

## Prerequisites

### Required Software

- **Node.js**: Version 14.0 or higher
  - Download: https://nodejs.org/
  - Verify installation: `node --version`

- **npm**: Usually comes with Node.js
  - Verify installation: `npm --version`

- **Tiled Map Editor** (for viewing output):
  - Download: https://www.mapeditor.org/
  - Version: 1.0 or higher

### Required Assets

- **Tileset File**: `stone_cave.tsx`
  - Must be in the same directory as generated TMX files
  - Referenced by output but not included in this repository
  - You must provide your own tileset or modify `temp.xml` to reference a different tileset

## Installation

### Step 1: Clone or Download Repository

```bash
# If using git
git clone <repository-url>
cd Maze-Generator

# Or download and extract ZIP file
```

### Step 2: Install Dependencies

```bash
npm install
```

This will install the required `xml2js` package.

### Step 3: Verify Installation

```bash
# Check that index.js is executable
ls -l index.js

# Verify dependencies are installed
ls node_modules/xml2js
```

## Basic Usage

### Generate a Maze

```bash
node index.js
```

This will:
1. Generate a randomized maze
2. Create a file named `giraffe.tmx` in the current directory
3. Complete in less than 1 second

### View the Generated Maze

1. Open Tiled Map Editor
2. File → Open → Select `giraffe.tmx`
3. Ensure `stone_cave.tsx` is in the same directory
4. View the three layers: background, walls, props

### Generate Multiple Mazes

```bash
# Generate maze
node index.js

# Rename output to preserve it
mv giraffe.tmx maze_001.tmx

# Generate another maze
node index.js

# Rename again
mv giraffe.tmx maze_002.tmx
```

## Configuration

All configuration is done by editing constants in `index.js`. There is currently no configuration file or command-line interface.

### Map Size Configuration

**Location**: `index.js` lines 18-19

```javascript
const width = 100;   // Map width in tiles
const height = 100;  // Map height in tiles
```

**Example**: Create a larger 150×150 map:
```javascript
const width = 150;
const height = 150;
```

**Note**: You must also adjust room grid size to fit:
```javascript
const COLS = 18;  // 18 × 8 = 144 tiles wide
const ROWS = 13;  // 13 × 11 = 143 tiles tall
```

### Room Grid Configuration

**Location**: `index.js` lines 21-22

```javascript
const COLS = 12;  // Number of room columns
const ROWS = 9;   // Number of room rows
```

**Example**: Create a larger maze with more rooms:
```javascript
const COLS = 15;
const ROWS = 12;
```

**Constraints**:
- `COLS × ROOM_WIDTH` must be ≤ `width`
- `ROWS × ROOM_HEIGHT` must be ≤ `height`
- ROOM_WIDTH = 8, ROOM_HEIGHT = 11

### Difficulty Configuration

**Location**: `index.js` lines 38-39

```javascript
const WACKING = false;           // Enable easier mazes
const WACKING_ITERATIONS = 35;   // Number of extra connections
```

**Example**: Create an easier maze with more loops:
```javascript
const WACKING = true;
const WACKING_ITERATIONS = 50;  // More connections = easier
```

**Effects**:
- `false`: Perfect maze, one path between any two rooms (harder)
- `true`: Imperfect maze, multiple paths, fewer dead ends (easier)

### Decoration Configuration

**Location**: `index.js` lines 41-44

```javascript
const SAND_DUNES_CHANCE = 0.15;      // 15% chance for sand tiles
const FLOOR_SHIT_CHANCE = 0.27;      // 27% chance for floor variations
const DISTANCE_BETWEEN_PROPS = 5;    // Grid spacing for props
const DISTANCE_BETWEEN_POOPS = 7;    // Minimum distance between props
```

**Example**: More decorations:
```javascript
const SAND_DUNES_CHANCE = 0.30;      // 30% sand
const FLOOR_SHIT_CHANCE = 0.40;      // 40% floor variations
const DISTANCE_BETWEEN_PROPS = 3;    // More props (closer spacing)
const DISTANCE_BETWEEN_POOPS = 5;    // Closer minimum distance
```

**Example**: Fewer decorations:
```javascript
const SAND_DUNES_CHANCE = 0.05;      // 5% sand
const FLOOR_SHIT_CHANCE = 0.10;      // 10% floor variations
const DISTANCE_BETWEEN_PROPS = 10;   // Fewer props (wider spacing)
const DISTANCE_BETWEEN_POOPS = 15;   // Greater minimum distance
```

### Output Filename Configuration

**Location**: `xml.js` line 59

```javascript
fs.writeFileSync('giraffe.tmx', xml);
```

**Example**: Change output filename:
```javascript
fs.writeFileSync('my_dungeon.tmx', xml);
```

### Tileset Reference Configuration

**Location**: `temp.xml` line 2

```xml
<tileset firstgid="1" source="stone_cave.tsx"/>
```

**Example**: Use a different tileset:
```xml
<tileset firstgid="1" source="my_custom_tileset.tsx"/>
```

## Advanced Usage

### Creating Custom Room Pieces

#### Step 1: Design Room in Tiled

1. Open Tiled Map Editor
2. Create a new map: 8×11 tiles, 16×16 pixels per tile
3. Design your room piece
4. Use appropriate door openings at edges
5. Save as `my_custom_piece.tmx`

#### Step 2: Export Room Data

1. Open the TMX file in a text editor
2. Locate the wall layer's `<data>` element
3. Copy the entire `<data encoding="base64" compression="zlib">...</data>` section

#### Step 3: Create Preset File

Create `presets/my_custom_piece.json`:

```json
{
  "name": "my_custom_piece",
  "variations": [
    {
      "map": {
        "layers": [
          {},
          {
            "data": {
              "_": "YOUR_BASE64_DATA_HERE"
            }
          }
        ]
      },
      "probability": 1.0
    }
  ]
}
```

#### Step 4: Import in pieces.js

Add to `pieces.js`:

```javascript
import MY_CUSTOM_PIECE_JSON from './presets/my_custom_piece.json' assert { type: 'json' };

export const my_custom_piece = [
  { map: MY_CUSTOM_PIECE_JSON.variations[0].map, probability: 1.0 }
];
```

#### Step 5: Update Room Selection Logic

Add case in `index.js` function `figureOutPiece()`:

```javascript
function figureOutPiece(doors) {
  // ... existing cases ...

  // Add your custom piece logic
  if (doors[0] && !doors[1] && !doors[2] && !doors[3]) {
    return my_custom_piece;  // North door only
  }

  // ... rest of function ...
}
```

#### Step 6: Test

```bash
node index.js
```

Open in Tiled to verify your custom piece appears in the maze.

### Batch Generation

Create a script to generate multiple mazes:

**generate_batch.sh**:
```bash
#!/bin/bash

for i in {1..10}
do
  node index.js
  mv giraffe.tmx "output/maze_$(printf "%03d" $i).tmx"
  echo "Generated maze $i"
done
```

Make executable and run:
```bash
chmod +x generate_batch.sh
mkdir output
./generate_batch.sh
```

### Integration with Game Engines

#### Phaser.js Example

```javascript
// Load the tilemap
this.load.tilemapTiledJSON('maze', 'giraffe.tmx');
this.load.image('tiles', 'stone_cave.png');

// Create the tilemap
const map = this.make.tilemap({ key: 'maze' });
const tileset = map.addTilesetImage('stone_cave', 'tiles');

// Create layers
const backgroundLayer = map.createLayer('background', tileset, 0, 0);
const wallsLayer = map.createLayer('walls', tileset, 0, 0);
const propsLayer = map.createLayer('props', tileset, 0, 0);

// Set collision for walls
wallsLayer.setCollisionByProperty({ collides: true });
```

#### Unity Example

```csharp
// Install Tiled2Unity or SuperTiled2Unity package
// Import giraffe.tmx into Unity project
// Unity will automatically create prefab with layers

// In your script:
GameObject maze = Instantiate(Resources.Load("giraffe"));

// Access layers
Transform backgroundLayer = maze.transform.Find("background");
Transform wallsLayer = maze.transform.Find("walls");
Transform propsLayer = maze.transform.Find("props");

// Add collision to walls layer
wallsLayer.gameObject.AddComponent<CompositeCollider2D>();
```

#### Godot Example

```gdscript
# Load the tilemap
var tilemap = preload("res://giraffe.tmx")

# Create TileMap node
var map = TileMap.new()
add_child(map)

# Set tile set
map.tile_set = preload("res://stone_cave.tres")

# Parse TMX and set tiles
# (Godot has built-in TMX support or use plugin)
```

## Troubleshooting

### Problem: "Cannot find module 'xml2js'"

**Solution**: Install dependencies
```bash
npm install
```

### Problem: Output file is empty or corrupted

**Solution**: Check that temp.xml exists and is valid
```bash
cat temp.xml
```

### Problem: Tiled shows "Cannot open tileset"

**Solution**: Ensure `stone_cave.tsx` is in the same directory as `giraffe.tmx`
```bash
ls -la *.tsx *.tmx
```

### Problem: Maze has disconnected rooms

**Solution**: This should not happen (auto-repair is built in). If it does:
1. Check that COLS and ROWS are reasonable (not too large)
2. Verify iteration limit wasn't reached (check console output)
3. Try running again (random variation may help)

### Problem: Too many/few decorations

**Solution**: Adjust probability constants in `index.js`:
```javascript
const SAND_DUNES_CHANCE = 0.15;  // Increase or decrease
const FLOOR_SHIT_CHANCE = 0.27;  // Increase or decrease
```

### Problem: Maze is too easy/hard

**Solution**: Enable/disable wacking:
```javascript
// Easier (more loops)
const WACKING = true;
const WACKING_ITERATIONS = 50;

// Harder (perfect maze)
const WACKING = false;
```

### Problem: "RangeError: Index out of range"

**Solution**: Check that map size is compatible with room grid:
```javascript
// Ensure: COLS × ROOM_WIDTH <= width
// Ensure: ROWS × ROOM_HEIGHT <= height

// Example fix:
const width = 100;
const height = 100;
const COLS = 12;   // 12 × 8 = 96 ✓
const ROWS = 9;    // 9 × 11 = 99 ✓
```

## Performance Optimization

### For Larger Maps

If generating very large maps (200×200+):

1. **Increase Node.js memory limit**:
   ```bash
   node --max-old-space-size=4096 index.js
   ```

2. **Reduce decoration density**:
   ```javascript
   const DISTANCE_BETWEEN_PROPS = 10;  // Fewer props
   const SAND_DUNES_CHANCE = 0.05;     // Less sand
   const FLOOR_SHIT_CHANCE = 0.10;     // Fewer variations
   ```

3. **Disable wacking** (faster generation):
   ```javascript
   const WACKING = false;
   ```

### For Batch Generation

When generating many mazes:

1. **Use a script** (see Batch Generation above)
2. **Monitor memory usage**:
   ```bash
   node --expose-gc index.js
   ```
3. **Clear output directory** periodically to avoid filesystem slowdown

## Common Use Cases

### Use Case 1: Roguelike Game

**Configuration**:
```javascript
const COLS = 15;
const ROWS = 12;
const WACKING = false;  // Hard mode
const SAND_DUNES_CHANCE = 0.20;
const FLOOR_SHIT_CHANCE = 0.30;
```

**Workflow**:
1. Generate maze at start of each game level
2. Place enemies in dead ends
3. Place treasures in distant rooms
4. Set player spawn at random room

### Use Case 2: Puzzle Game

**Configuration**:
```javascript
const COLS = 8;
const ROWS = 6;
const WACKING = false;  // Single solution
const DISTANCE_BETWEEN_PROPS = 10;  // Clean floors
```

**Workflow**:
1. Generate maze
2. Place puzzle elements in specific room types
3. Player must find correct path
4. No props to obstruct view

### Use Case 3: Exploration Game

**Configuration**:
```javascript
const COLS = 20;
const ROWS = 15;
const WACKING = true;  // Easy navigation
const WACKING_ITERATIONS = 75;
const SAND_DUNES_CHANCE = 0.25;
const DISTANCE_BETWEEN_PROPS = 3;  // Dense decoration
```

**Workflow**:
1. Generate large maze
2. Multiple paths encourage exploration
3. Props create landmarks
4. Less backtracking frustration

## Best Practices

### Development Workflow

1. **Version Control**: Commit changes to configuration before generating
2. **Naming**: Rename generated files with descriptive names
3. **Testing**: Test mazes in your game engine before final use
4. **Backup**: Keep successful configurations in separate files

### Code Modification

1. **Constants**: Always use named constants, avoid magic numbers
2. **Testing**: Test changes with small maps first (e.g., 3×3 rooms)
3. **Validation**: Verify output in Tiled after every change
4. **Documentation**: Comment your modifications for future reference

### Asset Management

1. **Tilesets**: Keep tileset files in version control
2. **Presets**: Document custom room pieces thoroughly
3. **Organization**: Use folders for different tileset themes
4. **Compatibility**: Test with different Tiled versions

## Next Steps

After successfully generating mazes:

1. **Integrate with Game Engine**: Follow integration examples above
2. **Customize Visuals**: Create custom tilesets and room pieces
3. **Add Gameplay**: Place enemies, items, spawn points programmatically
4. **Extend Functionality**: Add features like room tags, special rooms, boss arenas
5. **Optimize**: Profile and optimize for your specific use case

## Getting Help

If you encounter issues:

1. Check this documentation thoroughly
2. Verify all prerequisites are installed
3. Test with default configuration first
4. Check console output for error messages
5. Validate TMX files in Tiled Map Editor

## Additional Resources

- **Tiled Documentation**: https://doc.mapeditor.org/
- **Node.js Documentation**: https://nodejs.org/docs/
- **Maze Algorithms**: https://en.wikipedia.org/wiki/Maze_generation_algorithm
- **TMX Format Specification**: https://doc.mapeditor.org/en/stable/reference/tmx-map-format/

## Appendix: Quick Reference

### File Locations

| File | Purpose |
|------|---------|
| `index.js` | Main logic, configuration |
| `helpers.js` | Utility functions |
| `pieces.js` | Room piece definitions |
| `xml.js` | TMX generation |
| `temp.xml` | TMX template |
| `presets/*.json` | Room piece data |
| `giraffe.tmx` | Output file (generated) |

### Key Constants

| Constant | Default | Purpose |
|----------|---------|---------|
| `width` | 100 | Map width in tiles |
| `height` | 100 | Map height in tiles |
| `COLS` | 12 | Room columns |
| `ROWS` | 9 | Room rows |
| `ROOM_WIDTH` | 8 | Room width in tiles |
| `ROOM_HEIGHT` | 11 | Room height in tiles |
| `WACKING` | false | Enable easier mode |
| `WACKING_ITERATIONS` | 35 | Extra connections |
| `SAND_DUNES_CHANCE` | 0.15 | Sand probability |
| `FLOOR_SHIT_CHANCE` | 0.27 | Variation probability |
| `DISTANCE_BETWEEN_PROPS` | 5 | Prop spacing |
| `DISTANCE_BETWEEN_POOPS` | 7 | Prop minimum distance |

### Tile ID Ranges

| Range | Type |
|-------|------|
| 0 | Empty/transparent |
| 30-33 | Floor variations, sand |
| 50-59 | Walkable floors |
| 60-63 | Rock props |
| 80-89 | Pebbles |
| 91-110 | Walls and pillars |
| 111-130 | Shadows |

### Layer Names

| Layer | Contents |
|-------|----------|
| `background` | Floors, shadows, decorations |
| `walls` | Structural walls |
| `props` | Decorative rocks |
