---
description: Stable public Python API exported by nimbusos_sdk, including NimbusClient methods and typed data objects.
---

# API

This page documents the stable Python API exposed by `nimbusos_sdk`.

Import common public classes, type aliases, and helpers from `nimbusos_sdk`:

```python
from nimbusos_sdk import AttitudeTelemetry
from nimbusos_sdk import AutonomyStatus
from nimbusos_sdk import AutonomyStatusName
from nimbusos_sdk import BatteryTelemetry
from nimbusos_sdk import CameraFrame
from nimbusos_sdk import CameraOverlayLayerDict
from nimbusos_sdk import CameraOverlayPrimitiveDict
from nimbusos_sdk import CameraTopicName
from nimbusos_sdk import LinkTelemetry
from nimbusos_sdk import LocalFrameDirection
from nimbusos_sdk import LocalFramePosition
from nimbusos_sdk import LocalFrameVelocity
from nimbusos_sdk import NimbusClient
from nimbusos_sdk import ReceivedMessage
from nimbusos_sdk import State
from nimbusos_sdk import StateAttitude
from nimbusos_sdk import StateOrientation
from nimbusos_sdk import Telemetry
from nimbusos_sdk import WaypointStatus
from nimbusos_sdk import box
```

## NimbusClient

`NimbusClient` is the entry point for subscriptions and publications.

```python
NimbusClient(
    *,
    pub_endpoint: str = "tcp://127.0.0.1:7771",
    sub_endpoint: str = "tcp://127.0.0.1:7772",
)
```

The configured publish default is `DF_ZMQ_PUB_ENDPOINT` if set, otherwise `tcp://127.0.0.1:7771`. The configured subscribe default is `DF_ZMQ_SUB_ENDPOINT` if set, otherwise `tcp://127.0.0.1:7772`.

Use the client as a context manager when possible:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    for telemetry in client.telemetry(timeout_sec=1.0):
        print(telemetry.seq, telemetry.t_ns)
        break
