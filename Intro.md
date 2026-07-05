# Rubik's Cube Solver Engine -- Technical Introduction

## Overview

This project is a desktop application that solves a physical 3x3x3 Rubik's Cube. It uses a webcam
to scan the cube, computer vision to detect sticker colors, and a multi-stage algorithmic solver to
compute an optimized solution. The application is written in Java with OpenCV for image processing
and Java Swing for the graphical interface.

The average solution length is approximately 25 moves, with a maximum under 28 moves.

---

## System Architecture

The application is composed of three major subsystems:

1. **Image Capture and Display** -- Webcam integration, frame rendering, and user interaction.
2. **Computer Vision Pipeline** -- Contour detection, color extraction, color space conversion, and
   sticker classification.
3. **Solving Engine** -- Orientation normalization, four-stage lookup-table-based solving, and
   move optimization.

### Architectural Diagram

```
+-------------------------------+
|        MainFrame.java         |   Entry point. Swing JFrame with keyboard listener.
|   (GUI + Event Handling)      |   Captures SPACE/R/X key events.
+-------------------------------+
        |               |
        v               v
+---------------+   +-------------------+
| VideoCap.java |   | DisplayWindow.java|   Secondary Swing window showing the
| (Webcam Feed) |   | (Cube State GUI)  |   54-sticker grid and solution text.
+---------------+   +-------------------+
        |
        v
+-------------------------------+
|     AnalyzeFrame.java         |   Core computer vision: contour detection,
|  (CV Pipeline + Clustering)   |   color extraction, L*a*b* conversion,
|                               |   CIEDE2000 matching, k-means clustering.
+-------------------------------+
        |
        v
+-------------------------------+
|      SolveCube.java           |   Loads lookup tables, maps cube orientation,
|   (4-Stage Solver Engine)     |   runs 4-stage solve, optimizes move sequence.
+-------------------------------+
        |
        v
+-------------------------------+
|    TableGenerator.java        |   Cube data structure, all 18 face-turn
| (Cube Model + Table Gen)     |   operations, hashing functions, IDDFS
|                               |   with pruning for offline table generation.
+-------------------------------+
```

### Supporting Classes

| Class               | Purpose                                                                 |
|---------------------|-------------------------------------------------------------------------|
| `Mat2Image.java`    | Converts OpenCV `Mat` objects to Java `BufferedImage` (BGR to RGB).      |
| `Imshow.java`       | Utility providing an `imshow`-like display function for OpenCV in Java.  |
| `SortColors.java`   | Orders detected stickers spatially (top-to-bottom, left-to-right).      |
| `ColorAndIndex.java`| Data object holding L\*a\*b\* values, position index, and cluster info. |
| `TestEnvironment.java` | Offline test harness for debugging the color detection pipeline.     |

---

## Subsystem 1: Image Capture and Display

### MainFrame.java

This is the application entry point. It creates a Swing `JFrame` that:

- Opens a webcam feed via `VideoCap` and continuously repaints the window at approximately 33 fps
  (30 ms sleep interval between frames).
- Listens for keyboard input:
  - **SPACE**: Captures the current frame and sends it through the analysis pipeline.
  - **R**: Resets all progress and starts over.
  - **X**: Exits the application.
- After all six faces are captured, displays the computed solution in the `DisplayWindow`.

### VideoCap.java

Wraps OpenCV's `VideoCapture` class. On each frame:

1. Reads the current webcam frame into a `Mat`.
2. Passes it to `AnalyzeFrame.captureFrame()` with `captured = false` to draw contour overlays
   for visual feedback.
3. When the user presses SPACE, reads a fresh frame and passes it with `captured = true` to
   trigger color extraction.

### DisplayWindow.java

A secondary Swing window displaying:

- A grid of 54 `JButton` components arranged in the standard Rubik's Cube net layout (top, four
  middle faces, bottom). Each button's background color updates to reflect the detected sticker
  color.
- Status text showing capture progress and instructions.
- The final solution string and move count.

