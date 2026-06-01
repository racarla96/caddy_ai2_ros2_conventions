# VS Code Setup

Recommended extensions and workspace settings for developing Caddy AI2 ROS 2 packages.

---

## Extensions

| Extension | ID | Purpose |
|---|---|---|
| Better Jinja | `samuelcolvin.jinjahtml` | Syntax highlighting for Jinja2 templates (`.j2`) |

---

## File Associations

The project uses Jinja2 templates for robot descriptions and configuration:

| Pattern | Language mode | Used for |
|---|---|---|
| `*.urdf.j2` | `jinja-xml` | URDF robot description templates |
| `*.sdf.j2` | `jinja-xml` | SDF robot/world description templates |
| `*.sdf.xacro` | `xml` | Gazebo world files |
| `*.yaml.j2` | `jinja-yaml` | Controller and parameter templates |

---

## Applying the settings

VS Code reads `.vscode/` only from the workspace root. Copy the bundled config once after cloning:

```bash
cp -r src/caddy_ai2_ros2_conventions/.vscode <workspace_root>/
```

The `.vscode/` folder in this package contains:

- **`extensions.json`** — VS Code will offer to install recommended extensions automatically when the workspace is opened.
- **`settings.json`** — file associations so `.j2` files open with the correct language mode.
