# Agibot G2 Description

This package contains the description files for Agibot Genie G2 humanoid. The origin models could be found
at [GenieSim 3.0 Assets](https://modelscope.cn/datasets/agibot_world/GenieSimAssets).

Arm kinematics and **end effectors**
reuse [m6_ccs_description](../robot_descriptions/manipulator/Tianji/m6_ccs_description) (`eef_type_resolver.xacro` +
`eefs.xacro`). Type keys are the same as M6 §2.3 (humanoid gripper/hand set); G2 does **not** use M6-only scenes such as
`desktop` or `usb-90c`.

## 1. Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to agibot_g2_description --symlink-install
source install/setup.bash
```

## 2. End-effector configuration (launch)

Configure end effectors **via launch arguments** (optional `robot_profile:=` for a fixed machine).

| Setup | Launch args |
|--------|-------------|
| Symmetric (both arms) | `type:=…` |
| Asymmetric (left ≠ right) | `left_type:=…` `right_type:=…` (do **not** pass `type:=`) |

One key per side drives **URDF**, **OCS2 planning xacro**, and **ros2_control** compose together.

### 2.1 Supported `type` keys (symmetric, both arms)

| Key                                                 | End effector                                         |
|-----------------------------------------------------|------------------------------------------------------|
| *(empty)*                                           | Fixed `left_eef` / `right_eef` only (no tool joints) |
| `rg75`                                              | Jodell RG75                                          |
| `rg75_heavy`                                        | RG75 with heavy pad                                  |
| `ag2f90`                                            | ChangingTek AG2F90-D                                 |
| `ag2f90_c` / `ag2f90_d`                             | AG2F90-C / AG2F90-D                                  |
| `ag2f90_c_soft`                                     | AG2F90-C, soft pad                                   |
| `linkerhand_o6` / `linkerhand_l6` / `linkerhand_o7` | LinkerHand                                           |

Variant configs under `config/ros2_control/` include `rg75.yaml`, `ag2f90.yaml`, `linkerhand_o6.yaml`,
`linkerhand_l6.yaml`, `linkerhand_o7.yaml`, `ag2f90_c_soft.yaml`. **Asymmetric** setups use `robot_common_launch` *
*compose** (per-side templates from `eef_control_registry.yaml`) on top of `common.yaml`.

### 2.2 Symmetric — launch argument `type:=`

Works on **OCS2 / control** launches (`split_body`, `full_body`, `demo`, `manipulator_ocs2`) and on **RViz-only**
`manipulator.launch.py`.

```bash
# RViz — bare arms
ros2 launch robot_common_launch manipulator.launch.py robot:=agibot_g2 collider:=simple

# RViz — dual AG2F90-D
ros2 launch robot_common_launch manipulator.launch.py robot:=agibot_g2 type:=ag2f90 collider:=simple

# Split body — mock, symmetric RG75
ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2 type:=rg75

# Full body WBC
ros2 launch ocs2_arm_controller full_body.launch.py robot:=agibot_g2 type:=ag2f90
```

![G2](.images/agibot_g2.png)

**Split body note:** `ocs2_arm_controller` uses `robot_name: m6_ccs` for arm MPC. The same launch arguments are applied
to the **m6_ccs** planning URDF (`planning_scope:=arms`) and the **agibot_g2** ros2_control URDF — use matching types on
both sides.

### 2.3 Asymmetric end effectors (left ≠ right)

**Rules**

1. Do **not** pass `type:=` when using `left_type` / `right_type`.
2. Type keys are from §2.1 (`rg75`, `ag2f90_c`, `linkerhand_o7`, …). Leave a side empty for bare `*_eef` only.
3. `type:=ag2f90` means **90D** on both arms; use `ag2f90_c` explicitly for 90C.

**Split body — mock control (typical)**

Left RG75 + right LinkerHand O7:

```bash
ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2 \
  left_type:=rg75 right_type:=linkerhand_o7
```

Left AG2F90-C + right RG75:

```bash
ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2 \
  left_type:=ag2f90_c right_type:=rg75
```

Both hands, different models (O6 + O7):

```bash
ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2 \
  left_type:=linkerhand_o6 right_type:=linkerhand_o7
```

**Full body WBC**

```bash
ros2 launch ocs2_arm_controller full_body.launch.py robot:=agibot_g2 \
  left_type:=rg75 right_type:=linkerhand_o7
```

**RViz only**

```bash
ros2 launch robot_common_launch manipulator.launch.py robot:=agibot_g2 \
  left_type:=rg75 right_type:=linkerhand_o7 collider:=simple
```

Or planning-only:

```bash
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=agibot_g2 \
  left_type:=rg75 right_type:=linkerhand_o7
```

**Gazebo / Isaac**

Add `hardware:=gz` or `hardware:=isaac` to the same argument list as above.

**No tool controllers** (arms + body only):

```bash
ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2 enable_gripper:=false
```

### 2.4 Component visualization (`component.launch.py`)

`type:=arms` shows **arms only** (no end effectors). Use §2.2–2.3 launches to visualize tools on the full robot.

Arm mounts use `m6_ccs_description/xacro/arm_mount.xacro` (`DualArmMount`), ±0.069 m on **Y**. `arm_base_joint` on the
torso is translation-only (no extra roll on `arm_base`).

## 3. OCS2 Demo

### 3.1 Official OCS2 Mobile Manipulator Demo

```bash
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=agibot_g2
ros2 launch robot_common_launch manipulator_ocs2.launch.py robot_name:=agibot_g2 type:=rg75
```

### 3.2 Mock Components

* Split body (arms: `m6_ccs` OCS2 + `arm_base`; torso/head: `basic_joint_controller`)

  ```bash
  ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2
  ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2 type:=ag2f90
  ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2 type:=linkerhand_o7
  # Asymmetric (§2.3):
   ros2 launch ocs2_arm_controller split_body.launch.py robot:=agibot_g2 \
     left_type:=rg75 right_type:=linkerhand_o7
  ```

* Full body WBC

  ```bash
  ros2 launch ocs2_arm_controller full_body.launch.py robot:=agibot_g2 type:=ag2f90
  ```

### 3.3 Isaac Sim Simulation

```bash
ros2 launch ocs2_arm_controller full_body.launch.py robot:=agibot_g2 type:=ag2f90 hardware:=isaac
```
