# TASKS — Caddy AI2 ROS2

Registro centralizado de tareas pendientes para el conjunto de paquetes del proyecto.  
Las tareas están agrupadas por área temática, no por paquete, para facilitar la planificación.

---

## 1. Renombrado de paquetes de adaptadores

Los directorios ya tienen el prefijo `caddy_ai2_ros2_` pero el nombre interno de los paquetes no.

**Paquetes afectados:**
- `caddy_ai2_ros2_bicycle_to_ackermann_steering_adapter` → nombre actual: `bicycle_to_ackermann_steering_adapter`
- `caddy_ai2_ros2_bicycle_to_ackermann_traction_adapter` → nombre actual: `bicycle_to_ackermann_traction_adapter`

**Ficheros a modificar por cada paquete:**

### steering_adapter
- [ ] `package.xml` — `<name>` → `caddy_ai2_ros2_bicycle_to_ackermann_steering_adapter`
- [ ] `CMakeLists.txt` — `project()` → `caddy_ai2_ros2_bicycle_to_ackermann_steering_adapter`
- [ ] `plugin_description.xml` — atributo `path` y atributo `name` (el `type` C++ no cambia)
- [ ] `config/bicycle_to_ackermann_steering_adapter.yaml` — clave raíz del controller name y `type:` del plugin
- [ ] `launch/bicycle_to_ackermann_steering_adapter_example.launch.py` — `FindPackageShare()`

### traction_adapter
- [ ] `package.xml` — `<name>` → `caddy_ai2_ros2_bicycle_to_ackermann_traction_adapter`
- [ ] `CMakeLists.txt` — `project()` → `caddy_ai2_ros2_bicycle_to_ackermann_traction_adapter`
- [ ] `plugin_description.xml` — atributo `path` y atributo `name`
- [ ] `config/bicycle_to_ackermann_traction_adapter.yaml` — clave raíz y `type:` del plugin
- [ ] `launch/bicycle_to_ackermann_traction_adapter_example.launch.py` — `FindPackageShare()`

### Paquetes dependientes que referencian ambos adaptadores
- [ ] `caddy_ai2_ros2_sim/package.xml` — `<exec_depend>` × 2
- [ ] `caddy_ai2_ros2_gazebo_simulation/package.xml` — `<exec_depend>` × 2
- [ ] `caddy_ai2_ros2_gazebo_simulation/CMakeLists.txt` — `find_package()` × 2
- [ ] `caddy_ai2_ros2_gazebo_simulation/bringup/config/controllers_simulation.yaml.j2` — `type:` del plugin × 2
- [ ] `caddy_ai2_ros2_gazebo_simulation/bringup/launch/spawn_robot.launch.py` — argumentos del spawner × 2

---

## 2. Simulación Gazebo — física del robot

### 2.1 canonical_link sin masa (robot estático)
`base_footprint` es el `canonical_link` y no tiene `<inertial>`. En Gazebo Harmonic esto es válido — `base_footprint` actúa como frame de referencia en el suelo (convención ROS). La causa real del robot "estático" era el flotamiento de 2.2, no el canonical_link.

- [x] Diagnóstico revisado: `canonical_link=base_footprint` es correcto y se mantiene

### 2.2 spawn_z desplazado
Con `canonical_link=base_footprint`, Gazebo sitúa `base_footprint` en la posición de spawn. El SDF ya define `base_link` a `wheel_radius` por encima de `base_footprint`. Sumar `wheel_radius` al `spawn_z` duplicaba el offset → robot flotando.

- [x] Corregido: `spawn_z = z` (el argumento del usuario), sin suma adicional

### 2.3 Fricción en ruedas
- [x] Añadido `<surface><friction><ode><mu>1.0</mu><mu2>1.0</mu2></ode>` en las colisiones de las cuatro ruedas (traseras y delanteras) — valor inicial conservador, ajustar con pruebas

### 2.4 Parámetros físicos del vehículo real (en progreso)
- [ ] Medir centro de gravedad real y actualizar `inertial_origin_offset_x` en `robot_params.yaml`
- [ ] Medir masa real del vehículo completo
- [ ] Resolver asimetría de distribución de pesos (actualmente bloque de inercia desplazado hacia atrás)

---

## 3. Metadatos de paquetes sin completar

Varios `package.xml` tienen campos genéricos sin rellenar.

- [ ] `caddy_ai2_ros2_bicycle_to_ackermann_steering_adapter/package.xml` — `<description>` y `<license>`
- [ ] `caddy_ai2_ros2_bicycle_to_ackermann_traction_adapter/package.xml` — `<description>` y `<license>`
- [ ] `caddy_ai2_ros2_gazebo_simulation/package.xml` — `<description>` y `<license>`
- [ ] `caddy_ai2_ros2_description/package.xml` — `<description>`