```

You can also call `client.close()` manually.

## Typed subscription methods

Typed subscriptions are recommended for application code.

| Method                                                                                     | Returns                    | Default receive HWM |
| ------------------------------------------------------------------------------------------ | -------------------------- | ------------------- |
| `client.telemetry(timeout_sec=None, receive_hwm=128)`                                      | `Iterator[Telemetry]`      | `128`               |
| `client.selected_state(timeout_sec=None, receive_hwm=128)`                                 | `Iterator[State]`          | `128`               |
| `client.waypoint_status(timeout_sec=None, receive_hwm=128)`                                | `Iterator[WaypointStatus]` | `128`               |
| `client.autonomy_status(timeout_sec=None, receive_hwm=128)`                                | `Iterator[AutonomyStatus]` | `128`               |
| `client.camera_frames(topic="camera", timeout_sec=None, receive_hwm=8)`                    | `Iterator[CameraFrame]`    | `8`                 |
| `client.live_camera_frames(timeout_sec=None, receive_hwm=8)`                               | `Iterator[CameraFrame]`    | `8`                 |
| `client.latest_camera_frames(topic="camera", timeout_sec=None, receive_hwm=1, max_drain=64)` | `Iterator[CameraFrame]`  | `1`                 |
| `client.latest_live_camera_frames(timeout_sec=None, receive_hwm=1, max_drain=64)`          | `Iterator[CameraFrame]`    | `1`                 |

`timeout_sec` is the total timeout for the iterator. `receive_hwm` configures the ZeroMQ receive high-water mark. Use the `latest_*` camera methods when an application only needs the freshest queued camera frame.

## Raw subscription methods

Raw subscriptions yield `ReceivedMessage` objects with the decoded FlatBuffer root and the original payload bytes.

| Method                                                                       | Topic               | Returns                     |
| ---------------------------------------------------------------------------- | ------------------- | --------------------------- |
| `client.subscribe_telemetry(timeout_sec=None, receive_hwm=128)`              | `telemetry`         | `Iterator[ReceivedMessage]` |
| `client.subscribe_selected_state(timeout_sec=None, receive_hwm=128)`         | `selected_state`    | `Iterator[ReceivedMessage]` |
| `client.subscribe_waypoint_status(timeout_sec=None, receive_hwm=128)`        | `waypoint_status`   | `Iterator[ReceivedMessage]` |
| `client.subscribe_autonomy_status(timeout_sec=None, receive_hwm=128)`        | `autonomy_status`   | `Iterator[ReceivedMessage]` |
| `client.subscribe_camera(topic="camera", timeout_sec=None, receive_hwm=8)`   | `camera` or `live_camera` | `Iterator[ReceivedMessage]` |
| `client.subscribe_live_camera(timeout_sec=None, receive_hwm=8)`              | `live_camera`       | `Iterator[ReceivedMessage]` |
| `client.subscribe_camera_overlay(timeout_sec=None, receive_hwm=8)`           | `camera_overlay`    | `Iterator[ReceivedMessage]` |

`ReceivedMessage` fields are `topic`, `payload`, `received_at_s`, `decoded`, `root_type`, `seq`, and `t_ns`.

## Data objects

Typed subscription methods return frozen dataclasses.

| Object                | Fields                                                                                        |
| --------------------- | --------------------------------------------------------------------------------------------- |
| `Telemetry`           | `seq`, `t_ns`, `battery`, `attitude`, `link`                                                   |
| `BatteryTelemetry`    | `voltage`, `current`, `remaining_capacity`                                                     |
| `AttitudeTelemetry`   | `roll_deg`, `pitch_deg`, `yaw_deg`                                                            |
| `LinkTelemetry`       | `uplink_link_quality`, `rf_mode`                                                              |
| `State`               | `seq`, `t_ns`, `valid`, `position`, `velocity`, `forward`, `right`, `attitude`, `orientation` |
| `LocalFramePosition`  | `x_m`, `y_m`, `z_m`                                                                           |
| `LocalFrameVelocity`  | `x_mps`, `y_mps`, `z_mps`                                                                     |
| `LocalFrameDirection` | `x`, `y`, `z`                                                                                 |
| `StateAttitude`       | `roll_deg`, `pitch_deg`, `yaw_deg`                                                            |
| `StateOrientation`    | `w`, `x`, `y`, `z`                                                                            |
| `CameraFrame`         | `seq`, `t_ns`, `width`, `height`, `jpeg`                                                       |
| `WaypointStatus`      | `seq`, `t_ns`, `current_waypoint_seq`, `command_seq`, `waypoint_index`, `active`, `reached`, `held`, `distance_m` |
| `AutonomyStatus`      | `seq`, `t_ns`, `status`                                                                       |

`AutonomyStatus.status` is an `AutonomyStatusName`.

## Publish methods

### Arm state

```python
client.publish_arm_state(armed: bool = True) -> None
```

Publishes armed or disarmed state.

### Autonomy request

```python
client.publish_autonomy_request(
    request_type: str,
    *,
    forward: float = 0.0,
    right: float = 0.0,
    down: float = 0.0,
    mode: str = "override",
    threshold_m: float = 0.05,
    hold_time_s: float = 0.0,
) -> None
```

Publishes a high-level autonomy request. `request_type` must be one of `takeoff`, `land`, `relative_waypoint`, or `return_home`.

### Takeoff autonomy request

```python
client.publish_autonomy_request("takeoff")
```

Publishes a `takeoff` autonomy request.

### Relative waypoint

```python
client.publish_relative_waypoint(
    *,
    forward: float,
    right: float,
    down: float,
    mode: str = "override",
    threshold_m: float = 0.05,
    hold_time_s: float = 0.0,
) -> None
```

Publishes a `relative_waypoint` autonomy request. `mode` must be `override` or `queue`.

### Waypoint speed

```python
client.publish_waypoint_speed(speed_mps: float) -> None
```

Publishes waypoint path speed in meters per second. The accepted range is `0.05` to `0.75`.

### Yaw turn command

```python
client.publish_yaw_turn_command(delta_yaw_rad: float) -> None
```

Publishes a relative yaw turn command in radians.

### Camera overlay

```python
client.publish_camera_overlay(
    *,
    camera_seq: int,
    frame_width: int,
    frame_height: int,
    layers: Iterable[CameraOverlayLayerDict | Iterable[CameraOverlayPrimitiveDict]],
    ttl_ms: int = 250,
    source: str = "",
) -> None
```

Publishes drawing instructions on `camera_overlay`. Coordinates are in the image coordinate system described by `frame_width` and `frame_height`.

The `box()` helper builds a rectangle primitive:

```python
box(
    x: float,
    y: float,
    width: float,
    height: float,
    *,
    label: str = "",
    confidence: float = -1.0,
    tone: str | int = "info",
    track_id: int = -1,
) -> CameraOverlayPrimitiveDict
```

`CameraOverlayLayerDict` accepts `name`, `z_index`, `opacity`, `blend_mode`, and `primitives`. `CameraOverlayPrimitiveDict` accepts fields such as `type`, `tone`, `label`, `value`, `confidence`, `track_id`, geometry fields, point lists, and raster fields.

## Validation

The SDK validates command arguments before publishing.

| Argument                                    | Rule                                                                        |
| ------------------------------------------- | --------------------------------------------------------------------------- |
| `armed`                                     | Must be a `bool`.                                                           |
| `request_type`                              | Must be one of `takeoff`, `land`, `relative_waypoint`, or `return_home`.    |
| `mode`                                      | Must be `override` or `queue`.                                              |
| `forward`, `right`, `down`, `delta_yaw_rad` | Must be finite numbers.                                                     |
| `threshold_m`                               | Must be finite and greater than zero.                                       |
| `hold_time_s`                               | Must be finite and greater than or equal to zero.                           |
| `speed_mps`                                 | Must be finite and between `0.05` and `0.75`.                               |
| `camera_seq`                                | Must be a non-negative integer.                                             |
| `frame_width`, `frame_height`               | Must be positive integers.                                                  |
| `ttl_ms`                                    | Must be a non-negative integer.                                             |

Values rejected by these validation checks raise `ValueError`. Non-numeric values passed to numeric arguments may raise `TypeError` before publishing. Unsupported camera overlay enum names also raise `ValueError`.
