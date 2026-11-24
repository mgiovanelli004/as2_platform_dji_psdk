# as2_platform_dji_psdk

[![build](https://github.com/aerostack2/as2_platform_dji_psdk/actions/workflows/build.yaml/badge.svg)](https://github.com/aerostack2/as2_platform_dji_psdk/actions/workflows/build_humble.yaml)
[![ROS 2 Humble](https://img.shields.io/badge/ROS_2-Humble-22314E?logo=ros)](https://docs.ros.org/en/humble/)
[![License](https://img.shields.io/badge/License-BSD_3--Clause-blue.svg)](LICENSE)
[![arXiv](https://img.shields.io/badge/arXiv-2303.18237-b31b1b.svg)](https://arxiv.org/abs/2303.18237)

[Aerostack2](https://github.com/aerostack2/aerostack2) platform for DJI drones that use the
[DJI Payload SDK](https://github.com/dji-sdk/Payload-SDK) (PSDK), such as the Matrice 30,
Matrice 300 RTK and Matrice 350 RTK.

The platform talks to the drone through [`psdk_ros2`](https://github.com/umdlife/psdk_ros2), so a
running deployment has two nodes: the `psdk_wrapper` node, which owns the connection to the drone,
and the Aerostack2 platform node, which translates Aerostack2 commands into `psdk_wrapper` topics
and services.

> [!NOTE]
> For older drones that only support the deprecated Onboard SDK, such as the Matrice 210 V2, use
> [`as2_platform_dji_osdk`](https://github.com/aerostack2/as2_platform_dji_osdk).

> [!IMPORTANT]
> This package can't be released to the ROS build farm or listed on the ROS Index yet, because its
> `psdk_interfaces` dependency isn't released for any ROS 2 distribution newer than Humble. Building
> it from source, together with [`psdk_ros2`](https://github.com/umdlife/psdk_ros2), works.

## Requirements

- Ubuntu 22.04 with [ROS 2 Humble](https://docs.ros.org/en/humble/Installation.html)
- [Aerostack2](https://aerostack2.github.io/_00_getting_started/index.html)
- [`psdk_ros2`](https://umdlife.github.io/psdk_ros2/documentation/GettingStarted.html), which
  provides `psdk_interfaces` and the `psdk_wrapper` node
- A DJI developer app (`app_name`, `app_id`, `app_key` and `app_license`) from the
  [DJI Developer Center](https://developer.dji.com/)

## Build

From the root of your workspace:

```bash
rosdep install -y -r --ignore-src --from-paths src --packages-up-to as2_platform_dji_psdk
colcon build --packages-up-to as2_platform_dji_psdk
```

## Usage

Set your DJI app credentials in `config/psdk_authentication.yaml`:

```yaml
/**:
  ros__parameters:
    app_name: "YourPSDK"
    app_id: "YourID"
    app_key: "YourAppKey"
    app_license: "YourAppLicense"
    developer_account: "YourDeveloperAccount"
    baudrate: "921600"
```

Set the serial, USB bulk or network devices that connect to the drone in
`config/link_config.json`.

Then launch the `psdk_wrapper` node and the platform:

```bash
ros2 launch as2_platform_dji_psdk psdk_wrapper.launch.py namespace:=drone0
ros2 launch as2_platform_dji_psdk as2_platform_dji_psdk.launch.py namespace:=drone0
```

### `psdk_wrapper.launch.py`

| Argument | Default | Description |
| --- | --- | --- |
| `namespace` | `$AEROSTACK2_SIMULATION_DRONE_ID` | Drone namespace |
| `psdk_authentication_params_file` | `config/psdk_authentication.yaml` | DJI app credentials |
| `link_config_file_path` | `config/link_config.json` | Connection to the drone |
| `hms_return_codes_path` | `config/hms_2023_08_22.json` | Known DJI HMS return codes |
| `psdk_params_file` | `config/psdk_params.yaml` | Wrapper modules, frames and data rates |
| `tf_frame_prefix` | `''` | TF frame prefix |

The wrapper's GPS, IMU and main camera topics are remapped to the Aerostack2
`sensor_measurements/` namespace.

### `as2_platform_dji_psdk.launch.py`

| Argument | Default | Description |
| --- | --- | --- |
| `namespace` | `$AEROSTACK2_SIMULATION_DRONE_ID` | Drone namespace |
| `log_level` | `info` | Log severity level |
| `control_modes_file` | `config/control_modes.yaml` | Platform control modes |
| `config_file` | `config/platform_config_file.yaml` | Platform parameters |

Platform parameters (command and info rates, camera, gimbal, TF timeout) are in
`config/platform_config_file.yaml`, and each one can also be overridden as a launch argument.

### Matrice 4 series and Matrice 400 (DJI Manifold 3)

> [!WARNING]
> Build-tested only: not yet tested on a Manifold 3 or aircraft.

On these aircraft, PSDK flight control requires the DJI Manifold 3, which talks to the aircraft
over USB bulk only. Build `psdk_ros2` (PSDK >= 3.12, e.g. 3.18.0) with
`-DPSDK_PLATFORM=manifold3`, then launch the wrapper with the Manifold 3 configuration:

```bash
ros2 launch as2_platform_dji_psdk psdk_wrapper.launch.py namespace:=drone0 \
  link_config_file_path:=$(ros2 pkg prefix as2_platform_dji_psdk)/share/as2_platform_dji_psdk/config/link_config_manifold3.json \
  psdk_params_file:=$(ros2 pkg prefix as2_platform_dji_psdk)/share/as2_platform_dji_psdk/config/psdk_params_manifold3.yaml
```

Telemetry is capped at 50 Hz and `sensor_measurements/imu` is composed from separate topics
(no `HARD_SYNC` on these aircraft). Without GNSS and vision positioning the aircraft switches to
ATTI mode, where the velocity commands sent by this platform are not executed.

## Test

```bash
colcon test --packages-select as2_platform_dji_psdk
colcon test-result --verbose
```

## Citation

If you use Aerostack2 in your research, please cite:

> M. Fernandez-Cortizas, M. Molina, P. Arias-Perez, R. Perez-Segui, D. Perez-Saura and P. Campoy,
> 2023, ["Aerostack2: A software framework for developing multi-robot aerial systems"](https://arxiv.org/abs/2303.18237),
> arXiv 2303.18237.

## License

[BSD 3-Clause](LICENSE)
