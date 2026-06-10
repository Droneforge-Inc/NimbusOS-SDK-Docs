---
description: Current publish methods for arm state, autonomy requests, waypoint speed, yaw turns, and camera overlays.
---

# Publishing

Publishing methods send supported NimbusOS command messages to the configured publish endpoint.

{% hint style="warning" %}
Test publish commands only against a safe vehicle, simulator, or controlled NimbusOS test environment.
{% endhint %}

## Arm state

Publish armed or disarmed state.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_arm_state(True)
```

### Arguments

| Argument | Type   | Default | Meaning                                             |
| -------- | ------ | ------- | --------------------------------------------------- |
| `armed`  | `bool` | `True`  | `True` publishes armed, `False` publishes disarmed. |

### CLI equivalent

```bash
nimbusos-arm
nimbusos-arm --disarm
```

## Autonomy request

Publish a high-level autonomy request.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_autonomy_request("takeoff")
    client.publish_autonomy_request("return_home")
```

### Request types

| Request type        | Meaning                                                                                   |
| ------------------- | ----------------------------------------------------------------------------------------- |
| `takeoff`           | Request takeoff.                                                                          |
| `land`              | Request landing.                                                                          |
| `return_home`       | Request return-home behavior.                                                             |
| `relative_waypoint` | Request a relative waypoint using `forward`, `right`, `down`, `mode`, and hold settings. |

### Relative waypoint example

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_relative_waypoint(
        mode="override",
        forward=1.5,
        right=0.0,
        down=0.0,
        threshold_m=0.15,
        hold_time_s=0.0,
    )
```

`publish_relative_waypoint(...)` is a convenience wrapper around `publish_autonomy_request("relative_waypoint", ...)`.

### Arguments

| Argument       | Type    | Default    | Meaning                                                                                 |
| -------------- | ------- | ---------- | --------------------------------------------------------------------------------------- |
| `request_type` | `str`   | Required   | One of `takeoff`, `land`, `relative_waypoint`, `return_home`.                           |
| `forward`      | `float` | `0.0`      | Body-frame meters forward or backward. Positive is forward.                             |
| `right`        | `float` | `0.0`      | Body-frame meters right or left. Positive is right.                                     |
| `down`         | `float` | `0.0`      | Body-frame meters down or up relative to the current or queued waypoint base.           |
| `mode`         | `str`   | `override` | `override` replaces the active waypoint; `queue` appends after active/queued waypoints. |
| `threshold_m`  | `float` | `0.05`     | Meters from target that counts as reached.                                              |
| `hold_time_s`  | `float` | `0.0`      | Seconds to hold after reaching the requested waypoint.                                  |

### CLI equivalents

```bash
nimbusos-autonomy-request takeoff
nimbusos-autonomy-request land
nimbusos-autonomy-request return_home
nimbusos-autonomy-request relative_waypoint --mode override --forward 1.5 --right 0.0 --down 0.0
nimbusos-autonomy-request relative_waypoint --mode queue --forward 1.0 --right 0.5 --down 0.0 --threshold 0.15 --hold-time 2.0
```

## Waypoint speed

Publish the waypoint path speed in meters per second.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_waypoint_speed(0.45)
```

### Arguments

| Argument    | Type    | Default  | Meaning                                            |
| ----------- | ------- | -------- | -------------------------------------------------- |
| `speed_mps` | `float` | Required | Waypoint path speed, from `0.05` to `0.75` meters per second. |

### CLI equivalent

```bash
nimbusos-waypoint-speed 0.45
```

## Yaw turn command

Publish a relative yaw turn command in radians.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_yaw_turn_command(0.52)
```

### Arguments

| Argument        | Type    | Default  | Meaning                       |
| --------------- | ------- | -------- | ----------------------------- |
| `delta_yaw_rad` | `float` | Required | Relative yaw turn in radians. |

### CLI equivalent

```bash
nimbusos-yaw-turn-command 0.52
```

## Camera overlay

Publish drawing instructions for the `camera_overlay` topic.

```python
from nimbusos_sdk import NimbusClient
from nimbusos_sdk import box

with NimbusClient() as client:
    for frame in client.latest_camera_frames(timeout_sec=5.0):
        client.publish_camera_overlay(
            camera_seq=frame.seq,
            frame_width=frame.width,
            frame_height=frame.height,
            source="detector",
            layers=[
                {
                    "name": "detections",
                    "primitives": [
                        box(120, 80, 180, 240, label="target", confidence=0.91),
                    ],
                }
            ],
        )
        break
```

Overlay coordinates are in the image coordinate system described by `frame_width` and `frame_height`. The UI scales them to the displayed camera frame.

### Arguments

| Argument       | Type                      | Default  | Meaning                                              |
| -------------- | ------------------------- | -------- | ---------------------------------------------------- |
| `camera_seq`   | `int`                     | Required | Sequence number of the camera frame being annotated. |
| `frame_width`  | `int`                     | Required | Source frame width in pixels.                        |
| `frame_height` | `int`                     | Required | Source frame height in pixels.                       |
| `layers`       | `Iterable`                | Required | Overlay layers or primitive lists to publish.        |
| `ttl_ms`       | `int`                     | `250`    | Overlay lifetime in milliseconds.                    |
| `source`       | `str`                     | `""`     | Optional source label for the overlay publisher.     |

No command-line helper is installed for camera overlays.

## Validation rules

Values rejected by these validation checks raise `ValueError` before publishing. Non-numeric values passed to numeric arguments may raise `TypeError` before publishing.

| Argument                                    | Rule                                                                     |
| ------------------------------------------- | ------------------------------------------------------------------------ |
| `armed`                                     | Must be a `bool`.                                                        |
| `request_type`                              | Must be `takeoff`, `land`, `relative_waypoint`, or `return_home`.        |
| `mode`                                      | Must be `override` or `queue`.                                           |
| `forward`, `right`, `down`                  | Must be finite numbers.                                                  |
| `threshold_m`                               | Must be finite and greater than zero.                                    |
| `hold_time_s`                               | Must be finite and greater than or equal to zero.                        |
| `speed_mps`                                 | Must be finite and between `0.05` and `0.75`.                            |
| `delta_yaw_rad`                             | Must be finite.                                                          |
| `camera_seq`                                | Must be a non-negative integer.                                          |
| `frame_width`, `frame_height`               | Must be positive integers.                                               |
| `ttl_ms`                                    | Must be a non-negative integer.                                          |
| Camera overlay enum names                   | Must be supported primitive, tone, blend, raster, or color-map names.    |