---

## Subsystem 2: Computer Vision Pipeline

All computer vision logic resides in `AnalyzeFrame.java`.

### Step 1: Edge Detection and Contour Extraction

The raw webcam frame goes through the following processing chain:

```
RGB Frame --> Grayscale --> 3x3 Blur --> Canny (20, 40) --> Dilate (7x7) --> findContours
```

| Operation         | Function                          | Purpose                                      |
|-------------------|-----------------------------------|----------------------------------------------|
| Grayscale         | `Imgproc.cvtColor(RGB2GRAY)`      | Reduces data to single-channel for gradients. |
| Gaussian Blur     | `Imgproc.blur(3x3)`              | Smooths noise to prevent false edges.         |
| Canny Edge Detect | `Imgproc.Canny(20, 40)`          | Detects edges via dual-threshold gradients.   |
| Morphological Dilate | `Imgproc.dilate(7x7 kernel)`  | Thickens edges so contours close properly.    |
| Contour Finding   | `Imgproc.findContours(RETR_TREE)`| Extracts all closed shapes with hierarchy.    |

### Step 2: Contour Filtering

Not all detected contours are sticker squares. Two filters are applied:

**Squareness Filter**: The isoperimetric quotient (shape factor) is computed as:

```
squareness = 4 * pi * area / perimeter^2
```

A perfect circle scores 1.0, a perfect square approximately 0.785. Only contours with
squareness in [0.7, 0.9] and area >= 900 pixels are retained. This eliminates irregular shapes
while allowing for slight camera-angle distortion of the squares.

**Hierarchy Filter**: Using OpenCV's contour hierarchy (`RETR_TREE`), only contours that have a
parent (`hierarchy[3] != -1`) but no children (`hierarchy[2] == -1`) are kept. This selects the
innermost contour of each sticker, rejecting the outer cube border and any nested noise.

A valid capture requires exactly 9 contours (one per sticker on the face).

### Step 3: Color Extraction

For each detected sticker rectangle:

1. The bounding rectangle is extracted from the original frame as a sub-image.
2. Every pixel's RGB values are summed and then divided by the total pixel count to compute the
   **average RGB color** for that sticker.
3. The `SortColors` class orders all 9 stickers spatially -- sorted by Y coordinate first (with a
   tolerance threshold of 10 pixels to group rows), then by X coordinate within each row. This
   ensures stickers map to the correct positions regardless of detection order.
4. Colors are stored in a 54-element array, filled 9 at a time across all six faces.

### Step 4: Color Space Conversion (RGB to CIE L\*a\*b\*)

Once all 54 stickers are captured, the RGB values are converted to the CIE L\*a\*b\* color space
through a two-stage transformation:

**Stage A -- RGB to CIE XYZ (D65 illuminant)**:

1. Normalize RGB values to [0, 1].
2. Apply inverse sRGB companding (gamma correction).
3. Multiply by the sRGB-to-XYZ transformation matrix using the D65 reference white point
   (Xr = 95.047, Yr = 100.0, Zr = 108.883).

**Stage B -- CIE XYZ to CIE L\*a\*b\***:

1. Divide by the reference white point.
2. Apply the nonlinear cube-root transformation (with linear segment for small values).
3. Compute L\* (lightness), a\* (green-red axis), and b\* (blue-yellow axis).

The CIE L\*a\*b\* color space is perceptually uniform: the numerical distance between two colors
correlates closely with how different they appear to the human eye. This is critical for reliable
color classification under varying lighting conditions.

### Step 5: Center Color Identification (CIEDE2000)

The center sticker of each face (indices 4, 13, 22, 31, 40, 49) defines the face color. The
algorithm identifies which of the six standard Rubik's Cube colors each center corresponds to
using the **CIEDE2000** color-difference formula.

CIEDE2000 is the industry-standard perceptual color difference metric. Unlike simple Euclidean
distance in L\*a\*b\*, it applies correction factors for:

