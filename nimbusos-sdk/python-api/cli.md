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
nimbusos-subscribe state
nimbusos-subscribe camera
nimbusos-subscribe live_camera
nimbusos-subscribe waypoint_status
```

### Arguments

| Argument         | Required | Default            | Description                                                              |
| ---------------- | -------- | ------------------ | ------------------------------------------------------------------------ |
| `topic`          | Yes      | None               | One of `telemetry`, `state`, `camera`, `live_camera`, `waypoint_status`. |
| `--limit`        | No       | `0`                | Stop after N messages. `0` means unlimited.                              |
| `--timeout`      | No       | None               | Stop after N seconds.                                                    |
| `--sub-endpoint` | No       | Configured default | ZeroMQ subscribe endpoint.                                               |

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

## `nimbusos-guidance-request`

Publishes a guidance request.

```bash
nimbusos-guidance-request go
nimbusos-guidance-request land
nimbusos-guidance-request return_home
nimbusos-guidance-request relative_waypoint --forward 1.5 --right 0.0 --down -1.0
```

### Arguments

| Argument         | Required | Default            | Description                                              |
| ---------------- | -------- | ------------------ | -------------------------------------------------------- |
| `type`           | Yes      | None               | One of `go`, `land`, `relative_waypoint`, `return_home`. |
| `--forward`      | No       | `0.0`              | Meters forward or backward.                              |
| `--right`        | No       | `0.0`              | Meters right or left.                                    |
| `--down`         | No       | `0.0`              | Down-axis target in meters.                              |
| `--hold-time`    | No       | `0.0`              | Seconds to hold after reaching the waypoint.             |
| `--pub-endpoint` | No       | Configured default | ZeroMQ publish endpoint.                                 |

## `nimbusos-waypoint-command`

Publishes one waypoint command.

```bash
nimbusos-waypoint-command --mode override --forward 1.5 --right 0.0 --down -1.0
nimbusos-waypoint-command --mode queue --forward 1.0 --right 0.5 --down -1.0 --threshold 0.15 --hold-time 2.0
```

### Arguments

| Argument         | Required | Default            | Description                                                            |
| ---------------- | -------- | ------------------ | ---------------------------------------------------------------------- |
| `--mode`         | No       | `override`         | `override` replaces the active waypoint; `queue` appends to the queue. |
| `--forward`      | Yes      | None               | Meters forward or backward.                                            |
| `--right`        | Yes      | None               | Meters right or left.                                                  |
| `--down`         | Yes      | None               | Absolute local-frame down target.                                      |
| `--hold-time`    | No       | `0.0`              | Seconds to hold after reaching the waypoint.                           |
| `--threshold`    | No       | `0.05`             | Meters from target that counts as reached.                             |
| `--pub-endpoint` | No       | Configured default | ZeroMQ publish endpoint.                                               |

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
