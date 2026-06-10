---
description: Installed NimbusOS SDK command-line tools, topics, arguments, endpoints, and examples.
---

# CLI

The SDK installs command-line tools for smoke testing and manual operation.

All publish commands accept `--pub-endpoint`. The default is `DF_ZMQ_PUB_ENDPOINT` if set, otherwise `tcp://127.0.0.1:7771`.

The subscribe command accepts `--sub-endpoint`. The default is `DF_ZMQ_SUB_ENDPOINT` if set, otherwise `tcp://127.0.0.1:7772`.

{% hint style="warning" %}
Run publish commands only against a safe vehicle, simulator, or controlled NimbusOS test environment.
{% endhint %}

## `nimbusos-subscribe`

Subscribes to one supported SDK topic.

```bash
nimbusos-subscribe telemetry
nimbusos-subscribe selected_state
nimbusos-subscribe camera
nimbusos-subscribe camera_overlay
nimbusos-subscribe live_camera
nimbusos-subscribe waypoint_status
nimbusos-subscribe autonomy_status
```

### Arguments

| Argument         | Required | Default            | Description                                                                                                                   |
| ---------------- | -------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `topic`          | Yes      | None               | One of `telemetry`, `selected_state`, `camera`, `camera_overlay`, `live_camera`, `waypoint_status`, `autonomy_status`.        |
| `--limit`        | No       | `0`                | Stop after N messages. `0` means unlimited.                                                                                   |
| `--timeout`      | No       | None               | Stop after N seconds.                                                                                                         |
| `--sub-endpoint` | No       | Configured default | ZeroMQ subscribe endpoint.                                                                                                    |

### Examples

```bash
nimbusos-subscribe telemetry --limit 1 --timeout 5
nimbusos-subscribe camera --limit 10
```

Output includes the topic, payload byte length, decoded root type, and sequence/timestamp fields when present.

## `nimbusos-arm`

Publishes an arm-state command.

```bash
nimbusos-arm
nimbusos-arm --disarm
```

### Arguments

| Argument         | Required | Default            | Description                        |
| ---------------- | -------- | ------------------ | ---------------------------------- |
| `--disarm`       | No       | `false`            | Publish disarmed instead of armed. |
| `--pub-endpoint` | No       | Configured default | ZeroMQ publish endpoint.           |

## `nimbusos-autonomy-request`

Publishes an autonomy request.

```bash
nimbusos-autonomy-request takeoff
nimbusos-autonomy-request land
nimbusos-autonomy-request return_home
nimbusos-autonomy-request relative_waypoint --mode override --forward 1.5 --right 0.0 --down 0.0
nimbusos-autonomy-request relative_waypoint --mode queue --forward 1.0 --right 0.5 --down 0.0 --threshold 0.15 --hold-time 2.0
```

### Arguments

| Argument         | Required | Default            | Description                                                            |
| ---------------- | -------- | ------------------ | ---------------------------------------------------------------------- |
| `type`           | Yes      | None               | One of `takeoff`, `land`, `relative_waypoint`, `return_home`.          |
| `--forward`      | No       | `0.0`              | Meters forward or backward for `relative_waypoint`.                    |
| `--right`        | No       | `0.0`              | Meters right or left for `relative_waypoint`.                          |
| `--down`         | No       | `0.0`              | Meters down or up for `relative_waypoint`.                             |
| `--mode`         | No       | `override`         | `override` replaces the active waypoint; `queue` appends to the queue. |
| `--threshold`    | No       | `0.05`             | Meters from target that counts as reached.                             |
| `--hold-time`    | No       | `0.0`              | Seconds to hold after reaching the waypoint.                           |
| `--pub-endpoint` | No       | Configured default | ZeroMQ publish endpoint.                                               |

## `nimbusos-waypoint-speed`

Publishes waypoint path speed.

```bash
nimbusos-waypoint-speed 0.45
```

### Arguments

| Argument         | Required | Default            | Description                                |
| ---------------- | -------- | ------------------ | ------------------------------------------ |
| `speed_mps`      | Yes      | None               | Waypoint path speed in meters per second, from `0.05` to `0.75`. |
| `--pub-endpoint` | No       | Configured default | ZeroMQ publish endpoint.                   |

## `nimbusos-yaw-turn-command`

Publishes a relative yaw turn command.

```bash
nimbusos-yaw-turn-command 0.52
```

### Arguments

| Argument         | Required | Default            | Description                   |
| ---------------- | -------- | ------------------ | ----------------------------- |
| `delta_yaw_rad`  | Yes      | None               | Relative yaw turn in radians. |
| `--pub-endpoint` | No       | Configured default | ZeroMQ publish endpoint.      |