---

## 4. Driver de tracción (caddy_ai2_ros2_control_system_traction_driver)

- [ ] Verificar el signo del valor RPM en `src/traction_driver.cpp` (TODO en el código)
- [ ] Actualizar referencia del script en README: `asignar_can_motor_drv.sh` → `setup_can_trac_drv.sh` (se renombró)
- [ ] Añadir parámetro `prefix` al macro `system_traction_ros2_control` en `system_traction.ros2_control.xacro` para soporte multi-robot (actualmente `system_traction_joint` está hardcodeado sin prefijo)
- [ ] Implementar parámetros de temporización en el driver C++ (`src/traction_driver.cpp`) y exponerlos en el bloque `<ros2_control>` del SDF, igual que el driver de steering:
  - `controller_manager_frequency_hz` (sustituye a `update_rate`)
  - `hardware_sample_frequency_hz`
  - `read_multiplicity`, `write_multiplicity`, `read_offset`, `write_offset`
  - Referencia: `caddy_ai2_model.sdf.j2` líneas 374–393 (bloque `steering_system`)

---

## 4b. Driver IMU+GPS — SBG IG-500N (`caddy_ai2_ros2_control_sensors_sbg_ig-500n`)

Repo clonado en `src/`. El nodo publica `/imu`, `/imu_ned` y `/gps` (NavSatFix). El paquete companion de simulación (`caddy_ai2_ros2_sensors_imu_sbg_ig500n`) ya está creado como fragmento SDF inyectable.

### Problemas encontrados en el repo

- [x] **Renombrar paquete**: `package.xml` → `caddy_ai2_ros2_control_sensors_sbg_ig500n`. `CMakeLists.txt` actualizado.
- [x] **CMakeLists.txt no instala datos**: añadido `install(DIRECTORY bringup meshes params rviz2 ...)`. SDK opcional con `if(SBG_LIB)` — compila sin el SDK (warning esperado en PC de desarrollo).
- [x] **Topic GPS**: publisher renombrado de `gps` → `navsat` en `sbg_node.cpp`.
- [x] **Frame GPS**: parámetro `gps_frame_id` → `navsat_frame_id`, default `navsat_link`.
- [x] **Migrar `params/sbg.yaml`**: actualizado con `navsat_frame_id`. Creado `bringup/config/sensor_params.yaml` con estructura completa (hardware / operation / simulation).
- [x] **Crear `bringup/launch/real.launch.py`**: lanza `sbg_node` leyendo parámetros desde `sensor_params.yaml`. Argumento `use_rviz` opcional.
- [x] **Orientación IMU**: bloque `IMU2ROS` documentado con TODO explícito — pendiente validación con hardware real antes de activar.
- [x] **`checkError` incompleto**: corregido — ahora imprime el nombre de la operación y el código de error numérico.
- [ ] **SBG cubre también GPS en real**: cuando se usa el driver SBG, poner `navsat_generic.enabled: false` en `robot_params.yaml` para evitar conflicto de topics `/navsat`. Pendiente de integración en `caddy_ai2_ros2_robot`.
- [ ] **Validar transformación NED→ENU**: activar bloque `IMU2ROS` en `sbg_node.cpp` tras prueba con hardware real y verificar que `/imu` cumple REP-103.

### Arquitectura (referencia)
- **Simulación**: `caddy_ai2_ros2_sensors_imu_sbg_ig500n` → `description/sensor.sdf.j2` (plugin Gazebo IMU)
- **Hardware real**: `caddy_ai2_ros2_control_sensors_sbg_ig-500n` → `sbg_node` (driver serial SBG SDK)
- El sensor navsat en simulación lo provee `caddy_ai2_ros2_sensors_navsat_generic`; en real, lo provee el propio `sbg_node` vía el GPS integrado del IG-500N

---

## 5. Sensor LiDAR SICK LMS 291

- [ ] Medir la altura real del haz láser respecto a la base del sensor para ajustar el offset de montaje en el SDF
- [ ] Medir masa y momento de inercia del sensor con valores reales (actualmente estimados)
- [ ] Testear nodo real con hardware y verificar visualización en RViz2
- [ ] Obtener o medir valores de ruido gaussiano del sensor para la simulación

---

## 5b. Arquitectura de sensores — inyección dinámica en el modelo SDF

