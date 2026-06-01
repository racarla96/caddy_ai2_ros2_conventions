# Caddy AI2 ROS2 Conventions

> **Language policy:** All READMEs, commit messages, and code comments are written in **English**.

---

## Objective

Define the repository organization and development workflow used across all Caddy AI2 ROS 2 packages.

The core principle is a clear, enforced separation between:

- Production-ready code deployable on the real robot
- Development, testing, and simulation utilities

---

## Branch Strategy

```
feat/* ──► jazzy-dev ──► jazzy (selective)
```

### `jazzy` — Stable / Production

The minimum required to build, install, and run the package on the real robot.

**Must include:**
- Source code and headers
- Strictly necessary configuration (`config/`)
- Plugin description files
- `package.xml` with all dependencies declared
- `CMakeLists.txt` referencing only production targets
- `README.md`

**Must NOT include:**
- Test or simulation launch files
- RViz configurations
- Simulation worlds
- Documentation PDFs or unofficial SDK archives
- Mesh formats beyond STL (PLY, STEP, ZIP archives)
- Benchmarking or debugging scripts
- `ARCHITECTURE.md` or design notes
- ROS bag files

### `jazzy-dev` — Development / Staging

Contains everything in `jazzy` plus development utilities.

**May additionally include:**
- Launch files for testing and visualization (`bringup/launch/`, `bringup/rviz/`)
- Extended or experimental configuration
- Simulation worlds (`description/world.sdf.j2`)
- All mesh formats (PLY, STEP, ZIP)
- Reference documentation (`docs/`, `docs_official/`)
- Design documents (`ARCHITECTURE.md`)
- Calibration and debug scripts
- ROS bag files

### `feat/*` — Feature Branches

Always branched from `jazzy-dev`. Merged back into `jazzy-dev` after validation.

Naming:

```
feat/feature-name
fix/fix-name
refactor/name
experiment/name
```

---

## Promotion Workflow

### Development to stable

Promotion from `jazzy-dev` to `jazzy` is **selective**, not a blanket merge.

Some files in `jazzy-dev` (docs, simulation worlds, bringup) are permanently dev-only and must never enter `jazzy`.

Use cherry-pick or selective staging:

```bash
# Option A — cherry-pick a specific commit
git checkout jazzy
git cherry-pick <commit-hash>

# Option B — selectively stage files
git checkout jazzy
git checkout jazzy-dev -- src/ config/ include/
git add CMakeLists.txt package.xml
git commit
```

### New feature to development

```bash
git checkout jazzy-dev
git pull
git checkout -b feat/new-feature

# ... develop and test ...

git checkout jazzy-dev
git merge feat/new-feature
```

---

## Directory Structure

### ROS 2 controller plugin (chainable controller, hardware interface)

```
<package>/
├── include/<package>/       # C++ headers
├── src/                     # C++ implementation + parameters YAML
│   ├── <node>.cpp
│   └── <node>_parameters.yaml
├── config/                  # Production parameters
├── plugin_description.xml
├── CMakeLists.txt
├── package.xml
└── README.md
```

`jazzy-dev` adds:
```
├── launch/                  # Test launch files
```

### Sensor / hardware driver package

```
<package>/
├── code/src/                # C++ implementation
├── config/                  # Production sensor parameters
├── description/
│   ├── sensor.sdf.j2        # Injectable SDF fragment (Jinja2)
│   └── meshes/
│       └── sensor.stl       # STL only
├── sdk/                     # Bundled third-party SDK (if required)
├── startup/                 # Hardware init scripts (udev rules, port aliases)
├── plugin_description.xml   # If ros2_control plugin
├── CMakeLists.txt
├── package.xml
└── README.md
```

`jazzy-dev` adds:
```
├── ARCHITECTURE.md
├── bringup/
│   ├── launch/
│   └── rviz/
├── description/
│   ├── world.sdf.j2         # Test world
│   └── meshes/
│       ├── sensor.ply
│       └── sensor.zip
├── docs/                    # Reference papers / notes
└── docs_official/           # Manufacturer documentation
```

---

## `description/` Folder Convention

Used for sensor and robot description assets. Kept in both branches unless noted.

| Path | jazzy | jazzy-dev |
|---|---|---|
| `description/sensor.sdf.j2` | ✓ | ✓ |
| `description/world.sdf.j2` | — | ✓ |
| `description/meshes/*.stl` | ✓ | ✓ |
| `description/meshes/*.ply`, `*.zip` | — | ✓ |

SDF files use Jinja2 templating (`.sdf.j2`) to allow injection into larger robot descriptions.

---

## `package.xml` Requirements

Every package in `jazzy` must have a complete `package.xml`:

```xml
<package format="3">
  <name>package_name</name>
  <version>0.0.0</version>
  <description>One-line description of what the package does.</description>
  <maintainer email="racarla96@gmail.com">racarla96</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <!-- List every package used in CMakeLists.txt -->
  <depend>rclcpp</depend>
  ...
</package>
```

Rules:
- `<description>` must be filled in (not `TODO`)
- `<license>` must be declared (`Apache-2.0` unless otherwise required)
- Every `find_package()` call in `CMakeLists.txt` must have a corresponding `<depend>` (or `<build_depend>` / `<buildtool_depend>` as appropriate)
- Remove any dependency listed in `CMakeLists.txt` but not actually used in code

---

## `README.md` Requirements

Every package must have a README with at least:

1. **One-line description** — what the package does and why it exists
2. **Kinematics / algorithm** — if the package implements a non-trivial computation, document it (equations, diagrams)
3. **Controller chain / architecture** — how this package connects to upstream and downstream components
4. **Parameters table** — name, type, constraints, description for every parameter
5. **Example configuration** — a minimal working YAML snippet
6. **Build instructions**

---

## ROS 2 Overlay Recommendation

Pair this branch strategy with ROS 2 overlay workspaces:

```
underlay workspace  →  jazzy   (stable, always sourced)
overlay  workspace  →  jazzy-dev / feat/*  (experimental)
```

This allows switching between stable and experimental without rebuilding everything.

---

## VS Code

Recommended extensions and workspace settings are documented in [vscode.md](vscode.md) and bundled in the `.vscode/` folder of this package.

---

## Summary

| Branch | Purpose | Deployable on robot |
|---|---|---|
| `jazzy` | Stable, minimal, clean | Yes |
| `jazzy-dev` | Integration, staging, R&D | With caution |
| `feat/*` | Isolated feature development | No |