- Lightness weighting (SL)
- Chroma weighting (SC)
- Hue weighting (SH)
- A hue rotation term (RT) that corrects distortion in the blue region

The implementation follows the reference specification from Sharma, Wu, and Dalal (2005).

Centers are matched using a greedy nearest-neighbor approach: the closest unmatched standard
color is assigned to each center in sequence.

### Step 6: Sticker Classification (K-Means Clustering)

With the six center colors identified, all 54 stickers are classified into six groups:

1. The center L\*a\*b\* values serve as initial cluster centers.
2. For each cluster, the algorithm greedily assigns the 9 nearest unassigned stickers (by
   Euclidean distance in L\*a\*b\* space).
3. Each sticker receives a numeric label (0-5) corresponding to its assigned face color.

This is a constrained variant of k-means that enforces exactly 9 members per cluster, which is
appropriate given the fixed structure of a Rubik's Cube.

---

## Subsystem 3: Solving Engine

### Cube Representation

The cube is stored as a `byte[6][9]` array in `TableGenerator.java`:

- 6 faces: Top (0), Left (1), Front (2), Right (3), Back (4), Bottom (5).
- 9 stickers per face, indexed 0-8 in row-major order (top-left to bottom-right).
- Each sticker holds a value 0-5 representing its color (White, Orange, Green, Red, Blue, Yellow).

### Face Turn Operations

`TableGenerator.java` implements all 18 possible face turns:

| Operation | Clockwise | Counter-Clockwise | Half Turn |
|-----------|-----------|-------------------|-----------|
| Up        | `turn_U()`  | `turn_Ui()` | `turn_U2()` |
| Down      | `turn_D()`  | `turn_Di()` | `turn_D2()` |
| Left      | `turn_L()`  | `turn_Li()` | `turn_L2()` |
| Right     | `turn_R()`  | `turn_Ri()` | `turn_R2()` |
| Front     | `turn_F()`  | `turn_Fi()` | `turn_F2()` |
| Back      | `turn_B()`  | `turn_Bi()` | `turn_B2()` |

Each turn method performs in-place permutation of the relevant stickers on the face itself (using
`turn_side` / `turn_sideCC` / `turn_side2`) and on the four adjacent faces.

### Orientation Mapping

The solver requires the cube to be in a canonical orientation: White on top, Green in front. Since
the user may hold the cube in any of 24 possible orientations, `SolveCube.mapOrientation()` uses a
24-entry mapping table to find and apply the correct remapping.

The mapping is selected by finding the entry where `mappings[i][upCenter] == 0` (White) and
`mappings[i][frontCenter] == 2` (Green). All sticker values are then remapped through this table.

### Four-Stage Solving Algorithm

The solver decomposes the problem into four progressive stages, each solving a specific subset of
the cube. This approach is inspired by Thistlethwaite's algorithm, which reduces the problem into
nested subgroups with progressively restricted move sets.

| Stage | Hash Function          | Pieces Solved                                         | Table Entries |
|:-----:|------------------------|-------------------------------------------------------|-------------:|
| 0     | `hash_2x2x2()`        | First 2x2x2 block (FRD corner + FR, FD, RD edges)    | 253,440      |
| 1     | `hash_2x2x2B()`       | Second 2x2x2 block (BLD corner + BL, BD, LD edges)   | 84,672       |
| 2     | `hashBowTie()`         | Upper edges (UF, UR, UB, UL) + two corners (FUR, BUL)| 1,555,200    |
| 3     | `hash_1x1x3Stacks()`  | Remaining pieces (FL edge + FLU, FDL, BRU corners)    | 1,296        |

Each stage:

1. Computes a hash of the relevant cube sub-state.
2. Uses the hash as an index into a pre-computed lookup table (loaded from `stage0.txt` through
   `stage3.txt`) to retrieve the solution moves.
3. Applies the solution moves to the cube.
4. Passes through the move optimizer to reduce redundancy.

### Hash Functions

