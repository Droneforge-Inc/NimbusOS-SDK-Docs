---
description: Typed subscription examples for telemetry, selected state, camera frames, waypoint status, and autonomy status.
---

# Subscriptions

The SDK supports typed subscriptions for common NimbusOS receive workflows.

## Telemetry

Telemetry contains battery, attitude, and link data.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for telemetry in client.telemetry(timeout_sec=5.0):
        print(telemetry.seq, telemetry.t_ns)
        print(telemetry.battery.voltage)
        print(telemetry.attitude.roll_deg)
        print(telemetry.link.uplink_link_quality)
```

### Fields

| Object              | Fields                                       |
| ------------------- | -------------------------------------------- |
| `Telemetry`         | `seq`, `t_ns`, `battery`, `attitude`, `link` |
| `BatteryTelemetry`  | `voltage`, `current`, `remaining_capacity`   |
| `AttitudeTelemetry` | `roll_deg`, `pitch_deg`, `yaw_deg`           |
| `LinkTelemetry`     | `uplink_link_quality`, `rf_mode`             |

{% hint style="info" %}
Attitude values are exposed in degrees in the typed SDK object.
{% endhint %}

## Selected state

Selected state contains local-frame position, velocity, direction, attitude, and orientation data.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for state in client.selected_state(timeout_sec=5.0):
        if state.valid:
            print(state.position.x_m, state.position.y_m, state.position.z_m)
            print(state.velocity.x_mps, state.velocity.y_mps, state.velocity.z_mps)
```

### Fields

| Object                | Fields                                                                                        |
| --------------------- | --------------------------------------------------------------------------------------------- |
| `State`               | `seq`, `t_ns`, `valid`, `position`, `velocity`, `forward`, `right`, `attitude`, `orientation` |
| `LocalFramePosition`  | `x_m`, `y_m`, `z_m`                                                                           |
| `LocalFrameVelocity`  | `x_mps`, `y_mps`, `z_mps`                                                                     |
| `LocalFrameDirection` | `x`, `y`, `z`                                                                                 |
| `StateAttitude`       | `roll_deg`, `pitch_deg`, `yaw_deg`                                                            |
| `StateOrientation`    | `w`, `x`, `y`, `z`                                                                            |

{% hint style="info" %}
State attitude values are converted from radians to degrees.
{% endhint %}

## Camera

Camera subscriptions yield JPEG frames.

```python
from pathlib import Path

from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for frame in client.camera_frames(timeout_sec=5.0):
        Path("camera.jpg").write_bytes(frame.jpeg)
        print(frame.seq, frame.width, frame.height)
        break
```

### Fields

| Field    | Description                                     |
| -------- | ----------------------------------------------- |
| `seq`    | Message sequence number.                        |
| `t_ns`   | Message timestamp in nanoseconds from NimbusOS. |
| `width`  | Frame width in pixels.                          |
| `height` | Frame height in pixels.                         |
| `jpeg`   | JPEG image bytes.                               |

Use `client.camera_frames()` for the core-selected camera stream. Use `client.live_camera_frames()` for the raw live camera stream before core camera-source selection.

Use `client.latest_camera_frames()` or `client.latest_live_camera_frames()` when your application only needs the freshest queued camera frame.

## Waypoint status

Waypoint status reports active waypoint progress and reached/held state.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for status in client.waypoint_status(timeout_sec=5.0):
        print(status.active, status.reached, status.held, status.distance_m)
```

### Fields

| Field                  | Description                                                |
| ---------------------- | ---------------------------------------------------------- |
| `seq`                  | Status message sequence number.                            |
| `t_ns`                 | Status message timestamp in nanoseconds from NimbusOS.     |
| `current_waypoint_seq` | Sequence number of the current active waypoint.            |
| `command_seq`          | Sequence number of the command associated with the status. |
| `waypoint_index`       | Index of the waypoint in the active queue.                 |
| `active`               | Whether waypoint tracking is active.                       |
| `reached`              | Whether the current waypoint has been reached.             |
| `held`                 | Whether the waypoint hold period has completed.            |
| `distance_m`           | Current distance to the target in meters.                  |

## Autonomy status

Autonomy status reports the current high-level autonomy state.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for status in client.autonomy_status(timeout_sec=5.0):
        print(status.seq, status.status)
```

### Fields

| Field    | Description                                               |
| -------- | --------------------------------------------------------- |
| `seq`    | Status message sequence number.                           |
| `t_ns`   | Status message timestamp in nanoseconds from NimbusOS.    |
| `status` | Current autonomy status name, such as `idle` or `landed_manual`. |