Actualmente los sensores se añaden de forma ad hoc: los parámetros de montaje del SICK LMS 291 están como claves planas en `robot_params.yaml` (`lidar_sick_x`, `lidar_sick_y`, …) y el fragmento SDF se inyecta con una variable específica (`sick_lidar_fragment`). Esto no escala a múltiples sensores.

### Objetivo

Definir una estructura de sensores en `robot_params.yaml` que permita añadir sensores dinámicamente sin tocar el código Python del launch ni la plantilla SDF del modelo:

```yaml
sensors:
  sick_lms_291:
    package: caddy_ai2_ros2_sensors_lidar_sick_lms_291
    type: lidar           # identifica qué plantilla SDF usar dentro del paquete
    pose:                 # relativo a base_footprint (canonical link)
      x: 0.825
      y: 0.0
      z: 0.835
      roll: 0.0
      pitch: 0.0
      yaw: 0.0
  # imu_sbg:
  #   package: caddy_ai2_ros2_sensors_imu_sbg_ig500n
  #   type: imu
  #   pose: { x: 0.7, y: 0.0, z: 0.635, roll: 0.0, pitch: 0.0, yaw: 0.0 }
```

### Cambios necesarios

- [x] Reestructurar `robot_params.yaml` en `caddy_ai2_ros2_description`: eliminar claves planas `lidar_sick_*` y añadir bloque `sensors:` como arriba
- [x] Actualizar `spawn_robot.launch.py`: iterar sobre `robot_params['sensors']`, cargar el paquete de cada sensor, renderizar su `sensor.sdf.j2`, concatenar todos los fragmentos en `sensors_fragment`
- [x] Renombrar `{{ sick_lidar_fragment }}` → `{{ sensors_fragment }}` en `caddy_ai2_model.sdf.j2`
- [x] Verificar que las poses de cada sensor se expresan relativas a `base_footprint` (canonical link) — `parent_link` del joint ahora es `base_footprint`; z actualizada a 0.835 m (= 0.6 + wheel_radius)
- [ ] Verificar en simulación que el LiDAR aparece en la posición correcta tras el cambio de coordenadas

---

## 6. Meta-package simulación (caddy_ai2_ros2_sim)

### 6.1 Launch system
- [ ] Crear `launch/sim_gazebo.launch.py` — launch completo Gazebo Harmonic
  - [ ] Lanzar el mundo por parámetro
  - [ ] Incluir `robot_state_publisher` con URDF de `_description`
  - [ ] Incluir `controller_manager` con hardware interface simulado
  - [ ] Incluir los dos adaptadores bicycle→ackermann
  - [ ] RViz condicional a `use_rviz`
- [ ] Crear `launch/sim_mvsim.launch.py` — launch completo MVSim
- [ ] Crear `launch/sim.launch.py` — wrapper que selecciona simulador con argumento `simulator:=gazebo|mvsim`

### 6.2 Configuración
- [ ] Crear `config/controllers_sim.yaml` con todos los controladores de simulación
- [ ] Crear `config/rviz/sim.rviz` — configuración de RViz para simulación

### 6.3 Mundos
- [ ] Definir mundo vacío
- [ ] Definir mundo de campo agrícola básico (surcos)

### 6.4 Integración Nav2
- [ ] Definir parámetros de Nav2 adaptados al modelo cinemático Ackermann del Caddy AI2
- [ ] Crear `launch/nav2.launch.py`
- [ ] Evaluar `BicycleSteeringController` de ros2_controllers como alternativa a los adaptadores propios

### 6.5 Paridad simulación ↔ hardware real
- [ ] Verificar que los nombres de topics y TF frames son idénticos entre `_sim` y `_robot`
- [ ] Verificar que el URDF de simulación y hardware real es el mismo fichero (sin copias en `_description`)
- [ ] Script de smoke test: lanzar simulación, publicar `/cmd_vel`, verificar que llegan `/joint_states`

---

## 7. Meta-package hardware real (caddy_ai2_ros2_robot)

### 7.1 Launch system
- [x] Crear `launch/robot.launch.py` — launch principal del sistema completo
  - [x] Incluir `robot_state_publisher` con URDF de `_description` (xacro procesado en tiempo de launch)
  - [x] Incluir `controller_manager` con hardware interfaces steering + traction
  - [x] Sin adaptadores Ackermann — la física del mecanismo los hace innecesarios en real
  - [x] Incluir drivers de sensores habilitados via bucle sobre `robot_params['sensors']`
- [x] Argumento `use_rviz` (default: false en robot real)
- [ ] Argumento `use_joy` para control manual con joystick