Each hash function encodes the positions and orientations of specific edges and corners into a
unique integer. Edge positions are found by scanning a pre-defined edge list and counting how many
positions are checked before a match. Corner positions work similarly but account for all three
cyclic orientations of each corner. These hash values serve as direct array indices, making lookups
O(1).

### Lookup Table Generation (Offline)

The lookup tables are generated offline by `TableGenerator.startSearch()` using **Iterative
Deepening Depth-First Search (IDDFS)**:

1. For each target depth (0 to max), all possible move sequences of that length are explored.
2. At each terminal node, the cube state is checked against the stage goal conditions
   (`checkStage0`, `checkStage1`, `checkStage2`).
3. If the state has not been seen before, the move sequence is recorded in `foundStates[hash]`.

**Pruning**: A pruning table provides a lower bound on the number of moves required from any
given state. If `pruneValue + currentDepth > targetDepth`, the entire subtree is pruned, reducing
the search space by orders of magnitude.

**Move Pruning**: Consecutive moves on the same face or on opposite faces (which commute) are
skipped, as they can always be combined or reordered.

**Multi-Threading**: 18 threads are launched in parallel, each responsible for one initial move.
This provides near-linear speedup on multi-core machines.

### Time-Bounded Solution Search

Rather than returning the first solution found, the solver tries multiple pre-scramble sequences
(loaded from `testMoves.txt`) to find a shorter overall solution. It runs for up to 5 seconds and
keeps the best result. If a solution of 25 moves or fewer is found, it terminates early.

### Move Optimization

After each stage, the `readSingleTurn()` method scans the move sequence and applies the following
optimizations:

| Pattern             | Optimization         | Example          |
|---------------------|----------------------|------------------|
| Same move twice     | Combine to half turn | `R R` becomes `R2` |
| Move and its inverse| Cancel completely    | `R R'` becomes nothing |
| Two half turns      | Cancel completely    | `R2 R2` becomes nothing |
| Half turn + move    | Reduce to inverse    | `R2 R` becomes `R'` |
| Commuting move pair | Combine across       | `R L R` combines the two R moves |

---

## Data Files

| File            | Size      | Contents                                              |
|-----------------|----------:|-------------------------------------------------------|
| `stage0.txt`    | 7.7 MB    | Lookup table for Stage 0 (253,440 entries)            |
| `stage1.txt`    | 2.7 MB    | Lookup table for Stage 1 (84,672 entries)             |
| `stage2.txt`    | 75.6 MB   | Lookup table for Stage 2 (1,555,200 entries)          |
| `stage3.txt`    | 68 KB     | Lookup table for Stage 3 (1,296 entries)              |
| `testMoves.txt` | 1 MB      | Pre-scramble move sequences for solution optimization |

These files are generated once offline and shipped with the project. They are loaded into memory
at startup by the `SolveCube` constructor.

---

## End-to-End Flow

```
1. User launches MainFrame.java.
2. Webcam feed appears. Contour overlays show detected stickers in real time.
3. User presses SPACE six times, once per face (Top, Left, Front, Right, Back, Bottom).
4. After each capture:
   a. 9 contours are validated.
   b. Average RGB colors are extracted and spatially sorted.
   c. The DisplayWindow updates to show detected colors.
5. After the sixth face:
   a. RGB values are converted to CIE L*a*b*.
   b. Center stickers are matched to standard colors via CIEDE2000.
   c. All 54 stickers are classified via constrained k-means.
   d. The numeric cube representation is passed to SolveCube.
6. SolveCube normalizes the orientation and runs the 4-stage solver.
7. The optimized solution is displayed in standard Rubik's Cube notation.
```

---

## Performance Characteristics

| Metric                        | Value              |
|-------------------------------|--------------------|
| Average Solution Length       | ~25 moves          |
| Maximum Solution Length       | <28 moves          |
| Solver Execution Time         | <5 seconds         |
| Lookup Table Total Size       | ~86 MB (on disk)   |
| Supported Cube Size           | 3x3x3              |


