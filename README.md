# Rubik's Cube Solver Engine

A Java and OpenCV based 3x3x3 Rubik's Cube solver that scans a physical cube using a webcam,
reconstructs its state through computer vision, and computes an optimized solution automatically.

For a comprehensive technical explanation of the architecture, algorithms, and design decisions,
see [Intro.md](Intro.md).

---

## Table of Contents

- [Features](#features)
- [System Requirements](#system-requirements)
- [Installation](#installation)
- [Project Structure](#project-structure)
- [Usage](#usage)
- [Cube Notation](#cube-notation)
- [Example Output](#example-output)
- [Performance](#performance)
- [Future Improvements](#future-improvements)
- [Author](#author)
- [License](#license)

---

## Features

- Webcam-based cube scanning with real-time contour overlay
- Automatic sticker color detection using OpenCV
- Perceptually accurate color classification via CIE L\*a\*b\* and CIEDE2000
- Four-stage lookup-table-based solving algorithm
- Average solution length of approximately 25 moves
- Move optimization (combining, canceling, and reordering)
- Orientation-independent scanning (any cube orientation is accepted)
- Standard Rubik's Cube notation output
- Java Swing graphical interface
- Cross-platform support (Windows, Linux, macOS)

---

## System Requirements

- Java 8 or later
- OpenCV 3.x or 4.x with Java bindings
- A webcam
- An IDE such as Eclipse, IntelliJ IDEA, or VS Code (recommended for build path configuration)

---

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/SWAT-15/Rubix-Cube-Engine.git
cd Rubix-Cube-Engine
```

### 2. Install OpenCV

#### Windows

1. Download OpenCV from https://opencv.org/releases/.
2. Extract the archive.
3. Locate the Java library at `opencv/build/java/`.
4. Note the path to the native library directory (e.g., `opencv/build/java/x64/`).

#### Linux

```bash
git clone https://github.com/opencv/opencv.git
cd opencv
mkdir build && cd build
cmake ..
make -j$(nproc)
```

After compilation, locate the generated `opencv-xxx.jar` and the native `.so` library.

#### macOS

```bash
brew install opencv
```

Alternatively, build from source following the Linux instructions.

### 3. Configure the IDE

1. Add `opencv-xxx.jar` to the project build path as an external library.
2. Set the native library location to the directory containing the platform-specific OpenCV
   binaries:
   - Windows: `opencv/build/java/x64/`
   - Linux: the directory containing `libopencv_java*.so`
   - macOS: the directory containing `libopencv_java*.dylib`
3. Ensure the source folder is set to `src/`.

### 4. Verify Data Files

The following pre-computed lookup tables must be present in the project root directory:

| File              | Required |
|-------------------|----------|
| `stage0.txt`      | Yes      |
| `stage1.txt`      | Yes      |
| `stage2.txt`      | Yes      |
| `stage3.txt`      | Yes      |
| `testMoves.txt`   | Yes      |

These files are included in the repository. If any are missing, the solver will fail to
initialize.

---

## Project Structure

```
Rubix-Cube-Engine/
|
|-- src/
|   |-- MainFrame.java          Entry point. Swing GUI and keyboard event handling.
|   |-- VideoCap.java           Webcam capture wrapper using OpenCV VideoCapture.
|   |-- Mat2Image.java          Converts OpenCV Mat to Java BufferedImage.
|   |-- AnalyzeFrame.java       Computer vision pipeline: contour detection, color
|   |                           extraction, L*a*b* conversion, CIEDE2000, clustering.
|   |-- SolveCube.java          Four-stage solver with lookup tables and optimization.
|   |-- TableGenerator.java     Cube data model, 18 face-turn operations, hashing,
|   |                           IDDFS table generation, and pruning.
|   |-- DisplayWindow.java      Secondary Swing window showing cube state and solution.
|   |-- SortColors.java         Spatial ordering of detected sticker colors.
|   |-- ColorAndIndex.java      Data class for color classification results.
|   |-- Imshow.java             OpenCV imshow utility for Java.
|   |-- TestEnvironment.java    Offline test harness for color detection debugging.
|
|-- stage0.txt                   Lookup table for solving stage 0 (253,440 entries).
|-- stage1.txt                   Lookup table for solving stage 1 (84,672 entries).
|-- stage2.txt                   Lookup table for solving stage 2 (1,555,200 entries).
|-- stage3.txt                   Lookup table for solving stage 3 (1,296 entries).
|-- testMoves.txt                Pre-scramble sequences for solution optimization.
|-- Intro.md                     Detailed technical introduction.
|-- README.md                    This file.
|-- .classpath                   Eclipse classpath configuration.
|-- .project                     Eclipse project configuration.
|-- .gitignore                   Git ignore rules.
```

---

## Usage

### Running the Application

Run `MainFrame.java` as a Java application. Two windows will appear:

1. **Webcam Window** -- Shows the live camera feed with white rectangle overlays around detected
   sticker contours.
2. **Cube Display Window** -- Shows a grid of 54 colored buttons representing the cube state,
   along with status messages and the solution.

### Controls

| Key   | Action                         |
|-------|--------------------------------|
| SPACE | Capture the current face       |
| R     | Reset all progress             |
| X     | Exit the application           |

### Scanning Procedure

1. Hold the cube in front of the webcam so that one face is clearly visible.
2. Ensure all 9 stickers are detected (indicated by white rectangle overlays).
3. Press SPACE to capture.
4. Rotate the cube and repeat for all six faces.

**Capture the faces in this order:**

```
1. Top
2. Left
3. Front
4. Right
5. Back
6. Bottom
```

After the sixth face is captured, the solver runs automatically and displays the solution.

### Tips for Accurate Detection

- Use consistent, diffused lighting. Avoid direct spotlights.
- Minimize reflections on the cube surface.
- Keep the cube centered in the camera frame.
- Hold the cube steady during each capture.
- Ensure no other square-shaped objects are visible in the frame.
- If detection fails (status message turns red), retake the same face.

---

## Cube Notation

The solution uses standard Rubik's Cube notation:

| Symbol | Meaning                               |
|--------|---------------------------------------|
| R      | Right face, 90 degrees clockwise      |
| R'     | Right face, 90 degrees counter-clockwise |
| R2     | Right face, 180 degrees               |
| L      | Left face, 90 degrees clockwise       |
| L'     | Left face, 90 degrees counter-clockwise |
| L2     | Left face, 180 degrees                |
| U      | Upper face, 90 degrees clockwise      |
| U'     | Upper face, 90 degrees counter-clockwise |
| U2     | Upper face, 180 degrees               |
| D      | Down face, 90 degrees clockwise       |
| D'     | Down face, 90 degrees counter-clockwise |
| D2     | Down face, 180 degrees                |
| F      | Front face, 90 degrees clockwise      |
| F'     | Front face, 90 degrees counter-clockwise |
| F2     | Front face, 180 degrees               |
| B      | Back face, 90 degrees clockwise       |
| B'     | Back face, 90 degrees counter-clockwise |
| B2     | Back face, 180 degrees                |

Clockwise is defined as seen when looking directly at the face.

For a visual guide, see: https://ruwix.com/the-rubiks-cube/notation/

---

## Example Output

```
Your cube:

      OYG
      WYO
      YWG
  YBB RRR WWW OOB
  GBR BRB YGR YOR
  RBW BGO BOY OGG
      RWW
      OWG
      YYG

Calculating...

      WWW
      WWW
      WWW
  OOO GGG RRR BBB
  OOO GGG RRR BBB
  OOO GGG RRR BBB
      YYY
      YYY
      YYY

Solution:

R2 U' R B2 U' R2 U' B R2 U' L U
R2 U' D L D B D B' D' L2 F2 L
F2 R' D2 R

Number of Moves: 28
```

The first block shows the scanned cube state. The second block confirms the solved state. The
solution is given in standard notation. Before executing the moves, hold the cube in the same
orientation as shown in the output (White on top, Green in front).

---

## Performance

| Metric                        | Value              |
|-------------------------------|--------------------|
| Average Solution Length       | ~25 moves          |
| Maximum Solution Length       | <28 moves          |
| Solver Time Limit             | 5 seconds          |
| Lookup Table Size (on disk)   | ~86 MB             |
| Language                      | Java               |
| Computer Vision Library       | OpenCV             |




## Future Improvements

- Improved color calibration with adaptive lighting compensation
- Package structure and build system (Maven or Gradle)
- Unit tests with a proper testing framework
- Configurable file paths for lookup tables
- Faster image processing pipeline
- Real-time cube tracking across frames
- 3D cube visualization
- Solution playback animation
- JavaFX-based modern interface
- Mobile application support

---

## Author

**Swatantra Pratap Singh**

B.Tech -- Computer Science and Engineering
National Institute of Technology Raipur

GitHub: https://github.com/SWAT-15

---

## License

This project is licensed under the MIT License.