### 7.1b URDF para hardware real
- [x] Crear `caddy_ai2_ros2_description/description/urdf/caddy_ai2_robot.urdf.xacro`
  - [x] `base_footprint` → `base_link` (offset `wheel_radius`)
  - [x] `steering_joint` — revolute en centro del eje delantero, representa ángulo de bicicleta
  - [x] `system_traction_joint` — continuous en centro del eje trasero
  - [x] Ruedas delanteras fijas a `steering_link` (Ackermann mecánico) y traseras a `traction_link`
  - [x] Incluye macros ros2_control de ambos drivers
  - [x] Parámetros de calibración pasados como xacro args desde el launch

### 7.2 Configuración
- [x] Crear `config/controllers.yaml.j2` — `bicycle_steering_controller` apunta a `steering_joint` y `system_traction_joint` directamente (sin adapters)
- [x] Parámetros físicos en `caddy_ai2_ros2_description/config/robot_params.yaml` (fuente única de verdad)
- [x] Parámetros de calibración del encoder de dirección como xacro args en `robot.launch.py` (pendiente medir en hardware real):
  - `counts_per_radian` — actualmente 100000 (placeholder)
  - `abs_zero_position` — actualmente 3109 (placeholder)
  - `gain_inc_to_abs` — actualmente 0.00808 (placeholder)
- [ ] Mover parámetros de calibración de dirección a `robot_params.yaml` o a un fichero de calibración dedicado en `caddy_ai2_ros2_robot/config/`

### 7.3 Hardware y puesta en marcha
- [ ] Documentar procedimiento completo de arranque del robot
- [ ] Crear `scripts/check_hardware.sh` — verificación pre-arranque:
  - [ ] Interfaces CAN activas (`can_steer_drv`, `can_trac_drv`)
  - [ ] IMU respondiendo
  - [ ] LiDAR respondiendo
- [ ] Documentar configuración de `/etc/sudoers` para interfaces CAN sin contraseña
- [ ] Evaluar configuración de kernel RT-PREEMPT para el PC embarcado

### 7.4 Seguridad
- [ ] Implementar nodo de parada de emergencia (`/e_stop`)
- [ ] Definir comportamiento safe-state al perder comunicación CAN

---

## 8. Organización de repositorios

- [x] Alinear ramas de repos dependientes a `jazzy` — todos en `jazzy` salvo `gz_ground_truth` que usa `jazzy__harmonic` (correcto, rama específica de Gazebo Harmonic)
- [x] Clonar `caddy_ai2_ros2_control_system_steering_driver` en `src/`
- [x] Centralizar `caddy_ai2_model.sdf.j2` en `caddy_ai2_ros2_description/description/sdf/` (fuente única compartida por sim y real)
  - Flag `simulation` (bool) selecciona bloques Gazebo vs hardware real
  - `spawn_robot.launch.py` pasa `simulation=True`; `robot.launch.py` pasa `simulation=False`
  - `robot.launch.py` extrae `<ros2_control>` del SDF para envolver en URDF mínimo que necesita `ros2_control_node`
- [ ] Eliminar carpetas `bringup/` y `description/urdf/` de `_steering_driver` y `_traction_driver`; dejar solo el fragmento `ros2_control.urdf.xacro` (los standalone URDF son solo para test aislado)
- [ ] Mover meshes duplicados de `caddy_ai2_ros2_mvsim_simulation/meshes/` a `caddy_ai2_ros2_description/meshes/`
- [ ] Mover configuración de controladores de los repos de drivers a `_robot` y `_sim`

---

## 9. Alineamiento visual del chassis en simuladores

El mesh STL del chassis tiene un origen y orientación propios que no coinciden con el frame del robot. El offset de alineamiento puede variar entre simuladores (Gazebo y MVSim gestionan los ejes del modelo de forma diferente).

- [ ] Revisar el alineamiento del chassis en MVSim — actualmente está descolocado respecto al frame del vehículo
- [ ] Comprobar si el mismo desajuste ocurre en Gazebo Harmonic
- [ ] Determinar si el offset de corrección debe vivir en el propio URDF/SDF (transform visual) o en un fichero de alineamiento específico por simulador
- [ ] Centralizar la corrección en `caddy_ai2_ros2_description` para que ambos simuladores la hereden, evitando duplicar el ajuste

---

## 10. Documentación

- [ ] Actualizar README de `caddy_ai2_ros2_description` para reflejar estructura multirepo (actualmente documenta monorepo obsoleto)
- [ ] Añadir diagrama de topics y nodos (generado con `rqt_graph`) al README de `_sim`
- [ ] Documentar diferencias de comportamiento entre Gazebo y MVSim para este modelo
- [ ] Añadir instrucciones para crear mundos personalizados en `_sim`

