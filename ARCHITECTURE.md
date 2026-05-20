# Caddy AI2 — Package Architecture

## Package Relationships

```mermaid
graph TD

    %% ── External input ──────────────────────────────────────────
    NAV(["Navigation Stack\n― cmd_vel ―"])

    %% ── Bringup ─────────────────────────────────────────────────
    subgraph bringup["Bringup"]
        ROBOT["caddy_ai2_ros2_robot\n(real robot)"]
        SIM["caddy_ai2_ros2_sim\n(simulation)"]
    end

    %% ── Description ─────────────────────────────────────────────
    DESC["caddy_ai2_ros2_description\nURDF / SDF model"]

    %% ── ros2_control chain ───────────────────────────────────────
    subgraph chain["ros2_control chain"]
        BSC["bicycle_steering_controller\n― external ―"]

        subgraph adapters["Kinematic Adapters"]
            SA["bicycle_to_ackermann\nsteering_adapter"]
            TA["bicycle_to_ackermann\ntraction_adapter"]
        end

        subgraph hw_iface["Hardware Interfaces"]
            SD["control_system\nsteering_driver"]
            TD["control_system\ntraction_driver"]
        end
    end

    %% ── Physical endpoints ───────────────────────────────────────
    HW_S([Steering actuator])
    HW_T([Traction motors])

    %% ── Sensors ─────────────────────────────────────────────────
    subgraph sensors["Sensors"]
        SBG["sbg_ig500n\nIMU"]
        SICK["sick_lms_291\nLIDAR"]
        YDLIDAR["ydlidar_x4\nLIDAR"]
        NAVSAT["navsat_generic\nGPS"]
    end

    %% ── Safety ──────────────────────────────────────────────────
    ABS["automatic_brake_system"]

    %% ── Simulation backends ──────────────────────────────────────
    subgraph sim_back["Simulation Backends"]
        GZ["gazebo_simulation"]
        MVSIM["mvsim_simulation"]
        GGT["gz_ground_truth"]
    end

    %% ── Utility ─────────────────────────────────────────────────
    RDP["robot_description\npublisher"]

    %% ═══════════════════════════════════════════════════════════
    %% Control signal flow (solid)
    %% ═══════════════════════════════════════════════════════════
    NAV --> BSC
    BSC -- "δ center (position)" --> SA
    BSC -- "v center (velocity)" --> TA

    SA -- "δ left / δ right" --> SD
    TA -- "ω left / ω right" --> TD
    SA -- "δ left / δ right" --> GZ
    TA -- "ω left / ω right" --> GZ

    SD --> HW_S
    TD --> HW_T

    ABS -. "override cmd_vel" .-> BSC

    %% ═══════════════════════════════════════════════════════════
    %% Package dependencies (dashed)
    %% ═══════════════════════════════════════════════════════════

    %% Sensors feed data downward into the system
    SBG  -. "IMU data" .-> ROBOT
    NAVSAT -. "NavSat fix" .-> ROBOT
    SICK -. "LaserScan" .-> ROBOT
    SICK -. "LaserScan" .-> SIM
    YDLIDAR -. "LaserScan" .-> ROBOT
    YDLIDAR -. "LaserScan" .-> GZ

    %% Real robot bringup
    ROBOT -.-> DESC
    ROBOT -.-> BSC
    ROBOT -.-> SA & TA
    ROBOT -.-> SD & TD

    %% Simulation bringup
    SIM -.-> DESC
    SIM -.-> SA & TA
    SIM -.-> GZ & MVSIM
    GGT -.-> SIM

    %% Shared
    DESC -.-> RDP

    %% ═══════════════════════════════════════════════════════════
    %% Styles
    %% ═══════════════════════════════════════════════════════════
    classDef bringup   fill:#e2e8f0,stroke:#475569,color:#0f172a
    classDef adapter   fill:#fef9c3,stroke:#ca8a04,color:#713f12
    classDef driver    fill:#fce7f3,stroke:#db2777,color:#500724
    classDef sensor    fill:#dcfce7,stroke:#16a34a,color:#14532d
    classDef sim       fill:#ede9fe,stroke:#7c3aed,color:#2e1065
    classDef safety    fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef shared    fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef external  fill:#f1f5f9,stroke:#94a3b8,color:#334155,stroke-dasharray:4 4
    classDef hw        fill:#1e293b,stroke:#0f172a,color:#f8fafc

    class ROBOT,SIM bringup
    class SA,TA adapter
    class SD,TD driver
    class SBG,SICK,YDLIDAR,NAVSAT sensor
    class GZ,MVSIM,GGT sim
    class ABS safety
    class DESC,RDP shared
    class BSC,NAV external
    class HW_S,HW_T hw
```

---

## Package Index

| Package | Type | Description |
|---|---|---|
| `caddy_ai2_ros2_robot` | Bringup | Real robot launch: wires controllers, drivers and sensors |
| `caddy_ai2_ros2_sim` | Bringup | Simulation launch: Gazebo and MVSim |
| `caddy_ai2_ros2_description` | Description | URDF / SDF robot model shared by real and sim |
| `caddy_ai2_ros2_robot_description_publisher` | Utility | Publishes robot_description topic |
| `bicycle_to_ackermann_steering_adapter` | Controller | Chainable: bicycle center angle → individual Ackermann steering angles |
| `bicycle_to_ackermann_traction_adapter` | Controller | Chainable: bicycle center velocity → individual wheel velocities |
| `caddy_ai2_ros2_control_system_steering_driver` | Hardware Interface | ros2_control HW interface for real steering actuator |
| `caddy_ai2_ros2_control_system_traction_driver` | Hardware Interface | ros2_control HW interface for real traction motors |
| `caddy_ai2_ros2_control_sensors_sbg_ig500n` | Sensor | IMU driver (SBG IG-500N) — ROS 2 node + ros2_control HW interface |
| `caddy_ai2_ros2_sensors_sick_lms_291` | Sensor | LIDAR driver (SICK LMS 291) |
| `caddy_ai2_ros2_sensors_ydlidar_x4` | Sensor | LIDAR driver (YDLidar X4) |
| `caddy_ai2_ros2_sensors_navsat_generic` | Sensor | Generic GPS / NavSat bridge |
| `caddy_ai2_ros2_automatic_brake_system` | Safety | Monitors sensors and overrides cmd_vel to brake if needed |
| `caddy_ai2_ros2_gazebo_simulation` | Simulation | Gazebo world + ros_gz_bridge; reuses kinematic adapters |
| `caddy_ai2_ros2_mvsim_simulation` | Simulation | MVSim world definition |
| `gz_ground_truth` | Simulation | Gazebo ground-truth pose publisher |
