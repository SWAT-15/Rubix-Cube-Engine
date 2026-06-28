# 🧩 Rubik's Cube Solver

A **Java + OpenCV** based **3×3×3 Rubik's Cube Solver** that scans a physical cube using a webcam, reconstructs its state, and generates an optimized solution automatically.

The project combines **Computer Vision**, **Algorithm Design**, and **Object-Oriented Programming** to solve a scrambled Rubik's Cube with an average solution length of **approximately 25 moves**.

---

# Features

* Webcam-based cube scanning
* Automatic sticker color detection using OpenCV
* Cube state reconstruction
* Cube validity verification
* Optimized solving algorithm
* Average solution length of approximately **25 moves**
* Standard Rubik's Cube notation output
* Cross-platform support (Windows, Linux, macOS)
* Java Swing graphical interface

---

# Project Overview

The application performs the following steps:

1. Capture images of all six cube faces.
2. Detect sticker colors using OpenCV.
3. Reconstruct the cube state.
4. Validate the cube configuration.
5. Compute an optimized solving sequence.
6. Display the solution using standard Rubik's Cube notation.

---

# System Requirements

* Windows, Linux, or macOS
* Java 8 or later
* OpenCV 3.x or later
* Webcam
* IDE (Eclipse, IntelliJ IDEA, or VS Code)

---

# Installing OpenCV

## Windows

1. Download OpenCV from:

```
https://opencv.org/releases/
```

2. Extract the downloaded archive.

3. Locate the Java library:

```
opencv/build/java/
```

4. Add

```
opencv-xxx.jar
```

to your project's libraries.

5. Configure the native library path according to your operating system.

---

## Linux

Download OpenCV and build it:

```bash
mkdir build
cd build
cmake ..
make
```

After compilation, configure your IDE to use the generated Java libraries.

---

## macOS

Install OpenCV using Homebrew:

```bash
brew install opencv
```

or build it from source if preferred.

---

# Clone the Repository

```bash
git clone https://github.com/SWAT-15/Rubix-Cube-Engine.git

cd Rubix-Cube-Engine
```

Import the project into your preferred Java IDE and add the OpenCV library to the project's build path.

---

# Running the Application

Run

```
MainFrame.java
```

A webcam window will appear.

For the best detection accuracy:

* Use consistent lighting.
* Avoid reflections.
* Keep the cube centered.
* Hold the cube steady while capturing each face.

---

# Cube Scanning Order

Capture the six faces in the following order:

```
TOP
LEFT
FRONT
RIGHT
BACK
BOTTOM
```

---

# Controls

| Key   | Action               |
| ----- | -------------------- |
| SPACE | Capture current face |
| X     | Exit application     |

---

# Example Output

```
your cube:

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

Solution

R2 U' R B2 U' R2 U' B R2 U' L U
R2 U' D L D B D B' D' L2 F2 L
F2 R' D2 R

Number of Moves: 28
```

---

# Understanding the Output

The printed cube represents the scanned cube state.

```
      OYG
      WYO
      YWG
  YBB RRR WWW OOB
  GBR BRB YGR YOR
  RBW BGO BOY OGG
      RWW
      OWG
      YYG
```

The faces correspond to:

* Top
* Left
* Front
* Right
* Back
* Bottom

Before executing the generated solution, make sure you are holding the cube in the same orientation shown above.

---

# Cube Notation

The generated solution follows the standard Rubik's Cube notation.

| Move | Meaning                      |
| ---- | ---------------------------- |
| R    | Right face clockwise         |
| R'   | Right face counter-clockwise |
| R2   | Right face 180°              |
| L    | Left face clockwise          |
| U    | Upper face clockwise         |
| D    | Bottom face clockwise        |
| F    | Front face clockwise         |
| B    | Back face clockwise          |

Example:

```
R U R' U' F2 D2
```

If you are new to Rubik's Cube notation, you can learn it here:

https://ruwix.com/the-rubiks-cube/notation/

---

# Project Structure

```
Rubix-Cube-Engine
│
├── src
│   ├── cube
│   ├── gui
│   ├── imageprocessing
│   ├── solver
│   ├── util
│   └── MainFrame.java
│
├── README.md
└── LICENSE
```

---

# Performance

| Metric                  |                 Value |
| ----------------------- | --------------------: |
| Average Solution Length |             ~25 Moves |
| Maximum Solution Length |             <28 Moves |
| Language                |                  Java |
| Computer Vision         |                OpenCV |
| Platform                | Windows, Linux, macOS |

---

# Future Improvements

* Improved color calibration
* Automatic lighting compensation
* Faster image processing
* Real-time cube tracking
* 3D cube visualization
* Solution playback animation
* JavaFX interface
* Mobile application support

---

# Skills Demonstrated

* Computer Vision
* OpenCV
* Java Development
* Object-Oriented Programming
* Algorithm Design
* Search Algorithms
* Software Engineering
* GUI Development

---

# Repository

```
https://github.com/SWAT-15/Rubix-Cube-Engine
```

---

# Author

**Swatantra Pratap Singh**

B.Tech — Computer Science & Engineering
National Institute of Technology Raipur

GitHub: https://github.com/SWAT-15

---

# License

This project is licensed under the MIT License.

---

If you found this project useful, consider giving the repository a ⭐ on GitHub.
