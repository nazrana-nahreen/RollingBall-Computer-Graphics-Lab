# Rolling Ball 3D — OpenGL/GLUT Course Project

A 3D rolling-ball game built with **C++** and **OpenGL / GLUT (freeglut)** for
a Computer Graphics course, targeting **Code::Blocks**.

A red ball physically rolls (true rotation, not just translation) across a
large 200x200-unit checkered ground plane, dodging pillar obstacles and
collecting 20 spinning gold coins before the timer runs out. The camera
freely orbits around the ball on left-click drag, so the scene can be viewed
from literally any angle — top-down, side-on, close chase-cam, or a wide
pulled-back view of the whole area.

## Controls

| Action              | Input                                   |
|---------------------|-------------------------------------------|
| Roll forward/back    | `W`/`S` or Up/Down arrows (camera-relative) |
| Roll left/right      | `A`/`D` or Left/Right arrows              |
| Jump                 | `Space`                                    |
| Orbit camera (any view) | Left-click + drag mouse                |
| Zoom in/out           | Mouse scroll wheel                        |
| Restart               | `R`                                        |
| Quit                  | `Esc`                                      |

Movement is **camera-relative**: "forward" always means "away from the
camera, into the screen," exactly like a third-person game — so as you orbit
the camera, the WASD directions rotate with it.

## Project files

```
RollingBall3D/
├── RollingBall3D.cbp   <- Code::Blocks project file
├── src/
│   └── main.cpp         <- full game source (single file)
└── README.md
```

## How to run it — Code::Blocks (Windows)

1. Install **Code::Blocks** with the MinGW compiler bundle.
2. Install **freeglut** for MinGW:
   - Copy `freeglut.h` / `glut.h` into MinGW's `include/GL` folder.
   - Copy `libfreeglut.a` (and `libfreeglut_static.a`) into MinGW's `lib` folder.
   - Copy `freeglut.dll` next to the compiled `.exe` (or into `System32`).
3. Open `RollingBall3D.cbp` in Code::Blocks.
4. Build → Build and Run (F9).

Linker settings already included in the `.cbp`:
```
freeglut
opengl32
glu32
```

## How to run it — Linux (tested during development)

```bash
sudo apt-get install freeglut3-dev
g++ -std=c++11 src/main.cpp -o rollingball3d -lglut -lGLU -lGL
./rollingball3d
```

## How to run it — macOS

```bash
g++ -std=c++11 src/main.cpp -o rollingball3d -framework GLUT -framework OpenGL
./rollingball3d
```

## Implementation notes (useful for your report/viva)

- **3D projection**: `gluPerspective` (60° FOV) instead of an orthographic
  projection, with `GL_DEPTH_TEST` enabled for correct occlusion.
- **True rolling rotation**: the ball's spin is *not* faked. Each frame, the
  distance travelled `d` and movement direction are used to compute a
  rotation axis perpendicular to the velocity (`axis = (-dz, 0, dx)` in the
  horizontal plane) and an angle `θ = d / radius` (arc length ÷ radius —
  the real rolling-without-slipping relationship). That incremental rotation
  is composed onto a persistent 4x4 matrix (`ballRotation`) every frame using
  the standard OpenGL "load identity → rotate → multiply by previous matrix
  → read back" accumulation trick, so the sphere's surface visibly rolls in
  the correct direction as it moves.
- **Free-orbit camera**: implemented with spherical coordinates (`camYaw`,
  `camPitch`, `camDist`) around the ball's position, fed into `gluLookAt`
  every frame. Left-drag mouse motion updates yaw/pitch; the wheel updates
  distance. Pitch is clamped so the camera can't flip over the top or dive
  under the ground plane.
- **Camera-relative controls**: forward/right vectors are derived from
  `camYaw` so that "W" always means "away from the camera," matching what
  the player sees regardless of how the camera has been orbited.
- **Physics**: simple explicit-Euler integration for velocity/position, with
  ground friction (exponential-style speed decay), a speed cap, and gravity
  for the jump arc.
- **Collision**: sphere-vs-AABB against each pillar obstacle, resolved by
  pushing the ball out along the axis of least penetration and zeroing that
  velocity component (so the ball slides along a wall instead of sticking).
  Coin pickup uses a simple 2D (X-Z) distance check.
- **Lighting**: one directional-ish point light (`GL_LIGHT0`) with ambient +
  diffuse components; `GL_COLOR_MATERIAL` lets `glColor3f` drive the
  material color directly, which keeps the code simple for a course project.
- **HUD**: drawn as a 2D overlay by temporarily swapping to an orthographic
  projection matrix, disabling depth test/lighting, drawing bitmap text, then
  restoring the 3D perspective matrix — the standard technique for HUDs in
  immediate-mode OpenGL.

## Extension ideas (for extra credit)

- Replace the flat ground with a height-mapped terrain (sample a noise
  function for `y` and adjust ball-ground collision to follow the surface).
- Add ramps/slopes as tilted boxes and adjust collision to let the ball
  roll up them.
- Textures for the ground, ball, and coins (`glTexImage2D` + `.bmp`/`.png` loader).
- A minimap in the HUD showing the ball and remaining coins from above.
- Multiple ball skins or power-ups (speed boost, magnet for coins).
