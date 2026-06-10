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

## Guidance request

Publish a high-level guidance request.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_guidance_request("go")
    client.publish_guidance_request("return_home")
```

### Request types

| Request type        | Meaning                                                                               |
| ------------------- | ------------------------------------------------------------------------------------- |
| `go`                | Request normal guided operation.                                                      |
| `land`              | Request landing.                                                                      |
| `return_home`       | Request return-home behavior.                                                         |
| `relative_waypoint` | Request a relative waypoint using `forward`, `right`, `down`, and optional hold time. |

### Relative waypoint example

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_guidance_request(
        "relative_waypoint",
        forward=1.5,
        right=0.0,
        down=-1.0,
        hold_time_s=0.0,
    )
```

### Arguments

| Argument       | Type    | Default  | Meaning                                                  |
| -------------- | ------- | -------- | -------------------------------------------------------- |
| `request_type` | `str`   | Required | One of `go`, `land`, `relative_waypoint`, `return_home`. |
| `forward`      | `float` | `0.0`    | Meters forward or backward. Positive is forward.         |
| `right`        | `float` | `0.0`    | Meters right or left. Positive is right.                 |
| `down`         | `float` | `0.0`    | Down-axis target in meters.                              |
| `hold_time_s`  | `float` | `0.0`    | Seconds to hold after reaching the requested waypoint.   |

### CLI equivalents

```bash
nimbusos-guidance-request go
nimbusos-guidance-request land
nimbusos-guidance-request return_home
nimbusos-guidance-request relative_waypoint --forward 1.5 --right 0.0 --down -1.0
```

## Waypoint command

Publish an override or queued waypoint command.

```python
from nimbusos_sdk import NimbusClient

with NimbusClient() as client:
    client.publish_waypoint_command(
        mode="override",
        forward=1.5,
        right=0.0,
        down=-1.0,
        threshold_m=0.15,
        hold_time_s=0.0,
    )
```

### Arguments

| Argument      | Type    | Default    | Meaning                                                                                 |
| ------------- | ------- | ---------- | --------------------------------------------------------------------------------------- |
| `forward`     | `float` | Required   | Target meters forward or backward. Positive is forward.                                 |
| `right`       | `float` | Required   | Target meters right or left. Positive is right.                                         |
| `down`        | `float` | Required   | Absolute local-frame down target in meters.                                             |
| `mode`        | `str`   | `override` | `override` replaces the active waypoint; `queue` appends after active/queued waypoints. |
| `threshold_m` | `float` | `0.05`     | Meters from target that counts as reached.                                              |
| `hold_time_s` | `float` | `0.0`      | Seconds to hold after reaching the waypoint.                                            |

### CLI equivalents

```bash
nimbusos-waypoint-command --mode override --forward 1.5 --right 0.0 --down -1.0
nimbusos-waypoint-command --mode queue --forward 1.0 --right 0.5 --down -1.0 --threshold 0.15 --hold-time 2.0
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

## Validation rules

Values rejected by these validation checks raise `ValueError` before publishing. Non-numeric values passed to numeric arguments may raise `TypeError` before publishing.

| Argument                   | Rule                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `armed`                    | Must be a `bool`.                                            |
| `request_type`             | Must be `go`, `land`, `relative_waypoint`, or `return_home`. |
| `mode`                     | Must be `override` or `queue`.                               |
| `forward`, `right`, `down` | Must be finite numbers.                                      |
| `threshold_m`              | Must be finite and greater than zero.                        |
| `hold_time_s`              | Must be finite and greater than or equal to zero.            |
| `delta_yaw_rad`            | Must be finite.                                              |
