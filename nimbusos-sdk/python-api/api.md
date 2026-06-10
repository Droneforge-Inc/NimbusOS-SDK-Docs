# API

This page documents the stable Python API exposed by `nimbusos_sdk`.

Import common public classes and type aliases from `nimbusos_sdk`:

```python
from nimbusos_sdk import AttitudeTelemetry
from nimbusos_sdk import BatteryTelemetry
from nimbusos_sdk import CameraFrame
from nimbusos_sdk import CameraTopicName
from nimbusos_sdk import LinkTelemetry
from nimbusos_sdk import LocalFrameDirection
from nimbusos_sdk import LocalFramePosition
from nimbusos_sdk import LocalFrameVelocity
from nimbusos_sdk import NimbusClient
from nimbusos_sdk import State
from nimbusos_sdk import StateAttitude
from nimbusos_sdk import StateOrientation
from nimbusos_sdk import Telemetry
from nimbusos_sdk import WaypointStatus
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

The client manages the configured publish and subscribe endpoints.

Use it as a context manager when possible:

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_guidance_request("go")
```

You can also call `client.close()` manually.

## Typed subscription methods

Typed subscriptions are recommended for application code.

| Method                                                                  | Returns                    | Default receive HWM |
| ----------------------------------------------------------------------- | -------------------------- | ------------------- |
| `client.telemetry(timeout_sec=None, receive_hwm=128)`                   | `Iterator[Telemetry]`      | `128`               |
| `client.state(timeout_sec=None, receive_hwm=128)`                       | `Iterator[State]`          | `128`               |
| `client.waypoint_status(timeout_sec=None, receive_hwm=128)`             | `Iterator[WaypointStatus]` | `128`               |
| `client.camera_frames(topic="camera", timeout_sec=None, receive_hwm=8)` | `Iterator[CameraFrame]`    | `8`                 |
| `client.live_camera_frames(timeout_sec=None, receive_hwm=8)`            | `Iterator[CameraFrame]`    | `8`                 |

`timeout_sec` is the total timeout for the iterator. `receive_hwm` configures the ZeroMQ receive high-water mark.

## Publish methods

### Arm state

```python
client.publish_arm_state(armed: bool = True) -> None
```

Publishes armed or disarmed state.

### Guidance request

```python
client.publish_guidance_request(
    request_type: str,
    *,
    forward: float = 0.0,
    right: float = 0.0,
    down: float = 0.0,
    hold_time_s: float = 0.0,
) -> None
```

Publishes a high-level guidance request. `request_type` must be one of `go`, `land`, `relative_waypoint`, or `return_home`.

### Waypoint command

```python
client.publish_waypoint_command(
    *,
    forward: float,
    right: float,
    down: float,
    mode: str = "override",
    threshold_m: float = 0.05,
    hold_time_s: float = 0.0,
) -> None
```

Publishes an override or queued waypoint command. `mode` must be `override` or `queue`.

### Yaw turn command

```python
client.publish_yaw_turn_command(delta_yaw_rad: float) -> None
```

Publishes a relative yaw turn command in radians.

## Validation

The SDK validates command arguments before publishing.

| Argument                                    | Rule                                                                |
| ------------------------------------------- | ------------------------------------------------------------------- |
| `armed`                                     | Must be a `bool`.                                                   |
| `request_type`                              | Must be one of `go`, `land`, `relative_waypoint`, or `return_home`. |
| `mode`                                      | Must be `override` or `queue`.                                      |
| `forward`, `right`, `down`, `delta_yaw_rad` | Must be finite numbers.                                             |
| `threshold_m`                               | Must be finite and greater than zero.                               |
| `hold_time_s`                               | Must be finite and greater than or equal to zero.                   |

Values rejected by these validation checks raise `ValueError`. Non-numeric values passed to numeric arguments may raise `TypeError` before publishing.