---

## 11. Migración SDF → URDF (refactoring en curso)

Estado de la migración arquitectural en `caddy_ai2_ros2_robot`:

| Elemento | Estado |
|---|---|
| `caddy_ai2_model.urdf.j2` creado en `caddy_ai2_ros2_description` | ✓ Hecho |
| `display.launch.py` migrado a URDF | ✓ Hecho |
| `caddy_ai2_model.sdf.j2` movido a `caddy_ai2_ros2_gazebo_simulation` | ✓ Hecho |
| `caddy_ai2_ros2_description` reorganizado (`bringup/`, `description/model/`) | ✓ Hecho |
| `caddy_ai2_ros2_robot` reorganizado con `bringup/` | Pendiente |
| `description/ros2_control.urdf.j2` en `caddy_ai2_ros2_robot` | Pendiente |
| `robot.launch.py` migrado de SDF a URDF | Pendiente |

### T1 — Reorganizar `caddy_ai2_ros2_robot`

Mover `config/` y `launch/` dentro de `bringup/` para ser consistente con el resto de paquetes.

**Estructura objetivo:**

```
caddy_ai2_ros2_robot/
├── bringup/
│   ├── config/
│   │   └── controllers.yaml.j2     ← desde config/
│   └── launch/
│       └── robot.launch.py         ← desde launch/
└── description/
    └── ros2_control.urdf.j2        ← nuevo
```

### T2 — Crear `caddy_ai2_ros2_robot/description/ros2_control.urdf.j2`

Extraer los bloques `<ros2_control>` que actualmente están en el SDF y convertirlos a fragmento URDF inyectable en tiempo de launch.

**Contenido esperado:** los dos bloques de hardware real (steering + traction) actualmente en `caddy_ai2_ros2_description/description/sdf/caddy_ai2_model.sdf.j2` líneas 374–414, adaptados a formato URDF.

```xml
<ros2_control name="steering_system" type="system">
  <hardware>
    <plugin>caddy_ai2_ros2_control_system_steering_driver/SystemSteeringHardware</plugin>
    ...
  </hardware>
  <joint name="{{ prefix }}steering_joint">...</joint>
</ros2_control>

<ros2_control name="system_traction" type="actuator">
  <hardware>
    <plugin>caddy_ai2_ros2_control_system_traction_driver/SystemTractionHardwareInterface</plugin>
    ...
  </hardware>
  <joint name="{{ prefix }}system_traction_joint">...</joint>
</ros2_control>
```

### T3 — Migrar `robot.launch.py` de SDF a URDF

Reemplazar la lógica de `_launch_robot` en `caddy_ai2_ros2_robot/launch/robot.launch.py`:

**Actual (SDF):**
1. Renderiza `caddy_ai2_model.sdf.j2` con `simulation=False`
2. Extrae `<ros2_control>` con `_sdf_ros2_control_to_urdf()` → URDF mínimo para `ros2_control_node`
3. Pasa el SDF a `robot_state_publisher`

**Objetivo (URDF):**
1. Renderiza `caddy_ai2_model.urdf.j2` (URDF puro)
2. Renderiza `ros2_control.urdf.j2` (bloques hardware)
3. Inyecta el fragmento dentro del `<robot>` tag → URDF completo
4. Pasa el URDF completo tanto a `robot_state_publisher` como a `ros2_control_node`
5. Eliminar la función `_sdf_ros2_control_to_urdf()`

### T4 — Extraer parámetros de timing del driver a `robot_params.yaml`

Parámetros hardcodeados en el SDF que deberían venir de `robot_params.yaml`:

```yaml
steering_driver:
  hardware_sample_frequency_hz: 500
  read_multiplicity: 1
  write_multiplicity: 10
  read_offset: 0
  write_offset: 1

traction_driver:
  hardware_sample_frequency_hz: 500
```

### T5 — Inyectar `<ros2_control>` desde los paquetes de drivers (largo plazo)

Los bloques `<ros2_control>` de steering y traction están definidos en `caddy_ai2_ros2_robot`, pero idealmente cada driver debería ser su propio fragmento inyectable — igual que los sensores.

**Pasos:**
1. Añadir `description/hardware.urdf.j2` a `caddy_ai2_ros2_control_system_steering_driver`
2. Añadir `description/hardware.urdf.j2` a `caddy_ai2_ros2_control_system_traction_driver`
3. `robot.launch.py` renderiza y ensambla los fragmentos en lugar de tener el `ros2_control.urdf.j2` monolítico en `caddy_ai2_ros2_robot`

**Dependencia:** completar T2 y T3 primero.
